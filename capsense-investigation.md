# Leyden Jar Capacitive Scan Investigation

This note captures the current understanding of the Leyden Jar Model F
capacitive scan design and the firmware ideas worth testing once the F122
keyboard is available.

The goal is to decide whether the scan can use the analog behavior of the
capacitive matrix directly, instead of relying on generic key debounce after
the matrix has already been sampled.

## Preferred Direction

The preferred investigation path is now a fixed-reference, time-domain scanner:

```text
set one comparator reference voltage
strobe each column
sample the eight comparator outputs several times with PIO
use DMA to move the sample train into RAM
derive per-key dropout time and confidence on the RP2040 M-core
feed QMK only the resolved stable matrix
```

This is preferred over the current multi-bucket DAC scan because it measures
the signal the firmware actually sees: comparator output over time. It should
also be fast enough to keep a full matrix scan under the 1 ms USB poll interval
without pushing the analog timing to the edge.

The current DAC-bucket scheme remains important as:

- the baseline implementation to compare against
- a diagnostic and calibration reference
- a fallback if dropout timing is not stable enough

Adjacent-bucket hysteresis, both-edge sampling, and extra DAC passes should be
treated as backup or diagnostic tools unless the time-domain approach fails on
real hardware.

## Current Hardware Model

The Leyden Jar controller does not measure row voltage with the RP2040 ADC.
It uses:

- an MCP4716 I2C DAC to generate `V_REF`
- two LMV339 quad comparators for eight row channels
- RP2040 PIO to strobe columns and sample comparator outputs

The row sense node is analog, but the firmware only receives comparator bits.
The diagnostic "level" reported by the firmware is therefore a DAC threshold
crossing value, not a direct capacitance or ADC voltage reading.

From the schematic, `ROW_PULLUP` is not simply a digital pull-up on the row
input. It appears to be an analog row bias/precharge rail:

```text
3.3V -- 7.5k -- ROW_PULLUP -- 1k -- GND
```

So the bias point is approximately:

```text
Vbias = 3.3V * 1k / (7.5k + 1k)
      ~= 0.39V
```

Each analog row node, for example `A_ROW0`, is tied to that bias through a
47k resistor. Separately, each LMV339 digital output, for example `ROW0`, has
its own 10k pull-up because the comparator output is open collector.

That means the useful model for a row is not "discharge through a pull-down".
It is closer to:

```text
Vrow(t) = Vbias + (Vrow(0+) - Vbias) * exp(-t / tau)
```

The column rising edge kicks the row voltage in one direction relative to
`Vbias`. The falling edge should kick it in the opposite direction. The row
then returns toward the bias point through the row bias network and parasitic
paths.

## Current Comparator Reference

The MCP4716 DAC output is divided before it reaches the LMV339 comparator
reference input:

```text
MCP4716 VOUT -- 10k -- V_REF -- 4.7k -- GND
```

So:

```text
V_REF = V_DAC_OUT * 4.7k / (10k + 4.7k)
      ~= V_DAC_OUT * 0.3197
```

Assuming a 3.3 V DAC supply and the 10-bit DAC codes used by the firmware:

```text
V_DAC_OUT ~= code / 1024 * 3.3V
V_REF     ~= code / 1024 * 3.3V * 0.3197
```

Each DAC code is therefore about:

```text
1.03 mV at the comparator input
```

The current F122 `ACTIVATION_OFFSETS` value is `+7`, which is only about:

```text
7 * 1.03 mV ~= 7.2 mV
```

above the bucket reference at the comparator input. This means the current
scan likely spends much of its boundary behavior in the LMV339's low-overdrive
region. Strong pressed keys may have much larger overdrive, but marginal keys
and released/pressed decision edges will be affected by comparator propagation
delay, output pull-up rise time, row residual charge, and noise.

## Current PIO Timing

The reference PIO programs run at about 10 MHz, so each PIO cycle is about
0.1 us.

The current scan shape for a column is approximately:

```text
drive selected column high
sample row comparator bits about 0.9 us later
hold column high until about 10 us
drive column low around 13.1 us
wait until the 40 us column slot completes
```

