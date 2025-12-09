## Technical Overview
#### Introduction

This document presents a full technical description of a custom Kanthal Super high-temperature heating controller. The system has been deployed in a continuous industrial environment for over 3 years with zero failures, demonstrating robustness comparable to commercial SCR power controllers and industrial furnace safety systems. This paper describes the hardware, firmware architecture, control algorithms, safety mechanisms, and state machine logic that enable safe operation of Kanthal Super elements, which are known for extreme inrush current, thermal nonlinearity, and failure-critical behavior.
<br><br>
*The Controller is designed for high-power Industrial furnaces and has been validated on a 131 kW Khantal Super heating system.*
<br>

##### System Overview
The controller drives a Solid State Relay (SSR) using a ramping voltage firing algorithm. Instead of simple ON/OFF switching, the system modulates the trigger voltage to the SSR according to real-time current feedback, setpoint input, and operational state. The system employs a multi-layer safety architecture including fault detection, hardware watchdogs, high-fault amplifier thresholds, and a supervised cold-start workflow.
<br><br>
**Spec :** <br>
-	Rated Heater power : 131 kW<br>
-	Typical Line current : 150-200 A (3-phase equivalent)<br>
-	Load Type : Kanthal Super (MoSi2)<br>
-	Operation : Continuous industrial duty <br>

#### Hardware Architecture
Core hardware components: <br>
• STM32F030 MCU – runs control loops, RMS calculations, and state machine logic. <br>
• CT feedback chain – precision current sensing using a current transformer and analog filtering. <br>
• Offset and fault detection amplifiers – detect load failures, or over current SSR conditions. <br>
• SSR driver stage – isolated through optical couplers, providing high-integrity firing pulses. <br>
• MAX813 watchdog – independent hardware supervisor ensuring recovery from any firmware lockup. <br>
• Multi-rail power system – isolated DC-DC converters, analog reference regulators, and protection components. <br>
The hardware is designed with strict domain separation between high-voltage AC, isolated gate drive, analog sensing, and MCU digital logic to ensure safety and electromagnetic robustness.

<br>

#### Firmware Architecture 
The firmware is structured as a deterministic real-time control loop with the following responsibilities: <br>
• Sampling CT data and computing RMS current. <br>
• Verifying voltage offset and amplifier health. • Ramping trigger voltage based on load temperature and SV setpoint. <br>
• Executing a multi-stage safety state machine. <br>
• Driving SSR trigger pins with phase-angle-like voltage shaping. <br>
• Performing recovery logic or entering fail-safe modes during faults.<br> 

The firmware is written in C, with no dynamic allocation, no background threads, and deterministic timing suitable for safety-critical operation.

<br>

#### Ramping SSR Trigger Algorithm
Kanthal Super rods have extremely low resistance at cold start, causing 10–15× inrush current. To prevent catastrophic surge, the controller begins with minimal trigger voltage and increases gradually. The ramp slope is adaptive: <br>
• If current below expected rise → increase trigger voltage faster. <br>
• If current approaching setpoint → fine-step increments. <br>
• If current overshoots → decrease trigger voltage immediately. <br>

This behavior emulates soft-start SCR controllers found in industrial furnaces, but implemented entirely in firmware for a simple SSR module.<br>


#### Current Measurement and RMS Calculation
The system computes RMS current from CT samples using averaged absolute-value accumulation. Offset calibration is performed continuously to ensure accuracy even with component drift. The RMS value drives: <br>
• Overcurrent comparator limits. <br>
• Safety thresholds. <br>
• Ramping speed of the trigger voltage. <br>
• Load-presence detection.<br>

#### Safety State Machine
The safety architecture is a multi-state progression: <br>
1. Startup – ensure offsets are valid. <br>
2. Volt1–Volt7 – controlled ramping stages with strict boundaries. <br>
3. Trigger On – allow normal power flow. <br>
4. Fault – reset trigger voltage, short SSR pin, log alarm. <br>
5. Recovery – verify conditions before re-entry into normal mode. <br>
Transitions require both electrical and logical conditions to be satisfied. Any violation immediately forces entry into Fail mode with isolated SSR shutdown.

<br>

#### Protection Layers
Multiple levels of protection exist: <br>
• Overcurrent shutdown. <br>
• No-load detection (voltage rising without current).<br>
 • Offset-fault detection. <br>
• High-fault amplifier threshold crossing. <br>
• Hardware watchdog reset. <br>
• Trigger-pin shorting during faults. <br>
• Ramping out-of-range detection. <br>
This layered approach is comparable to IEC 61508 SIL-2 safety logic patterns used in industrial heating and motion systems.
<br>
At 131 kW, fault energy is sufficient to cause catastrophic damage. Therefore, the system employs a four-layer defense-in-depth safety architecture combining passive, hardware, and firmware protection
<br>

#### Fault Detection and Recovery
A dedicated recovery path evaluates whether the system can return to normal operation: <br>
• Offset within bounds. <br>
• Load detected. <br>
• SSR not latched or shorted. <br>
• Current stable. <br>
• Setpoint nonzero. <br>
If any condition is violated, the system remains in fail-safe mode until manual or automatic recovery is allowed.
<br>




#### 	Long-Term Reliability
The controller has operated continuously on a 131 kW furnace load for over three years, demonstrating stability under extreme electrical stress, thermal cycling, and component aging.
<br>
After two years of continuous industrial operation, the SSR module experienced conduction loss due to internal wear. This is a normal end-of-life behavior for SSRs operating in high-temperature and high-cycle environments. Key results: 
- No smoke, cracking, or thermal event. <br>
- No uncontrolled heating occurred. <br>
- The firmware correctly identified the no-load condition. <br>
- The system transitioned into safe fault mode automatically. <br>
- The furnace was protected from thermal runaway. <br>
This event validates the robustness of the fault detection and safety architecture.<br>


#### 	Conclusion
The Kanthal Super Controller represents an advanced embedded control solution integrating power electronics control, real-time firmware, functional safety, and industrial reliability. The design is a complete end-to-end system engineered by a single developer and demonstrates expertise typically requiring multiple engineers across several disciplines.
