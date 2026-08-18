<div align="center">

# MTP Forge

### Extract. Quantize. Verify. Embed. One clear CLI for MTP/NextN GGUF workflows.

**Architecture-aware extraction through current llama.cpp, plus byte-preserving embedded MTP injection.**

[Quick start](#quick-start) · [Extract](#extract-a-standalone-mtp-gguf) · [Embed](#embed-mtp-without-requantizing-the-backbone) · [Publish](PUBLISHING.md)

</div>

---

MTP Forge is the MTP half of the former QwenQuant Toolkit, rebuilt as an independent repository.

It provides two deliberately different operations:

| Command | Input | Output | How it stays safe |
| --- | --- | --- | --- |
| `mtp-forge extract` | Original Hugging Face checkpoint with MTP/NextN weights | Standalone speculative-draft GGUF | Uses the current llama.cpp `--mtp` architecture mapping |
| `mtp-forge embed` | MTP-free target GGUF + compatible GGUF with embedded extra MTP blocks | Target backbone with embedded donor MTP | Copies raw tensor types/data and verifies hashes |

The tool does not guess how an unknown architecture stores MTP. Extraction support automatically follows the `convert_hf_to_gguf.py` version you supply.

## Quick start

### 1. Clone and install MTP Forge

Linux/macOS:

```bash
git clone https://github.com/YOUR_USERNAME/mtp-forge.git
cd mtp-forge
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e .
```

Windows PowerShell:

```powershell
git clone https://github.com/YOUR_USERNAME/mtp-forge.git
cd mtp-forge
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e .
```

### 2. Install a current llama.cpp checkout

```bash
git clone https://github.com/ggml-org/llama.cpp.git
python -m pip install -r llama.cpp/requirements/requirements-convert_hf_to_gguf.txt
cmake -S llama.cpp -B llama.cpp/build -DCMAKE_BUILD_TYPE=Release
cmake --build llama.cpp/build --config Release -j
```

MTP Forge uses:

- `convert_hf_to_gguf.py --mtp` for architecture-aware extraction;
- `llama-quantize` for Q/K/IQ output;
- `gguf-py` for structural verification and embedding.

Pass `--llama-cpp /path/to/llama.cpp`, or set `LLAMA_CPP` once.

### 3. Verify discovery

```bash
mtp-forge doctor --llama-cpp /path/to/llama.cpp
```

## Extract a standalone MTP GGUF

Safest source-precision output:

```bash
mtp-forge extract /models/model-hf ./model-MTP.gguf --llama-cpp /opt/llama.cpp
```

From a Hub repository:

```bash
mtp-forge extract OWNER/MODEL ./model-MTP.gguf --llama-cpp /opt/llama.cpp
```

Choose an output type:

```bash
mtp-forge extract /models/model-hf ./model-MTP-F16.gguf    --to f16 --llama-cpp /opt/llama.cpp
mtp-forge extract /models/model-hf ./model-MTP-Q8_0.gguf   --to q8  --llama-cpp /opt/llama.cpp
mtp-forge extract /models/model-hf ./model-MTP-Q4_K_M.gguf --to q4  --llama-cpp /opt/llama.cpp
mtp-forge extract /models/model-hf ./model-MTP-IQ4_XS.gguf --to iq4 --llama-cpp /opt/llama.cpp
```

`auto`, `f16`, `bf16`, and `q8` can be written directly by the current HF converter. Q4/Q6/IQ targets automatically use a temporary F16 MTP GGUF and `llama-quantize`.

Current llama.cpp versions may prefix `mtp-` to the converter's requested filename. MTP Forge detects both naming behaviors, verifies the staged GGUF, and publishes it under the exact output name you requested.

Run a real architecture preflight without writing weights:

```bash
mtp-forge extract /models/model-hf ./model-MTP.gguf --check-only --llama-cpp /opt/llama.cpp
```

Print commands only:

```bash
mtp-forge extract /models/model-hf ./model-MTP-Q4.gguf --to q4 --dry-run --llama-cpp /opt/llama.cpp
```

After writing, MTP Forge reopens the result with `gguf-py`, verifies GGUF magic, requires at least one tensor, and requires MTP/NextN evidence in the architecture, metadata, or tensor names.

Extraction and embedding are verified under a staging name first. The final filename appears only after validation succeeds, and an existing output is never replaced.

## Use the standalone MTP draft

With a current llama.cpp server:

```bash
llama-server \
  -m /models/base.gguf \
  -md /models/model-MTP.gguf \
  --spec-type draft-mtp \
  --spec-draft-n-max 2
```

Tune `--spec-draft-n-max` for the model and benchmark acceptance and throughput. Speculation is not guaranteed to be faster on every backend or context size.

## Embed MTP without requantizing the backbone

Use this only when the donor is the same architecture and contains embedded extra MTP blocks:

```bash
mtp-forge embed \
  /models/target-without-mtp.gguf \
  /models/compatible-donor-with-embedded-mtp.gguf \
  /models/final-with-mtp.gguf \
  --llama-cpp /opt/llama.cpp
```

Preflight only:

```bash
mtp-forge embed target.gguf donor.gguf final.gguf --check-only --llama-cpp /opt/llama.cpp
```

The embedder checks:

- identical `general.architecture` and endianness;
- target regular block count equals donor total blocks minus donor MTP blocks;
- donor declares `<architecture>.nextn_predict_layers`;
- target does not already declare or contain those MTP tensors;
- critical hidden-size, attention, MoE, SSM, rope, and tokenizer metadata match;
- MTP tensors occupy the donor's declared extra block indices;
- all target tensor types are preserved;
- all donor MTP tensor types are preserved;
- target and MTP tensor bytes match SHA-256 after writing.

Full hashing is the default. `--skip-full-hash` exists for expert use but weakens the post-write guarantee.

Use an embedded result with:

```bash
llama-server -m /models/final-with-mtp.gguf --spec-type draft-mtp --spec-draft-n-max 2
```

## Standalone extraction versus embedding

These operations are not interchangeable.

- `extract` starts from the original HF checkpoint. llama.cpp knows the model-specific mapping and writes a standalone draft architecture.
- `embed` starts from two GGUF files. It supports the common embedded NextN representation where donor MTP tensors are extra `blk.N.*` blocks and the GGUF declares `nextn_predict_layers`.
- MTP Forge intentionally does not reverse-engineer an arbitrary embedded GGUF into a standalone draft. That transformation can require architecture-specific metadata rewriting and is safest in llama.cpp's HF converter.

## Inspect metadata

```bash
mtp-forge inspect /models/model-hf --json
mtp-forge inspect /models/model.gguf --json --llama-cpp /opt/llama.cpp
```

## Important limitations

- An MTP head is trained for a particular base model. Matching dimensions do not guarantee a good acceptance rate after the backbone has been modified or fine-tuned.
- The target and donor tokenizers must match for embedding.
- Quantizing a draft can change acceptance and speed.
- llama.cpp MTP support is architecture-specific and evolving. Update llama.cpp when a newly released model is not recognized.
- Model and donor licenses must permit your copying, use, and redistribution.
- Outputs must not exist; in-place mutation is never performed.

## Tests

```bash
python -m unittest discover -s tests -v
python -m compileall -q src
```

## Upstream contracts

MTP Forge v2.0.0 follows [llama.cpp's maintained `--mtp` exporter](https://github.com/ggml-org/llama.cpp/blob/master/convert_hf_to_gguf.py) for architecture-specific extraction and its [speculative decoding guide](https://github.com/ggml-org/llama.cpp/blob/master/docs/speculative.md) for standalone and embedded MTP runtime usage. Run `mtp-forge doctor` and the default preflight after updating llama.cpp.

## Publishing

Follow [PUBLISHING.md](PUBLISHING.md) for exact GitHub setup, push, tag, and release commands.

## License

MIT. See [LICENSE](LICENSE). MTP Forge is a community project and is not affiliated with Qwen, Alibaba, Hugging Face, or llama.cpp.