So each column gets about:

```text
40 us total slot
~27 us low/recovery time before the next column's rising-edge sample
```

For F122:

```text
18 columns * 40 us = 720 us per threshold scan
```

## Current DAC And Bucket Behavior

The F122 configuration uses:

```c
#define NB_CAL_BINS         7
#define NB_CUSTOM_CAL_BINS  2
#define ACTIVATION_OFFSETS  {7,7,7,7,7,7,7}
```

The number of buckets is fixed at compile time by `NB_CAL_BINS`; calibration
does not choose it dynamically.

Calibration does this:

1. Sweep DAC values from 350 to 499.
2. At each DAC value, wait 100 us and run a full PIO scan.
3. Record the last/maximum threshold where each key position appears active.
4. Sort key positions by custom bin and measured level.
5. Divide the non-custom positions into equal-size standard buckets.
6. Use the median level of each bucket as the bucket reference.
7. Set each bucket threshold to:

```text
threshold = bucket_median + activation_offset
```

Then the firmware reassigns keys to the closest bucket reference level, as
long as the activation offset matches, and merges buckets that ended up with
identical reference levels.

During normal scanning, the firmware scans once per active bucket:

```text
for each active bucket:
    set DAC threshold for that bucket
    wait 100 us
    scan all columns with PIO
    merge only the keys assigned to this bucket
```

Worst case for seven active buckets:

```text
7 * 720 us = 5.04 ms of PIO scan time
7 DAC writes over 400 kHz I2C
7 explicit 100 us waits
firmware overhead
```

A practical full raw scan is likely around 6-7 ms before normal QMK debounce.

The DAC write itself cannot be done inside a 40 us column slot. The firmware
writes two bytes over 400 kHz I2C:

```text
address byte + ack: 9 clocks
data byte 1 + ack:  9 clocks
data byte 2 + ack:  9 clocks
total:              27 clocks

27 / 400 kHz ~= 67.5 us
```

That is before API overhead and before the current explicit 100 us wait after
changing the DAC value.

## Open Questions

The important unknowns are empirical:

- How much margin is there between pressed and released threshold levels?
- How much do levels drift across columns, rows, temperature, and time?
- Is the current 27 us low/recovery interval enough for the previous column's
  falling-edge transient to stop disturbing the next column?
- How often do apparent false positives occur as one-sided comparator noise?
- Do marginal keys fail in a way that follows the previous column, which would
  indicate residual row charge or recovery interaction?
- How many calibration buckets are actually active after merge on a real F122?
- Is generic QMK debounce masking analog scan instability, or just adding
  unnecessary latency after an already stable capsense result?

## Backup Approach: Adjacent-Bucket Hysteresis

The current design already scans several ordered thresholds. That can be used
as a Schmitt-style state machine instead of a time debounce.

For each key, keep its assigned bucket as the center point. Then choose the
test threshold based on the current logical state:

```text
currently released:
    require active at a stricter adjacent bucket before declaring pressed

currently pressed:
    require inactive at an easier adjacent bucket before declaring released
```

Assuming Model F polarity means a higher DAC threshold is stricter:

```text
press test bucket:   assigned_bin + 1
release test bucket: assigned_bin - 1
```

Both values would be clamped at the valid bucket range.

This would require preserving per-bucket scan results instead of immediately
collapsing them into one merged matrix:

```c
uint8_t raw_by_bin[NB_CAL_BINS][CONTROLLER_COLS];
```

For the F122, that is small:

```text
7 bins * 18 columns = 126 bytes
```

Advantages:

- Uses scans the firmware is already doing.
- Adds little RAM cost.
- Can replace time debounce with threshold hysteresis.
- Should reduce latency compared with millisecond debounce.

Risks:

- Adjacent buckets may be too widely spaced for some keys.
- Bucket spacing is based on population medians, not guaranteed equal voltage
  intervals.
- Keys at the first or last bucket have one-sided hysteresis unless special
  handling is added.
- If a key is assigned to a custom bucket, adjacent bucket semantics may not
  be physically meaningful.

Possible refinement:

