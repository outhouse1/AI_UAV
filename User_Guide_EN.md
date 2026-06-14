# User Guide: Reproducing the K-HLIDF UAV Emergency Logistics Experiments with AI-KM and Local Programs

This guide helps users reproduce the Section 4 experiments from the paper *Knowledge-based Human-LLM Interaction Decision Support Framework for UAV Emergency Logistics Network Optimization*. It describes only the public reproduction workflow, data, and program entry points used by the paper experiments.

For general AI-KM platform usage, see `AI-KM_README.md` in this folder. This project also provides directly runnable Python programs, so users can reproduce result tables, case outputs, and figures without rerunning the full AI-KM interaction process.

---

## 1. Reproduction Goals

This project evaluates the role of K-HLIDF in UAV emergency logistics network optimization. It does not propose a new routing model or a single new metaheuristic. The reproduction goals are:

1. Map natural-language emergency dispatching requirements into MDVRP model elements through a structured semantic template;
2. Generate executable solution programs through AI-KM knowledge bases, workflows, skill mode, and feedback verification;
3. Run local programs on public data to reproduce benchmark experiments, real-world cases, and dynamic re-dispatching experiments;
4. Use path-level reverse verification to check customer coverage, UAV count, payload, range or mission duration, and objective-value consistency.

---

## 2. Environment and Directory Layout

### 2.1 Recommended Environment

- Windows;
- Python 3.x;
- NumPy, pandas, matplotlib, and Numba;
- Gurobi 11.0 and `gurobipy`, required only for Alg-1 and exact-solution experiments;
- AI-KM platform, with general usage described in `AI-KM_README.md`. The AI-KM project is available at https://github.com/whl1207/Knowledge.
- DeepSeek-v4-flash or an equivalent LLM interface, used when rerunning AI-KM workflow interactions.

Install common Python dependencies with:

```powershell
pip install numpy pandas matplotlib numba
pip install gurobipy
```

If reproducing only non-Gurobi algorithms, real-world cases, and dynamic re-dispatching experiments, Gurobi is not required.

### 2.2 Key Directories

```text
date/
  Prior knowledge, benchmark instances, BKS references, and prompt resources.

Results/
  4.1/
    Modeling_Accuracy_Evaluation_Results/
      4.1.1/
      4.1.2/
    Algorithm_Solution_Accuracy_Evaluation_Results/
      4.1.3/
      4.1.4/
  4.2.1/
  4.2.2/
```

---

## 3. Preparing the AI-KM Knowledge Base

Create a UAV emergency logistics optimization knowledge base in AI-KM. Recommended public content includes:

| Knowledge type | Description | AI-KM storage mode |
|:---|:---|:---|
| Problem templates | MDVRP model structure, objective function, constraint types, and model-element descriptions | Knowledge chunks, ontology entities, table views |
| Algorithm templates | Gurobi exact solution, constructive heuristic, standard ALNS, advanced/intensified ALNS, enhanced ALNS, genetic algorithm, simulated annealing, and large neighborhood search templates | Workflow skill packages, `.flow` files, skill-mode associated files |
| Benchmark instances | Cordeau MDVRP instances `p01-p23` and `pr01-pr10` | File nodes and table views |
| BKS references | Known optimal or best-known solutions for Gap statistics | Metadata tables and read-only reference files |
| Verification rules | Model-element verification and path-level reverse verification rules | Python nodes, decision nodes, and feedback nodes |

Recommended ontology extraction covers:

- Problem entities: MDVRP, TSP, VRP;
- Constraint entities: payload constraints, range or mission-duration constraints, vehicle-count constraints, depot departure and return constraints, subtour-elimination constraints;
- Algorithm entities: Gurobi, constructive heuristic, ALNS, advanced ALNS, enhanced ALNS, genetic algorithm, simulated annealing;
- Parameter entities: `A_b`, `Q_b`, `T_b`, `q_i`, `s_i`, `d_{ij}`.

---

## 4. Reproducing Experiments in AI-KM

### 4.1 Standard Benchmark Instance Experiments

This part corresponds to Section 4.1 of the paper and uses Cordeau MDVRP benchmark instances to verify model construction, algorithm generation, and result verification capabilities.

