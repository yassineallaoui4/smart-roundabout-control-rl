# Methodology

## Reference and scope

This description follows the owner's selected neural implementation and the supplied tabular implementation. It distinguishes their actual behavior from richer variants described in academic materials. The executable implementation remains private.

## Network

The supplied SUMO network contains five controlled entries with two lanes each. The approach edge IDs are E0, E1, E2.77, E3, and E4; the signal IDs are J5, J9, J10, J1, and J3 respectively. The central ring includes E10, E11, E7, E8, and E9. The tabular controller also inspects specified internal junction edges.

Demand is defined by probabilistic flows with origin/destination edges over a 3,600-second injection interval. The supplied routes define a passenger-car type. No ambulance flow is present in that route file. The academic report describes field observations; underlying collection records have not been verified.

## Tabular state and actions

State classification first checks whether any monitored ring segment contains a stopped vehicle. If so, the state is 4. Otherwise, three queue totals are compared: E1 + E4, E3 + E0, and E2.77. An empty set of queues gives state 0; the dominant group gives state 1, 2, or 3, with ties resolved by the order of the conditions.

Four actions open one of those groups or set all signals red. The configuration remains active for seven simulation steps. This implementation has no intervening yellow sequence.

At each step, the reward subtracts one for every vehicle with speed below 0.1 m/s and three for each monitored ring segment containing a stopped vehicle. Rewards are summed over the action interval, then used for a Q-table update. This is not the -50 per blocked ring vehicle described in some presentation material.

## Neural state and actions

The eleven inputs consist of five stopped-vehicle counts, one ring vehicle count, and five phase indices. The western approach count is multiplied by 1.2. Counts are divided by 30; phase indices are divided by the maximum phase index in the current observation, with a zero-vector fallback. This changing phase scale requires review, particularly when direct signal-state changes are used.

The twelve actions are hand-selected joint signal strings. Eleven use combinations of `Gr` and `GG`; one uses `rr` at every signal. `Gr` is not all-red: its first controlled link stays green. The exact movement mapping must be checked before drawing conclusions about conflicts.

## Neural reward

The reference implementation uses:

- Squared queue terms, with count features divided again by 50 after state normalization.
- A penalty of 5 per approach vehicle stopped for more than 20 seconds.
- A ring penalty of 50 when the normalized ring count reaches 10/50. Because the state divides the count by 30, this triggers at 6 vehicles rather than 10.
- An ambulance-wait penalty based on vehicle IDs containing `ambulance`.

These details are recorded to preserve the distinction between intended reward design and implemented behavior. The queue scaling and ring threshold should be corrected or explicitly justified before retraining. The ambulance accumulator can count the same vehicle again while traversing subsequent edges.

## Learning and timing

The neural update selects a future action using the main network and evaluates it using the target network. Replay samples transitions uniformly from a deque. The two networks are initialized independently and are not synchronized immediately; initial synchronization is a required correction.

The transition records omit a terminal flag, so the learning target continues to bootstrap at the episode boundary. The time counter also begins after an initial simulation step. These issues need explicit treatment before comparing runs.

The neural action lasts up to fifteen simulation steps, with three yellow and three clearance steps when a change requires them. Action durations can therefore differ. A comparison should state whether discounting is per decision or per unit of simulated time.

Epsilon decreases after each tabular decision but only after each neural episode. Both implementations multiply epsilon conditionally without clamping it to the intended minimum, allowing a slight undershoot.

## What a comparison can establish

The controllers differ in observation richness, action space, reward, action duration, and transition handling. A shared-scenario comparison can evaluate these two complete controller designs. It cannot isolate the effect of replacing a table with a neural network unless those other factors are controlled in a separate experiment.
