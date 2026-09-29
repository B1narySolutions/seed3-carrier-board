# TODO

Remaining review items, updated 2026-09-15.

## Fix before fab

- [ ] **Finish 5 V input protection.** Q1 now provides P-FET reverse-polarity protection. J3 remains unkeyed, and overvoltage/overcurrent protection is still absent.
  - Replace J3 with a keyed connector (barrel jack or JST).
  - Select suitable overvoltage and overcurrent protection for the 5 V rail.
- [ ] **Bench-check input plug-in pop.** J1's TN switch now grounds the input when unplugged. Verify plug-in behavior on hardware; a permanent tip-to-GND pulldown has not been added.

## Should fix

- [ ] **Check bulk capacitance at J3.** C9 is 10 µF ceramic; C18 provides 47 µF electrolytic capacitance near the headphone amplifier. Check supply ringing with the intended leads and whether additional bulk capacitance is needed at J3.
- [ ] **Restore an AUDIO_IN test point.** RV1 level calibration relies on measuring AUDIO_IN (target ≤ ~3 Vpp, below the Seed's 3.6 Vpp limit), but no test point exists on that net.
- [ ] **Check mono-plug behavior on J5.** A TS plug shorts R to sleeve, so U2 VOUT2 drives about 4.7 Ω. The TPA6111A2 has short-circuit and thermal protection, but bench-check heating.

## Housekeeping

- [ ] **Verify the Seed3 pinout.** A1 uses the Daisy Seed Rev4 symbol and footprint. Confirm it matches the Seed3, especially VIN (pin 39) and the audio pins (16–19).
