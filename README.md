# Reinforcement Learning for Treatment Recommendation using Multi-Armed Bandits

## Problem Statement

A hospital is evaluating multiple medicines for treating a chronic disease. The true effectiveness of each medicine is initially unknown. The objective is to develop a recommendation system that learns from patient outcomes over time and progressively identifies the most effective medicine.

This problem is modeled as a **Multi-Armed Bandit (MAB)** problem, where:

* Each medicine represents an arm.
* Each patient treatment produces a reward.
* The system must balance exploration and exploitation to maximize long-term clinical benefit.

---

# Project Configuration

## Group Number

```python
G = 121
```

To ensure reproducibility:

```python
random.seed(G)
np.random.seed(G)
```

---

## Number of Medicines

The assignment defines the number of available medicines as:

```text
K = (G mod 3) + 5
```

For our group:

```text
K = (121 mod 3) + 5
K = 1 + 5
K = 6
```

Therefore, we have:

```text
Medicine 0
Medicine 1
Medicine 2
Medicine 3
Medicine 4
Medicine 5
```

---

## Hidden Success Probabilities

Each medicine has a hidden recovery probability:

```text
Pi = 0.4 + ((G + i) mod 6) × 0.07
```

For Group 121:

| Medicine | Hidden Success Probability |
| -------- | -------------------------- |
| M0       | 0.47                       |
| M1       | 0.54                       |
| M2       | 0.61                       |
| M3       | 0.68                       |
| M4       | 0.75                       |
| M5       | 0.40                       |

Important:

The recommendation algorithms do not know these values.

Only the environment knows them.

---

# Dataset Design

## Patient Generation

The environment contains exactly 1000 patients:

```text
Patient IDs = 0 to 999
```

---

## Severity Score

Each patient receives a disease severity score:

```text
Severity = (patient_id mod 5) + 1
```

This produces:

| Patient ID | Severity |
| ---------- | -------- |
| 0          | 1        |
| 1          | 2        |
| 2          | 3        |
| 3          | 4        |
| 4          | 5        |
| 5          | 1        |

The pattern repeats every 5 patients.

Interpretation:

```text
1 = Mild
2 = Moderate
3 = Serious
4 = Severe
5 = Critical
```

---

## Clinical Outcome

When a medicine is assigned:

```text
Recovered (1) with probability Pi

Not Recovered (0) with probability 1 - Pi
```

This is simulated using:

```python
np.random.binomial(1, Pi)
```

---

## Utility Score

The assignment introduces a utility score that decreases for highly severe patients.

Formula:

```text
Utility Score =
clinical_outcome × (1 - severity / 10)
```

Examples:

| Outcome | Severity | Utility |
| ------- | -------- | ------- |
| 1       | 1        | 0.9     |
| 1       | 2        | 0.8     |
| 1       | 3        | 0.7     |
| 1       | 4        | 0.6     |
| 1       | 5        | 0.5     |
| 0       | Any      | 0       |

Important:

* Clinical Outcome is used for learning.
* Utility Score is used for evaluation.

---

# Understanding the Multi-Armed Bandit Problem

Imagine a casino containing six slot machines.

```text
Machine 0
Machine 1
Machine 2
Machine 3
Machine 4
Machine 5
```

Each machine pays out with a different probability.

You do not know which machine is best.

Every pull teaches you something.

In our problem:

```text
Medicine = Slot Machine

Recovery = Win

No Recovery = Loss
```

The goal is to discover the best medicine while treating patients.

---

# Learning Through Rewards

For each medicine we maintain:

```text
Q(Medicine)
```

which represents:

```text
Estimated Success Rate
```

Initially:

```text
Q0 = 0
Q1 = 0
Q2 = 0
Q3 = 0
Q4 = 0
Q5 = 0
```

As patients are treated, these estimates become more accurate.

Example:

Medicine 4 outcomes:

```text
1
1
1
0
1
```

Estimated success:

```text
Q4 = 4/5 = 0.8
```

The algorithm begins to believe Medicine 4 is effective.

---

# Task 2: Immediate Exploitation Strategy

## Idea

The hospital administrator proposes:

> Once a treatment appears best, continue prescribing only that treatment.

---

## Step 1: Initial Testing

Each medicine is tested exactly 10 times.

```text
Medicine 0 → 10 patients
Medicine 1 → 10 patients
Medicine 2 → 10 patients
Medicine 3 → 10 patients
Medicine 4 → 10 patients
Medicine 5 → 10 patients
```

Total:

```text
60 patients
```

