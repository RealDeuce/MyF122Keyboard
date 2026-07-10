# F122 Model F Linux Setup Manual

Source manual reviewed: https://www.modelfkeyboards.com/manual/

This is a Linux-focused, F122-specific rewrite of the Model F Labs manual. It keeps the parts that apply to a new-production F122 buckling spring keyboard using a Leyden Jar controller, and removes the unrelated host-platform, xwhatsit/Atmel, original-IBM-controller, F62/F77-only, and beam-spring-module material unless it affects the F122 directly.

## What This Keyboard Is

The new-production F122 is not delivered as a finished appliance in the normal commodity-keyboard sense. It is a capacitive buckling-spring assembly that usually needs final setup after shipping:

- Springs can shift during shipping.
- Keycaps can need reseating.
- Stabilizer inserts must be installed correctly before the wide keys.
- The spacebar may need wire/tab adjustment.
- Some keys may initially miss, double, transpose, or feel wrong until the spring and keycap are corrected.

That is normal setup work for this design. Do not plug the keyboard in until the physical setup steps say to do so. With no keycaps installed, the flippers can rest on the PCB and confuse calibration.

## Useful Tools

Have these available before starting:

- Precision screwdriver set with Phillips and Torx T8 bits.
- Wire keycap puller.
- Tweezers for spring work.
- Pliers.
- Five or six 2 inch C clamps if you ever open the inner assembly.
- Soldering iron only for rare ribbon/trace repair.
- Dry lint-free cloth or microfiber cloth.
- Mild dish soap and lukewarm water for keycap cleaning.

For normal first setup, the key puller and tweezers matter most. The clamps and soldering iron are only for deeper repair.

## Parts To Inspect First

Inspect everything within a few days of delivery.

- Confirm keycaps, springs, flippers, feet/bumpers, cable, and any ordered accessories are present.
- Do not install a keycap if its stem is chipped or visibly different from the others.
- Check every spring and flipper before installing keycaps. The spring should be present and the flipper should move freely.
- If a flipper is stuck, gently nudge it with tweezers. If it cannot be freed from above, the keyboard may need to be opened, but remove all keycaps first.
- Barrels without springs/flippers are intentional under stabilized wide-key positions. Do not "fix" those by adding flippers unless you are deliberately changing the physical layout.

For ANSI-style horizontal Enter and a 2U Backspace, the left stabilizer barrel for each wide key normally has no flipper. For full-size Shift keys, one of the two barrels is also intentionally empty. On your current F122 setup, the real physical layout is ANSI with split Backspace, so treat the actual installed barrels/flippers as authoritative.

## Stabilizer Inserts

Install stabilizer inserts before installing ordinary 1U keys. If an insert is installed in the wrong barrel or wrong orientation, removal can require opening the keyboard or using specialized extraction methods.

There are two stabilizer insert styles:

- Horizontal inserts are normally white. They are used for 2U and wider horizontal keys, except the spacebar.
- Vertical inserts are normally black. They are used for 2U vertical keys, such as numpad vertical keys.

Color is not enough by itself: identify the insert by shape. Vertical inserts have uneven ears.

Rules:

- Do not put a stabilizer insert into a barrel that has a flipper.
- Spacebar does not use these inserts. It uses the wire stabilizer and metal tabs.
- Install horizontal inserts with the internal cavity number toward the top of the keyboard, ears left/right, pressed flush with the top of the barrel.
- Install vertical inserts initially about 0.5 to 1 mm proud of the barrel. If a vertical key binds, height can matter.
- ISO Enter has special vertical-insert orientation, but that is not relevant to your ANSI F122 layout.

If a wide key binds later, the insert height, insert type, spring position, and key stem fit are the first things to suspect.

## Spacebar Setup

Do the spacebar before filling the board with the rest of the keycaps. It is much easier to correct while the surrounding keys are absent.

The Model F spacebar is intentionally adjustable. Some looseness, rattle, and small rotation are normal for the design. The target is reliable return and acceptable feel, not Cherry-style immobility.

Check:

- The spacebar actuates from the center and from off-center presses.
- It returns immediately and fully.
- It does not scrape or stick.
- It does not hit adjacent keys.
- It has the sound and weight you can tolerate before the rest of the keys make access harder.

If the spacebar does not actuate, first test the spring position with a 1U keycap in that barrel. If the 1U key works, the problem is the spacebar installation or stabilizer geometry, not the switch position.