Instead of blindly using `bin +/- 1`, compute per-key `press_bin` and
`release_bin` during calibration based on measured level and desired margin.

## Backup Approach: Per-Key Window And Bucket Coverage

A stronger version of adjacent-bucket hysteresis is to define a fixed
hysteresis window for each key, then use the bucket progression to determine
which side of the window the key is on.

For each key, calibration would derive or store:

```text
baseline_level
release_edge
press_edge
```

At runtime:

```text
currently released:
    become pressed only when the observed strength is beyond press_edge

currently pressed:
    become released only when the observed strength is below release_edge

inside the window:
    keep the previous state
```

For the Model F level convention used by the diagnostic tool, higher inferred
level means a stronger pressed-looking signal. So the state machine is:

```text
if released:
    press only when inferred_level >= press_edge

if pressed:
    release only when inferred_level <= release_edge

if release_edge < inferred_level < press_edge:
    retain previous state
```

The useful part is that the existing bucket scans can approximate
`inferred_level`. Once per-bucket raw data is preserved, the firmware can
determine how far the key's current response got through the ordered threshold
set.

Ideally, each key should have enough scanned thresholds around its window:

```text
at least two thresholds below the release edge
release edge
hysteresis window
press edge
at least two thresholds above the press edge
```

This gives the resolver room to distinguish:

```text
clearly released
weak release candidate
inside hysteresis window
weak press candidate
clearly pressed
```

The important measure should be threshold distance, not just bucket index.
The current buckets are population buckets, so adjacent bucket indices are not
guaranteed to represent equal DAC spacing. Custom bins and the post-calibration
optimization pass also make pure index distance less reliable.

Better checks:

```text
do at least two active thresholds exist below release_edge?
do at least two active thresholds exist above press_edge?
are those thresholds close enough to be useful?
are they far enough away to reject noise?
```

If a key does not have enough threshold coverage, possible fallbacks are:

- use a smaller per-key window
- use a wider confirmation distance for only that key
- keep ordinary QMK debounce for that key
- add more calibration buckets
- compute bucket thresholds by voltage spacing rather than equal population

This model turns debounce into a threshold/state problem rather than a time
delay problem.

## Backup Approach: Preserve Per-Bucket Strength Information

Once per-bucket raw data is kept, a key's "strength" can be inferred from the
highest threshold bucket at which it still appears active.

That gives more than a binary key state:

```text
not active in any bucket  -> weak/inactive
active only in easy bins  -> marginal
active in assigned bin    -> normal
active in stricter bins   -> strong
```

This can be used for:

- transition confidence
- diagnostics
- adaptive thresholds
- identifying noisy or marginal positions
- deciding whether a transition needs confirmation

The distance from the key's hysteresis window gives an even stronger signal
than a binary bucket result. For example:

```text
far below release edge: clearly released
near release edge:      marginal release
inside window:          retain previous state
near press edge:        marginal press
far above press edge:   clearly pressed
```

In bucket-count form, the resolver could use:

```text
distance >= 2 buckets:
    accept transition immediately

distance == 1 bucket:
    require another agreeing scan, or mark marginal

inside window:
    keep previous state

single opposite blip:
    ignore
```

The better version uses DAC-unit distance instead of bucket count:

```text
confidence = abs(observed_edge_level - window_edge)
```

This produces a physically meaningful confidence score:

```text
far from threshold = stable
near threshold     = marginal/noisy
inside window      = state hold
```

It should also make diagnostics more useful. The firmware or diagnostic tool
can report keys that spend time near their window, require repeated
confirmation, or change confidence depending on previous column order. Those
are the positions where the analog scan needs attention.

This should be tested before making the state machine too clever.

## Diagnostic Approach: Both-Edge Sampling

Because the row is biased around 0.39V, both the rising and falling column
edges may be useful.

Expected behavior:

```text
rising column edge:   row voltage moves above Vbias
falling column edge:  row voltage moves below Vbias
```

The existing PIO slot has unused time after the column is driven low. A second
PIO sample after the falling edge should fit in the column slot.

A real key event should produce a correlated opposite-polarity pair:

