# Safety Architecture

## Overview

This document describes the safety architecture of the Kanthal Super Heating Controller.  
The controller is a safety-conscious embedded system designed to drive high-power Kanthal Super (Si-Mo) heating elements while preventing catastrophic failure modes (thermal runaway, element/mount explosion, fire, personnel harm). The architecture is intentionally layered: independent hardware protections operate in parallel with firmware diagnostics and supervised state transitions.<br>
At 131 kW, fault energy is sufficient to cause catastrophic damage. Therefore, the system employs a four-layer defense-in-depth safety architecture combining passive, hardware, and firmware protections.<br>

The implementation is proven in field operation (3+ years continuous runtime). The design philosophy is *fail-safe*, *defensive*, and *redundant where practical* given a single-operator engineering scope..<br>

## Safety Goals

1. Prevent uncontrolled application of full mains power to cold Kanthal Super elements (inrush / thermal shock).  .<br>
2. Detect and safely respond to overcurrent, short-circuit and SSR/driver failures.  .<br>
3. Detect no-load and open-circuit heater conditions and prevent false heating. .<br> 
4. Guarantee deterministic, verifiable shutdown in the event of firmware hang or critical hardware fault.  .<br>
5. Maintain safe recovery/restart paths and provide diagnostic visibility for maintenance and root-cause analysis..<br>


## Safety Principles

- **Fail-safe by design:** any detected fault or missing precondition transitions the system to a safe state (isolated SSR + alarm).  .<br>
- **Defense-in-depth:** multiple, independent detection and response layers (analog comparators, watchdog, firmware checks).  .<br>
- **Domain separation:** distinct HV (mains), isolated SSR drive, analog sensing, and MCU domains with galvanic isolation.  .<br>
- **Observable degradation:** monitor wear-prone parts (SSR) and record/log the event for maintenance.  .<br>
- **Deterministic behavior:** no dynamic allocation and simple, time-bounded loops to simplify verification and reduce unexpected behavior..<br>

## Safety Functions (Summary)

1. **Cold-Start Soft-Start Supervision**  .<br>
   - Purpose: Prevent excessive inrush and thermal shock during element warm-up.  .<br>
   - Implementation: Adaptive ramping algorithm in firmware (Volt1→Volt7) with per-stage electrical limits.  .<br>
   - Hardware: SSR + opto isolation (driver), isolated DC-DC rails for drive.  .<br>
   - Fail-safe action: If ramp constraints violated → ResetVoltageOut() → SSR isolate → Alarm..<br>

2. **Overcurrent Protection**  .<br>
   - Purpose: Detect sustained or peak overcurrent and remove power.  .<br>
   - Implementation: CT sampling → IRMS calculation in firmware; analog comparator path for fast trip recommended/implemented in schematic.  .<br>
   - Hardware: CT/shunt + LM324 conditioning + (recommended) fast comparator + latch to cut main contactor/relay.  .<br>
   - Fail-safe action: Immediate SSR isolation, log alarm, latched hardware trip if comparator used..<br>

3. **No-Load / Broken-Load Detection**  .<br>
   - Purpose: Detect when voltage is present but current is absent (open element or connection).  .<br>
   - Implementation: Firmware checks (Voltage level vs IRMS threshold) and offset validation.  .<br>
   - Fail-safe action: ResetVoltageOut(); set AlarmIdx=kAlarm_NoLoad..<br>

4. **Offset / Amplifier Fault Detection**  .<br>
   - Purpose: Ensure ADC and analog front-end are healthy; detect drift or saturation.  .<br>
   - Implementation: Continuous Offset measurement and range checks (kOffset_LO..kOffset_HI).  .<br>
   - Fail-safe action: ResetVoltageOut(); set AlarmIdx=kAlarm_OffsetFail..<br>

5. **Firmware Watchdog / Supervisor**  .<br>
   - Purpose: Guarantee recovery from firmware hang or livelock.  .<br>
   - Implementation: MAX813 independent watchdog controlling reset/power supervisor. Firmware toggles heartbeat; missing heartbeat causes hardware reset and SSR isolation.  .<br>
   - Fail-safe action: Hardware reset + SSR off; operator alert..<br>

6. **SSR End-of-Life Detection (Field Observed)**  .<br>
   - Purpose: Detect conduction loss due to SSR aging and safely handle it.  .<br>
   - Implementation: Firmware detects "no-load after ramp" or sudden conduction loss and flags SSR aging.  .<br>
   - Fail-safe action: Log SSR end-of-life, isolate SSR, require replacement before re-entry..<br>

7. **Ramping Out-of-Range & State Machine Enforcement**  .<br>
   - Purpose: Prevent entering higher voltage stages when preconditions fail.  .<br>
   - Implementation: `StateRunTarget()` enforces thresholds between kState_LevelN limits; violation leads to FailState and SSR isolation..<br>