---

## Step 2: Select Best Medicine

Suppose estimates become:

```text
M0 = 0.4
M1 = 0.5
M2 = 0.6
M3 = 0.7
M4 = 0.8
M5 = 0.3
```

The algorithm selects:

```text
Medicine 4
```

---

## Step 3: Exploit Forever

All remaining patients receive Medicine 4.

```text
Patient 61 → M4
Patient 62 → M4
...
Patient 1000 → M4
```

---

## Risk

A medicine may appear superior due to luck.

The algorithm may lock onto a suboptimal medicine and never recover.

This is called:

```text
Premature Convergence
```

---

# Task 3: Controlled Clinical Trial (ε-Greedy)

## Idea

Doctors suggest:

> Most patients should receive the current best treatment, but occasionally another treatment should be tested.

---

## Exploration vs Exploitation

With:

```text
ε = 10%
```

The algorithm behaves as:

```text
90% Exploitation

10% Exploration
```

---

### Exploitation

```python
action = np.argmax(Q)
```

Choose the currently best medicine.

---

### Exploration

```python
action = random.randint(0, K-1)
```

Choose a random medicine.

---

## Why Explore?

Because our current belief may be wrong.

Exploration prevents us from missing a better medicine.

---

## ε = 1%

```text
99% Exploitation
1% Exploration
```

Advantages:

* Quickly focuses on the best medicine.

Disadvantages:

* Can become stuck with a poor choice.

---

## ε = 50%

```text
50% Exploitation
50% Exploration
```

Advantages:

* Learns a lot.

Disadvantages:

* Continues testing poor medicines unnecessarily.

---

## ε = 10%

Provides a balance between:

```text
Learning
and
Reward Maximization
```

---

# Task 4: UCB1 Strategy

## Idea

A senior physician proposes:

> Treatments with fewer observations should initially be given more chances, but this preference should reduce as evidence grows.

---

## Core Principle

UCB1 evaluates:

```text
Score =
Estimated Success
+
Confidence Bonus
```

The confidence bonus measures uncertainty.

---

### Example

Medicine 4:

```text
75 successes from 100 patients
```

Medicine 2:

```text
7 successes from 10 patients
```

Medicine 2 receives a larger uncertainty bonus because it has been tested fewer times.

---

## Behavior

Initially:

```text
Explore under-tested medicines
```

Later:

```text
Focus on medicines with consistently strong outcomes
```

The exploration naturally decreases over time.

---

## Why UCB1 Works

Unlike ε-Greedy:

* No exploration parameter is required.
* Exploration is automatic.
* Uncertainty is explicitly considered.

---

# Cumulative Reward

Each algorithm records:

```text
Cumulative Reward
```

Example:

Patient rewards:

```text
0.9
0
0.7
0.5
```

Cumulative rewards:

```text
0.9
0.9
1.6
2.1
```

---

## Interpretation

Higher cumulative reward means:

```text
More successful treatment decisions
```

Lower cumulative reward means:

```text
Less effective decision-making
```

---

# Task 5: Comparative Analysis

All strategies are compared using:

```text
Cumulative Reward vs Number of Patients
```

The graph helps visualize:

* Learning speed
* Long-term performance
* Exploration efficiency

---

# Expected Results

True hidden probabilities:

```text
M0 = 0.47
M1 = 0.54
M2 = 0.61
M3 = 0.68
M4 = 0.75
M5 = 0.40
```

Medicine 4 is the true best medicine.

Good algorithms should eventually discover this.

Typical performance ordering:

```text
UCB1
≈
Epsilon Greedy (10%)

↓

Immediate Exploitation

↓

Epsilon Greedy (1%)

↓

Epsilon Greedy (50%)
```

Actual results may vary slightly due to randomness.

---

# End-to-End Patient Journey

Consider Patient #237.

Severity:

```text
(237 mod 5) + 1 = 3
```

Suppose ε-Greedy chooses:

```text
Medicine 4
```

The environment simulates recovery:

```text
75% chance of recovery
```

Suppose the patient recovers:

```text
clinical_outcome = 1
```

Utility:

```text
1 × (1 - 3/10)

= 0.7
```

The algorithm updates its estimate of Medicine 4.

The cumulative reward increases by 0.7.

The system has now learned slightly more than before.

This cycle repeats for every patient:

```text
Choose Treatment
        ↓
Observe Outcome
        ↓
Update Knowledge
        ↓
Make Better Future Decisions
```

This loop is the essence of Reinforcement Learning and the central objective of this assignment.