```text
positive transient on rising edge
negative transient on falling edge
```

One-sided events are more likely to be noise, residual charge, comparator
chatter, or a threshold artifact.

The clean two-threshold test would be:

```text
rising edge:  A_ROW > V_HIGH
falling edge: A_ROW < V_LOW
```

But the board has one I2C DAC threshold, and it cannot switch fast enough
inside a column slot. A true `V_HIGH`/`V_LOW` implementation would need two
passes:

```text
set V_HIGH
scan positive edge

set V_LOW
scan negative edge

combine results
```

That would roughly double scan time if done for every bucket every scan.

However, the two thresholds do not need independent calibration tables. If the
row bias and positive threshold are known, the low threshold can likely be
derived:

```text
V_LOW = Vbias - k * (V_HIGH - Vbias)
```

or in DAC units:

```text
low_threshold = bias_dac - k * (high_threshold - bias_dac)
```

The coefficient `k` may not be exactly 1 because the falling-edge response is
not necessarily symmetric with the rising-edge response. The column has been
held high for about 10 us before the falling edge, so the row state at the
falling edge depends on the prior high-time recovery.

Possible both-edge strategies:

- Single-threshold both-edge sample: cheap, but likely fragile.
- Full two-pass high/low threshold scan: strongest signal, highest cost.
- Transition-only negative-edge confirmation: normal scan stays fast; only
  changed or marginal keys get extra confirmation.
- Diagnostic-only both-edge scan: gather data first, then decide whether it is
  worth using in normal typing.

## Support Experiment: Tune Scan Timing

The current timing appears empirical:

```text
sample about 0.9 us after rising edge
drive low around 13.1 us
next column samples about 27 us after low
```

Once the keyboard is available, useful experiments are:

1. Increase the low/recovery interval and see whether detected levels or
   marginal positions change.
2. Decrease the low/recovery interval until errors appear.
3. Change column scan order and see whether failures follow a previous column.
4. Add a later positive sample during the high interval and compare it to the
   early sample.
5. Add a falling-edge sample and record whether it correlates with the rising
   sample.
6. Measure how many bins remain active after calibration on the real board.

If a longer recovery interval significantly changes levels, the current scan
is interacting with residual row state from the previous column.

If changing column order moves failures, that strongly suggests inter-column
coupling or incomplete recovery.

## Preferred Approach: Time-Domain Calibration

The current firmware creates numeric-looking "levels" by sweeping the DAC
threshold and doing repeated digital comparator scans. A time-domain scheme
would invert that:

```text
current scheme:
    fixed sample time
    sweep comparator threshold
    record the highest DAC threshold where a key still reads active

time-domain scheme:
    fixed comparator threshold
    sample repeatedly after the column strobe
    record when the comparator output drops out
```

The measured value would be a crossing/dropout time, not a voltage:

```text
observed_duration = last sample time where the key still reads active
```

For a simple row decay:

```text
Vrow(t) = Vbias + A * exp(-t / tau)
```

with a fixed comparator threshold:

```text
t_cross = tau * ln(A / margin)
```

So a larger initial transient, or a slower decay, stays active longer and
produces a later dropout time.

This maps well to what the RP2040 actually sees. The firmware does not get an
ADC voltage; it gets digital comparator outputs. Repeated PIO samples over
time can therefore produce a real runtime measurement without extra DAC
threshold passes.

Boot/autocalibration would change meaning. Instead of finding a threshold that
is above the resting/released level, it would find the time by which released
keys have reliably dropped out:

```text
released_dropout_time[key] =
    latest time a released key still appears active after a strobe
```

The simplest runtime decision is then:

```text
sample after released_dropout_time + margin
if still active:
    pressed
else:
    released
```

The global safe version would be:

```text
global_sample_time = max(released_dropout_time for all keys) + margin
```

That is simple, but may be slower than necessary. A per-key or bucketed version
could keep the scan fast:

```text
fast released dropout -> earlier decision point
slow released dropout -> later decision point
```

The time-domain version also gives the missing data needed for dynamic
calibration:

