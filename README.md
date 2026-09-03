# Lab Notes: Research the sensitivity boundaries and determine defensive solutions if BOR threshold vulnerable state on Nucleo-L552ZE-Q board

## Objective
To research heavy loaded with NTT computations Nucleo board conditions to determine stable and unstable states(the sensitivity boundaries) to determine vulnerability points to deduce defensive solutions and implement them using STM32CubeIDE.

---

## Sensitivity Margins Search and Evaluation
1. Connect shunt resistors to the Nucleo board via breadboard to simulate aging by increasing resistance
2. Add NOP code before/after a PQC ML_KEM algorithm implementation to research margins during the higher load
3. Add bus-switching code to increase load to research margins during the high load

---

## Sensitive Margins Defence
1. Clock Gating
2. Frequency Change
3. Capacitors (ceramic and electrolytic)

---

## Hardware Security Evaluation of BOR Sensitivity Under Dynamic Workloads

| Impedance | Wiring / hardware configuration | Code workload / execution state | Boot / switcher state | Observed LED hardware behavior | Inferred silicon / power-system state |
|---|---|---|---|---|---|
| **52 Ω**<br>47 Ω + two 10 Ω resistors in parallel | IDD: `Pin1-10a, Pin2-15a`<br>R47: `10b–13b`<br>R10: `13c–15c` and another one `13d–15d`<br>Switcher: `10e–13e` | ML-KEM-512 PQC execution | Cold / warm<br>Switcher OFF | Solid, dimmed LEDs | **No BOR reset.** Series resistance is insufficient; the rail remains safely above the 2.54 V BOR threshold point. |
| **57 Ω**<br>47 Ω + 10 Ω | IDD: `Pin1-10a, Pin2-15a`<br>R47: `10b–13b`<br>R10: `13c–15c`<br>Switcher: `10e–13e` | Additional load code with NOP | Cold boot<br>Switcher OFF | Bright, beaming LEDs | **Dynamic BOR behavior.** Clean cold-inrush reset oscillations with bright LED indication. |
| **57 Ω**<br>47 Ω + 10 Ω | IDD: `Pin1-10a, Pin2-15a`<br>R47: `10b–13b`<br>R10: `13c–15c`<br>Switcher: `10e–13e` | High-density ALU / bus-switching code<br>and no-NOP code | Cold boot<br>Switcher OFF | Dimmed, beaming LEDs | **Level 3 BOR threshold state.** High dynamic loading lowers the baseline voltage; cold inrush triggers rapid, dimmed reset oscillations. |
| **57 Ω**<br>47 Ω + 10 Ω | IDD: `Pin 1-10a` and `Pin2–15a`<br>R47: `10b–13b`<br>R10: `13c–15c`<br>Switcher: `10e–13e` | ML-KEM-512 with parallel background workloads | Warm / cold<br>Switcher ON | LD3 red, beaming or dim | **ST-LINK regulator throttling.** Extreme transient current bypasses the MCU BOR response and triggers the onboard STEF01 e-fuse protection. |
| **67 Ω**<br>47 Ω + two 10 Ω resistors in series | IDD: `Pin 1: 10a` and `Pin2: 17a`<br>R47: `10b–13b`<br>R10: `13c–15c` and `15c–17c` | PQC mathematics / bus-switching code | Warm / cold<br>Switcher ON | Solid, dimmed LEDs<br>Execution stalled | **Undervoltage sag / lockout.** Excessive static voltage loss pulls the rail below the operational logic threshold. |
| **57 Ω — frequency test** | IDD: `Pin 1: 10a` and `Pin2–15a`<br>R47: `10b–13b`<br>R10: `13c–15c`<br>Switcher: `10e–13e` | ML-KEM-512 at **110 MHz** | Cold / warm<br>Switcher ON | Continuous LD3 red beaming | **Severe over-current protection.** Higher clock frequency increases dynamic current, hard-tripping the ST-LINK regulator. |
| **57 Ω — frequency test** | IDD: `Pin 1-10a` and `Pin2–15a`<br>R47: `10b–13b`<br>R10: `13c–15c`<br>Switcher: `10e–13e` | ML-KEM-512 at **80 or 64 MHz** | Cold / warm<br>Switcher ON or OFF | Bright-ish solid LEDs<br>No beaming | **Voltage has no drops (no big drops).** Lower frequency reduces dynamic-current peaks, preventing both BOR resets and regulator lockouts. |
