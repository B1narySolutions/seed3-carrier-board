# seed3-carrier

KiCad carrier board for the Daisy Seed, part of a capstone project: a portable touchscreen practice guitar amp.
The firmware is in [realtime-nam-seed3](https://github.com/B1narySolutions/realtime-nam-seed3).

## Layout

- `rev1/`: the board sent to fabrication on 2026-09-15 (also tagged `rev1`). Keep it unchanged while it is being tested. Manufacturing files are in `rev1/fabrication/`.
- `rev2/`: the next revision. The schematic starts as a copy of rev 1; the PCB layout starts empty.
- `Seed3_Carrier.pretty/` and `seed3-carrier.kicad_sym`: footprint and symbol libraries shared by both revisions.
- [`studies/`](studies/): analysis of the board's behaviour (scripts, figures, write-ups). Current studies cover rev 1.

Open `rev1/seed3-carrier.kicad_pro` or `rev2/seed3-carrier.kicad_pro` in KiCad 10.

## Project documentation

Whole-project documentation lives in
[capstone-documentation](https://github.com/B1narySolutions/capstone-documentation): requirements,
board measurements, test plans, the rev 2 change list and the decision log. Check there before making a
decision that affects more than this board, and record board measurements and project-wide decisions there.