Adjustment points:

- The wire should sit under the top of both metal tabs.
- If the wire catches the back of the tabs, bend the wire slightly so there is roughly a small visible clearance. The manual describes about 0.5 to 1 mm as the kind of scale involved.
- If the spacebar contacts nearby keys, adjust the wire shape so the spacebar sits farther from the interfering key. A small piece of card or foam between wire and tab has been used as a spacer, but use this only if geometry adjustment is not enough.
- If the wire ends are too wide and hit nearby Alt keys, remove the wire and gently move the left/right ends inward with pliers.
- Always remove the wire from the spacebar before bending it. Pulling or pushing the wire while it is clipped into the spacebar can break the small plastic tabs.
- If the spacebar is squeaky or does not buckle cleanly, suspect the spring first. Adjust, rotate, flip, or replace the spring before blaming the wire.
- Carefully pushing the metal tabs down can reduce rattle, but too much will increase force or cause sticking. If overdone, nudge the stabilizer wire upward with a screwdriver or tweezers to lift the tab slightly.

Do not glue the spacebar stabilizer parts into rigidity. The spacebar needs some movement to work reliably.

## Spring And Keycap Installation

Key installation is also spring inspection. Do not just press all keycaps onto the keyboard and then test at the end.

Use this process:

1. Hold the keyboard vertically with the spacebar side up.
2. Inspect the spring in the barrel before installing the keycap.
3. The spring should be close to the barrel edge, not centered in the barrel.
4. The spring end should be around the 12 o'clock position relative to the flipper.
5. The spring should be fully seated on the flipper nub.
6. Install the keycap slowly so the spring enters the key stem cleanly.
7. Press the key several times and listen for a clean buckle/click.
8. Do not set the keyboard flat until the key has been fully seated.

Spring correction:

- Remove a spring by gently twisting counterclockwise while looking down at it.
- Never pull a spring straight up.
- Reinstall by pressing straight down onto the flipper nub.
- If a spring buzzes, misses, double-presses, or produces a bad click, remove and reseat it.
- If reseating does not work, flip the spring upside down and try again.
- If that still does not work, replace the spring. A bent spring is usually not worth saving.

Some keys may need a few dozen presses to settle after correct installation. Reinstalling the keycap over and over without changing the spring position is usually wasted effort.

## Wide Keys And Binding

Install and validate all 2U and larger keys before the rest of the board. They are the keys most likely to bind.

If a wide key is physically stuck, use the "wiggle method" only for that kind of binding. It is not a cure for a key that fails electrically or fails to buckle.

Process:

- Confirm the spring is correctly placed first.
- Confirm the stabilizer insert is the right type, in the right barrel, and at a workable height.
- Gently pinch the affected key's outer stem/ear and stabilizer post area and wiggle side to side 20 to 30 times.
- Gently wiggle the stabilizer post in the side-to-side direction about 10 times.
- Burnish the back and angled bottom edge of the key stem with a fingernail.
- Reinstall and test 10 to 20 presses.
- Repeat only a few times if it improves.

Do not sand keycaps or add lubricant as a first-line fix. Lubricant and foam spacers are last-resort community fixes, not normal setup.

For 2U vertical keys, be especially gentle. Their stems are easier to break, and sometimes horizontal inserts work better than vertical ones, but that is a troubleshooting exception.

If a key is too loose and pops off, gently widen the two legs of the key stem slightly, using other key stems as a reference.

## First Electrical Test

Before plugging in:

- Every physical key position that should have a keycap must have one.
- Every installed key should produce a clean snap/click when pressed.
- No spring should feel like it is just compressing without buckling.

Then plug the keyboard directly into the computer. Avoid hubs, KVMs, extension cables, underpowered front-panel USB ports, and suspect third-party cables during troubleshooting.

Basic test:

- Use an ordinary keyboard tester or a text editor for a first pass.
- Press every key several times.
- A factory Fn key may not show on ordinary key testers. That is normal because layer keys are consumed by firmware.
- Close the Leyden Jar diagnostic tool before opening Vial or Vial.Rocks.

If a key misses, repeats, or transposes with another key, go back to spring correction before assuming electronics are bad.

## Leyden Jar Signal-Level Testing

The F122 uses the Leyden Jar controller, so use the Leyden Jar diagnostic tool, not the pandrew/xwhatsit tool.

Source:

