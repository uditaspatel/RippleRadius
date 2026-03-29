# Model Migrator

## What This Is
A pipeline-aware framework for recovering multi-stage 
LLM pipelines after a component model is migrated to 
a new version. This is NOT a prompt optimizer, NOT a 
monitoring tool. It is a migration-time recovery system 
triggered by a deliberate model swap event.

## Core Concepts

**Semantic cascade degradation**: when stage i is 
migrated to a new model, downstream stages i+1, i+2 
degrade silently because their prompts were implicitly 
calibrated to the old model's output distribution.
Visible only at pipeline level, not per-stage evaluation.

**Blast radius**: the set of downstream stages that 
experience secondary degradation from a migration at 
stage i. Estimated from embedding-space distributional 
shift at stage i.

**Delta type**: each observed behavioral change between 
old and new model is classified as:
  - Type A (regression): new model worse — fix via rewriting
  - Type B (neutral): different but equivalent — manage 
    interface compatibility only
  - Type C (improvement): new model better — propagate 
    downstream, do not normalize away

**Non-migratable flag**: raised when a stage cannot 
be recovered through prompt rewriting — architecture 
crossing or hard capability gap. Requires hybrid routing.

**Task success rate (TSR)**: primary quality metric. 
Measured at each stage and end-to-end. Grounded in 
task ground truth, NOT old model outputs.

## Module Inventory

core/pipeline.py        — Stage and PipelineGraph 
                          data structures
core/characterize.py    — ModelCharacterizationModule: 
                          profiles source and target 
                          model before any pipeline analysis
core/delta.py           — BehavioralDeltaCharacterizer: 
                          types behavioral shifts between 
                          old and new model outputs
core/drift.py           — SemanticDriftTracker: 
                          embedding-based distributional 
                          shift measurement per stage
core/cascade.py         — CascadeRiskEstimator: blast 
                          radius identification and 
                          counterfactual attribution
core/rewrite.py         — PromptRewriter: three-step 
                          rewriting (best practices → 
                          delta correction → cascade-aware)
core/gradient.py        — TextualGradientEngine: 
                          pipeline-level error 
                          backpropagation across stages
core/orchestrator.py    — MigrationOrchestrator: 
                          top-level workflow controller

pipelines/book_sum.py   — 5-stage BookSum pipeline
pipelines/sop_bench.py  — 6-stage SOP-Bench pipeline

eval/metrics.py         — TSR measurement at stage 
                          and pipeline level
eval/stats.py           — promptstats integration for 
                          statistical significance

experiments/run_migration.py — experiment runner

## Key Constraints
- Every pipeline stage must be an LLM prompt invocation 
  via Anthropic API. No retrieval stages, no rule-based 
  stages, no deterministic transforms.
- Source and target models: Anthropic models only 
  (claude-sonnet-3-7, claude-haiku-4-5, claude-sonnet-4)
- Each Stage has: prompt, model_id, input_schema, 
  output_schema, stage_index, stage_name
- DeltaReport per stage contains: delta_type (A/B/C), 
  severity, specific_losses, format_shift, 
  confidence_score
- MigrationReport contains: per-stage TSR before/after, 
  blast_radius, non_migratable_flags, 
  confidence_scores, recommendation

## Data Contracts

Stage:
  name: str
  index: int
  prompt: str
  model_id: str
  input_schema: dict
  output_schema: dict

PipelineGraph:
  stages: List[Stage]
  get_downstream(stage_index) -> List[Stage]
  get_blast_radius(stage_index, drift_scores) -> List[int]

DeltaReport:
  stage_index: int
  delta_type: Literal["A", "B", "C"]
  severity: Literal["LOW", "MEDIUM", "HIGH"]
  embedding_distance: float
  content_losses: List[str]
  format_shift: str
  confidence: float

MigrationReport:
  source_model: str
  target_model: str
  per_stage_tsr_before: Dict[int, float]
  per_stage_tsr_after: Dict[int, float]
  blast_radius: List[int]
  delta_reports: Dict[int, DeltaReport]
  non_migratable_stages: List[int]
  recommendation: str

## What Is Built and Working
[ nothing yet — update this as modules complete ]