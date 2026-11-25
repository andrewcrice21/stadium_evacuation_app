# Stadium Evacuation Simulator

## Purpose
Provide a single-file browser app, implemented in the top-level `index.html`, to simulate crowd evacuation from a venue floor plan. Animate movement and door queuing while tracking evacuation milestones and remaining crowd over time.

## Inputs
- Floor plan: select a built-in sample, draw/edit a custom layout, import a structured layout, or import an image for tracing/editing; an imported image must be traced into editable layout geometry before simulation (supported file formats TBD)
- Layout content: walls, exits (including door widths), and obstacles such as rows of seats; dimensions and door widths are in feet
- Crowd size (100 to 2,000 people) and initial placement, randomized within a user-selected region
- User-configurable walking speed from 0.5 to 3.0 miles per hour (mph), default 2.7 mph
- Simulation playback controls (start, pause, reset, and animation playback speed); playback speed affects animation only, while walking speed affects simulation movement

## Calculations
- Find walkable routes around walls and seat blocks on a 0.5-foot grid; during the single-exit movement stage, reject a crowd region with any walkable grid area that cannot reach the exit
- Move people at their selected constant walking speed (0.5–3.0 mph, default 2.7 mph); do not apply density slowdown or door queues until the later flow-model stage
- Later, choose among exits by estimated walking time plus queue delay, allow exit switching as congestion changes, and apply the selected published density/flow model (model, source, and reassessment timing TBD)
- Count a person as evacuated when they cross an exit; track the times when 50%, 90%, and 100% of the crowd have crossed, and finish when everyone has crossed

## Outputs
- Animated floor plan showing people walking toward exits and queuing at doors
- During a run, elapsed time and evacuated/remaining crowd counts, updated live
- A live plot with time in minutes on the horizontal axis and people remaining on the vertical axis
- Total evacuation time and the times when 50% and 90% of the crowd have evacuated
- CSV export of milestone times and the remaining-people time series
- PNG capture of the currently displayed floor-plan frame
- If a layout is rejected, an explanation that the selected crowd region contains an area with no route to an exit

## Look and feel
Educational and approachable, using simple visual cues. Keep the floor plan as the main focus, with controls and live results in supporting panels. Support desktop, tablet, and phone layouts, with readable contrast and labels or other non-color cues so information is not conveyed by color alone. Load a sample floor plan and crowd at startup, ready for the user to start the simulation.

## Validation
- Door-flow check: 500 people through a 6 ft door at approximately 24 people/min/ft (SFPE Handbook; exact edition and page TBD) gives a theoretical door-flow time of about 3.47 minutes (500 / (24 × 6) = 3.472 min). Validate this theoretical benchmark separately; Stage 3 movement does not model door flow or queues, and the pass/fail tolerance is TBD.
- Total-evacuation check: once the flow model is implemented, include walking time to the door and compare total evacuation time with the door-flow-only estimate. Room dimensions, crowd-to-door distance, initial crowd distribution, and pass/fail criterion are TBD.
- Validate routing around walls and obstacles, exit selection and switching under congestion, and rejection of a crowd region with no route to an exit.
- Validate 50%, 90%, and 100% evacuation milestones, the remaining-people plot, and CSV/PNG exports.

## Out of scope
Stairs and elevation changes, built-in real-venue presets (users may draw their own venue layouts), and automated exit-layout optimization.
