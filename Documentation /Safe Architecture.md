## Firmware Architecture

The firmware is implemented as a deterministic real-time control loop with the following responsibilities:<br>

- sampling CT data and computing RMS current<br>
- verifying offset stability and amplifier health<br>
- ramping SSR trigger voltage based on load behavior and setpoint<br>
- executing a multi-stage safety state machine<br>
- driving SSR trigger outputs with phase-angle-like shaping<br>
- managing fault recovery and fail-safe shutdown paths<br>

The firmware is written in C, with:<br>
- no dynamic memory allocation<br>
- no background threads<br>
- deterministic timing<br>

This design is suitable for safety-critical industrial operation.<br>
