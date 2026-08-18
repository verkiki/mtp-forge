# Contributing

Useful contributions include reproducible architecture mappings, compatibility checks, clearer failure messages, and tests using synthetic or redistributable fixtures.

Run before submitting:

```bash
python -m unittest discover -s tests -v
python -m compileall -q src
```

Include the exact llama.cpp revision, model architecture, source metadata, command, and error. Never attach restricted model weights, tokens, credentials, or private paths.

Architecture-specific extraction logic belongs upstream in llama.cpp. MTP Forge should orchestrate and verify it, not carry a stale fork of every mapping.
