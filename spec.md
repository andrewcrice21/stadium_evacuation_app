# Stadium Evacuation Simulator

## Purpose
Provide a single-file browser app, implemented in the top-level `index.html`, to simulate crowd evacuation from a venue floor plan. Animate movement and door queuing while tracking evacuation milestones and remaining crowd over time.

## Inputs
- Floor plan: select a built-in sample, draw/edit a custom layout on either of two floors, import a version 1 or version 2 JSON layout, or load a PNG/JPEG image as a tracing guide; simulation uses editable geometry, not the reference image
- Structured layout files are JSON files up to 1 MB. Version 1 uses the fixed 70 × 40 ft canvas and contains an optional `name`, `region` rectangle or `null`, and an `items` array of up to 500 uniquely identified `seat`, `wall`, and normal `exit` objects. Version 2 contains two floor definitions, each with a crowd count, region, and floor-local items; it also contains stair links connecting endpoints on the two floors. Version 1 files remain supported as single-floor layouts.
- A `seat` has `id`, `type`, `x`, `y`, `width`, and `height`; a `wall` has `id`, `type`, `x1`, `y1`, `x2`, and `y2`; an `exit` has `id`, `type`, `kind`, `side`, `position`, `width`, and `label`. `kind` is `normal` or `emergency` (omitted means `normal`); exit position runs along its named room edge, and labels must be unique within a floor.
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
- A version 2 layout uses `floors: [{id, crowdCount, region, items}, ...]` for floor IDs 1 and 2, and `stairs: [{id, width, travelTime, ends: [{floorId, x, y}, {floorId, x, y}]}]`. Each stair link must have one endpoint on each floor. Item IDs and exit labels are unique within each floor. For example:
  ```json
  {
    "version": 2,
    "name": "Two-level hall",
    "floors": [
      { "id": 1, "crowdCount": 250, "region": { "x": 5, "y": 5, "width": 70, "height": 40 }, "items": [
        { "id": 1, "type": "exit", "kind": "normal", "side": "east", "position": 25, "width": 6, "label": "Ground Exit" }
      ] },
      { "id": 2, "crowdCount": 250, "region": { "x": 5, "y": 5, "width": 70, "height": 40 }, "items": [] }
    ],
    "stairs": [
      { "id": 1, "width": 4, "travelTime": 10, "ends": [
        { "floorId": 1, "x": 35, "y": 25 },
        { "floorId": 2, "x": 35, "y": 25 }
      ] }
    ]
  }
  ```
- A PNG/JPEG tracing image up to 10 MB and 20 megapixels is fitted, aspect-preserving, as a faded background on the 70 × 40 ft canvas. Add editable walls, exits, obstacles, and a crowd region with the drawing tools; the image itself does not define dimensions or routes.
- Layout content: walls, normal and emergency exits (including clear widths), obstacles such as rows of seats, and paired stair endpoints; dimensions and widths are in feet
- Crowd count and initial placement for each floor, randomized within that floor's user-selected region; the combined population is 100 to 2,000 people
- User-configurable walking speed from 0.5 to 3.0 miles per hour (mph), default 2.7 mph
- Simulation playback controls (start, pause, reset, and animation playback speed); playback speed changes wall-clock pacing but not modeled walking speed or simulated evacuation times
- An operator can activate emergency exits while a run is in progress. Until activated, they are unavailable to route choice. Once activated, they receive a fixed 30-second route-choice cost; this keeps normal exits preferred unless using an emergency exit meaningfully reduces projected delay. Emergency-exit queue capacity is based on its clear width, like a normal exit.
- Stair links connect paired locations between the two floors. Each link has a configured traversal time, a FIFO queue, and 24 people/min/ft of stair clear width for service capacity. This shared specific-flow rate is a simplifying assumption for this educational model. Stairs are modeled connectors, not exterior exits.

