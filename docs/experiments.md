# Experimental Evidence and Evaluation Plan

## Evidence available

| Item | What is established | What remains unknown |
|---|---|---|
| Tabular reference | Five states, four actions, seven configured episodes | Completed run count and associated test outputs |
| Neural reference | Eleven inputs, two 32-unit hidden layers, twelve outputs, 120 configured episodes | Completed run count and matching saved models |
| Archived neural weights | All 79 inspected archives describe 16 inputs, hidden layers 64/64/32, twelve outputs, and Huber loss | Exact producing script, training configuration, and input/action semantics |
| Training CSV | Three rows with epochs 1–3 and seven columns | Producing script, measurement semantics, seed, and model association |
| Report and slides | Training and comparison figures are included | Raw test series and reproducible mapping to the reference code |

The three-row CSV includes epsilon, cumulative reward, total waiting time, maximum queue length, arrived vehicles, and ambulance waiting time. It must not be interpreted as 78 completed training episodes merely because similarly named model files are available. Filenames alone also do not identify the best model.

## Local scenario check

A scenario-only execution reached 3,600 simulated seconds under installed SUMO 1.22.0. The input configuration indicates it was generated with SUMO 1.26.0. The run reported 70 vehicle teleports and unfinished demand. No RL agent participated in that check. This is a reproducibility diagnostic, not a validated performance benchmark or a comparison with the academic results.

## Planned evaluation

1. Freeze code, network, routes, dependencies, and controller configurations; record their hashes and random seeds.
2. Resolve model/reference incompatibility, reward scaling, initial target synchronization, terminal handling, and metric definitions.
3. Evaluate fixed signals, Q-Learning, and Double DQN using identical traffic demand and paired random seeds.
4. Disable exploration during evaluation and keep evaluation data separate from training.
5. Repeat across independent seeds and traffic conditions; report distributions and uncertainty, not only the best run.
6. Record incomplete trips, vehicles waiting to enter, collisions reported by the simulator, and teleports. Report the teleport policy explicitly.
7. Evaluate emergency-vehicle behavior separately using a documented ambulance scenario.

## Metric definitions to settle before measurement

| Metric | Intended measurement |
|---|---|
| Waiting time | Per-vehicle waiting time with a stated vehicle population and treatment of unfinished trips |
| Queue length | Stopped vehicles per approach at a fixed sampling interval; summarize mean and maximum |
| Throughput | Vehicles completing trips within a defined interval, alongside injected and pending demand |
| Travel time | Departure-to-arrival duration, with unfinished trips reported separately |
| Speed | A stated vehicle-weighted or time-weighted aggregation |
| Return | Reward sum within each controller; not a cross-controller score when rewards differ |
| Loss | Training loss recorded against update count; diagnostic rather than traffic performance |
| Emergency waiting | Per-ambulance waiting time counted once per observation, with identification rules stated |

Repeatedly summing a vehicle's accumulated waiting time is not equivalent to measuring mean waiting time per vehicle. Existing column labels alone cannot resolve that distinction.

## Figures to publish after verification

- Matched waiting-time and queue-length comparisons with uncertainty across runs.
- Throughput with unfinished and not-yet-inserted demand visible.
- Separate learning curves for each algorithm using clearly labeled horizontal axes.
- Emergency-vehicle results only after verifying the relevant scenario and controller.

No numerical improvement claim or reconstructed training chart is included in this showcase until the underlying evidence supports it.
