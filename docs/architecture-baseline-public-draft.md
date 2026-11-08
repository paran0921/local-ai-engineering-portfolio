# Local AI Architecture Baseline

- Decision date: 2026-10-11
- Status: Adopted baseline; subject to evidence-based revision
- Scope: Personal local AI coding and documentation environment

## 1. Goals

- Prioritize a practical local AI development workflow.
- Avoid dependence on paid AI services.
- Preserve reproducibility by recording actual commands and outputs.
- Add infrastructure only when a demonstrated need justifies its cost and complexity.

## 2. Current Architecture

### Inference server

- Device: Windows laptop used as the local inference server
- Operating system: Windows 11 Home (OS version 10.0.26300, display version 26H2, build 26300, 64-bit)
- OS verification date: 2026-10-27; reflects the operating system information verified on that date.
- Inference runtime: Ollama
- Network endpoint recorded in the project history: local network endpoint on port 11434
- Recorded model: `qwen3:8b`
- Recorded GPU: GPU with 12 GB VRAM

These are recorded environment facts from prior checks, not a claim that every value has been rechecked on this date.

### Remote inference verification

- The ThinkPad development client successfully connected to the OMEN inference server over TCP port 11434.
- The remote `/api/version` and `/api/tags` endpoints returned valid responses.
- A direct chat request to the remote `qwen3:8b` model returned the expected response `LOCAL_AI_INFERENCE_OK`.
- The response reported `done=true`.
- The measured total processing time was 2,871 ms in this single test.

This verification confirms remote API access and model inference for the tested request. The processing time is a single measurement, not a performance benchmark.

### Verification scope and limitations

- The complete Aider workflow, from code modification through testing, was not verified.
- GPU utilization, sustained-load performance, and repeated inference latency were not measured.
- Compatibility across VS Code extensions was not comprehensively verified.

### Development client

- Device: Windows laptop used as the development client
- Operating system: Windows 11 Pro (OS version 10.0.26300, display version 26H2, build 26300, 64-bit)
- OS verification date: 2026-10-27; reflects the operating system information verified on that date.
- Main development environment: VS Code
- Main application project: local Laravel project directory
- Laravel target: Laravel 12

### Coding-agent workflow

- Primary working approach: Aider with the local Ollama model
- CECLI: 2026-10-15에 Aider 대체 후보로 추가했다. CECLI 1.8.0에서 로컬 Ollama provider 연결을 확인했으나, 실제 코드 수정부터 테스트까지의 전체 에이전트 워크플로 검증과 채택 결정은 완료하지 않았다.
- OpenCode: 2026-10-11 아키텍처 결정에 보조 후보로 기록되었다. OpenCode v2 평가는 진단 범위와 진행 승인이 정해질 때까지 보류한다. CECLI와 OpenCode는 검토 대상 후보이며, 두 도구의 실제 평가 완료 또는 아키텍처 전환을 의미하지 않는다.
- Cline and Kilo Code: comparison and reference tools, not assumed to be the primary workflow
- Jan.ai: planned for a separate practical evaluation; not yet adopted as a replacement for Ollama

The agent choices are workflow decisions, not a claim that one tool is universally superior.

## 3. Operating Principles

1. Prefer native Windows execution first; use WSL2 only when a concrete requirement warrants it.
2. Prioritize the simplest working local workflow.
3. Introduce RAG, MCP, or additional agents only when testing shows a practical benefit.
4. Do not expand broad tool comparisons at the expense of real application development.
5. Keep the Laravel application and its history separate from this documentation repository.
6. Do not delete backups or source records without explicit authorization.
7. Do not record an operation as successful until its actual output has been checked.
8. Separate verified facts, hypotheses, plans, and historical observations.

## 4. Laravel Development Verification Cycle

For scoped coding tasks:

1. Inspect the actual project state and target files.
2. Define the permitted change scope.
3. Run Aider with the explicit local model and `--no-auto-commits`.
4. Run the relevant tests.
5. Inspect `git diff` and `git status`.
6. Commit only after reviewing the results and obtaining explicit authorization.

Example command pattern:

`aider --model ollama_chat/qwen3:8b --no-auto-commits <verified-file-paths>`

Replace file paths only after confirming the actual target files.

## 5. RAG Policy

RAG is an optional capability, not a default requirement.

Prioritize measurable retrieval quality and practical usefulness. Retain the existing local RAG experiment results as historical evidence; rerun evaluation before claiming that a configuration remains optimal.

## 6. Security and Publication

- Keep credentials, tokens, `.env` files, and private agent histories out of public repositories.
- Review diffs and ignore rules before committing.
- Keep development records local until the publication scope and repository destination have been reviewed.
- Review every document for private paths, personal data, secrets, and unsupported claims before publication.

## 7. Review Triggers

Revisit this baseline when one of the following occurs:

- A material hardware or operating-system change
- A verified Ollama or model-serving limitation
- Repeated Aider failures in real development work
- Evidence that an alternative agent materially improves the workflow
- RAG evaluation shows a reproducible improvement
- A concrete requirement for WSL2, MCP, or additional agents

## 8. Current Limitations

- The 2026-10-11 baseline is a dated snapshot; it does not establish that every installed tool version was verified on that date.
- The dated AI-generated environment reviews are references, not authoritative verification.
