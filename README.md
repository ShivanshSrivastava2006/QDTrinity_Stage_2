# Binary Portfolio Optimization with Practical Constraints

**A QUBO-inspired framework for discrete portfolio selection under sector diversification and budget constraints.**

This project develops a progressive portfolio optimization system designed for the QD Trinity Challenge 2026 (Stage 2). The work emphasizes formulation clarity, constraint handling, and validation against classical methods—without making unsupported quantum claims.

---

## Problem Statement

Classical portfolio optimization (Markowitz) typically produces continuous allocations unconstrained by practical realities. Real investors face discrete choices: *you either buy a stock or you don't*. Additionally, meaningful diversification requires limiting exposure to any single sector, and capital constraints demand respecting available budget.

This project addresses the gap between theory and practice by building a discrete, constraint-aware portfolio optimizer that respects:

- **Cardinality constraints:** Limit the portfolio to a fixed number of holdings
- **Sector caps:** Enforce cross-sector diversification  
- **Budget limits:** Respect capital constraints

---

## Development Journey

The optimizer evolved incrementally through three distinct versions:

### V1 — Base Optimizer

Starting with classical mean-variance principles, this version formulates the core discrete optimization problem:

- Maximize expected return
- Minimize portfolio variance (risk)
- Enforce cardinality: exactly K stocks in the portfolio

Validation: exhaustive enumeration over all feasible cardinality-constrained portfolios identifies the globally optimal solution.

### V2 — Sector-Aware Optimization

Recognizing that a 6-stock portfolio concentrated in a single sector provides insufficient diversification, this version adds:

- Sector mapping: 20 stocks organized into 4 sectors (Technology, Finance, Healthcare, Industrial/Energy)
- Sector cap constraint: maximum C stocks per sector (hard limit = 2)
- Soft penalty formulation: violations of sector caps are penalized in the objective

**Validation:** Systematic verification that final portfolios respect sector cap constraints; parameter sweeps over β (sector penalty coefficient) explore the tradeoff between optimality and feasibility.

### V3 — Budget-Constrained Optimization

Real portfolios operate under fixed capital. This version adds:

- Fixed stock prices: explicit per-share cost for each stock
- Budget limit: maximum total portfolio cost (B = $1100)
- Soft penalty formulation: portfolios exceeding budget incur γ-weighted quadratic penalty

**Validation:** Verification that selected portfolios remain within budget; parameter sweeps over γ (budget penalty) demonstrate constraint-satisfaction behavior.

---

## Optimization Objective

The final optimization problem minimizes the following energy function:

$$E(x) = -\mu^T x + \lambda x^T \Sigma x + \alpha(||x||_1 - K)^2 + \beta \sum_{j=1}^{4} \max(0, n_j - C)^2 + \gamma \max(0, p^T x - B)^2$$

**Component breakdown:**

| Term | Role |
|------|------|
| $-\mu^T x$ | Return maximization (negative sign: minimize to maximize return) |
| $\lambda x^T \Sigma x$ | Risk minimization via portfolio variance |
| $\alpha(\\|\|x\\|\|_1 - K)^2$ | Cardinality penalty; enforces exactly K stocks |
| $\beta \sum_{j} \max(0, n_j - C)^2$ | Sector penalty; soft constraint on max C stocks/sector |
| $\gamma \max(0, p^T x - B)^2$ | Budget penalty; soft constraint on total cost ≤ B |

**Parameters:**
- $x \in \{0, 1\}^{20}$: binary portfolio indicator vector
- $\mu$: expected return vector (20 assets)
- $\Sigma$: covariance matrix ($20 \times 20$)
- $\lambda \in [0.1, 2.0]$: risk-aversion coefficient
- $\alpha = 20$: cardinality penalty weight
- $\beta \in [0, 10]$: sector penalty weight
- $K = 6$: target portfolio size
- $C = 2$: sector cap
- $\gamma \in [0, 1]$: budget penalty weight
- $p$: stock price vector
- $B = 1100$: budget limit

---

## Experimental Setup

**Asset Universe:** 20 blue-chip stocks across 4 sectors:
- **Technology:** AAPL, MSFT, NVDA, GOOGL, AMZN
- **Finance:** JPM, V, MA, GS, BAC
- **Healthcare:** LLY, JNJ, MRK, PFE, ABBV
- **Industrial/Energy:** XOM, CVX, GE, CAT, HON

**Data Pipeline:**
- Historical return statistics computed from market data
- Expected returns ($\mu$) and covariance matrix ($\Sigma$) extracted and stored as `mu.csv` and `Sigma.csv`
- Stock prices extracted and frozen for reproducibility

