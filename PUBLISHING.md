# Publish MTP Forge on GitHub

Replace `YOUR_USERNAME` before running the commands below.

## 1. Verify the release

```bash
python -m pip install -e .
python -m unittest discover -s tests -v
python -m compileall -q src
mtp-forge --version
```

## 2. Replace placeholder URLs

Replace `YOUR_USERNAME` in `README.md` and `pyproject.toml`, then run the tests again.

## 3. Initialize Git

```bash
git init -b main
git add .
git status
git commit -m "Release MTP Forge v2.0.0"
```

Confirm that no `.gguf`, model weights, credentials, or local environments are staged.

## 4. Create the empty GitHub repository

Use these GitHub settings:

- Repository name: `mtp-forge`
- Description: `Extract and verify MTP/NextN GGUF speculative draft models`
- Visibility: Public
- Initialize with README: No
- Add `.gitignore`: No
- Add license: No

## 5. Push and tag

```bash
git remote add origin https://github.com/YOUR_USERNAME/mtp-forge.git
git push -u origin main
git tag -a v2.0.0 -m "MTP Forge v2.0.0"
git push origin v2.0.0
```

SSH alternative:

```bash
git remote add origin git@github.com:YOUR_USERNAME/mtp-forge.git
git push -u origin main
git tag -a v2.0.0 -m "MTP Forge v2.0.0"
git push origin v2.0.0
```

## 6. Create the GitHub release

Open **Releases → Draft a new release**, choose `v2.0.0`, use the title `MTP Forge v2.0.0`, summarize extraction, quantization, structural verification, and byte-preserving embedding, then publish.

With GitHub CLI:

```bash
gh repo create mtp-forge --public --source=. --remote=origin --push
git tag -a v2.0.0 -m "MTP Forge v2.0.0"
git push origin v2.0.0
gh release create v2.0.0 --title "MTP Forge v2.0.0" --generate-notes
```

## 7. Recommended settings

1. Add topics: `mtp`, `nextn`, `gguf`, `llama-cpp`, `speculative-decoding`.
2. Enable Issues.
3. Protect `main` and require the `tests` workflow.
4. Disable force pushes to `main`.

## Release checklist

- [ ] Clean-environment tests pass.
- [ ] Placeholder username is gone.
- [ ] No GGUF/model files or credentials are tracked.
- [ ] README commands match `mtp-forge --help`.
- [ ] llama.cpp requirements use the current path.
- [ ] Package and tag versions are both `2.0.0`.
