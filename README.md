**Cutting Stock (AOS5) — Column Generation**

- **Purpose:** Study the cutting stock problem using column generation, evaluate scalability, and compare to a conventional LP (all-patterns) formulation when feasible.
- **Scope:** Run on random-generated instances and a small provided test instance; collect runtime and solution quality metrics.

**Repository Structure**

- `cutting-stock.ipynb`: Main notebook with a working pattern-based LP using PuLP; base for adding column generation and experiments.
- `other-problems-solved/`: Supporting notebooks/examples (diet, production) not central to this project.

**Requirements**

- **Python:** 3.9+ (3.10 recommended)
- **Packages:** `pulp`, `numpy`, `matplotlib`, `jupyter`
- **Solver:** PuLP’s CBC works by default; you can also use other LP/MIP solvers supported by PuLP if available.

Install into a virtual environment:

- `python -m venv .venv && source .venv/bin/activate`
- `pip install pulp numpy matplotlib jupyter`

**Quick Start**

- Launch Jupyter: `jupyter lab` (or `jupyter notebook`).
- Open `cutting-stock.ipynb` and run all cells.
- The current notebook solves a cutting stock instance via the pattern-based LP (enumerates feasible patterns and minimizes stock used subject to demand).

**Small Test Instance**

- In the notebook, set instance parameters near the top (example):
- `stock_length = 100`
- `item_lengths = [45, 36, 31, 14]`
- `demands = [97, 610, 395, 211]`
- Replace these with your small test instance values, then Run All.

**Random Instance Runs**

- To test scalability quickly, add a cell before pattern generation with something like:
- ```python
  import numpy as np
  rng = np.random.default_rng(seed=42)

  stock_length = 100
  n_types = 5                  # try 5, 8, 10, ...
  item_lengths = sorted(rng.integers(5, stock_length//2, size=n_types).tolist(), reverse=True)
  base_demand = 50
  demands = (base_demand + rng.integers(0, 50, size=n_types)).tolist()
  print(dict(stock_length=stock_length, item_lengths=item_lengths, demands=demands))
  ```
- Then run the remaining cells to solve. Increase `n_types` and demand scale to stress-test.

**Column Generation Plan**

- **Master LP:** Start with a small set of patterns; minimize number of stocks with demand-cover constraints. Variables are pattern usage counts.
- **Pricing (Subproblem):** Given master dual prices, solve a 0/1 knapsack variation to find a new pattern with negative reduced cost. Repeat until no improving pattern exists.
- **Stop Criterion:** No pattern with reduced cost < 0 (or within a small tolerance) is found.
- **Integerization (optional):** If needed, add integrality or apply rounding/branch-and-price only for small cases.

Suggested notebook sections to add:

- Pattern initialization (e.g., one item per stock patterns)
- Master LP build + resolve loop with updated columns
- Pricing via DP/knapsack using dual prices
- Logging: iterations, patterns added, objective, runtime

**Comparison vs. Conventional LP**

- For small instances, the existing all-patterns LP is a strong baseline.
- Compare on the same instances:
- **Metrics:** objective value, total runtime, number of patterns generated, memory footprint, LP iterations.
- **Expectations:** Column generation should handle larger item-type counts by avoiding full enumeration; baseline may be competitive on tiny instances.

**Scalability Study Guide**

- **Vary problem size:** number of item types (e.g., 5–40), demand scale, and stock length ranges.
- **Trials:** ≥5 random instances per size; fix seeds for reproducibility.
- **Record:** instance params, objective, runtime (master/pricing/total), patterns added, final columns, solver used.
- **Output:** save to CSV (e.g., `results/experiments.csv`) for plotting runtime vs. size.

Example timing snippet to wrap a solve in the notebook:

- ```python
  import time, csv, os
  t0 = time.perf_counter()
  # solve model here
  t1 = time.perf_counter()
  os.makedirs('results', exist_ok=True)
  with open('results/experiments.csv', 'a', newline='') as f:
      w = csv.writer(f)
      w.writerow([stock_length, len(item_lengths), sum(demands), t1 - t0, 'baseline'])
  ```

**Reproducibility**

- Set seeds for random generators (`numpy.random.default_rng(seed=...)`).
- Capture solver version (CBC/others) in the results CSV header or a separate metadata file.

**References**

- Gilmore, P. C., & Gomory, R. E. (1961, 1963): Foundational papers on cutting stock and column generation.
- Desaulniers, Desrosiers, Solomon (2005): Column Generation, Springer.

**Notes**

- The current notebook solves the baseline (all-patterns) LP. Column generation scaffolding and experiment logging are to be added following the plan above.
- If you have a specific small test instance, place its values at the top of the notebook and re-run to reproduce results.

