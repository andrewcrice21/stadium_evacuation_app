# Stadium Evacuation Simulator

## Purpose
Provide a single-file browser app, implemented in the top-level `index.html`, to simulate crowd evacuation from a venue floor plan. Animate movement and door queuing while tracking evacuation milestones and remaining crowd over time.

## Inputs
- Floor plan: select a built-in sample, draw/edit a custom layout, import a structured layout, or import an image for tracing/editing; an imported image must be traced into editable layout geometry before simulation (supported file formats TBD)
- Layout content: walls, exits (including door widths), and obstacles such as rows of seats; dimensions and door widths are in feet
- Crowd size (100 to 2,000 people) and initial placement, randomized within a user-selected region
- User-configurable walking speed (units, allowed range, and default TBD)
- Simulation playback controls (start, pause, reset, and animation playback speed); playback speed affects animation only, while walking speed affects simulation movement

## Calculations
- Find walkable routes to exits around walls and obstacles; choose the exit with the lowest estimated total travel time, including walking time and queue delay, and allow people to switch exits as congestion changes (reassessment timing TBD)
- Apply walking speed, density-based slowdown, and queuing at bottlenecks and exits using a published model/formula (model and source TBD)
- Reject a layout before simulation if the selected crowd region contains an area with no walkable route to an exit
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
- Door-flow check: 500 people through a 6 ft door at approximately 24 people/min/ft (SFPE Handbook; exact edition and page TBD) gives a theoretical door-flow time of about 3.47 minutes (500 / (24 × 6) = 3.472 min). Validate door flow separately against this estimate; the pass/fail tolerance is TBD.
- Total-evacuation check: include walking time to the door, so total evacuation time should exceed the door-flow-only estimate. Room dimensions, crowd-to-door distance, initial crowd distribution, and pass/fail criterion are TBD.
- Validate routing around walls and obstacles, exit selection and switching under congestion, and rejection of a crowd region with no route to an exit.
- Validate 50%, 90%, and 100% evacuation milestones, the remaining-people plot, and CSV/PNG exports.

## Out of scope
Stairs and elevation changes, built-in real-venue presets (users may draw their own venue layouts), and automated exit-layout optimization.
