# Benchmark Generation Pipeline

Generate EDA tool call benchmark datasets with structured ground truth and validated natural language prompts.

## Pipeline Demo

```bash
# Navigate to the benchmark generation directory:
cd src/benchmark_generation

# Run complete pipeline
python demo_benchmark_generation.py --api_key your_openai_api_key

# Skip prompt generation (no API key needed)
python demo_benchmark_generation.py
```

## Files

In this directory:  

- `demo_benchmark_generation.py` - Demonstrate the full benchmark generation workflow using all scripts
- `gen_answer_tcl.py` - Generate structured EDA tool call ground truth
- `gen_prompt_tcl.py` - Convert to natural language prompts  
- `validate_benchmark.py` - Validate answer-prompt consistency

In the `data` directory at the project root (all generated files are located there):  

- `schema.json` - Parameter definitions
- `tcl_answers_example.jsonl` - Examples of generated ground truth
- `tcl_prompts_example.jsonl` - Examples of generated natural language prompts

## Usage Examples

```bash
# Generate mixed tool sequences
python gen_answer_tcl.py --output tcl_answers.jsonl --num 10 --mixed

# Generate prompts (requires OPENAI API key)
export API_KEY="your-key"
python gen_prompt_tcl.py --input tcl_answers.jsonl --output tcl_prompts.jsonl

# Validate results
python validate_benchmark.py --answers tcl_answers.jsonl --prompts tcl_prompts.jsonl --result validation_results.json
```
