# Rev 2 changes

Changes from rev 1, with the reason for each. Keep this list current as the schematic changes.

## Done in the schematic

### Pots removed, input level set in firmware
- **Removed:** RV1 (input level), RV2 (volume), RV3 (tone), C10, C11.
- **Added:** R17 (1M) from the C3 node to VREF. It keeps the 1M guitar input load that RV1 provided.
  C3 now feeds R1 and U1A directly, so the input buffer runs at unity gain.
- **Why:** with RV1 at mid travel, U1A saw about 250 kΩ of source impedance, which added noise and an ~80 kHz pole
  (see `studies/signal-path-response`). Tone and input trim move to the touchscreen.

### Master volume encoder
- **Added:** SW1, an EC11E-footprint rotary encoder with push switch (push = mute).
  Each line has a 10k pull-up, 10k series resistor and 10nF capacitor (R18–R23, C19–C21).
- **Daisy pins:** `ENC_A` = pin 22 (was VOLUME_POT), `ENC_B` = pin 23 (was TONE_POT),
  `ENC_SW` = pin 24 (was A2). These pins are also renamed on the J7 breakout.
- **Why:** a relative encoder never disagrees with the touchscreen, unlike an absolute pot.

### Output mute watchdog
- **Added:** U3 (SN74LVC1G123 retriggerable one-shot), Q2 (2N7002), R24–R27, C22, C23 and TP10.
- **How it works:** U2 SHUTDOWN (pin 5) was tied to GND. It now connects to `AMP_SHUTDOWN`, which R27 pulls up to +5V,
  so the amp is muted by default. The firmware toggles `AUDIO_ALIVE` (Daisy pin 33, was D26) once per audio callback.
  Each rising edge retriggers U3, whose Q output holds Q2 on. Q2 pulls SHUTDOWN low, which unmutes the amp.
  If the edges stop for ~10 ms (R24 × C22), the amp mutes.
- **Also muted when:** the Daisy is absent, booting, in DFU, or hung. R25 holds the trigger input low and R26 holds Q2 off.
- **Why:** a hung Daisy can loop its last audio buffer at full level. Mute must not depend on the software that has failed.

### Speaker feed from AUDIO_OUT_2, headphone detect in firmware
- **Removed:** the tip-switch line out. J5's TN contact no longer drives `LINE_SEND`, and TN and RN are both no-connects.
  In rev 1, plugging in headphones lifted the tip spring and muted J2 mechanically.
- **Added:** a speaker / line-send stage driven by the Daisy's second output.
  The path is `AUDIO_OUT_2` → R28/C24 (3.3k/1nF, ~48 kHz low-pass, same as the headphone path) → C25/R29 (AC coupling, biased to VREF)
  → U1C buffer → C26 (10µF) → `LINE_SEND` → R3 → J2. J2 and R3/R4 are unchanged and remain the feed to the speaker amp.
- **Changed:** U1 is now a TLV9064 quad (SOIC-14) instead of the TLV9062 dual, using the same TL07x-style symbol.
  Units A and B are unchanged. Unit C is the speaker buffer. Unit D is a spare, parked as a VREF follower for the AUX input buffer.
  Power moves to pin 4 (+5V) and pin 11 (GND).
- **Added:** `HP_DETECT` on Daisy pin 34 (was D27, also renamed on J7), with a 10k pull-up, 1k series resistor (R30, R31) and 100nF (C27).
  TP11 sits on the switch side (`HP_DETECT_SW`).
- **Why:** muting in firmware means the speaker amp can be shut down while headphones are in (saving battery), mute and unmute can ramp
  without clicks, and the speaker and headphones get independent levels.

## Firmware changes these require
- Read the encoder on pins 22/23/24 instead of the volume/tone ADCs. Ramp volume changes and use push to mute.
- Toggle `AUDIO_ALIVE` (pin 33) once per audio callback. Initialise it as a low output.
- Send the speaker mix to output 2 and headphones to output 1. Read `HP_DETECT` (pin 34). Its polarity depends on the jack:
  with a normally-closed switch, high means headphones are in. Ramp the speaker down and shut its amp off when headphones are in.
- Add an output clamp or limiter and NaN/Inf guard at the end of the chain. Enable the STM32 internal watchdog.

## Open items
- **Choose J5:** a 3.5 mm stereo jack with an isolated detect switch, whose switch contacts aren't connected to tip or ring.
  KiCad's stock library has none, so this needs a part pick (check the datasheet's switch drawing), plus a symbol and footprint.
  Connect its switch between `HP_DETECT_SW` and GND, and update the firmware polarity to match normally-open or normally-closed.
- **Confirm the U3 timeout:** check tw ≈ K·Rext·Cext against the SN74LVC1G123 datasheet, then measure on the board.
  The target is ~10 ms, comfortably longer than the 2 ms edge period.
- **Choose an encoder part number** that fits the EC11E footprint (Alps EC11E or Bourns PEC11R with switch).
- **Wire `AMP_SHUTDOWN`** to the speaker amp's mute/shutdown pin, and check that pin's polarity.
- **Measure the maximum headphone level** on rev 1, then decide whether to add a fixed attenuator before U2.
- **AUX input** (1/8" TRS → stereo-to-mono sum → U1D buffer → Daisy AUDIO_IN_2): not drawn yet. U1D is reserved for it.
- **XT30 power input** in place of J3, keeping Q1 for reverse polarity: not drawn yet.
- **Polarity:** U2 inverts the signal. Decide whether the firmware negates the output.
- **TEST_PLAN check 3B.4:** the low-end corner is ≈ 12.6 Hz, not 9 Hz.
- **Layout:** 4-layer stackup, fresh board.