#### 4.1.1 Create the Structured Semantic Input Template

In AI-KM Knowledge Management, use a table view to create an MDVRP model-element template.

| Model element | User input in natural language | AI-KM mapping result |
|:---|:---|:---|
| Sets and indices | “There are several UAV bases and several affected demand points.” | `\mathcal{B}={n+1,...,n+t}`, `\mathcal{C}={1,...,n}`, `\mathcal{V}=\mathcal{C}\cup\mathcal{B}` |
| Parameters | “Each base has available UAV count, payload limit, maximum mission duration/range, and demand information.” | `A_b`, `Q_b`, `T_b`, `q_i`, `s_i`, `d_{ij}` |
| Decision variables | “Decide whether UAV k from base b flies from node i to node j.” | `x_{ij}^{bk}\in{0,1}`, auxiliary variable `u_i^{bk}` |
| Objective | “Minimize total flight distance or total travel cost of the UAV fleet.” | `min Z=\sum_{b\in\mathcal{B}}\sum_{k\in\mathcal{K}_b}\sum d_{ij}x_{ij}^{bk}` |
| Constraints | “Each demand point is served once, UAVs depart from and return to their assigned bases, and resource limits are satisfied.” | Coverage, flow conservation, departure and return, payload, mission duration, MTZ, and variable-domain constraints in the paper model |

#### 4.1.2 Configure the Knowledge-Guided Reasoning Workflow

Build a four-stage workflow in the workflow editor:

```text
User requirement input
  -> Retrieve MDVRP templates and algorithm-selection rules from the SDKB
  -> Problem classification and model-structure recognition
  -> Use DTDR to complete instance data, distance matrices, and parameters
  -> Algorithm configuration and code generation
  -> Execute solution programs
  -> Model-element verification or path-level reverse verification
  -> Output dispatching plans and verification reports
```

The workflow should retain execution logs, error messages, repair records, and verification results as PMFR feedback memory.

#### 4.1.3 Model-Element Verification

This corresponds to Sections 4.1.1 and 4.1.2 of the paper. Recommended modeling workflows include:

- Without structured model prior knowledge: directly ask the LLM to generate an MDVRP model;
- With structured model prior knowledge: use the SSSME table template to constrain the input, then generate the model through KGRW.

Recommended checks in the model-element verification node:

- Whether the objective function is consistent with `d_{ij}` and `x_{ij}^{bk}`;
- Whether sets, indices, and parameters cover `\mathcal{B}`, `\mathcal{C}`, `\mathcal{K}_b`, `A_b`, `Q_b`, `T_b`, `q_i`, and `s_i`;
- Whether the depot-departure constraint is modeled as “at most once”;
- Whether payload, mission-duration, flow-conservation, and subtour-elimination constraints are included;
- Whether variable domains are consistent with the paper model.

Public results are located at:

```text
Results/4.1/Modeling_Accuracy_Evaluation_Results/4.1.1/
Results/4.1/Modeling_Accuracy_Evaluation_Results/4.1.2/
```

#### 4.1.4 Algorithm Generation, Feedback Repair, and Five-Algorithm Expansion

This corresponds to Sections 4.1.3 and 4.1.4 of the paper. Section 4.1.3 uses a Gurobi mathematical-programming workflow as a representative chain to test the effect of one feedback-repair round on code compilation, execution, valid route output, and constraint violations.

Section 4.1.4 further uses AI-KM skill mode and multi-stage interaction to generate five types of solution programs under the same MDVRP definition, instance data, output format, and verification protocol. The five programs are not a manually isolated set of fixed algorithms. They are progressively generated and corrected by AI-KM during the algorithm-configuration and code-generation stage around solution strategies, search structures, execution feedback, and result verification.