## Calculations
- Find walkable routes around walls and seat blocks on a 0.5-foot grid on each floor; connect the two floor grids through stair links and require every walkable cell in each occupied starting region to reach at least one exterior exit
- Calculate average starting density from the crowd count and walkable area in the selected region. Below 0.05 people/ft², use the selected free-flow walking speed; otherwise use the level-floor hydraulic relation `S = 275(1 − 2.86D)` ft/min, capped at the selected free-flow speed. Apply this density-adjusted walking speed throughout the run; model congestion with exit queues rather than re-estimating local density from the moving agents. Reject a starting region at or above the relation's zero-speed density (about 0.35 people/ft²).
- Cap each exit at 24 people/min per foot of its clear width, without subtracting a jamb boundary layer, to match the accepted 500-person / 6-ft door benchmark. Queue people first-in, first-out and estimate queue delay from the current queue and people already assigned to that exit.
- Choose routes by estimated walking time, stair and exit queue delay, and any emergency-exit preference cost. Reassess every 5 simulated seconds and switch only when another route's estimate is at least 10% lower; do not switch a person after joining a stair or exit queue.
- The hydraulic model is summarized by [DGA Consulting](https://dgaconsulting.net/tools/egress-time-calculator/) and references Fruin (1971), Predtechenskii and Milinskii (1978), Pauls (1987), and [NIST Technical Note 1680](https://nvlpubs.nist.gov/nistpubs/TechnicalNotes/NIST.TN.1680.pdf). This aggregate-density approximation does not resolve changing local congestion, counterflow, or crowd interactions; it is an educational model, not an engineering analysis.
- Count a person as evacuated when they cross an exit; track the times when 50%, 90%, and 100% of the crowd have crossed, and finish when everyone has crossed

## Outputs
- Animated selected-floor plan showing people walking toward stairs and exits, with queue counts for stairs and exits
- Floor selector for the first- and second-floor layouts; crowd totals, progress plots, and evacuation milestones aggregate both floors
- During a run, elapsed time and evacuated/remaining crowd counts, updated live
- A live plot with time in minutes on the horizontal axis and people remaining on the vertical axis
- Total evacuation time and the times when 50% and 90% of the crowd have evacuated
- After completion, CSV export of per-floor starting populations, aggregate 50%, 90%, and 100% milestone times, and the all-floor remaining-people time series in simulated seconds and minutes
- PNG capture of the currently displayed floor-plan frame, including people, stair/exit queues, layout geometry, and any tracing image; the selected floor is named in the downloaded file
- If a layout is rejected, an explanation that the selected crowd region contains an area with no route to an exit

## Look and feel
Educational and approachable, using simple visual cues. Keep the floor plan as the main focus, with controls and live results in supporting panels. Support desktop, tablet, and phone layouts, with readable contrast and labels or other non-color cues so information is not conveyed by color alone. Load a sample floor plan and crowd at startup, ready for the user to start the simulation. The sample starting region covers the walkable floor area to avoid an unrealistic initial crowd density.

## Validation
- Door-flow check: 500 people through a 6 ft clear door at 24 people/min/ft gives a theoretical door-flow time of about 3.47 minutes (500 / (24 × 6) = 3.472 min). The simulation uses the full clear width without a jamb deduction so it matches this benchmark; validate door flow separately from walking time.
- Emergency-exit check: before manual activation, no person is assigned to an emergency exit. After activation, normal exits remain preferred when projected costs are comparable, while sufficiently larger normal-exit delays can make a narrower emergency exit a useful relief route. Verify the added 30-second preference cost and width-limited queue service.
- Multi-floor check: each floor's population, starting density, and geometry are validated independently; all occupants can use a direct exterior exit or a connected stair route to another floor's exit. Verify stair traversal time, FIFO service at 24 people/min/ft, and no completion until all people cross an exterior exit.
- Total-evacuation check: include walking time to the door plus queue service time. Room dimensions, crowd-to-door distance, initial crowd distribution, and pass/fail criterion are TBD.
- Validate the average-density speed adjustment, exit queue formation and service rate, multiple-exit assignment and switching at the 5-second / 10% rule, and rejection of a starting region with no route or excessive average density.
- Validate the live remaining-people plot and 50%, 90%, and 100% milestone times at exit crossings; reset clears both.
- Verify pause stops simulated time and playback speed changes wall-clock pacing without changing simulated walking speed or evacuation times.
- Validate version 1 single-floor and version 2 two-floor JSON imports, including clear errors for invalid files or unpaired stair endpoints. Verify PNG/JPEG tracing images remain reference-only and editable geometry is required for simulation.
- Validate completed-run CSV milestone and time-series rows, PNG capture of the displayed floor-plan frame, and responsive/accessibility behavior for the import/export controls.

## Out of scope
More than two floors, elevators, fire/smoke effects, built-in real-venue presets (users may draw their own venue layouts), and automated exit-layout optimization. This simplified stair-flow assumption is not an engineering design criterion.