- Diagnostic tool: https://github.com/mymakercorner/Leyden_Jar_Diagnostic_Tool
- Firmware tree: https://github.com/mymakercorner/vial-qmk/tree/leyden_jar

Use:

1. Plug in the keyboard.
2. Open the Leyden Jar diagnostic tool.
3. Click Refresh.
4. Select the keyboard if needed.
5. Open the Level Monitor.

Important behavior:

- The diagnostic tool does not auto-refresh after unplug/replug. Click Refresh again.
- Quick taps are not enough. Press and hold a key for several seconds when checking levels.
- The three numbers shown for a key are highest/current/lowest style values.
- A normal unpressed value is often around 110 to 125.
- A pressed value should rise enough to cross the bin reference threshold; 150+ is the rough scale from the manual, with 155 to 175+ described as a healthier range.
- Leyden Jar may show lower-strength keys in light blue when they are close to the bin reference level. Variation is normal. The problem is a pressed key not reaching the reference level.
- Do not use the Key Press Monitor in the diagnostic tools, Vial, QMK Toolbox, or similar tools for Model F capacitive diagnosis. Use Level Monitor.

If removing the keycap lets the flipper move freely and the level changes, the keycap/spring installation is still suspect. If the key is inconsistent after spring correction, suspect debris, PCB position, grounding, or a rare solder/ribbon issue.

## Troubleshooting Order

Use this order instead of randomly changing things.

1. Reseat the keycap slowly with the keyboard vertical.
2. Reseat the spring.
3. Flip the spring and retry.
4. Replace the spring.
5. Try another known-good keycap in the same barrel.
6. Check that the flipper moves freely.
7. Unplug the keyboard for several minutes, then plug it back in and retest.
8. Tighten the two controller grounding screws.
9. Use the Leyden Jar Level Monitor.
10. If values vary widely for the same held key, suspect dust/debris or PCB position.
11. If an entire matrix row/column is affected, suspect ribbon soldering or a short.
12. Only then consider opening the inner assembly.

Common symptoms:

- No click or bad click: spring/keycap installation.
- Double presses or transposed fast key order: usually spring seating, damaged spring, loose controller grounding screws, or debris.
- One scattered key not working: almost never a bad controller. Check spring, cap, flipper, and debris first.
- Entire matrix row/column not responding: possible ribbon solder joint or short.
- Many keys reading 0 in matrix view: possible short to ground near controller/ribbon area.
- Random disconnect/reconnect: avoid hubs/KVMs/extensions, try another USB port, try a motherboard USB 2.0 port, try another cable.
- Keyboard not detected at all: test another cable/port/computer, ensure the USB-C connector is fully seated on the controller, then consider boot pads/bootloader recovery.

If dust/debris is suspected but you do not want to open the keyboard yet:

- Remove the affected and nearby keycaps.
- Hold the keyboard at different angles.
- Move the springs/flippers by hand.
- Retest.

Compressed air can sometimes move debris, but it can also move debris somewhere else. Opening and wiping the PCB with a dry lint-free cloth is the more deterministic fix if the problem persists.

## Grounding, PCB Position, And Ribbon Issues

The controller grounding screws matter. If they loosen in shipping, the keyboard can produce spurious output, repeated keys, transposed keys, or unreliable modifier chords.

For deeper issues:

- The large capacitive PCB must be seated correctly on the bottom inner assembly posts.
- The ribbon cable should not be flexed unnecessarily, especially at its ends.
- If a whole trace group is dead, use matrix view and PCB trace diagrams to determine the affected through-hole. The manual's trace gallery is here: https://www.modelfkeyboards.com/rl_gallery/capacitive-pcbs/
- Many ribbon/through-hole repairs can be touched up from the accessible side after loosening the controller, without opening the inner assembly.
- If two key groups fire together, inspect for solder bridges and remove only the bridge, taking care not to cut traces.

Soldering is a rare repair path, not normal setup.

## Optional Solenoid

The F122 can use the solenoid and solenoid driver. The instructions are electrically the same for Leyden Jar and xwhatsit controllers, but the physical mounting differs by case.

Critical rules:

- Match VCC to VCC. On the connectors, the square pad marks the voltage pin. Connect square pad to square pad.
- If the cable is reversed, you can destroy the controller, solenoid driver, or solenoid.
- Push the cable fully into both connectors.
- Do not overtighten screws into the solenoid.
- Use the rubber washers/spacers where required so the screws do not puncture the solenoid's blue tape.
- For classic Model M style F104/FSSK/F122 cases, mount the solenoid directly to the bottom case with the two small M3 x 3 mm bolts. The F62/F77 L bracket is not needed for the F122.
- The F122/F104/FSSK die-cast Model M style plates are thick. The manual suggests a solenoid throw around half to three-quarters of maximum as a reasonable starting point.

If the solenoid does not work:

- Verify firmware supports it.
- Verify EEPROM/config has not disabled it.
- Move the plunger by hand a few times.
- Ensure the movable bracket is not touching the cylinder.
- Loosen and reposition the bracket if needed.

Stock firmware shortcuts may include:

- `Fn+Space+T`: toggle solenoid.
- `Fn+Space+=`: increase dwell.
- `Fn+Space+-`: decrease dwell.

Your RealDeuce firmware/keymap may intentionally differ from the stock manual.

## Firmware Flashing On Linux

The F122 with Leyden Jar is RP2040-based and uses UF2 flashing. Do not follow the dfu-programmer/Atmel/xwhatsit `.hex` instructions for this keyboard.

Firmware files:

- Factory F122 Leyden Jar firmware releases: https://github.com/mymakercorner/vial-qmk/releases
- Your local custom firmware tree: `/usr/home/admin/F122/vial-qmk`

Build your custom firmware from the local tree:

```sh
cd /usr/home/admin/F122/vial-qmk
gmake leyden_jar/f122:RealDeuce
```

The resulting UF2 will be under the QMK build output, typically in `.build/`.

Enter bootloader by one of these methods:

- Stock firmware may support `Fn+Space+R`, but on a Model F it only works after the keys are installed and the keyboard is functional.
- Use the Leyden Jar diagnostic tool: plug in, click Refresh, then click Enter Bootloader.
- If the keyboard is not detected or cannot enter bootloader from software, short the controller boot/prog pads while plugging in USB. Keep the pads shorted during plug-in, then remove the short after about a second. It may take several tries. Use the correct boot/prog pads, not reset pads.

Flash:

1. Enter bootloader.
2. Linux should see an `RPI-RP2` style mass-storage device.
3. Mount it if your desktop does not mount it automatically.
4. Copy the `.uf2` file to that drive.
5. Wait for the copy to finish and the device to reboot/disappear.
6. Unplug the keyboard, wait several seconds, and plug it back in.
7. In the diagnostic tool, click Refresh after reconnecting.

Factory F122 UF2 names from the manual include:

- `leyden_jar_f122_vial-ansi.uf2`
- `leyden_jar_f122_vial-iso.uf2`
- variants with HHKB-style split right shift, split Backspace, and Ctrl/Caps choices
- `leyden_jar_f122_vial.uf2`, described as the special all-pads-by-default firmware

For your custom physical layout, prefer your local RealDeuce build rather than reflashing a generic factory layout.

## Vial On Linux

Vial edits the keyboard configuration without reflashing firmware, when the firmware has Vial support enabled.

Rules:

- Close the Leyden Jar diagnostic tool before opening Vial or Vial.Rocks.
- Vial.Rocks requires a Chromium-family browser with WebHID support.
- The native Vial GUI is the offline alternative.
- Use only Vial, not VIA.
- The Layout tab controls split-key options when the firmware exposes them.
- Leyden Jar diagnostic-tool layout changes are temporary; persistent keymap changes belong in Vial.
- Reflashing firmware can erase dynamic settings.
- Vial features such as Combos can add latency or contribute to key-order weirdness if overused.

Stock Leyden Jar mappings mentioned by the manual may include:

- `Fn+Space+T`: solenoid toggle.
- `Fn+Space+E`: EEPROM reset.
- `Fn+Space+R`: bootloader/reset.
- `Fn+Space+D`: debug.
- `Fn+Space+N`: NKRO toggle.
- `Fn+Esc`, `Fn+F1/F2`, `Fn+F6` through `Fn+F12`, and `Fn+PrintScreen` for system/media keys.

Your RealDeuce keymap is not the stock manual keymap. Treat the manual's shortcuts as factory references only.

## NKRO

The manual says stock firmware defaults to 6KRO for broad compatibility, with NKRO available as a toggle. On the RealDeuce firmware, you have been discussing forcing NKRO, which is reasonable for a modern host stack as long as you retain a way to toggle out of it for broken firmware, KVMs, or boot environments.