```text
if confidently released:
    update released_duration_estimate

if confidently pressed:
    update pressed_duration_estimate

if near the decision window:
    do not update calibration
```

The state window becomes time-based:

```text
release_edge_time = released_estimate + low_margin
press_edge_time   = released_estimate + high_margin
```

or, after pressed behavior has been learned:

```text
release_edge_time =
    released_estimate + k1 * (pressed_estimate - released_estimate)

press_edge_time =
    released_estimate + k2 * (pressed_estimate - released_estimate)

where 0 < k1 < k2 < 1
```

This would let the keyboard track drift without extra DAC sweeps:

- released baseline drift changes released dropout time
- pressed response changes pressed duration
- boot-held-key mistakes can be corrected after later confident samples
- marginal keys can be detected because they spend time near the window

This does not make the analog model trivial. The measured dropout time still
includes:

- row transient amplitude
- row decay time constant
- comparator propagation delay
- comparator overdrive
- open-collector output rise through the 10k pull-up
- prior row/column residual charge
- column edge rate through the 1k series resistors

But those effects are part of the actual signal path being sampled. A
time-domain calibration may therefore be more honest than pretending the
firmware has a direct voltage measurement.

Useful experiments:

1. Keep a fixed DAC threshold and sample each column at several times, such as
   0.5 us, 1 us, 2 us, 4 us, and 8 us.
2. Encode each row as the last active sample index.
3. Compare released keys, pressed keys, and same-row multi-key cases.
4. Check whether dropout times are stable over repeated scans.
5. Check whether dropout times move with column order or held same-row keys.
6. Compare dropout-time classification against the existing DAC-bucket
   classification.

If stable, this approach could replace the current multi-pass DAC bucketing
with a single full-matrix scan that still returns a strength/confidence value.

### Reference Control

Under the time-domain scheme, the comparator reference should normally stay
fixed during runtime scanning. The DAC becomes a slow control input, not a
per-bucket scan input.

The desired released-key waveform is:

```text
early samples: active / 1
middle samples: dropout occurs
late samples: inactive / 0
```

So initial calibration should tune `V_REF` until released keys drop out inside
the sample train:

```text
zero too early, such as by sample 3:
    V_REF is probably too high, or the first useful samples are too late
    lower V_REF

ones run off the end for released keys:
    V_REF is probably too low, or the sample window is too short
    raise V_REF

both happen across released keys:
    the sample window is too narrow for the population
    increase the sample span or report invalid timing
```

Pressed keys running off the end with ones is not an error. That is the
expected strong-press/saturated case. Calibration adjustment should be based on
confident released keys, not on keys that are currently pressed or marginal.

The reference should be held near the useful knee of the decay curve:

```text
too high:
    dropout happens too early and timing is dominated by comparator delay,
    edge rate, and tiny differences

too low:
    dropout happens too late or runs off the end, where residual charge and
    noise dominate

near the knee:
    dropout timing has useful slope and gives the per-key resolver room
```

After boot, runtime adjustment should be damped:

- small DAC steps
- rate limited
- based on repeated evidence
- driven only by confident released-state samples
- disabled or slowed during heavy typing
- rejected if early and late failures both persist

This gives a stable loop:

```text
boot calibration finds a useful operating region
runtime scans produce dropout measurements
released-state evidence slowly corrects drift
pressed-state evidence learns press duration/window
QMK sees only resolved stable key states
```

Because meeting the 1 ms scan target should be easy with one fixed-threshold
scan, the sample train should have a conservative minimum length. There is no
need to tune the column slot to the shortest possible value if a longer train
gives better recovery visibility and more stable calibration.

## Preferred Runtime Architecture: PIO And DMA

If the time-domain scheme works, the clean runtime architecture is:

```text
M-core:
    configure scan buffer
    start DMA from PIO RX FIFO to RAM
    feed/start PIO scan
    wait for scan complete / DMA complete
    parse waveform buffer into dropout times
    resolve key states
    hand final matrix to QMK

PIO:
    drive columns with deterministic timing
    sample row comparator outputs at fixed delay points
    push packed row samples to RX FIFO
    signal scan completion

DMA:
    move PIO RX FIFO words into the scan buffer
    pace transfers from the PIO RX DREQ
```

