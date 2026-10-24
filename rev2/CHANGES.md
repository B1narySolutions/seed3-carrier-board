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
- **Changed:** J5 is now a Same Sky **SJ3-35063AG** (3.5 mm stereo, through hole; [datasheet](https://www.sameskydevices.com/product/resource/sj3-3506x.pdf)).
  Pins 1/2/3 are sleeve/ring/tip on GND/HP_R/HP_L. Pins 5 and 6 are an isolated normally-open switch that closes when a plug
  is inserted: pin 5 goes to GND and pin 6 to `HP_DETECT_SW`. Pin 4 isn't fitted on this variant.
  The symbol is new in `seed3-carrier.kicad_sym`, and the footprint is new in
  `Seed3_Carrier.pretty/Jack_3.5mm_SameSky_SJ3-35063AG_Horizontal.kicad_mod`, drawn from the datasheet's recommended PCB layout.
  The jack face sits at the board edge (X = −2.4 mm from pin 1), with the nose overhanging.
  DigiKey had 398 in stock when chosen.
- **Why:** muting in firmware means the speaker amp can be shut down while headphones are in (saving battery), mute and unmute can ramp
  without clicks, and the speaker and headphones get independent levels.

### Power board connector (J3 → 8-pin JST-XH)
- **Changed:** J3 is now an 8-pin JST-XH (B8B-XH-A, vertical) instead of the 2-pin 5 V header. It carries power and the control
  signals to the separate power board, which handles the battery, the power button and the speaker amp. J3 moved to a new
  "Power board connector" block at the bottom of the sheet. Q1 still provides reverse-polarity protection, now on the `5V_IN` net.

  | Pin | Net | Direction | Daisy pin | Use |
  |---|---|---|---|---|
  | 1, 2 | `5V_IN` | Power board → carrier | 39 (VIN) via Q1 | 5 V supply, two pins share the current |
  | 3, 4 | GND | — | 20 / 40 | Ground, two pins |
  | 5 | `BATT_SENSE_CONN` | Power board → carrier | 25 (`BATT_SENSE`, was A3) | Battery voltage scaled to ≤ 3.3 V. R32 (1k) and C28 (100nF) filter it into the ADC |
  | 6 | `PWR_BTN_CONN` | Power board → carrier | 26 (`PWR_BTN`, was A4) | Low while the power button is pressed. R36 (10k) pull-up to 3V3_D, R35 (1k) series, C29 (100nF) |
  | 7 | `PWR_HOLD_CONN` | Carrier → power board | 27 (`PWR_HOLD`, was A5) | High keeps the power on. R33 (1k) series, R34 (100k) pull-down so it is low while the Daisy is absent or booting |
  | 8 | `SPK_MUTE` | Carrier → power board | 28 (`SPK_EN`, was A6) | Open drain speaker-amp mute (see below) |

- **Added:** `SPK_MUTE` gate. Q3 and Q4 (2N7002) are in series from `SPK_MUTE` to GND. Q3's gate is `WD_OK` (U3 Q, the watchdog
  signal that already drives Q2, now named). Q4's gate is `SPK_EN` with R37 (100k) pull-down. `SPK_MUTE` is pulled low (speaker on) only
  when the watchdog is running **and** the firmware enables the speaker. `AMP_SHUTDOWN` now mutes only the headphone amp (U2).
- **Added:** FB1 (ferrite bead, 1206) and C30 (47µF, 1210) between `+5V` and J4 pin 4, which is now `DISPLAY_5V`.
  They keep the display's current pulses (ESP32 Wi-Fi, backlight PWM) off the rail that feeds the analog circuits.
- **Power board requirements:** nothing above 3.3 V on pins 5–7. Pull `SPK_MUTE` up on the amp side so the amp is muted when this
  board is unpowered or unplugged. Keep the power on for about 2 s after a button press until the Daisy raises `PWR_HOLD`, and force
  the power off when the button is held for about 5 s. The power button itself is panel mounted and wired to the power board.
- **Why:** one connector for power and signals, a clean firmware-controlled shutdown, battery level on the display, and a speaker
  mute that is independent of the headphone amp.

## Firmware changes these require
- Read the encoder on pins 22/23/24 instead of the volume/tone ADCs. Ramp volume changes and use push to mute.
- Toggle `AUDIO_ALIVE` (pin 33) once per audio callback. Initialise it as a low output.
- Send the speaker mix to output 2 and headphones to output 1. Read `HP_DETECT` (pin 34):
  **low = headphones in** (J5's normally-open switch closes to GND). Debounce it in firmware. Ramp the speaker down and shut its amp off when headphones are in.
- Add an output clamp or limiter and NaN/Inf guard at the end of the chain. Enable the STM32 internal watchdog.
- Drive `PWR_HOLD` (pin 27) high early in boot. On a `PWR_BTN` (pin 26, active low) press while running, fade out, mute,
  save settings, then drive `PWR_HOLD` low.
- Drive `SPK_EN` (pin 28) high to enable the speaker amp, low when headphones are in. This replaces shutting the speaker off
  through `AMP_SHUTDOWN`.
- Read `BATT_SENSE` (pin 25, ADC) and send the battery level to the display. Shut down cleanly when it gets low.

## Open items
- **Check the J5 footprint** against a physical part (or a 1:1 print) before ordering boards. It was drawn by hand from the datasheet.
- **Confirm the U3 timeout:** check tw ≈ K·Rext·Cext against the SN74LVC1G123 datasheet, then measure on the board.
  The target is ~10 ms, comfortably longer than the 2 ms edge period.
- **Choose an encoder part number** that fits the EC11E footprint (Alps EC11E or Bourns PEC11R with switch).
- **Speaker amp mute polarity:** `SPK_MUTE` floats (pulled up = muted) or is pulled low (= playing). Check that the chosen amp's
  mute/shutdown pin matches, or invert it on the power board.
- **`BATT_SENSE` divider:** the power board sets the ratio. Record it here so the firmware can convert the reading to volts.
- **Display current:** check the display's datasheet. If it draws well over 1 A, feed it from the power board instead of J4.
- **J3 orientation:** vertical (B8B-XH-A) is drawn. Switch to the right-angle S8B-XH-A if the cable has to leave along the board.
- **2N7002 gate drive:** Q2–Q4 are driven from 3.3 V logic. That should be enough for the small pull-up currents on `SPK_MUTE` and `AMP_SHUTDOWN`, but check it on the board.
- **Measure the maximum headphone level** on rev 1, then decide whether to add a fixed attenuator before U2.
- **AUX input** (1/8" TRS → stereo-to-mono sum → U1D buffer → Daisy AUDIO_IN_2): not drawn yet. U1D is reserved for it.
- **Polarity:** U2 inverts the signal. Decide whether the firmware negates the output.
- **TEST_PLAN check 3B.4:** the low-end corner is ≈ 12.6 Hz, not 9 Hz.
- **Layout:** 4-layer stackup, fresh board.
