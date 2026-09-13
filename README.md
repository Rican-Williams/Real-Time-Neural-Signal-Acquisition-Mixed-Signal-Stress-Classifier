# Real-Time Neural Signal Acquisition & Mixed-Signal Stress Classifier

[![Award: 1st Place](https://img.shields.io/badge/Award-1st%20Place%20Winner-gold?style=for-the-badge)](#)
[![Domain: Mixed-Signal Design](https://img.shields.io/badge/Focus-Analog%20Front--End%20%7C%20DSP-blue?style=for-the-badge)](#)

> **First Place Winner** in the 2026 Biomedical Engineering Course Union Competition. Designed an low-noise Analog Front-End (AFE) and mixed-signal processing pipeline capable of resolving microvolt-level ($\mu\text{V}$) EEG signals to classify real-time neurological stress.

---

## 1. System Architecture Overview

The system captures biopotentials across the scalp, isolates microvolt-level neural dynamics from large common-mode biological and environmental interference, conditions the band-limited signal, and translates frequency-domain spectral energy into cognitive stress classifications.

## 2. Video Demonstration & Media Showcase

### Live Demo & Dynamic Waveform Capture
[![Watch the Video Demo](https://img.shields.io/badge/Watch-Video%20Demonstration-red?style=for-the-badge&logo=youtube)]()

> *Click the banner above or watch the demo video below showcasing real-time Alpha/Beta wave transitions under varying cognitive load:*

https://github.com/user-attachments/assets/your-video-demo-hash

---

### Hardware Schematics & Breadboard Implementation

| Schematic Capture (Multisim/SPICE) | Physical Hardware Layout & Probing |
| :---: | :---: |
| ![Analog Front End Schematic](./hardware/schematics/afe_schematic.png) | ![Breadboard Prototype](./hardware/images/afe_breadboard.jpg) |
| *Figure 1: 6-stage discrete AFE with decoupled split-rail power.* | *Figure 2: Physical hardware implementation and scope probing.* |

---

## 3. Analog Front-End (AFE) Engineering Deep-Dive

Human EEG voltages range between **$10\,\mu\text{V}$ and $100\,\mu\text{V}$**, making biological acquisition vulnerable to source loading, contact impedance drift, and environmental $60\text{ Hz}$ mains coupling. The discrete AFE employs a 6-stage cascaded architecture:

### Stage 1: Ultra-High Input Impedance Buffers ($U_1, U_2$)
* **Topology:** Voltage followers ($A_v = 1$).
* **Design Goal:** Biosensing electrodes exhibit high source impedance ($k\Omega \text{ to } M\Omega$). The buffer stage ensures $Z_{\text{in}} \gg Z_{\text{source}}$, preventing signal degradation due to voltage-divider loading effects.
* **Biasing:** $100\text{ k}\Omega$ pull-down resistors establish a stable DC reference path for input bias currents without pulling down scalp potentials.

### Stage 2: Differential Subtractor ($U_3$)
* **Topology:** Inverting differential configuration ($R_5 = R_6 = 10\text{ k}\Omega$, $R_7 = 10\text{ k}\Omega$, $R_8 = 20\text{ k}\Omega$).
* **Differential Gain:** $A_{v,\text{diff}} \approx 2\text{ V/V}$ ($6\text{ dB}$).
* **Design Goal:** Rejects large common-mode potentials (electrodermal artifacts, electromagnetic hum) present across both the active and reference cranial sites.

### Stage 3: Active High-Pass Filter & Initial Drift Reduction ($U_4$)
* **Topology:** Inverting gain with series input capacitance ($C_1 = 0.1\,\mu\text{F}$, $R_9 = 400\text{ k}\Omega$, $R_{10} = 10\text{ k}\Omega$, $R_{11} = 100\text{ k}\Omega$).
* **Pole Calculation:**
  $$f_c = \frac{1}{2\pi R_9 C_1} = \frac{1}{2\pi \cdot (400\text{ k}\Omega) \cdot (0.1\,\mu\text{F})} \approx 3.98\text{ Hz}$$
* **Design Goal:** Eliminates DC half-cell electrode offset voltages ($>100\text{ mV}$) before high-order amplification. Distributes an initial stage gain of:
  $$A_{v1} = -\frac{R_{11}}{R_{10}} = -\frac{100\text{ k}\Omega}{10\text{ k}\Omega} = -10\text{ V/V}$$
  Keeping this initial gain moderate prevents DC drift from driving op-amp output rails into hard saturation.

### Stages 4 & 5: Cascaded AC Gain Blocks ($U_5, U_6$)
* **Topology:** Dual inverting stages ($R_{13} = R_{15} = 10\text{ k}\Omega$; $R_{14} = R_{16} = 100\text{ k}\Omega$).
* **Cascaded Midband Gain:**
  $$A_{v2} = -\frac{R_{14}}{R_{13}} = -10\text{ V/V}, \quad A_{v3} = -\frac{R_{16}}{R_{15}} = -10\text{ V/V}$$
* **Cumulative Midband Voltage Gain:**
  $$A_{v,\text{total}} = A_{v,\text{diff}} \times A_{v1} \times A_{v2} \times A_{v3} \approx 2 \times (-10) \times (-10) \times (-10) = -2000\text{ V/V} \quad (\approx 66.02\text{ dB})$$
* **Gain-Bandwidth Product (GBWP) Budgeting:** Splitting $1000\times$ AC gain across three identical operational amplifiers preserves stage-level bandwidth ($BW \approx GBWP / A_v$), maintaining linearity well past the $40\text{ Hz}$ cutoff without introducing phase distortions.

### Stage 6: Anti-Aliasing Low-Pass Filter
* **Topology:** First-order passive $RC$ filter ($R_{12} = 40\text{ k}\Omega$, $C_2 = 0.1\,\mu\text{F}$).
* **Cutoff Frequency:**
  $$f_{c,\text{LPF}} = \frac{1}{2\pi R_{12} C_2} = \frac{1}{2\pi \cdot (40\text{ k}\Omega) \cdot (0.1\,\mu\text{F})} \approx 39.79\text{ Hz}$$
* **Design Goal:** Attenuates muscular electromyographic (EMG) artifacts and eliminates spectral components above $40\text{ Hz}$.

---

## 4. Power Topology & Signal Integrity Management

* **Split-Rail Power Distribution:** Driven by dual $9\text{V}$ batteries configured in series with a virtual ground reference ($\pm 9\text{V}$ rails). Battery power inherently severs the system from the $60\text{ Hz}$ AC power line return path, suppressing common-mode noise.
* **Rail Decoupling:** Every active gain block includes parallel ceramic decoupling capacitors ($C_{3}-C_{14} = 0.1\,\mu\text{F}$) located at the supply pins to minimize high-frequency rail impedance and suppress inter-stage supply coupling.

---

## 5. Mixed-Signal Acquisition & Real-Time Classification

### Digitization Pipeline
The system accommodates two conversion options:
1. **Precision 24-Bit ADC (NAU7802 via $\text{I}^2\text{C}$):** Utilizes an internal PGA, differential low-noise delta-sigma modulation, and ultra-high dynamic range to resolve sub-millivolt swings without quantization-induced noise.
2. **Direct Embedded ADC (Arduino Nano):** Successive-Approximation Register (SAR) configuration for compact benchtop verification.

### Digital Signal Processing (DSP) Logic
The microcontroller computes a real-time radix-2 Fast Fourier Transform (FFT) across a sliding window of digitized biosignals:

| Neural Band | Frequency Range | Physiological Correlate |
| :--- | :--- | :--- |
| **Theta ($\theta$)** | $4 - 7\text{ Hz}$ | Drowsiness, deep relaxation |
| **Alpha ($\alpha$)** | $8 - 13\text{ Hz}$ | Calm, alert, resting state |
| **Beta ($\beta$)** | $14 - 30\text{ Hz}$ | Active concentration, cognitive stress |

### Stress Metric Computation
A continuous stress index ($\mathcal{S}$) evaluates spectral power balance:
$$\mathcal{S} = \frac{P_{\text{Beta}}}{P_{\text{Alpha}}}$$
* **$\mathcal{S} > \text{Threshold}$:** Sustained high Beta power triggers an alert LED indicating elevated cognitive stress.
* **$\mathcal{S} \le \text{Threshold}$:** Dominant Alpha spectral density maps to a relaxed, unstressed neurological state.

