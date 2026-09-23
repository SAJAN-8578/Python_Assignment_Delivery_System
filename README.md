# Delivery Simulation
## Overview
This project implements a Python-based delivery simulation for FastBox. It reads delivery data from a JSON file containing warehouses, delivery agents, and packages.
The simulation:
1. Reads and parses the input JSON.
2. Supports two different JSON input formats.
3. Assigns each package to the nearest available agent.
4. Simulates the route:

   `Agent → Warehouse → Destination`

5. Calculates the total distance traveled by each agent.
6. Calculates delivery efficiency.
7. Identifies the most efficient agent.
8. Saves the final result to `report.json`.
---
## Files

```text
project/
│
├── base_case.json       # Sample input format
├── new_case.json        # Alternate input format (if provided)
├── delivery_simulation.py
├── report.json          # Generated output
└── README.md
```
If the Python code is being executed in a Jupyter Notebook, the notebook can be used instead of `delivery_simulation.py`.
---
## Requirements
- Python 3.x
- No external packages are required.

The program uses only Python standard libraries:

```python
import json
import math
```
## How to Run

### 1. Place the input JSON file in the project directory

For example:

```text
base_case.json
```

### 2. Set the input filename

Inside the Python program:

```python
input_file = "base_case.json"
```

For another input file:

```python
input_file = "new_case.json"
```

### 3. Run the program

```bash
python delivery_simulation.py
```
The program will print the package assignments and distances and generate:

```text
report.json
```
# Input Formats Supported

The program supports both formats supplied for the assignment.

## Format 1 — List-Based Format

Example:

```json
{
    "warehouses": [
        {
            "id": "W1",
            "location": [0, 0]
        },
        {
            "id": "W2",
            "location": [50, 75]
        }
    ],
    "agents": [
        {
            "id": "A1",
            "location": [5, 5]
        },
        {
            "id": "A2",
            "location": [60, 60]
        }
    ],
    "packages": [
        {
            "id": "P1",
            "warehouse_id": "W1",
            "destination": [30, 40]
        }
    ]
}
```

## Format 2 — Dictionary-Based Format

Example:

```json
{
    "warehouses": {
        "W1": [34, 29],
        "W2": [95, 4]
    },
    "agents": {
        "A1": [89, 16],
        "A2": [52, 21]
    },
    "packages": [
        {
            "id": "P1",
            "warehouse": "W1",
            "destination": [12, 7]
        }
    ]
}
```

The program automatically detects the format and normalizes the data internally.

It also supports both package warehouse fields:

```text
warehouse_id
```
and:

```text
warehouse
```
# Delivery Logic

Each package follows this route:

```text
Agent → Warehouse → Destination
```
The total distance for a package is:

```text
Agent → Warehouse
+
Warehouse → Destination
```
For example:
```text
Agent = [5, 5]
Warehouse = [0, 0]
Destination = [30, 40]
```
The calculation is:

```text
Agent → Warehouse = 7.071
Warehouse → Destination = 50.000

Total = 57.071
```

# Distance Calculation
Euclidean distance is used:
```text
distance = √((x2-x1)² + (y2-y1)²)
```

The implementation is:

```python
def calculate_distance(point1, point2):

    x1, y1 = point1
    x2, y2 = point2

    return math.sqrt(
        (x2 - x1) ** 2 +
        (y2 - y1) ** 2
    )
```
# Agent Assignment

For each package, the nearest agent to the package's warehouse is selected.
Importantly, the agent's **current location** is used.
For example:

```text
Agent starts at [5,5]
        ↓
Delivers package
        ↓
Agent moves to destination
        ↓
Agent's current location changes
        ↓
Next package uses the new location
```

This prevents every package from incorrectly assuming that the agent starts from the original location.

---

# Assumptions and Engineering Decisions

The assignment allows reasonable engineering decisions when logic is ambiguous. The following assumptions are used.

## 1. Package Processing Order

Packages are processed in the same order in which they appear in the input JSON.

Example:

```text
P1 → P2 → P3 → P4 → ...
```

