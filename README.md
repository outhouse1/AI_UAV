# K-HLIDSS for UAV Emergency Logistics Network Optimization

This repository contains the public experimental data, workflow configurations,
source code, and certified result tables used in the paper on a
knowledge-based human-LLM interaction decision support system (K-HLIDSS) for
UAV emergency logistics network optimization.

The study does not propose a new routing model or a single new metaheuristic.
Instead, it evaluates how a structured human-LLM workflow can transform
operational emergency-dispatch descriptions into formal MDVRP models,
executable solution programs, and independently verified route results.

For detailed reproduction instructions, see `User_Guide_EN.md`. For the AI-KM
platform overview, see `AI-KM_README.md` and the AI-KM project at
https://github.com/whl1207/Knowledge. The AI-KM platform was developed as a 
companion research output to support the study reported in this paper.

## Method Overview

K-HLIDF embeds an LLM into a controlled decision chain rather than using it as a
one-shot solver. The framework contains three connected mechanisms.

1. Structured Semantic Schema based on Model Elements (SSSME)

   User requirements are decomposed into standard optimization model elements:
   sets and indices, parameters, decision variables, objectives, and
   constraints. This schema reduces ambiguity before model generation and
   provides a verification baseline for later stages.

2. Knowledge-Guided Reasoning Workflow (KGRW)

   The workflow integrates a static domain knowledge base, a dynamic tool and
   data registry, and a procedural memory and feedback repository. The
   reasoning process covers problem classification and mathematical modeling,
   scenario instantiation and parameter retrieval, algorithm configuration and
   code generation, and result verification with adaptive correction.

3. Feedback Verification

   Model-element verification checks whether generated mathematical models
   correctly express objectives, parameters, decision variables, and
   constraints. Path-level reverse verification independently checks generated
   route results, including customer coverage, UAV count, payload capacity,
   route duration, and objective-value consistency.

## Repository Layout

```text
date/
  Prior knowledge, benchmark data, BKS references, and prompt resources.

Results/
  4.1/
    Modeling_Accuracy_Evaluation_Results/
      4.1.1/  Structured model prior knowledge experiment.
      4.1.2/  Model-element verification experiment.
    Algorithm_Solution_Accuracy_Evaluation_Results/
      4.1.3/  Structured workflow and feedback repair experiment.
      4.1.4/  Five-algorithm certified benchmark experiment.
  4.2.1/
    Real-world Shenzhen emergency logistics case.
  4.2.2/
    Dynamic new-demand re-dispatching experiment.
    GIS/      GIS-based visualization inputs and outputs.
```

## Experimental Sections

### Section 4.1.1: Structured Model Prior Knowledge

This experiment compares two AI-KM modeling workflows under the same MDVRP
model-evaluation protocol:

- without structured model prior knowledge;
- with structured model prior knowledge.

Main files:

```text
Results/4.1/Modeling_Accuracy_Evaluation_Results/4.1.1/
  Modeling_Workflow_without_Prior_Knowledge_2026-05-30-07-17-39.flow
  Modeling_Workflow_with_Prior_Knowledge_2026-05-30-07-17-39.flow
  without_Prior_Knowledge-batch_results_2026-05-30T07-39-11-692Z.xlsx
  with_Prior_Knowledge-batch_results_2026-05-30T08-32-08-544Z.xlsx
```

### Section 4.1.2: Model-Element Verification

This experiment adds model-element verification after structured model prior
knowledge and evaluates the incremental correction effect.

Main files:

```text
Results/4.1/Modeling_Accuracy_Evaluation_Results/4.1.2/
  FK1_Workflow_2026-05-30-16-28-21.flow
  FK1_batch_results_2026-05-31T07-04-29-209Z.xlsx
```

### Section 4.1.3: Structured Workflow and Feedback Repair

This experiment evaluates whether feedback repair improves algorithm-code
generation and route-output reliability. It uses a Gurobi-based mathematical
programming workflow as the representative solution chain.

Main files:

```text
Results/4.1/Algorithm_Solution_Accuracy_Evaluation_Results/4.1.3/
  code_used/
    FK1_Workflow_2026-06-02-14-24-36.flow
    mdvrp_structured_workflow_experiment.py
  results_used/workflow_experiment_4_3_exact_fk1_main20_p_instances/
    analysis_by_algorithm_mode_bks_corrected.csv
    analysis_feedback_comparison_bks_corrected.csv
    analysis_report_bks_corrected.md
    experiment_results_bks_corrected.csv
    summary_bks_corrected.txt
```

