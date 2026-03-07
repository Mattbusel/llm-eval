# CLAUDE.md -- llm-eval

## Build
```bash
cmake -B build && cmake --build build
```

## THE ONE RULE: SINGLE HEADER
`include/llm_eval.hpp` is the entire library. Never split it.

## Common Notes
- Jaccard similarity on word sets (tokenized by whitespace)
- high_variance = CV > 0.3, inconsistent = avg_similarity < 0.5
- compare() winner uses: avg_similarity - length_cv + success_rate
- score_response: case-insensitive substring match for each keyword