### Reason
No package priority, deadline, or scheduling rule is specified. Preserving input order provides a deterministic and reproducible simulation.
## 2. Current Agent Location

Agent assignment uses the agent's **current location**, not the original starting location.
After an agent delivers a package, its current location becomes the package destination.
### Reason

This represents realistic movement during a delivery day and avoids incorrectly resetting agents to their starting locations.
## 3. Delivery Route
Every package follows:
```text
Agent → Warehouse → Destination
```
No return trip to the original location is assumed after delivery.

---

## 4. Nearest-Agent Assignment
The nearest agent to the warehouse is selected using Euclidean distance.
### Reason
The assignment explicitly specifies nearest-agent assignment based on distance.
---

## 5. Tie-Breaking

If two agents have exactly the same minimum distance from a warehouse, the agent appearing first in the input data is selected.

### Reason

This provides deterministic behavior without introducing an arbitrary random choice.

Python dictionaries preserve insertion order, so the result is reproducible.

---

## 6. Multiple Packages

An agent can deliver multiple packages during the same simulation.

There is no restriction that an agent can only receive one package.

---

## 7. Every Package Must Be Delivered

Each package is assigned to exactly one agent.

No package is intentionally skipped.

---

## 8. Package Order Is Not Optimized

The program does not reorder packages to minimize the global total distance.

### Reason

The input does not provide package priorities or a required route-optimization algorithm. Processing the supplied order is predictable and easy to reproduce.

---

## 9. Efficiency

Efficiency is calculated as:

```text
Efficiency = Total Distance / Packages Delivered
```
A lower value means fewer distance units traveled per delivered package.
---
## 10. Agents With No Deliveries

An agent that delivers zero packages receives:

```text
packages_delivered = 0
total_distance = 0.0
efficiency = 0.0
```
However, an agent with zero deliveries is excluded when selecting `best_agent`.

### Reason
An agent that delivered no packages should not be considered the top-performing delivery agent.
---
## 11. Best Agent

The best agent is the active agent with the lowest average distance per delivered package.

```text
best_agent =
agent with minimum efficiency
```
Only agents who delivered at least one package are considered.

## 12. Input Data

The JSON input is treated as the source of truth.

The program does not hard-code:

- Warehouse coordinates
- Agent coordinates
- Package destinations
- Number of packages
- Number of warehouses
- Number of agents

This allows the program to work with different valid JSON datasets.
---
# Output
The program generates `report.json`.

Example structure:

```json
{
    "A1": {
        "packages_delivered": 2,
        "total_distance": 108.28,
        "efficiency": 54.14
    },
    "A2": {
        "packages_delivered": 2,
        "total_distance": 142.42,
        "efficiency": 71.21
    },
    "A3": {
        "packages_delivered": 1,
        "total_distance": 25.0,
        "efficiency": 25.0
    },
    "best_agent": "A3"
}
```
The exact values depend on the input JSON.
# Error Handling

The program checks whether the referenced warehouse exists.
If a package refers to an unknown warehouse, the program raises an error such as:

```text
ValueError: Warehouse 'W99' does not exist.
```
This prevents invalid package data from silently producing incorrect results.
# Design Approach

The implementation separates the problem into small functions:

```text
calculate_distance()
        ↓
find_nearest_agent()
        ↓
simulate_delivery()
        ↓
calculate efficiency
        ↓
find best agent
        ↓
save report.json

This makes the code easier to understand, test, and modify.
# Testing

The program should be tested with:

1. The provided `base_case.json`.
2. The alternate dictionary-based JSON.
3. Different numbers of agents.
4. Different numbers of warehouses.
5. Multiple packages assigned to the same warehouse.
6. Agents starting at different locations.
7. A package with a destination equal to its warehouse.
8. Cases where two agents are equally distant from a warehouse.
9. Cases where an agent receives multiple packages.

The important invariant is:

```text
Total packages delivered
=
Total packages in input
```
# Result Verification

After running the program, verify:

```python
import json

with open("report.json", "r") as file:
    report = json.load(file)

print(json.dumps(report, indent=4))
```
