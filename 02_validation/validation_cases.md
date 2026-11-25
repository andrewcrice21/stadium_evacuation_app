# Validation Cases

## FLOW-001: 500 people through one 6 ft door

**Purpose:** Check the idealized exit-flow benchmark separately from the time people need to walk to the exit.

### Inputs

- People: 500
- Exits: one
- Clear exit width: 6 ft
- Specific flow: 24 people/min/ft
- Jamb-width deduction: none

### Hand calculation

```text
Exit capacity = 24 people/min/ft × 6 ft = 144 people/min
               = 2.4 people/s

Idealized door-flow time = 500 people ÷ 144 people/min
                         = 3.472 min
                         = 208.3 s
```

### Expected results

- The theoretical door-service benchmark is approximately **3.47 minutes (208.3 seconds)**.
- The app uses the full 6 ft clear width, so its modeled exit capacity is 144 people/min.
- In an app run with 500 people and this single exit, the queue should form when arrivals exceed service capacity, and all 500 people should eventually evacuate.
- The app's total elapsed time includes travel to the exit as well as queue service. Do not compare total elapsed time directly with 3.47 minutes; a fixed geometry and walking-time acceptance criterion are still needed for that comparison.

### Actual test result

- Measured result: **209.4 seconds**
- Outcome: **Success**

### Procedure

1. Use a valid layout with one 6 ft exit and a reachable starting region.
2. Set the crowd size to 500 and start the simulation.
3. Confirm the exit queue forms, the evacuated count reaches 500, and the run completes without stalling.
4. Record the full elapsed time for comparison only after the walking-time conditions and pass/fail tolerance have been defined.

**Acceptance:** The hand calculation rounds to 3.47 minutes. The recorded test completed successfully in 209.4 seconds. The app smoke check confirms queueing and completion; it does not treat 3.47 minutes as the full-simulation target.
