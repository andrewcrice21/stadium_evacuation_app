# Stadium Evacuation Simulator

## Purpose
Provide a single-file browser app, implemented in the top-level `index.html`, to simulate crowd evacuation from a venue floor plan. Animate movement and door queuing while tracking evacuation milestones and remaining crowd over time.

## Inputs
- Floor plan: select a built-in sample, draw/edit a custom layout, import a version 1 JSON layout, or load a PNG/JPEG image as a tracing guide; simulation uses editable geometry, not the reference image
- Structured layout files are JSON files up to 1 MB. They use the fixed 70 × 40 ft canvas and contain a `version: 1`, optional `name`, `region` rectangle or `null`, and an `items` array of up to 500 uniquely identified `seat`, `wall`, and `exit` objects. Coordinates and dimensions are in feet; imported items must fit the canvas.
- A `seat` has `id`, `type`, `x`, `y`, `width`, and `height`; a `wall` has `id`, `type`, `x1`, `y1`, `x2`, and `y2`; an `exit` has `id`, `type`, `side`, `position`, `width`, and `label`. Exit position runs along its named room edge; exit labels must be unique.
- Minimal layout example:
  ```json
  {
    "version": 1,
    "name": "Training Hall",
    "region": { "x": 5, "y": 5, "width": 70, "height": 40 },
    "items": [
      { "id": 1, "type": "exit", "side": "east", "position": 25, "width": 6, "label": "Exit A" }
    ]
  }
  ```
- A PNG/JPEG tracing image up to 10 MB and 20 megapixels is fitted, aspect-preserving, as a faded background on the 70 × 40 ft canvas. Add editable walls, exits, obstacles, and a crowd region with the drawing tools; the image itself does not define dimensions or routes.
- Layout content: walls, exits (including door widths), and obstacles such as rows of seats; dimensions and door widths are in feet
- Crowd size (100 to 2,000 people) and initial placement, randomized within a user-selected region
- User-configurable walking speed from 0.5 to 3.0 miles per hour (mph), default 2.7 mph
- Simulation playback controls (start, pause, reset, and animation playback speed); playback speed changes wall-clock pacing but not modeled walking speed or simulated evacuation times

## Calculations
- Find walkable routes around walls and seat blocks on a 0.5-foot grid; require every walkable cell in the starting region to reach at least one exit
- Calculate average starting density from the crowd count and walkable area in the selected region. Below 0.05 people/ft², use the selected free-flow walking speed; otherwise use the level-floor hydraulic relation `S = 275(1 − 2.86D)` ft/min, capped at the selected free-flow speed. Apply this density-adjusted walking speed throughout the run; model congestion with exit queues rather than re-estimating local density from the moving agents. Reject a starting region at or above the relation's zero-speed density (about 0.35 people/ft²).
- Cap each exit at 24 people/min per foot of its clear width, without subtracting a jamb boundary layer, to match the accepted 500-person / 6-ft door benchmark. Queue people first-in, first-out and estimate queue delay from the current queue and people already assigned to that exit.
- Choose exits by estimated walking time plus queue delay. Reassess every 5 simulated seconds and switch only when another exit's estimate is at least 10% lower; do not switch a person after they join an exit queue.
- The hydraulic model is summarized by [DGA Consulting](https://dgaconsulting.net/tools/egress-time-calculator/) and references Fruin (1971), Predtechenskii and Milinskii (1978), Pauls (1987), and [NIST Technical Note 1680](https://nvlpubs.nist.gov/nistpubs/TechnicalNotes/NIST.TN.1680.pdf). This aggregate-density approximation does not resolve changing local congestion, counterflow, or crowd interactions; it is an educational model, not an engineering analysis.
- Count a person as evacuated when they cross an exit; track the times when 50%, 90%, and 100% of the crowd have crossed, and finish when everyone has crossed

## Outputs
- Animated floor plan showing people walking toward exits, exit queues, and queue counts by exit
- During a run, elapsed time and evacuated/remaining crowd counts, updated live
- A live plot with time in minutes on the horizontal axis and people remaining on the vertical axis
- Total evacuation time and the times when 50% and 90% of the crowd have evacuated
- After completion, CSV export of 50%, 90%, and 100% milestone times and the remaining-people time series in simulated seconds and minutes
- PNG capture of the currently displayed floor-plan frame, including people, queues, layout geometry, and any tracing image
- If a layout is rejected, an explanation that the selected crowd region contains an area with no route to an exit

## Look and feel
Educational and approachable, using simple visual cues. Keep the floor plan as the main focus, with controls and live results in supporting panels. Support desktop, tablet, and phone layouts, with readable contrast and labels or other non-color cues so information is not conveyed by color alone. Load a sample floor plan and crowd at startup, ready for the user to start the simulation. The sample starting region covers the walkable floor area to avoid an unrealistic initial crowd density.

## Validation
- Door-flow check: 500 people through a 6 ft clear door at 24 people/min/ft gives a theoretical door-flow time of about 3.47 minutes (500 / (24 × 6) = 3.472 min). The simulation uses the full clear width without a jamb deduction so it matches this benchmark; validate door flow separately from walking time.
- Total-evacuation check: include walking time to the door plus queue service time. Room dimensions, crowd-to-door distance, initial crowd distribution, and pass/fail criterion are TBD.
- Validate the average-density speed adjustment, exit queue formation and service rate, multiple-exit assignment and switching at the 5-second / 10% rule, and rejection of a starting region with no route or excessive average density.
- Validate the live remaining-people plot and 50%, 90%, and 100% milestone times at exit crossings; reset clears both.
- Verify pause stops simulated time and playback speed changes wall-clock pacing without changing simulated walking speed or evacuation times.
- Validate supported JSON/PNG/JPEG imports and clear errors for invalid files; verify tracing images remain reference-only and editable geometry is required for simulation.
- Validate completed-run CSV milestone and time-series rows, PNG capture of the displayed floor-plan frame, and responsive/accessibility behavior for the import/export controls.

## Out of scope
Stairs and elevation changes, built-in real-venue presets (users may draw their own venue layouts), and automated exit-layout optimization.
