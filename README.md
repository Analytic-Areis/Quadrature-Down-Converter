# Quadrature Down Converter (QDC)

> **Design, Simulation, and Hardware Implementation of a Quadrature Down Converter**  
> *Course Project — Analog Electronic Circuits (S26)*  
> *Center for VLSI and Embedded Systems Technology (CVEST), IIIT Hyderabad*  
> **Authors:** Katta Sri Kaushik, Lakshmi Sai Bhargav V, Kushwanth Lanka

---

## 📌 Project Overview

A **Quadrature Down Converter (QDC)** is a critical subsystem in modern RF transceivers (Wi-Fi, Bluetooth, 5G, and IoT), responsible for translating high-frequency RF signals down to baseband or intermediate frequencies (IF) while separating the signal into **In-Phase ($I$)** and **Quadrature ($Q$)** components.

This project covers the full end-to-end design lifecycle:
1. **Mathematical Derivations & Analytical Sizing:** Analytical derivations for oscillator oscillation condition, switch mixer conversion gain, cascaded filter transfer functions, and small-signal BJT amplifier gain.
2. **SPICE Circuit Simulations:** Comprehensive schematic modeling and transient/AC/FFT characterization in LTSpice using TSMC 180nm CMOS models, UA741 op-amps, and discrete BJTs.
3. **Laboratory Hardware Prototyping:** Physical breadboard implementation using CD4007 CMOS arrays, discrete BJTs, and op-amps, verified on Digital Storage Oscilloscopes (DSO) across transient waveforms and FFT frequency spectra.
4. **Formal Academic Documentation:** Full IEEE conference paper format report and Beamer slide deck.

---

## ⚙️ System Architecture & Specifications

```
                     +---------------------------------------+
                     |         Quadrature Oscillator         |
                     |         f_LO = 170 kHz, 1 Vpp         |
                     +-------------------+-------------------+
                                         |
                       v_OSC_I (cos)     |      v_OSC_Q (sin)
                             |           |            |
                             v           |            v
    RF Input v_in    +---------------+   |    +---------------+
    (165 - 175 kHz) -+-> NMOS Switch |   |    |  NMOS Switch  | <- RF Input v_in
                     |   (I-Mixer)   |   |    |   (Q-Mixer)   |
                     +-------+-------+   |    +-------+-------+
                             |           |            |
                             v           |            v
                     +---------------+   |    +---------------+
                     | Cascaded LPF  |   |    | Cascaded LPF  |
                     | (fc ~ 4.65kHz)|   |    | (fc ~ 4.65kHz)|
                     +-------+-------+   |    +-------+-------+
                             |           |            |
                             v           |            v
                     +---------------+   |    +---------------+
                     |  CE BJT Amp   |   |    |  CE BJT Amp   |
                     | (Av ~ 10-15x) |   |    | (Av ~ 10-15x) |
                     +-------+-------+   |    +-------+-------+
                             |                        |
                             v                        v
                        v_IF_FINAL_I             v_IF_FINAL_Q
                        (5 kHz IF,               (5 kHz IF,
                         >= 400 mVpp)             >= 400 mVpp, 90° shifted)
```

### Key Target Specifications

| Parameter | Specification / Target | Achieved (Sim / Lab) |
|---|---|---|
| **Local Oscillator Frequency ($f_{\text{LO}}$)** | $170\text{ kHz}$ ($\pm 1\%$) | $170.9\text{ kHz}$ |
| **Quadrature Phase Separation** | $90^\circ$ between $I$ and $Q$ | $89.4^\circ$ |
| **LO Signal Amplitude** | $1.0\text{ V}_{\text{pp}}$ | $1.02\text{ V}_{\text{pp}}$ |
| **RF Input Test Frequency Range** | $165\text{ kHz} - 175\text{ kHz}$ | $165, 168, 169, 171, 172, 175\text{ kHz}$ |
| **Intermediate Frequency (IF)** | $\lvert f_{\text{RF}} - f_{\text{LO}}\rvert = 5\text{ kHz}$ | $5.0\text{ kHz}$ |
| **LPF Cutoff Frequency ($f_c$)** | Cascaded 2-stage RC ($\approx 4.65\text{ kHz}$) | Clean attenuation of $340\text{ kHz}$ sum term |
| **Final IF Output Swing** | $\ge 400\text{ mV}_{\text{pp}}$ into load | $\approx 450 - 520\text{ mV}_{\text{pp}}$ |
| **Technology / Components** | TSMC 180nm CMOS / CD4007, UA741, 2N2222 / 2N3904 | Verified in LTSpice & Lab Hardware |

---

## 📁 Repository Structure