## Failure Modes & Mitigations (Top items)

1. **SSR welds short (fail-closed)**  .<br>
   - Risk: Continuous mains power applied.  <br>
   - Mitigation: Use mechanical contactor in series (recommended) as the primary isolator for maintenance/emergency. Add fast fuse upstream. Use two different switching technologies (mechanical + SSR) for redundancy.<br>

2. **MCU lock or firmware fault**  <br>
   - Risk: SSR stuck on or uncontrolled behavior.  <br>
   - Mitigation: MAX813 hardware watchdog; hardware logic that defaults SSR to OFF on supervisor reset; SSR drive default bias to safe state when isolated.<br>

3. **CT/shunt open or short**  <br>
   - Risk: Wrong IRMS calculation → false safe/unsafe state.  <br>
   - Mitigation: ADC sanity checks, offset verification, dual-sensor voting recommended for certification.<br>
4. **Analog amplifier saturation or drift**  <br>
   - Risk: Wrong sensor input → missed trip.  <br>
   - Mitigation: Continuous offset range checks (OffsetOK), high-fault comparators with independent thresholds.<br>

5. **Mains transients & EMI**  <br>
   - Risk: False triggering or component damage.  <br>
   - Mitigation: MOVs, line filters, TVS devices, and good PCB layout/grounding (see hardware notes).<br>

6. **SSR end-of-life (aged)**  <br>
   - Risk: Increased Rds(on)/loss of conduction leading to heating or open-circuit.  <br>
   - Mitigation: Firmware detection, preventive maintenance, documented replacement schedule.<br>

## Diagnostics & Logging

- **Alarms & Codes**: `AlarmIdx` entries (kAlarm_NoLoad, kAlarm_OverCurrent, kAlarm_OffsetFail, kFailStateX) are logged via UART/Modbus for remote diagnostics.  <br>
- **Operational telemetry**: VRMS, IRMS, Offset, Voltage, CurrentSV, and state variables are exposed over Modbus for monitoring and trending.  <br>
- **Event logging**: Maintain event counts for trips and SSR end-of-life to support maintenance planning.<br>

## Recommended Safety Enhancements (Prioritized)

1. **Add a series mechanical contactor** upstream of SSR (acts as hard disconnect & maintenance switch).  <br>
2. **Add a fast analog overcurrent comparator + latch** to remove mains rapidly if a high-current spike occurs faster than firmware sampling. This comparator should directly drive contactor or a hardware trip input.  <br>
3. **Add non-resettable thermal fuse near hot zone** as absolute last resort.  <br>
4. **Add RCD / earth leakage detection** for shock/fire protection.  <br>
5. **Add surge protection (MOVs, TVS)** and common-mode chokes for EMI robustness.  <br>
6. **Consider redundant temperature/current sensing** for critical safety paths (voting).  <br>
7. **Document FMEA and proof test procedures** (periodic tests for watchdog, comparator, SSR, and thermal fuses).<br>

## Verification & Test Plan (Minimum)
**Unit tests / bench tests**<br>
- Analog front-end verification (gain, offset, range).  <br>
- CT/shunt calibration & ADC reading validation.  <br>
- Firmware test harness for state transitions and boundary conditions.<br>

**Integration tests**<br>
- Cold-start ramp verification across Volt1→Volt7 with oscilloscope capture of SSR trigger and current waveform.  <br>
- Simulated overcurrent injection to validate fast comparator + firmware reaction.  <br>
- SSR end-of-life simulation (force open/close behavior) and verify safe shutdown.<br>

**Field proof tests**<br>
- Burn-in (continuous operation) with telemetry logging for at least 1,000 hours. (You have 3+ years; preserve logs.)  <br>
- Proof test after maintenance: verify watchdog, comparator trip, SSR isolation, and thermal fuse behavior.<br>

**Certification prep**<br>
- Compile FMEA and traceability matrix mapping safety functions to hardware & firmware tests.  <br>
- Prepare EMC / dielectric and insulation tests for your operating voltage and environment.  <br>
- If pursuing formal SIL/PL, define target level (SIL2/PL d typical) and calculate diagnostic coverage and redundancy requirements.<br>

## Maintenance & Operations

- **SSR**: inspect and schedule replacement based on duty-cycle and logged event counts (observed EoL ~2 years in field for given workload).  <br>
- **Proof testing**: periodic functional test of watchdog, comparator, and no-load detection. Frequency based on operational criticality (recommend quarterly for critical furnaces).  <br>
- **Logging**: keep a rolling log of all alarm events and power cycles for root cause analysis.<br>

## Final Notes

This safety architecture balances **field-proven practical reliability** with recommended improvements to reach an auditable safety posture suitable for industrial certification. The current system shows strong real-world results; implementing the prioritized hardware redundancies and documented proof-test regime is the next step toward formal functional-safety compliance.
