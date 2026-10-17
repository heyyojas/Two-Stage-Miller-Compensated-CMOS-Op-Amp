# Two-Stage-Miller-Compensated-CMOS-Op-Amp
Transistor-level design of a 1.2 V two-stage Miller-compensated operational amplifier in 90 nm CMOS (Cadence GPDK090). Devices are sized with the gm/ID methodology instead of square-law hand calculations, because of short-channel effects at this node. Designed and simulated in Cadence Virtuoso (ADE L, Spectre). 

Course: Analog Electronics, IIIT Kottayam
Roll Number: 2025BEC0010

## Specifications and Results

All results: TT corner, 27 °C, unity-gain feedback configuration, CL = 1 pF.

| Specification | Target | Achieved | Status |
|---|---|---|---|
| Technology / supply | GPDK090, 1.2 V | GPDK090, 1.2 V | Pass |
| Open-loop DC gain | ≥ 50 dB | ≈ 68 dB (TODO: exact value from marker) | Pass |
| Unity-gain bandwidth | ≥ 20 MHz | 43.9 MHz | Pass |
| Phase margin | ≥ 60° | 70.7° | Pass |
| Slew rate | ≥ 10 V/µs | 26.4 V/µs rising, 22.6 V/µs falling (avg 24.5) | Pass |
| Power dissipation | ≤ 1 mW | ≈ 98 µW | Pass |
| Load capacitance | 1 pF | 1 pF | Pass |

Gain margin: 35.0 dB at 549 MHz.

---

## Architecture

| Block | Devices | Notes |
|---|---|---|
| Stage 1 | M1, M2 (NMOS diff pair), M3, M4 (PMOS mirror load), M5 (NMOS tail) | M1 gate is the inverting input, M2 gate the non-inverting input |
| Stage 2 | M6 (PMOS common-source), M7 (NMOS current-source load) | Output at the M6/M7 drains |
| Bias | M8 (diode-connected NMOS), 10 µA reference | M5 = 2x M8, M7 = 5x M8 |
| Compensation | Cc in series with Rz, from stage-1 output to op-amp output | Rz nulls the right-half-plane zero |

All devices use L = 500 nm. Schematic: `results/schematic.png`.

---

## Design Methodology

1. **Device characterization.** `nmos1v` and `pmos1v` (W = 10 µm, VDS = 0.6 V, VSB = 0), DC sweep of VGS with L = 100, 180, 300, 500 nm. Extracted gm/ID vs ID/W and gm/ID vs gm·ro (`characterization/`).
2. **Choose L from the gain budget.** At gm/ID = 10, NMOS gm·ro rises strongly with L, so L = 500 nm was used for every device to meet the 50 dB gain target with margin.

   | L | ID/W at gm/ID = 10 (µA/µm) | gm·ro at gm/ID = 10 |
   |---|---|---|
   | 100 nm | 61.2 | 7.0 |
   | 180 nm | 28.1 | 13.2 |
   | 300 nm | ≈ 16 | 28.4 |
   | 500 nm | 9.7 | 64.7 |

3. **Size from the charts.** W = ID / (ID/W read at the chosen gm/ID and L). Mirror ratios fix the currents, and (W/L)6 = 5 x (W/L)4 removes systematic offset.
4. **Compensate.** Cc sets the dominant pole and UGB, Rz places the compensating zero.
5. **Verify.** DC operating point, `stb` analysis (loop gain and phase margin), transient slew.

---

## Final Design

DC operating point, TT, 27 °C, VCM = 0.6 V. Output sits at 0.6 V (systematic offset ≈ 0).

| Device | Role | W | L | ID | gm | gm/ID |
|---|---|---|---|---|---|---|
| M1, M2 | Input pair | 10 µm | 0.5 µm | 9.89 µA | 220.9 µS | 22.3 |
| M3, M4 | Mirror load | 5 µm | 0.5 µm | 9.89 µA | 139.7 µS | 14.1 |
| M5 | Tail | 4 µm | 0.5 µm | 19.78 µA | 267.2 µS | 13.5 |
| M6 | 2nd-stage driver | 25 µm | 0.5 µm | 51.57 µA | 724.2 µS | 14.0 |
| M7 | 2nd-stage load | 10 µm | 0.5 µm | 51.57 µA | 691.8 µS | 13.4 |
| M8 | Bias diode | 2 µm | 0.5 µm | 10.0 µA | 134.2 µS | 13.4 |

Cc = 700 fF, Rz = 3 kΩ, CL = 1 pF, bias current = 10 µA. Total supply current ≈ 81.3 µA, so power ≈ 98 µW.

The input pair runs at a higher gm/ID (22.3) than the other devices (13 to 14), giving gm1 = 221 µS.

---

## Results

### Stability (stb, 1 Hz to 1 GHz)

| Case | Cc | Rz | Phase margin | UGB | Gain margin |
|---|---|---|---|---|---|
| Initial | 0.5 pF | TODO | 51.6° | 53.4 MHz | 23.1 dB at 244 MHz |
| Final | 0.7 pF | 3 kΩ | 70.7° | 43.9 MHz | 35.0 dB at 549 MHz |

Raising Cc and retuning Rz traded about 10 MHz of UGB for 19° of phase margin. UGB still clears the 20 MHz spec.

### Transient slew rate

Unity-gain buffer, input step 0.4 V to 0.8 V, 1 ns edges. Slew measured between the 20% (480 mV) and 80% (720 mV) points of the output.

| Edge | Δt | Slew rate |
|---|---|---|
| Rising | 9.10 ns | 26.38 V/µs |
| Falling | 10.64 ns | 22.55 V/µs |
| Average | n/a | 24.47 V/µs |

The falling edge shows a small undershoot (about 15 mV) that settles back to 0.4 V.

---

## Verification Status

| Item | Status |
|---|---|
| DC operating point | Done |
| AC / stability (stb) | Done |
| Transient slew rate | Done |
| PVT corners (TT / SS / FF at 0, 27, 85 °C) | Planned, not yet run |
| CMRR and PSRR | Planned, not yet run |
| Layout, DRC/LVS, post-layout (PEX) | Planned, not yet run |

---

## Repository Structure

```
.
├── README.md
├── docs/                 # abstract and final report (PDF)
├── characterization/     # gm/ID plots for NMOS and PMOS
├── design/               # exported Spectre netlist, sizing table
├── results/              # schematic, DC OP, stb, transient plots
└── .gitignore
```

---

## Reproducing the Results

1. Create a cell `opamp2stage` with the topology above (library with GPDK090).
2. ADE L design variables: `Cc = 700f`, `Rz = 3k`, `vcm = 600m`, `W1 = 10u`, `W3 = 5u`, `W5 = 4u`, `W6 = 25u`, `W7 = 10u`, `W8 = 2u`. All lengths 500n.
3. Bias: 10 µA ideal current source into M8, VDD = 1.2 V.
4. Analyses:
   - `stb`: 1 Hz to 1 GHz, 20 points/decade, probe `IPRB0` (placed in the feedback path from `out` to M1's gate).
   - `tran`: 0 to 2 µs, conservative. Input is a `vpulse` on M2's gate: v1 = 400 mV, v2 = 800 mV, delay 100 ns, rise/fall 1 ns, width 1 µs.
5. Read phase margin and UGB from the `stb` log, and measure slew between the 20% and 80% points of the output.

---

## Notes

- **PDK not included.** GPDK090 is Cadence's proprietary academic PDK. Only the exported netlist (which references the model path) and result plots are in this repo. A licensed GPDK090 install is required to re-run the simulations.
- Results are for the TT corner at 27 °C only until the planned corner runs are completed.
