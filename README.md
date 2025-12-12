![GitHub stars](https://img.shields.io/github/stars/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture)
![GitHub forks](https://img.shields.io/github/forks/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture)
# Kanthal-Super-High-Temperature-Controller (131 KW)

Custom embedded controller for Kanthal Super (Si-Mo) heating elements, designed for safety-critical industrial furnaces.
The controller is designed for high-power industrial furnaces and has been validated on a 131 kW Kanthal Super heating system.

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

**Spec :**
- Rated Heater Power: 131 kW<br>
- Typical Line Current: 150–200 A (3-phase equivalent)<br>
- Load Type: Kanthal Super (MoSi2)<br>
- Operation: Continuous industrial duty<br>

## System Features
- 131 kW heater control
- SSR-300 power switching with voltage ramping
- RMS current feedback with continuous calibration
- Redundant thermal sensors with safety watchdog
- Hardware + firmware + safety logic
  
## Documentation
- 📄 [Technical Overview](https://github.com/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture/blob/bec7cec90d52a5f2b84a17f64c2f4486e8544c4c/Documentation/Technical%20Overview.md)
- 🛡️ [Safety Architecture](https://github.com/sinfu98android/Kanthal-Super-High-Temperature-Controller/blob/aea99e4c84bd27a27627e6e728dc2e6159ed56b1/Documentation%20/Safety%20Architecture.md)
- 🔁 [State Machine](https://github.com/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture/blob/d838d45998e24ebbc05e2599479edca739a953b8/Documentation/State%20Machine.jpg)
- 📊 [Long-Term Reliability](https://github.com/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture/blob/bec7cec90d52a5f2b84a17f64c2f4486e8544c4c/Documentation/Long%20Time%20Reliablity.md)
- 📎 [Block Diagram](https://github.com/sinfu98android/131-kW-Kanthal-Super-Heater-Controller-with-Safety-Architecture/blob/ce26d7bc2504f890bb98c31c97792afacd08d716/Documentation/Block%20Diagram.jpg)

## Repository Structure
- `/firmware` – embedded source code
- `/hardware` – schematics and BOM
- `/documentation` – design documentation
<br>
Designing and validating a 131 kW heating controller requires consideration of fault energy, component aging, and safe failure modes typically addressed only in multi-engineer industrial teams
