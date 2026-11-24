# hf-downloader

`hfdl` is a focused Rust downloader for Hugging Face model artifacts. It writes
files directly to a directory; it does not create a Hugging Face cache, blobs,
symlinks, manifests, or persistent metadata.

## Install

### Arch Linux (x86_64)

The `huggingface-downloader-bin` package installs `/usr/bin/hfdl`. When upgrading, accept pacman’s prompt to remove `huggingface-dl-bin`.

Download the `.pkg.tar.zst` from [GitHub Releases](https://github.com/delfianto/hf-downloader/releases/latest)
and install it with `sudo pacman -U ./<package-file>`, or build the package
from the checksum-verified release archive with `makepkg -si`:

```sh
git clone --branch v0.3.1 https://github.com/delfianto/hf-downloader.git
cd hf-downloader/arch
makepkg -si
```

### Build from source

Source builds require Rust 1.100.0 or later. Starting with v0.3.0, run
`hfdl` in scripts that used `hfd`.

```sh
cargo install --path .
# or: just install
```

## Usage

```sh
hfdl black-forest-labs/FLUX.1-dev-FP8
# writes ./flux1-dev-fp8.safetensors
```

By default hfdl selects `.safetensors`, `.gguf`, `.bin`, `.pt`, `.pth`, `.ckpt`,
and `.onnx` files, plus shard index companions such as
`.safetensors.index.json`, and writes their basenames into the output directory.

```sh
hfdl org/model -o models -r main -j 8
hfdl org/model --all --repo-dir -o models
hfdl org/model -i Q4           # downloads all matching Q4 GGUFs
hfdl org/model -i Q4_K_M       # downloads only that specific quant
hfdl org/model -i 'weights/**' -e '*.json'
hfdl org/model -y              # bypass interactive selection prompt
hfdl org/private-model -t "$HF_TOKEN"
```

When running `hfdl org/model` interactively without `--include`, an interactive
TUI selection prompt lets you choose which files to download.

`--all` selects every file. `--include` and `--exclude` match remote relative
paths using glob, substring, or regex patterns (excludes win). Flat output
refuses basename collisions; use `--repo-dir` to write to
`<output>/<owner>/<repo>/` while preserving remote directories. Use `-y` /
`--yes` to skip interactive selection and download all matching artifacts.

Authentication uses `--token` or the `HF_TOKEN` environment variable. Use
`--endpoint` for a compatible Hub endpoint. Downloads resume from local `.part`
files when the server validates byte ranges, and completed same-size files are
skipped.

## Configuration

By default hfdl reads `$XDG_CONFIG_HOME/hfd.toml`, falling back to
`~/.config/hfd.toml` on Linux and macOS. A missing default file is fine. Pass
`--config /path/to/hfd.toml` to use another file.

```toml
# Defaults for downloads that do not pass -j.
max_download_jobs = 4

# Concurrent HTTP range requests per model file. Default: 1.
max_parallel_parts_per_job = 1

# Used when neither --token nor HF_TOKEN is provided.
huggingface_token = "hf_..."
```

Run `hfdl --help` for the complete command reference. Development checks:

```sh
just full-gate
```

Optional live discovery tests use public Hub metadata only—never model downloads:

```sh
just test --integration
```
