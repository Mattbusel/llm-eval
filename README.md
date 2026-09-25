# llm-eval

[![CI](https://github.com/Mattbusel/llm-eval/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-eval/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
![Single header](https://img.shields.io/badge/single-header-green.svg)

Measure how consistent an LLM is from C++: run a prompt N times, score the variance, compare models or system prompts.

> Part of **[llm-cpp](https://github.com/Mattbusel/llm-cpp)**, a family of 26 single-header C++ libraries for building on LLM APIs. Each one stands alone: copy one header, include it, done.

An LLM that gives a different answer every time is hard to build on, and one good sample tells you little. llm-eval runs the same prompt repeatedly and turns the results into numbers: how similar the answers are, how much their length varies, latency, and success rate, with flags when a prompt or model looks unreliable.

## Features

- `evaluate()`: run a prompt `runs` times (default 10) and compute consistency metrics
- Metrics: pairwise Jaccard word similarity (avg/min/max), length mean/stddev/coefficient of variation, average and p95 latency, success rate
- Automatic flags: `high_variance` (CV > 0.3), `inconsistent` (similarity < 0.5), `unreliable` (success < 90%)
- `compare()`: two model configs on the same prompt, with a winner
- `compare_prompts()`: two system prompts on the same model
- `score_response()`: keyword coverage score with found/missing lists
- Human-readable summary string for every result

## Quick start

Requirements: a C++17 compiler and libcurl (`apt install libcurl4-openssl-dev`, `brew install curl`, or `vcpkg install curl`).

1. Copy [`include/llm_eval.hpp`](include/llm_eval.hpp) into your project.
2. In exactly one `.cpp` file, `#define LLM_EVAL_IMPLEMENTATION` before including it. Other files just `#include "llm_eval.hpp"`.

```cpp
#define LLM_EVAL_IMPLEMENTATION
#include "llm_eval.hpp"
#include <cstdlib>
#include <iostream>

int main() {
    llm::EvalConfig cfg;
    cfg.api_key = std::getenv("OPENAI_API_KEY");
    cfg.model   = "gpt-4o-mini";
    cfg.runs    = 5;

    llm::EvalResult r = llm::evaluate("List three uses of a hash map.", cfg);
    std::cout << r.summary << "\n";
    std::cout << "avg similarity: " << r.metrics.avg_similarity
              << ", inconsistent: " << std::boolalpha << r.metrics.inconsistent << "\n";

    llm::ScoreResult s = llm::score_response(r.runs[0].response, {"cache", "lookup", "count"});
    std::cout << "keyword score: " << s.score << "\n";
}
```

Build and run:

```bash
g++ -std=c++17 -I include example.cpp -o example -lcurl
export OPENAI_API_KEY=sk-...
./example
```

## API

Everything lives in namespace `llm`.

| Function / type | What it does |
|---|---|
| `evaluate(prompt, cfg)` | Return an `EvalResult` with every run and its `EvalMetrics` |
| `compare(prompt, cfg_a, cfg_b)` | Evaluate both configs and pick `"a"`, `"b"` or `"tie"` |
| `compare_prompts(user_prompt, system_a, system_b, cfg)` | Same comparison for two system prompts |
| `score_response(response, keywords)` | Fraction of expected keywords present, plus found/missing lists |
| `EvalConfig` | API key, model (default `gpt-4o-mini`), runs, temperature, system prompt, verbose |

## How it works

Each run is a separate chat completions request through libcurl, timed individually. Similarity is Jaccard overlap between the word sets of every pair of responses. `compare()` scores each side as average similarity minus length CV plus success rate and picks the higher one.

## Examples

The [`examples/`](examples) folder has runnable programs:

- [`basic_eval.cpp`](examples/basic_eval.cpp)
- [`compare_models.cpp`](examples/compare_models.cpp)

Build the examples with CMake (needs libcurl):

```bash
cmake -B build
cmake --build build
```

Examples that call the API read `OPENAI_API_KEY` from the environment.

## Limitations

- Runs are sequential; 10 runs means 10 round trips.
- Jaccard word overlap measures surface consistency, not semantic equivalence.
- The public header includes `<curl/curl.h>`, so curl headers are needed wherever it is included.

## License

MIT. See [LICENSE](LICENSE).