| Algorithm | Program type | AI-KM formation path | Main verification role |
|:---|:---|:---|:---|
| Alg-1 | Gurobi exact optimization program | Generate mathematical-programming code based on the unified MDVRP model and invoke an optimization solver | Verify executability from model formulation to exact solver invocation |
| Alg-2 | Fast construction and local improvement program | Generate lightweight route-construction logic under the unified constraint boundary | Verify the ability to rapidly generate feasible route structures |
| Alg-3 | Standard ALNS program | Generate destroy, repair, and acceptance mechanisms based on domain prior knowledge | Verify generation capability for problem-oriented metaheuristic structures |
| Alg-4 | Intensified search program | Extend search depth and operator combinations based on standard ALNS | Verify the role of multi-stage interaction in expanding search-strategy complexity |
| Alg-5 | Computationally enhanced search program | Introduce computational-efficiency enhancement on top of a complex search structure | Verify execution stability and efficiency of complex search strategies in batch instances |

Recommended AI-KM operations:

1. Enter the MDVRP solution task and output-format requirements in skill mode;
2. Retrieve model templates, algorithm templates, and BKS references from the SDKB;
3. Generate Alg-1 to Alg-5 by solution-strategy type;
4. Feed compilation errors, execution logs, and path-verification failures into subsequent correction nodes;
5. Export `.flow` workflows, generated code, and verification results for local batch reproduction.

Path-level reverse verification does not directly trust objective values reported by generated programs. Instead, it converts program outputs into a unified route format and checks:

- Whether each demand point is served exactly once;
- Whether duplicate service or missing service exists;
- Whether UAV usage at each base does not exceed `A_b`;
- Whether each route payload does not exceed `Q_b`;
- Whether each route travel distance plus service time does not exceed `T_b`;
- Whether the recalculated objective value is consistent with the output record.

---

## 5. Reproducing Experiments with Local Programs

This section is for reproducing public experimental data and results without rerunning AI-KM interactions. Run all commands from the project root `AI_UAV/`, or enter the directory specified by each `cd` command.

### 5.1 Reproduce the Section 4.1.3 Feedback-Repair Experiment

Main files:

```text
Results/4.1/Algorithm_Solution_Accuracy_Evaluation_Results/4.1.3/code_used/
  FK1_Workflow_2026-06-02-14-24-36.flow
  mdvrp_structured_workflow_experiment.py
```

Example command:

```powershell
set DEEPSEEK_API_KEY=<your_api_key>

python Results\4.1\Algorithm_Solution_Accuracy_Evaluation_Results\4.1.3\code_used\mdvrp_structured_workflow_experiment.py ^
  --flow-file Results\4.1\Algorithm_Solution_Accuracy_Evaluation_Results\4.1.3\code_used\FK1_Workflow_2026-06-02-14-24-36.flow ^
  --instances-dir date\6-mdvrp ^
  --bks-file date\mdvrp-sol-BKS.md ^
  --rounds 20 ^
  --algorithms exact ^
  --modes structured_no_feedback,structured ^
  --solver-time-limit 180 ^
  --execution-timeout 300
```

Public result summaries are located at:

```text
Results/4.1/Algorithm_Solution_Accuracy_Evaluation_Results/4.1.3/results_used/workflow_experiment_4_3_exact_fk1_main20_p_instances/
```

### 5.2 Reproduce the Section 4.1.4 Five-Algorithm Certified Experiment

Main files:

```text
Results/4.1/Algorithm_Solution_Accuracy_Evaluation_Results/4.1.4/
  benchmark_data/
  code_used/
  source_results_used/
  final_merged_results/
```

Example command:

```powershell
cd Results\4.1\Algorithm_Solution_Accuracy_Evaluation_Results\4.1.4\code_used

python run_5_algorithm_certified_experiment.py ^
  --output-root ..\rerun_certified_5_algorithm_experiment ^
  --instances all ^
  --algorithms all ^
  --time-limit 120 ^
  --turbo-max-iters 10000
```

If Gurobi is not installed, reproduce only the non-Gurobi algorithms. The final paper tables mainly come from:

```text
final_merged_results/combined_algorithm_statistics.csv
final_merged_results/certified_scope_statistics.csv
final_merged_results/p_series_turbo_alns_detail.csv
final_merged_results/best_certified_by_instance.csv
final_merged_results/combined_certified_solution_results.csv
```

### 5.3 Reproduce the Section 4.2.1 Shenzhen Real-World Case

