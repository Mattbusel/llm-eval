# AGENTS.md -- llm-eval

## Purpose
Single-header C++ LLM evaluation library. Run prompts N times, compute
consistency metrics (response length CV, pairwise Jaccard similarity,
p95 latency, success rate), compare models or system prompts, and score
responses against expected keywords.

## Architecture
```
llm-eval/
  include/llm_eval.hpp   <- THE ENTIRE LIBRARY. Do not split.
  examples/
    basic_eval.cpp
    compare_models.cpp
  CMakeLists.txt
```

## Build
```bash
cmake -B build && cmake --build build
```

## Rules
- Single header only.
- Only libcurl as external dep.
- All public API in namespace llm.
- Implementation inside #ifdef LLM_EVAL_IMPLEMENTATION guard.

## API Surface
- evaluate(prompt, config) -> EvalResult with EvalMetrics
- compare(prompt, config_a, config_b) -> ComparisonResult
- compare_prompts(user_prompt, system_a, system_b, base) -> ComparisonResult
- score_response(response, expected_keywords) -> ScoreResult
