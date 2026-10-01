# testmind

# Coverage-Based Regression Test Selection for CI/CD

## Overview

This project implements and evaluates a **coverage-based regression test selection (RTS) strategy** for Python projects in a CI/CD environment.

The project is based on the research paper:

> **"Regression Test Selection Tool for Python in Continuous Integration Process"**  
> Eero O. Kauhanen, Jukka K. Nurminen, Tommi Mikkonen, Matvey Pashkovskiy  
> VST 2021 / SANER 2021

The main idea is to avoid running the entire test suite every time code changes.

Instead, our system will:

```text
Code Change
     ↓
Identify Changed Files
     ↓
Analyze Test Coverage
     ↓
Identify Relevant Tests
     ↓
Run Only Selected Tests
     ↓
Evaluate Performance & Fault Detection
```

We will compare this against the traditional approach of running the **entire test suite**.

---

# Research Question

Our primary research question is:

> **Can coverage-based regression test selection reduce CI testing time while maintaining effective regression-fault detection?**

We will investigate:

1. How many tests can be safely excluded?
2. How much execution time can be saved?
3. How does selective testing affect fault detection?
4. How does the approach behave across different Python projects?

---

# Project Goals

## Required

Our implementation should be able to:

- Detect changed source files.
- Collect/process test coverage information.
- Determine which tests are related to changed files.
- Select only the relevant tests.
- Run the selected tests.
- Run the complete test suite for comparison.
- Measure execution time.
- Measure the number/percentage of tests selected.
- Evaluate fault detection using controlled mutations.
- Produce experimental results that can be analyzed in the final report.

## Optional

Only implement these if the core project is already working:

- Multiple levels of selection granularity.
- Additional open-source Python projects.
- More sophisticated mutation experiments.
- GitHub Actions demonstration.
- Additional performance/scalability experiments.
- Visualization/dashboard for results.

---

# Important Scope Boundary

This is a **research implementation and evaluation**, not a production CI/CD platform.

We are NOT trying to build:

- Jenkins
- GitHub Actions
- Kubernetes infrastructure
- A complete CI/CD platform
- A production-grade mutation testing framework
- A general-purpose dependency analysis platform
- An AI-based test selection system

The core research contribution is the **test-selection algorithm and its evaluation**.

Supporting tools/libraries may be used for things such as:

- Git operations
- Python test execution
- Coverage collection
- Data processing
- Plotting
- Mutation generation

However, the **core regression test selection strategy must be implemented by us** rather than simply calling an existing RTS library.

---

# Experimental Subjects

We will evaluate the approach on open-source Python projects hosted on GitHub.

A suitable project should ideally have:

- Python source code
- A working automated test suite
- pytest or another easily executable test framework
- Multiple source files/modules
- Sufficient test coverage
- A reasonable project size
- An open-source license
- A reproducible build/test process

We should aim for **2–3 projects** if practical.

One project is acceptable for an initial prototype, but multiple projects will make the evaluation stronger.

The repositories should NOT be permanently copied into this repository.

Instead, document the repositories and commit/version used in:

```text
datasets/README.md
```

---

# Project Architecture

```text
                         Git Repository
                              │
                              ▼
                     Change Detector
                              │
                              ▼
                    Changed Source Files
                              │
                              ▼
                     Coverage Analyzer
                              │
                              ▼
                       Test Mapper
                              │
                              ▼
                       Test Selector
                     [CORE ALGORITHM]
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              Selected Tests       Full Test Suite
                    │                   │
                    └─────────┬─────────┘
                              ▼
                         Test Runner
                              │
                              ▼
                          Evaluator
                              │
                              ▼
                           Results
```

---

# Repository Structure