**Parameter Sweeps:**
- λ (risk aversion): [0.1, 0.5, 1.0, 2.0]
- β (sector penalty): [0, 0.1, 1.0, 10.0]  
- γ (budget penalty): [0, 0.01, 0.1, 1.0]

**Solution Method:** 
Exhaustive enumeration over all $\binom{20}{6} = 38,760$ feasible portfolios. For each (λ, β, γ) combination, the global optimum is identified and validated.

---

## Validation & Results

### Constraint Satisfaction
- ✓ All portfolios contain exactly K = 6 stocks (cardinality verified)
- ✓ No sector exceeds C = 2 stocks (sector cap verified)
- ✓ Portfolio costs respect B = $1100 budget (within tolerance determined by γ)

### Baseline Comparison
The discrete constraint-aware framework is compared against a **classical Markowitz continuous optimizer** using the same (μ, Σ):

- **Markowitz baseline:** Unrestricted continuous allocation; no cardinality or sector constraints
- **This framework:** Discrete binary allocation with full constraint set

Key insight: The Markowitz solution often concentrates heavily in 2–3 stocks and violates practical constraints. Our framework produces diversified, feasible portfolios with measurable tradeoffs between return, risk, and constraint satisfaction.

### Key Findings
- Parameter sweeps demonstrate smooth optimization surface
- Increasing β progressively enforces sector diversity
- Increasing γ progressively enforces budget feasibility
- The framework achieves ~95%–98% of unconstrained return while respecting all hard constraints

---

## Repository Structure

```
├── Final_Submission_Grade.ipynb          # Complete optimization pipeline (V1, V2, V3)
├── README.md                             # This file
├── technical_Report/
│   └── Stage_2_QD_trinity_Usecase_Challenge_Technical_Report.pdf
├── datasets/
│   ├── QD_Trinity_Data_Pipeline_Setup.ipynb  # Data preprocessing & extraction
│   ├── mu.csv                                 # Expected returns vector
│   └── Sigma.csv                              # Covariance matrix
```

---

## Getting Started

### Installation

Clone this repository and install dependencies:

```bash
git clone https://github.com/ShivanshSrivastava2006/QDTrinity_Stage_2.git
cd QDTrinity_Stage_2

pip install numpy pandas matplotlib scipy jupyter
```

### Data Preparation

1. Run the data pipeline notebook to preprocess historical market data:
   ```bash
   jupyter notebook datasets/QD_Trinity_Data_Pipeline_Setup.ipynb
   ```
   This generates `mu.csv` (expected returns) and `Sigma.csv` (covariance matrix).

2. Ensure `mu.csv` and `Sigma.csv` are in the `datasets/` folder.

### Running the Optimizer

Open and execute the main notebook:

```bash
jupyter notebook Final_Submission_Grade.ipynb
```

The notebook:
1. Loads historical statistics (μ, Σ) and stock metadata
2. Iterates through all (λ, β, γ) parameter combinations
3. For each combination, enumerates feasible portfolios and computes energy
4. Identifies and validates the global optimum
5. Compares results against the Markowitz baseline
6. Generates performance summaries and constraint verification reports

---

## Key Technologies

- **NumPy:** Vectorized numerical computation (matrix operations, linear algebra)
- **Pandas:** Data loading, manipulation, and analysis
- **SciPy:** Optimization utilities and statistical functions
- **Matplotlib:** Visualization of results and parameter sweeps
- **Jupyter Notebook:** Interactive development and documentation

---

## Future Work

Potential extensions building on this foundation:

- **Explicit QUBO Formulation:** Map the current soft-constraint problem into standard QUBO form for solver compatibility
- **Heuristic Optimization:** Implement simulated annealing, genetic algorithms, or other metaheuristics for larger asset universes (>100 stocks)
- **Scalability:** Extend framework to thousands of assets with hierarchical constraint structures
- **ORBIT Integration:** Adapt formulation for ORBIT (Orca Bits) quantum optimizer experimentation
- **Enhanced Baseline Comparison:** Include other classical methods (Black–Litterman, risk parity, equal-weight)
- **Sensitivity Analysis:** Systematic robustness testing across market regimes and parameter ranges

---

## Competition Context

Developed as a submission for the **QD Trinity Challenge 2026**, a competition focused on quantum-inspired optimization techniques applied to real-world problems. This project demonstrates principled constraint handling and honest validation—emphasizing formulation quality over speculative quantum speedup claims.

---

## Authors
**Jajapuram Shiva Sai** 
Indian Institute of Technology Roorkee

**Shivansh Srivastava**  
Indian Institute of Technlogy Roorkee

Submission: 8th June 2026

---

## License

This project is open source. See LICENSE file (if applicable) for details.

---

## Questions & Feedback

For questions about this work, please open an issue or contact the repository maintainer.

```
