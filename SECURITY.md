# Security Policy

Report vulnerabilities privately to the repository maintainer before publishing details.

MTP Forge reads large memory-mapped GGUF files and writes new model artifacts. It refuses in-place changes and overwrites, validates exact paths, executes subprocesses without a shell, checks tokenizer and architecture compatibility, and reopens outputs before success.

Do not include model weights, access tokens, credentials, or private paths in reports. Prefer a minimal synthetic GGUF reproduction.