```text
regression-test-selection/
│
├── src/
│   ├── __init__.py
│   ├── change_detector.py
│   ├── coverage_analyzer.py
│   ├── test_mapper.py
│   ├── test_selector.py
│   ├── test_runner.py
│   └── evaluator.py
│
├── experiments/
│   ├── run_experiment.py
│   ├── run_mutations.py
│   └── configurations/
│       └── experiment_config.json
│
├── datasets/
│   └── README.md
│
├── results/
│   ├── raw/
│   └── processed/
│
├── tests/
│   ├── __init__.py
│   ├── test_change_detector.py
│   ├── test_coverage_analyzer.py
│   ├── test_test_mapper.py
│   ├── test_test_selector.py
│   └── test_evaluator.py
│
├── docs/
│   ├── research-notes.md
│   ├── algorithm.md
│   └── experiment-plan.md
│
├── requirements.txt
├── README.md
├── .gitignore
└── LICENSE
```

---

# Components

## 1. Change Detector

**File:**

```text
src/change_detector.py
```

Purpose:

Determine which source files changed between two versions/commits.

Example:

```text
Commit A → Commit B

Changed:
    auth.py
    database.py
```

Possible supporting technology:

- Git
- GitPython
- Git command line

The Git library itself is not the research contribution.

---

## 2. Coverage Analyzer

**File:**

```text
src/coverage_analyzer.py
```

Purpose:

Determine which source files are executed by each test.

Example:

```text
test_login.py
    → auth.py
    → database.py

test_logout.py
    → auth.py

test_products.py
    → products.py
```

Possible supporting technology:

- coverage.py
- pytest

---

## 3. Test Mapper

**File:**

```text
src/test_mapper.py
```

Purpose:

Create a usable mapping between tests and source files.

For example:

```text
{
    "test_login.py": [
        "auth.py",
        "database.py"
    ],

    "test_logout.py": [
        "auth.py"
    ],

    "test_products.py": [
        "products.py"
    ]
}
```

This data will be used by the selection algorithm.

---

## 4. Test Selector — CORE ALGORITHM

**File:**

```text
src/test_selector.py
```

This is the most important component of the project.

Input:

```text
Changed files
+
Test → source-file coverage mapping
```

Output:

```text
Tests that should be executed
```

Example:

```text
Changed:

auth.py
database.py

Coverage:

test_login.py    → auth.py, database.py
test_logout.py   → auth.py
test_products.py → products.py

Selected:

test_login.py
test_logout.py

Skipped:

test_products.py
```

The exact selection strategy should follow the research paper we are implementing.

The implementation should be our own rather than simply using an existing RTS implementation.

---

# 5. Test Runner

**File:**

```text
src/test_runner.py
```

Purpose:

Run:

### Baseline

```text
ALL TESTS
```

and:

### RTS

```text
SELECTED TESTS ONLY
```

The runner should record information such as:

```text
Number of tests
Execution time
Pass/fail status
```

---

# 6. Evaluator

**File:**

```text
src/evaluator.py
```

Purpose:

Compare the baseline and RTS approaches.

Potential metrics:

### Test reduction

```text
(number of all tests - selected tests)
--------------------------------------- × 100
             all tests
```

### Time reduction

```text
(full execution time - RTS execution time)
------------------------------------------- × 100
             full execution time
```

### Fault detection

Measure whether the selected tests detect faults that the complete test suite detects.

### Missed-fault rate

Determine how many injected faults were not detected by the selected tests.

---

# Mutation Testing

Mutation testing will be used to evaluate whether selective testing misses faults.

The basic idea:

```text
Original Code
     ↓
Introduce Small Fault
     ↓
Run Full Test Suite
     ↓
Run Selected Test Suite
     ↓
Compare Detection
```

Example:

```python
if user.is_valid():
```

could be mutated into something such as:

```python
if not user.is_valid():
```

If the full suite detects the mutation but our selected tests do not, that mutation represents a missed fault.

We should use existing mutation tooling where appropriate rather than spending most of the project building a complete mutation-testing framework.

The RTS algorithm itself remains our implementation.

---

# Experimental Comparison

For every experiment we want to compare:

| Metric | Full Test Suite | Selected Tests |
|---|---:|---:|
| Tests executed | X | X |
| Execution time | X sec | X sec |
| Faults detected | X | X |
| Faults missed | X | X |

From this we can calculate:

- Test reduction %
- Time reduction %
- Fault detection rate
- Missed-fault rate

---

# Experimental Process

For each selected GitHub project:

```text
1. Clone/project setup
        ↓
2. Establish baseline
        ↓
3. Collect coverage
        ↓
4. Create/identify code changes
        ↓
5. Detect changed files
        ↓
6. Select relevant tests
        ↓
7. Run ALL tests
        ↓
8. Run SELECTED tests
        ↓
9. Compare execution
        ↓
10. Perform mutation experiments
        ↓
11. Record results
```

Experiments should be reproducible.

Record:

- Repository
- Commit/version
- Number of source files
- Number of tests
- Changed files
- Selected tests
- Full-suite runtime
- Selected-suite runtime
- Mutation information
- Fault detection results

---

# Work Division

We are a team of two.

The work can initially be divided into:

## Member 1

Focus:

- Change detection
- Coverage analysis
- Test mapping
- Experiment infrastructure
- Test execution

## Member 2

Focus:

- Core test-selection algorithm
- Evaluation logic
- Mutation experiments
- Result processing

Both members should understand the entire system before submission.

The division is for parallel development, not for creating two independent projects.

---

# Development Strategy

We should build the project incrementally.

## Phase 1 — Basic Prototype

Goal:

```text
Changed file
    ↓
Relevant tests
```

No mutation testing yet.

---

## Phase 2 — Full Experiment

Add:

- Full-suite execution
- Selected-suite execution
- Runtime measurement
- Test reduction measurement

At this point we should already have a working RTS system.

---

## Phase 3 — Fault Detection

Add:

- Mutation experiments
- Fault detection comparison
- Missed-fault measurement

---

## Phase 4 — Multiple Projects

Run the system against additional GitHub projects.

---

## Phase 5 — Analysis

Generate:

- Tables
- Graphs
- Statistical summaries where appropriate
- Conclusions

---

# Definition of Done

The project is considered technically complete when:

- [ ] We can identify changed files.
- [ ] We can obtain test coverage.
- [ ] We can map tests to source files.
- [ ] Our RTS algorithm selects relevant tests.
- [ ] We can run the selected tests automatically.
- [ ] We can run the complete test suite automatically.
- [ ] We can compare execution times.
- [ ] We can calculate test reduction.
- [ ] We can perform mutation/fault experiments.
- [ ] We can compare fault detection.
- [ ] Experiments can be reproduced.
- [ ] Results are saved in a structured format.
- [ ] We have results from at least one suitable open-source project.
- [ ] Ideally, we have results from 2–3 projects.

---

# Timeline Target

Although the final deadline is later in the semester, our goal should be to finish the **technical implementation and primary experiments before midterms**.

### Target

```text
September
    ↓
Paper + proposal + repository setup

Early October
    ↓
Prototype + coverage + test mapping

Mid October
    ↓
Core RTS algorithm complete

Late October
    ↓
Experiments + mutation testing

End of October
    ↓
TECHNICAL PROJECT COMPLETE

November
    ↓
Analysis + graphs + report + polishing

Final week
    ↓
Demo preparation
```

Finishing the technical work early gives us time to fix unexpected problems and improve the evaluation rather than rushing before the final deadline.

---

# What We Should NOT Do

Avoid scope creep.

Do not start building:

- A web dashboard
- A full CI/CD platform
- Kubernetes infrastructure
- A Jenkins replacement
- An AI/ML test-selection model
- A custom mutation-testing framework
- A distributed testing system
- A VS Code extension
- A complicated frontend

Unless the core experiment is already complete, these are distractions.

**The research question comes first.**

---

# Immediate Next Steps

1. Finish reading the selected research paper.
2. Extract the exact RTS strategy we need to implement.
3. Choose 2–3 candidate GitHub Python projects.
4. Verify that their tests run successfully.
5. Establish baseline test execution.
6. Finalize the proposal.
7. Begin implementing the RTS pipeline.

---

# Final Deliverable

The final project should demonstrate:

> Given a change to a Python project, our system can identify and execute a subset of tests that are relevant to that change, and we can experimentally measure the trade-off between testing efficiency and fault detection.

The final report will document:

1. What strategy we implemented.
2. How we implemented it.
3. How we evaluated it.
4. What results we obtained.
5. What conclusions can be drawn from those results.