To rerun:

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

### Section 4.1.4: Five-Algorithm Certified Benchmark Experiment

This experiment evaluates five generated solution programs on 33 Cordeau MDVRP
benchmark instances:

- Alg-1: Gurobi direct model;
- Alg-2: constructive heuristic with simulated annealing;
- Alg-3: ALNS;
- Alg-4: Deep ALNS;
- Alg-5: Turbo ALNS / enhanced ALNS.

Main files:

```text
Results/4.1/Algorithm_Solution_Accuracy_Evaluation_Results/4.1.4/
  benchmark_data/
    C-mdvrp/
    C-mdvrp-sol/
  code_used/
    algorithm_assets/
    appendix_5_algorithm_workflow_integration.py
    multi_stage_algorithm_asset_builder.py
    run_5_algorithm_certified_experiment.py
  source_results_used/
    certified_5_algorithm_full33_gurobi_v3/
    certified_5_algorithm_full33_non_gurobi/
    certified_5_algorithm_full33_turbo_alns_swap_local_search_120s/
  final_merged_results/
    combined_algorithm_statistics.csv
    certified_scope_statistics.csv
    p_series_turbo_alns_detail.csv
    best_certified_by_instance.csv
    combined_certified_solution_results.csv
    per_instance_certification_matrix.csv
```

To rerun:

```powershell
cd Results\4.1\Algorithm_Solution_Accuracy_Evaluation_Results\4.1.4\code_used

python run_5_algorithm_certified_experiment.py ^
  --output-root ..\rerun_certified_5_algorithm_experiment ^
  --instances all ^
  --algorithms all ^
  --time-limit 120 ^
  --turbo-max-iters 10000
```

Gurobi 11.0 is required for the Gurobi direct model. The non-Gurobi algorithms
can be rerun without Gurobi by selecting the corresponding algorithm ids.

### Section 4.2.1: Real-World Shenzhen Case

This experiment applies Alg-5 to four Shenzhen emergency logistics instances:

- C1: single-district case;
- C2: cross-district case;
- C3: multi-district case;
- C4: city-wide case.

Main files:

```text
Results/4.2.1/
  A5_Turbo_ALNS.py
  case_input_data/
    demand_S.csv
    demand_MS.csv
    demand_M.csv
    demand_L.csv
    supplyTable_S.csv
    supplyTable_MS.csv
    supplyTable_M.csv
    supplyTable_L.csv
  Turbo_ALNS_Results/
    A5_*_routes.csv
    A5_*_summary.csv
    A5_*_turbo.png
    A5_Turbo_ALNS_Summary.csv
    A5_Turbo_ALNS_Summary.txt
  fig7_resource_utilization/
    plot_utilization_boxplots.py
    Fig_7_resource_utilization.png
  case_analysis_results_table5_fig7.xlsx
  Fig_8_route_characteristics.png
```

To rerun:

```powershell
cd Results\4.2.1
python A5_Turbo_ALNS.py
python fig7_resource_utilization\plot_utilization_boxplots.py
```

### Section 4.2.2: Dynamic New-Demand Re-Dispatching

This experiment simulates new demand appearing during execution and reconstructs
the dispatching problem from current UAV states, unfinished original demands,
and newly generated demand nodes. The updated figures use GIS-based route
visualizations.

Main files:

```text
Results/4.2.2/
  A5_Disruption_Analysis.py
  Disruption_Results/
    A5_MS_disruption_routes.csv
    A5_MS_disruption_summary.txt
    A5_MS_disruption.png
    A5_MS_disruption.svg
    A5_M_disruption_routes.csv
    A5_M_disruption_summary.txt
    A5_M_disruption.png
    A5_M_disruption.svg
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

To rerun the disruption analysis:

```powershell
cd Results\4.2.2
python A5_Disruption_Analysis.py
```

To regenerate the GIS figures:

```powershell
cd Results\4.2.2\GIS
python plot_disruption_gis_routes.py
```

## Environment

The experiments used:

- AI-KM workflow platform with DeepSeek-v4-flash;
- Python 3.x;
- Gurobi 11.0 for mathematical programming experiments;
- NumPy, pandas, matplotlib, and Numba for data processing, visualization, and
  accelerated heuristic components;
- AMD Ryzen AI 9 H 365 with 32 GB RAM.
