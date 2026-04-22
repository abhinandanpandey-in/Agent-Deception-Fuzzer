# Agent-Deception-Fuzzer

An automated fuzzer for testing cross-boundary prompt injections and multi-agent LLM deception.

## What it does

Tests whether malicious payloads can propagate between chained LLM agents in a multi-agent pipeline. Models a realistic agentic system with two agents and a structural guardrail between them.

## Pipeline Architecture

```
[User Input] → [Agent A: HR Parser] → [Guardrail: Schema Validator] → [Agent B: DB Executor]
```

- **Agent A** parses resume text and outputs structured JSON
- **Guardrail** strips any unauthorized keys before passing downstream
- **Agent B** processes the validated JSON and executes actions

## Attack Vectors Tested

| Vector | Description |
|--------|-------------|
| Benign Baseline | Clean input, normal flow |
| Few-Shot Poisoning | Injects malicious examples into resume text to manipulate Agent A's output |
| Base64 Encoding Evasion | Hides malicious JSON in base64 hoping Agent A decodes and outputs it |
| Diagnostic Roleplay | Tricks Agent A into thinking it's an "Emergency Root Terminal" |

## Key Finding

The LLM boundary (Agent A) was successfully breached by Few-Shot Poisoning and Diagnostic Roleplay — Agent A produced unauthorized `admin_action` keys. However, the Python-level guardrail stripped them before reaching Agent B.

> **Conclusion: You cannot rely on LLMs to enforce inter-agent security. Structural, code-level validation at every boundary is essential.**

## Requirements

- Python 3.x
- [Ollama](https://ollama.com) running locally
- Mistral model: `ollama pull mistral`

## Setup

```bash
git clone https://github.com/abhinandanpandey-in/Agent-Deception-Fuzzer
cd Agent-Deception-Fuzzer
pip install -r requirements.txt
```

## Usage

```bash
# Make sure Ollama is running first
ollama serve

# Run the fuzzer
python agent_deception_fuzzer.py --model mistral
```

## Output

Results are saved to `fuzzing_telemetry_{model}.json` with full per-vector telemetry including raw agent output, validated output, and whether the boundary was compromised.

## License

MIT