Section 4.2.1 uses a Shenzhen flood emergency logistics case and constructs four instance types: single-district, cross-district, multi-district, and city-wide. Alg-5 is selected as the case solution configuration in the paper, and all route outputs use the same path-level reverse verification protocol as the benchmark experiments.

Main files:

```text
Results/4.2.1/
  A5_Turbo_ALNS.py
  case_input_data/
  Turbo_ALNS_Results/
  fig7_resource_utilization/
  case_analysis_results_table5_fig7.xlsx
  Fig_8_route_characteristics.png
```

Example command:

```powershell
cd Results\4.2.1
python A5_Turbo_ALNS.py
python fig7_resource_utilization\plot_utilization_boxplots.py
```

Input data include affected demand points, supply points, and case parameters.

### 5.4 Reproduce the Section 4.2.2 Dynamic New-Demand Re-Dispatching Experiment

Section 4.2.2 simulates new demand appearing during mission execution. The system maps departed UAVs into dynamic supply units with current position, residual payload, and residual range, keeps non-departed UAVs as standby resources, and reconstructs unfinished original demand plus new demand into an updated optimization problem.

Main files:

```text
Results/4.2.2/
  A5_Disruption_Analysis.py
  Disruption_Results/
  Fig_9a_redisp_case_C2.png
  Fig_9b_redisp_case_C3.png
  GIS/
    plot_disruption_gis_routes.py
    shenzhen_districts.geojson
    shenzhen_population_density.tif
    outputs/
      Fig. 9a.png
      Fig. 9b.png
      disruption_gis_routes_interactive.html
```

Example command:

```powershell
cd Results\4.2.2
python A5_Disruption_Analysis.py
```

To generate GIS-version figures, run:

```powershell
cd Results\4.2.2\GIS
python plot_disruption_gis_routes.py
```

This guide does not hard-code specific coordinates or demand quantities for dynamic disturbances. In reproduction, use the program or case data to generate new demand points within the longitude-latitude range, demand range, and `T_b` reachability conditions of the original case.

---

## 6. Recommended Result Records

After reproduction, record the following files or indicators:

| Experiment | Recommended records |
|:---|:---|
| 4.1.1/4.1.2 | Model-element accuracy, error type, and verification-corrected results |
| 4.1.3 | Compilation rate, run rate, valid route-output rate, constraint-violation rate, Gap |
| 4.1.4 | Five-algorithm verification pass rate, average Gap, median Gap, runtime, instance-wise best results |
| 4.2.1 | Cost, CPU time, UAVs used, payload utilization, and route figures for the four Shenzhen cases |
| 4.2.2 | Re-dispatching routes after new demand, residual-resource mapping, local insertion, or route reconstruction results |

For paper tables and figures, prioritize final files in `final_merged_results/`, `Turbo_ALNS_Results/`, `Disruption_Results/`, and `GIS/outputs/` rather than intermediate run directories.

---

## 7. FAQ

### Q1: Do I need to rerun AI-KM if I only want to reproduce final results?

No. AI-KM is used to reproduce the structured interaction, workflow generation, and multi-stage algorithm generation process. Final results can be reproduced directly through the Python programs and public data under `Results/`.

### Q2: Can I reproduce all experiments without Gurobi?

No. Alg-1 and Gurobi exact-solution results require Gurobi. You can still reproduce non-Gurobi algorithms, the Shenzhen case, and the dynamic re-dispatching experiment. The five-algorithm experiment can also be run with only non-Gurobi algorithm selections.

### Q3: How do I determine whether an algorithm output can be used for Gap statistics?

Only results that pass path-level reverse verification are used for Gap statistics. The verifier checks route feasibility and recalculates the objective value, rather than directly using the cost reported by the generated program.

---

## 8. Summary

This guide provides two reproduction paths. The first reconstructs structured semantic input, knowledge-guided reasoning, multi-stage algorithm generation, and feedback verification in AI-KM. The second directly reproduces experimental results with the public local programs in this repository. The five algorithms in Section 4.1.4 should be understood as solution programs generated, corrected, and uniformly verified through AI-KM multi-stage interaction. The real-world case and dynamic disturbance experiments in Section 4.2 use Alg-5 and local programs to demonstrate how K-HLIDF maps real spatial data into verifiable dispatching solutions.
