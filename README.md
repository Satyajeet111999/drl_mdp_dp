## **Autonomous Drone Rescue using Dynamic Programming** 

## **Problem Statement** 

A disaster-hit city requires autonomous rescue drones capable of navigating dangerous environments, rescuing stranded civilians, avoiding hazardous regions, and managing battery resources intelligently. 

The objective of this assignment is to design a custom reinforcement learning environment modeled as a finite Markov Decision Process (MDP) and solve it using Dynamic Programming techniques. 

The drone must learn: 

- Efficient rescue planning 

- Safe navigation 

- Battery management 

- Charging station utilization 

- Avoidance of dangerous zones 

- Handling environmental uncertainty caused by wind 

The problem is solved using: 

```
Value Iteration
```

## **Project Configuration** 

## **Group Number** 

```
G=121
```

Since the last digit of the group ID is: 

```
1
```

the environment follows the assignment configuration for IDs ending with:  

```
0–4
```

## **Environment Configuration** 

## **Grid Size** 

```
5 × 5 Grid
```

## **Maximum Battery Capacity** 

Since the last digit is odd: 

```
Maximum Battery = 15
```

## **Maximum Episode Steps** 

For 5×5 grids: 

```
Maximum Steps = 50
```

## **Environment Layout** 

The environment contains: 

|Element|Count|
|---|---|
|Rescue Targets|2|
|Charging Stations|1|
|Danger Zones|3|
|Wind Zones|2|
|Blocked Cells|2|



## **Grid Symbols** 

|Symbol|Meaning|
|---|---|
|S|Start Position|
|F|Free Cell|
|D|Danger Zone|
|R|Rescue Target|
|C|Charging Station|
|W|Wind Zone|
|X|Blocked Cell|
|A|Drone Position|



## **Grid Placement** 

## **Start Position** 

```
S=(0,0)
```

The drone always starts from the top-left corner. 

## **Rescue Targets** 

```
[(1,3),(4,1)]
```

These cells contain stranded civilians. 

When reached: 

- reward is granted 

- rescue target disappears 

- cell becomes free 

## **Charging Station** 

```
[(2,4)]
```

When the drone enters a charging station: 

```
Battery becomes fully recharged
```

If the drone performs: 

```
HOVER
```

while on charging station: 

```
Battery increases by +2
```

without exceeding maximum capacity. 

## **Danger Zones** 

```
[(1,1),(3,2),(4,4)]
```

Entering danger zones produces: 

```
Heavy negative reward
```

but does not terminate the episode immediately. 

## **Wind Zones** 

```
[(0,2),(2,3)]
```

Wind zones introduce stochastic transitions. 

If the drone attempts movement inside a wind zone: 

```
20% probability
```

exists that movement direction changes randomly. 

Possible disturbed directions: 

```
UP
DOWN
LEFT
RIGHT
```

## **Blocked Cells** 

```
[(1,4),(3,0)]
```

Blocked cells are inaccessible. 

If movement attempts to enter blocked cell: 

- drone remains in current position • battery is still consumed 

## **Action Space** 

The drone can perform five actions. 

|Action ID|Action|Movement|
|---|---|---|
|0|UP|(-1, 0)|
|1|DOWN|(1, 0)|
|2|LEFT|(0, -1)|
|3|RIGHT|(0, 1)|
|4|HOVER|(0, 0)|



## **Understanding Movement Representation** 

Movement is represented as: 

```
(row_change, column_change)
```

Example: 

```
(-1,0)
```

means: 

- move one row upward • column unchanged 

Similarly: 

```
(0,1)
```

means: 

- row unchanged • move one column right 

## **State Space Representation** 

Each state is represented as: 

```
(row,column,battery_level,rescue_status)
```

Example: 

```
(2,3,10,(True,False))
```

means: 

- drone located at row 2, column 3 • battery remaining = 10 • first rescue completed • second rescue pending 

## **Rescue Status Tracking** 

The rescue status tuple: 

```
(False,False)
```

tracks whether rescue targets are completed. 

For two rescue targets: 