This keeps each part doing the job it is good at:

- PIO provides deterministic column strobe and sample timing.
- DMA drains the PIO RX FIFO without CPU polling jitter.
- The M-core parses the waveform and runs the capsense state machine.
- QMK receives only the resolved keyboard matrix.

The raw data rate is small for the RP2040, but too large to treat casually with
the tiny PIO RX FIFO. For example:

```text
8 samples per column
18 columns
1 byte row bitmap per sample
= 144 bytes per full scan

4 row samples packed per 32-bit word
8 samples per column = 2 words per column
18 columns = 36 words = 144 bytes per scan
```

At 1 kHz full scans, that is only about:

```text
144 KB/s
```

which is trivial for the M-core and SRAM. The constraint is not CPU bandwidth;
it is keeping the PIO state machine from blocking on a full RX FIFO while
preserving deterministic sample timing. DMA is the right fit for that.

A scan buffer can be very small:

```c
uint32_t scan_words[36];  // example: 8 samples/column, 18 columns
```

Double buffering is still cheap:

```text
2 * 144 bytes = 288 bytes
```

The parser would convert the packed waveform into values such as:

```text
dropout_index[col][row]
observed_duration[col][row]
confidence[col][row]
resolved_matrix[row]
diagnostic counters
```

Important implementation constraints:

- PIO should push a fixed, known number of words per scan.
- DMA transfer count should exactly match the expected scan word count.
- DMA completion and PIO scan completion must describe the same scan.
- The RX FIFO must never fill in a way that stalls PIO and changes timing.
- The column schedule should leave enough recovery time for row decay.
- Diagnostic capture should reuse this buffer path rather than add a separate
  scanner.

For diagnostics, the firmware should not stream the full waveform to the host
continuously. Instead, use:

- snapshot capture: host requests one scan or a short burst
- summary capture: host reads dropout/confidence values
- triggered capture: store waveforms only for marginal or suspicious events

Normal USB/HID reports should remain the resolved QMK matrix/report path.

## Final Integration: Reduce Or Remove QMK Debounce

If the time-domain scanner produces stable logical transitions, generic QMK
debounce should be reduced or removed.

The desired final model is:

```text
capsense scan produces stable logical transitions
QMK debounce is disabled or reduced to a minimal safety net
```

Reasons:

- Model F capacitive sensing has no metal contact bounce.
- A long debounce hides analog threshold problems instead of describing them.
- Time-domain dropout hysteresis can react faster than time debounce.
- Confidence from dropout duration can reject marginal one-scan noise without
  adding a fixed millisecond delay.
- Backup methods such as adjacent-bucket hysteresis or both-edge validation can
  still be tested if the preferred dropout-time resolver is not sufficient.

This should only be changed after collecting real keyboard data.

## Latency Target And USB Polling

The RP2040 USB device is full-speed USB. For HID interrupt IN endpoints, the
device descriptor requests a polling interval, and the host polls the endpoint.
The device does not asynchronously interrupt the host.

In this QMK tree, the USB endpoint descriptor uses:

```c
#ifndef USB_POLLING_INTERVAL_MS
#    define USB_POLLING_INTERVAL_MS 1
#endif
```

So the normal USB poll interval is 1 ms unless explicitly overridden. For
full-speed USB, 1 ms is effectively the practical floor. High-speed USB can use
125 us microframes, but the RP2040 device controller is not high-speed.

This means there is little value in making the matrix scan radically faster
than the USB reporting cadence if doing so harms analog margin.

Useful target:

```text
full matrix scan + capsense resolver < 1 ms
```

With a sub-1 ms scan, the remaining host-visible latency is mostly:

```text
time until next matrix scan observes the change
time until next USB host poll collects the report
firmware/report overhead
OS/application latency
```

For example, if a future single-pass scan used the existing 40 us column slot:

```text
18 columns * 40 us = 720 us
```