Useful stock concept:

- `MAGIC_TOGGLE_NKRO` / `NK_TOGG` toggles NKRO mode.
- Stock manual mentions `Fn+Space+N` for this on Leyden Jar layouts.

If a specific machine, BIOS/UEFI, KVM, or recovery environment cannot handle NKRO, toggle to the ordinary keyboard report before using that environment.

## Opening The F122

Do not open the inner assembly casually. Remove every keycap first.

Reasons to open:

- Persistent dust/debris issue after spring/keycap fixes.
- PCB position problem.
- Layout changes involving barrels/flippers.
- Inner assembly or case alignment repair.
- Deep cleaning after liquid spill.

F122-specific notes:

- F122 case wobble can sometimes be corrected with a thin washer on an internal post, or by careful case adjustment.
- The F122 may contain two nuts/spacers used to adjust the internal angle of the higher function-key area. If found during disassembly, that is intentional.
- Use the correct screwdriver bit and firm downward pressure. A loose bit can strip screws.
- For the larger Model M style assemblies, use C clamps to close the inner assembly. The manual explicitly discourages improvised kneeling/force methods.
- When reassembling classic F104/FSSK/F122 cases, reinstall the four bolts that attach the bottom inner assembly to the top case.
- Keep the keyboard upside down while attaching the bottom case.
- Internal paint flaws or scratches are normal and not functionally important.

Some physical layout changes are possible by moving barrels/flippers, such as ANSI/ISO and split/unsplit Backspace, Shift, Enter, or spacebar positions. Not every split right-shift geometry can be changed without replacing larger parts.

## Cleaning And Spills

Routine cleaning:

- Unplug the keyboard.
- Remove keycaps with a wire puller.
- Clean keycaps and barrels with mild dish soap and lukewarm water.
- Do not clean springs/flippers unless there is residue that cannot be removed dry.
- Air-dry keycaps overnight in a non-humid place with good airflow.
- Do not reinstall keycaps after only a few hours; trapped droplets can cause capacitive sensing problems.
- Clean the case gently with soap and water. Be careful with melamine sponge, which can damage paint.

Controller warning:

- Do not get the controller wet.

Liquid spill:

- Unplug immediately.
- Disassemble.
- Dry everything thoroughly.
- Spills usually require cleaning the PCB and flippers with mild dish soap/water, then at least a day of drying.

## Noise And Feel Mods

The manual lists community mods, but none are required for setup.

Potential mods:

- Floss mod to reduce spring ringing, at the cost of possible added key weight.
- Very small amounts of grease/lubricant inside springs to reduce ringing.
- Spacebar tape, tubing, foam, or grease at stabilizer contact points.
- Solenoid lubrication if the plunger becomes sluggish.

Treat these as experiments. First get the keyboard working correctly in stock mechanical condition.

## What To Ignore For This F122

Ignore these manual sections unless you are working on a different keyboard:

- Beam spring Round 1/Round 2 module setup.
- Flyplate reattachment.
- Beam module washer, Part A/Part B, and MX stabilizer instructions.
- xwhatsit firmware flashing.
- Atmel/ATmega/dfu-programmer `.hex` flashing.
- QMK Toolbox instructions for original IBM controllers.
- F62/F77 compact/classic case conversion.
- F77 right-side-block firmware discussion.
- Non-Linux host platform setup.

For this keyboard, the relevant firmware model is: Leyden Jar, RP2040, UF2, Vial-QMK.

## Practical Setup Checklist

1. Inspect parts and keycap stems.
2. Check every flipper and spring before installing keycaps.
3. Install stabilizer inserts for wide keys.
4. Adjust/test the spacebar.
5. Hold the keyboard vertically, spacebar side up.
6. Install and test all 2U and larger keys.
7. Install remaining keys, testing click and spring behavior as you go.
8. Only after all intended keys are installed, plug the keyboard directly into the computer.
9. Test all keys in a text editor or keyboard tester.
10. Use Leyden Jar Level Monitor for any unreliable key.
11. Fix spring/keycap issues first.
12. Tighten/check controller grounding screws if there are transposed, repeated, or unreliable key events.
13. Consider debris/PCB/ribbon issues only after the above.
14. Use Vial only after closing the diagnostic tool.
15. Flash UF2 firmware only through RP2040 bootloader mass storage.