```
Quadrature-Down-Converter/
├── README.md                      # Comprehensive project documentation
├── .gitignore                     # Ignores LaTeX build artifacts & temporary files
├── final_report.pdf               # Symlink to compiled IEEE conference report
├── questions.pdf                  # Symlink to course problem statement specification
│
├── QDC-simulations/               # LTSpice circuit schematics & device models
│   ├── final.asc                  # Full integrated system simulation (Osc + Mixer + LPF + Amp)
│   ├── QDC.asc                    # Core QDC mixer and down-conversion testbench
│   ├── amplified_output_QDC.asc   # Full path with BJT amplifier output stages
│   ├── Amplifier.asc              # Standalone CE BJT amplifier stage characterization
│   ├── mixer.asc                  # Standalone NMOS switch mixer schematic
│   ├── mixer_test.asc             # Mixer parametric transient testbench
│   ├── LPF.asc                    # Single-stage RC filter testbench
│   ├── LPF2.asc                   # Cascaded 2nd-order RC filter testbench
│   ├── TSMC_180nm.txt             # TSMC 180nm Level-49 BSIM3 MOSFET model card
│   ├── cd4007.lib                 # CD4007 CMOS dual complementary pair model
│   ├── UA741.asy / UA741.301      # UA741 operational amplifier symbol & subcircuit
│   └── LM741.ASY / LM741.lib      # Alternative LM741 op-amp library & symbol
│
├── report/                        # IEEE conference report
│   ├── main.tex                   # Main LaTeX source document
│   ├── main.pdf                   # Compiled IEEE 2-column final paper
│   ├── IEEEtran.cls               # Official IEEE Transactions LaTeX class file
│   ├── images/                    # Schematics, waveform captures, DSO & FFT graphs
│   └── ref/                       # Reference materials & project specification
│
└── presentation/                  # Project presentation
    ├── presentation.tex           # LaTeX Beamer slide deck source
    ├── presentation.pdf           # Compiled slide presentation
    └── images -> ../report/images # Shared images symlink (zero duplication)
```

---

## 🔬 Subsystem Details

### 1. Quadrature Local Oscillator ($170\text{ kHz}$)
- An active op-amp based loop consisting of a non-inverting integrator and an inverting stage to satisfy the Barkhausen criterion ($\lvert A\beta\rvert = 1$, $\angle A\beta = 0^\circ / 360^\circ$).
- Simultaneous $I$ ($\cos$) and $Q$ ($\sin$) outputs generated with high spectral purity ($>35\text{ dB}$ harmonic suppression) and precise $90^\circ$ relative phase shift.

### 2. NMOS Switch Mixer
- Implemented as a passive switching mixer utilizing NMOS transistors ($W/L = 10\mu\text{m}/180\text{nm}$ in TSMC 180nm; CD4007 in hardware).
- Periodic gating multiplies the incoming RF voltage $v_{\text{in}}(t) = A_{\text{RF}}\cos(\omega_{\text{RF}}t)$ by the square-wave switching function, producing sum ($\omega_{\text{RF}} + \omega_{\text{LO}}$) and difference ($\lvert \omega_{\text{RF}} - \omega_{\text{LO}}\rvert$) components.

### 3. Cascaded Low-Pass Filter
- 2nd-order passive $RC$ filter cascaded to provide $\approx -40\text{ dB/decade}$ roll-off beyond $f_c \approx 4.65\text{ kHz}$.
- Effectively eliminates the $\sim 340\text{ kHz}$ sum products and LO feedthrough while preserving the $5\text{ kHz}$ IF signal with negligible phase distortion.

### 4. Common-Emitter BJT Amplifier
- Discrete BJT amplifier with emitter degeneration ($R_E$) for thermal stability and high linearity.
- Provides a loaded voltage gain $A_v \approx 10 - 15\text{ V/V}$, stepping up the filtered $\sim 35\text{ mV}_{\text{pp}}$ IF signal to $\ge 400\text{ mV}_{\text{pp}}$ into a load resistor.

---

## 🛠️ How to Build & Run

### 1. Compiling the LaTeX Report
Requires `pdflatex` or `latexmk`:
```bash
cd report/
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex   # Run twice for cross-references
```
The output PDF is generated at `report/main.pdf`.

### 2. Compiling the Beamer Presentation
```bash
cd presentation/
pdflatex -interaction=nonstopmode presentation.tex
pdflatex -interaction=nonstopmode presentation.tex
```
The output presentation PDF is generated at `presentation/presentation.pdf`.

### 3. Running LTSpice Simulations
1. Open [LTSpice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html).
2. Open `QDC-simulations/final.asc` to simulate the complete end-to-end quadrature down converter.
3. Run the transient simulation (`.tran 2m`).
4. Inspect the nodes:
   - `V(vosc_i)` and `V(vosc_q)`: Quadrature LO waveforms.
   - `V(v_if_final_i)` and `V(v_if_final_q)`: Final amplified $5\text{ kHz}$ $I$ and $Q$ intermediate outputs.