That is already below the USB poll interval. If in-slot confidence or
delay-based sampling can replace the current multi-bucket DAC passes and QMK
time debounce, the result would already be very competitive.

The current design is much slower because it does:

```text
~720 us per bucket scan
up to 7 bucket scans
7 DAC writes
7 explicit DAC settle waits
normal QMK debounce, currently 10 ms in f122/keyboard.json
```

The performance goal should therefore be:

1. classify the analog state reliably
2. avoid generic millisecond-scale debounce
3. avoid multiple full-matrix DAC threshold passes if possible
4. keep the full scan under about 1 ms
5. only then tune column timing lower

There is no strong reason to chase the absolute minimum possible delay between
columns if a conservative column slot gives better analog fidelity and the full
scan still stays under the USB polling interval.

## Data To Collect When The Keyboard Arrives

Capture these values from the existing utility/protocol paths before changing
the scan logic:

- per-column/per-row detected levels
- bucket reference levels
- bucket thresholds
- bin map
- active bucket count after merge
- results with no keys pressed
- results with representative keys held
- results with several keys held during boot calibration

Then repeat with experimental firmware:

- fixed-`V_REF` time-domain scan
- several sample schedules, including conservative long-window schedules
- dropout index/duration capture per key
- first-sample and last-sample anchor failures
- same-row multi-key dropout changes
- PIO+DMA waveform capture
- reduced QMK debounce
- disabled QMK debounce
- adjacent-bucket hysteresis only as a backup comparison
- falling-edge diagnostic sampling only as a secondary experiment

For each run, record:

- calibration levels
- fixed `V_REF` DAC code
- dropout duration distribution
- early-fail count, where released keys drop out too soon
- late-fail count, where released keys run off the end
- false press count
- missed press count
- perceived latency
- whether failures cluster by row/column
- whether failures depend on previous scanned column

## First Implementation Candidate

The first serious implementation candidate is the fixed-reference time-domain
scanner with PIO+DMA capture.

Suggested implementation shape:

1. Keep the current DAC-bucket scan available as the known-good baseline.
2. Add an experimental fixed-`V_REF` scan path.
3. Use PIO to sample row comparator outputs at fixed time points after each
   column strobe.
4. Use DMA to transfer packed row sample words from PIO RX FIFO to RAM.
5. Parse each column/row waveform into dropout index and confidence.
6. During boot calibration, adjust `V_REF` until released-key dropout lands
   inside the sample train.
7. During runtime, use damped released-state feedback to correct drift.
8. Feed QMK a resolved stable matrix.
9. Disable or reduce QMK debounce only after the capsense resolver is proven
   stable.

The state resolver should start simple:

```text
released key:
    become pressed only if observed dropout duration is beyond press_edge_time

pressed key:
    become released only if observed dropout duration is below release_edge_time

inside the time window:
    retain previous state
```

The first calibration target should be conservative:

```text
released keys start with 1
released keys end with 0
released dropout is not before the minimum sample-time floor
released dropout is not off the end
full scan plus resolver stays below about 1 ms
```

If this path fails because dropout timing is too noisy or too dependent on
same-row held keys, fall back to preserving per-bucket DAC scan results and
using adjacent-bucket hysteresis. That fallback uses the current analog method
but adds a better state resolver.

## Files Of Interest

Firmware:

- `vial-qmk/keyboards/leyden_jar/common.c`
- `vial-qmk/keyboards/leyden_jar/dac.c`
- `vial-qmk/keyboards/leyden_jar/pio_matrix_scan.c`
- `vial-qmk/keyboards/leyden_jar/reference_pio_programs/col_0_7_pio.pio`
- `vial-qmk/keyboards/leyden_jar/reference_pio_programs/col_8_15_pio.pio`
- `vial-qmk/keyboards/leyden_jar/reference_pio_programs/col_16_17_pio.pio`
- `vial-qmk/keyboards/leyden_jar/f122/config.h`

Hardware:

- `Leyden_Jar/pcb/leyden_jar_controller/Leyden_Jar.kicad_sch`
- `Leyden_Jar/pcb/leyden_jar_controller/Leyden_Jar.kicad_pcb`
