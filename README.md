# Kanthal-Super-High-Temperature-Controller

Custom embedded controller for Kanthal Super (Si-Mo) heating elements, designed for safety-critical industrial furnaces.

## Impact Overview
- 3+ years continuous industrial operation with zero uncontrolled failures<br>
- Adaptive ramping algorithm preventing 10–15× cold-start inrush<br>
- Multi-layer safety architecture (IEC 61508 SIL-2 inspired)<br>
- Safely handled real SSR end-of-life with automatic fail-safe shutdown<br>
- End-to-end system (hardware + firmware + safety) designed by a single engineer<br>

## System Highlights
- STM32-based real-time control firmware (deterministic, no RTOS)<br>
- RMS current feedback via CT with continuous offset calibration<br>
- Voltage-ramped SSR trigger (SCR-like behavior)<br>
- Supervised cold-start and recovery state machine<br>
- Hardware watchdog and analog fault thresholds<br>

## Documentation
- 📄 [Technical Overview](docs/Technical_Overview.md)
- 🛡️ [Safety Architecture](Documentations/Safety_Architecture.md)
- 🔁 [State Machine](docs/State_Machine.md)
- 📊 [Long-Term Reliability](docs/Long_Term_Reliability.md)

## Repository Structure
- `/firmware` – embedded source code
- `/hardware` – schematics and BOM
- `/docs` – design documentation