|Status|Meaning|
|---|---|
|(False, False)|No rescues completed|
|(True, False)|First rescue completed|
|(False, True)|Second rescue completed|
|(True, True)|All rescues completed|



This is required because future rewards depend on remaining rescue targets. 

## **Total Number of States** 

The total state space is computed as: 

```
(Valid Positions)
×
(Battery Levels)
×
(Rescue Status Combinations)
```

## **Step 1 – Valid Positions** 

Total grid cells: 

```
5 × 5 = 25
```

Blocked cells: 

```
2
```

Valid positions: 

```
25 - 2 = 23
```

## **Step 2 – Battery Levels** 

Battery values: 

```
1 to 15
```

Total battery states: 

```
15
```

## **Step 3 – Rescue Combinations** 

Two rescue targets: 

```
2² = 4 combinations
```

## **Final State Space** 

```
23 × 15 × 4
=
1380 states
```

## **Reward Structure** 

The assignment reward design is strictly followed. 

|Event|Reward|
|---|---|
|Rescue Target Reached|+20|
|Enter Danger Zone|-10|
|Battery Exhausted|-20|
|Reach Charging Station|+5|
|Regular Movement|-1|



## **Episode Termination Conditions** 

An episode terminates when: 

|Condition|Description|
|---|---|
|Battery becomes zero|Drone power exhausted|
|All rescues completed|Mission successful|
|Maximum steps reached|50 steps exceeded|

 

## **Dynamic Programming Approach** 

The environment is solved using: 

```
Value Iteration
```

## **Bellman Optimality Update** 

The value function update is: 

```
V(s) = max [ R + γV(s') ]
```

Where: 

|Symbol|Meaning|
|---|---|
|V(s)|Current state value|
|R|Immediate reward|
|γ|Discount factor|
|V(s')|Next state value|



## **Convergence Criteria** 

The algorithm iteratively updates state values until: 

```
Delta < Theta
```

Where: 

Theta = 10^-3


## **Policy Extraction** 

After convergence: 

- optimal action for every state is selected • resulting optimal policy guides drone navigation 

The policy attempts to: 

- maximize rescue reward • avoid danger • manage battery intelligently 

## **Policy Visualization** 

The optimal policy is visualized using arrows. 

|Symbol|Action|
|---|---|
|↑|UP|
|↓|DOWN|
|←|LEFT|
|→|RIGHT|
|H|HOVER|



This helps visualize: 

- preferred movement directions 

- rescue paths 

- charging behaviour 

- danger avoidance 

## **State Value Heatmap** 

A heatmap of optimal state values is generated. 

Higher values indicate: 

- safe regions 

- proximity to rescue targets 

- efficient battery access 

Lower values indicate: 

- danger zones 

- inefficient regions 

- risky movement paths 

## **Wind Transition Dynamics** 

Wind zones introduce uncertainty. 

Inside wind zones: 

```
20% probability
```

exists that movement direction changes randomly. 

This simulates real-world environmental disturbances. 

## **Observed Policy Behaviour** 

The learned policy demonstrates: 

- intelligent rescue planning 

- charging station usage 

- danger avoidance 

- battery-aware navigation 

Some simulations terminate due to: 

```
Maximum step limit
```

instead of rescue completion because charging rewards may occasionally encourage repeated chargingstation visits. 

However, this still satisfies assignment requirements because maximum-step termination is a valid episodeending condition. 

## **Curse of Dimensionality** 

Dynamic Programming becomes computationally expensive as state space grows. 

For example: 

## **Current State Space** 

```
23 × 15 × 4
=
1380 states
```

## **If Grid Becomes 10×10** 

```
100 × 15 × 4
=
6000 states
```

## **If 5 Rescue Targets Exist** 

```
100 × 15 × 2^5
=
48000 states
```

State explosion makes classical DP difficult. 
 

## **Conclusion** 

successfully implementation of: 

- a custom Drone Rescue MDP environment 

- Value Iteration based Dynamic Programming 

- optimal policy computation 

- policy visualization 

- heatmap-based state value analysis 

This demonstrates how reinforcement learning principles can be applied to autonomous disaster-response systems. 


