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

## FLOW-002: Emergency exit activation and priority

**Purpose:** Confirm that emergency exits remain unavailable until activated, have lower route priority than normal exits, and provide extra capacity only when they materially reduce projected delay.

### Setup

- Use a custom layout with at least one reachable normal exit and one reachable emergency exit.
- Set the emergency exit narrower than the normal exit.
- Place enough people to form a queue at the normal exit.

### Procedure

1. Start the run without activating emergency exits.
2. Confirm no one is assigned to or enters the emergency-exit queue.
3. Activate emergency exits during the run.
4. Confirm the normal exit remains preferred while its projected delay is comparable, then confirm the emergency exit can be selected when it offers a meaningful delay reduction after its 30-second route-choice cost.
5. Confirm its queue service rate is `24 × emergency clear width` people/min.

**Acceptance:** Emergency exits are inactive before activation, receive the documented 30-second route-choice cost when activated, and their narrower width limits service capacity. All occupants eventually cross an exterior exit.

## FLOW-003: Second-floor evacuation through stairs

**Purpose:** Confirm that people can evacuate from the second floor through a capacity-limited stair link to a ground-floor exit.

### Hand calculation

```text
Stair service rate = 24 people/min/ft × 4 ft = 96 people/min
                    = 1.6 people/s

Idealized service time for 500 people = 500 ÷ 96
                                      = 5.208 min
                                      = 312.5 s
```

### Setup and procedure

1. Create a reachable ground-floor exit and a reachable second-floor starting region.
2. Add a stair link with a 4 ft clear width and 10 second traversal time, pairing one endpoint on each floor.
3. Set the second-floor crowd to 500 and the ground-floor crowd to 0.
4. Run the simulation and confirm a stair queue forms, people transfer to the ground floor, and all 500 eventually cross the ground-floor exit.
5. Confirm overall counts, plot, and milestones include both floors.

**Acceptance:** The stair's idealized queue service rate is 96 people/min and each occupant has the configured 10-second stair travel time after queue service. Total run time also includes walking and exit service, so 312.5 seconds is not a full-run target.
