# Changelog

## v2.0.0 — MTP Forge split release

- Split MTP workflows into an independent repository.
- Added one-command standalone MTP/NextN extraction through current llama.cpp `--mtp` mappings.
- Added automatic Q/K/IQ quantization through a temporary F16 MTP GGUF.
- Added dry-run, check-only, work preservation, GGUF reopening, and MTP evidence verification.
- Normalized llama.cpp's automatic `mtp-` output prefix and published only verified staged files.
- Generalized embedded MTP injection beyond hard-coded Qwen architecture names while retaining strict metadata checks.
- Added tokenizer compatibility checks and endianness validation.
- Preserved raw target and donor tensor types and retained default SHA-256 verification.
- Added packaging, CI, tests, security documentation, and GitHub publishing steps.

## v1.0.0 — QwenQuant Toolkit

- Initial Qwen embedded NextN/MTP GGUF injector.
