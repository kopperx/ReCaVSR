<h1 align="center">ReCaVSR: One-Step Streaming Diffusion Video Super-Resolution<br>with Recycled Latents and Learned Cache Routing</h1>

<p align="center">Xijun Wang · Xin Li · Suhang Yao · Zirui Lang · Bingchen Li · Zhibo Chen</p>

<p align="center">University of Science and Technology of China</p>

<p align="center">
  <a href="https://kopperx.github.io/ReCaVSR/">Project Page</a> ·
  <a href="https://arxiv.org/abs/2609.37831">Paper</a> ·
  <a href="https://huggingface.co/kopper/ReCaVSR">Pretrained Models</a> ·
  <a href="https://github.com/kopperx/ReCaVSR">Code</a>
</p>

ReCaVSR is a one-step streaming video super-resolution method built on Wan2.2. It
reuses previously generated super-resolution latents, assigns different temporal
cache scopes to transformer layers, and decodes with a low-resolution-conditioned
FlashDecoder. This repository provides the inference code and links to pretrained
weights.

## Visual Results

| Low-resolution input | ReCaVSR output (4×) |
| :---: | :---: |
| https://github.com/user-attachments/assets/e78d3868-b06e-4a57-b025-fac737351230 | https://github.com/user-attachments/assets/da6ef150-e0ce-4f4d-bbd5-afb0858c588f |

Both videos retain all 361 frames at their original pixel scale after cropping
the bottom label. Source MP4s: [input](assets/demo/input.mp4) ·
[output](assets/demo/output.mp4).

## Quick Start

**Requirements:** Linux x86-64, Python 3.11–3.13, an NVIDIA GPU, and a CUDA
13.0-compatible driver. Install [uv](https://docs.astral.sh/uv/getting-started/installation/)
before running these commands.

1. Clone the repository and install the locked dependencies:

   ```bash
   git clone https://github.com/kopperx/ReCaVSR.git
   cd ReCaVSR
   uv sync --locked
   ```

2. Download the default model weights into `checkpoints/`:

   ```bash
   uvx hf download kopper/ReCaVSR \
     transformer.safetensors prompt.safetensors flashdecoder.safetensors \
     --local-dir checkpoints
   ```

3. Run a 31-frame preview:

   ```bash
   uv run python inference.py \
     --input assets/demo/input.mp4 \
     --output outputs/demo_preview_x4.mp4 \
     --scale 4 \
     --frames 31
   ```

The first run may take longer because compilation is enabled by default. To
process the full demo, omit `--frames` and choose a new `--output` path; the
script does not overwrite existing results.

## Pretrained Models

The [Hugging Face repository](https://huggingface.co/kopper/ReCaVSR) hosts the
weights. The matching JSON configuration files are already in `checkpoints/`.

| Weight file | Role | Needed for |
| --- | --- | --- |
| `transformer.safetensors` | Merged diffusion transformer | All runs |
| `prompt.safetensors` | Prompt embedding | All runs |
| `flashdecoder.safetensors` | Low-resolution-conditioned decoder | Default decoder |
| `vae.safetensors` | Original Wan decoder | Optional `--decoder wan` |

The transformer checkpoint is already merged; no adapter conversion is needed.
If you use a different `--model-dir`, copy `model_config.json`, `vae_config.json`,
and `flashdecoder_config.json` into it as well.

For the optional Wan decoder, download its weight and pass `--decoder wan` to
the inference command:

```bash
uvx hf download kopper/ReCaVSR vae.safetensors --local-dir checkpoints
```

## Inference Options

- `--scale 2` or `--scale 1.5` changes the spatial upscaling factor.
- `--device cuda:1` selects a different GPU.
- `--color-fix wavelet` or `--color-fix adain` enables color correction.
- `--model-dir /path/to/checkpoints` selects another model directory.
- `--frames 31` limits the run to a short preview.
- `--no-compile-blocks --no-compile-decoder` disables compilation.

Input may also be a numbered image directory; use `--fps` to set its playback
rate. See `uv run python inference.py --help` for the full CLI reference.

## Method at a Glance

1. **Recycled latents:** predictions from earlier blocks provide local temporal
   context to later blocks.
2. **Layer-wise cache routing:** transformer layers use different history scopes
   under a fixed cache budget.
3. **LR-conditioned FlashDecoder:** low-resolution observations help decode the
   generated latents efficiently.

For the architecture and experiments, see the [paper](https://arxiv.org/abs/2609.37831).

## Scope and Limitations

- The runtime requires a CUDA GPU and uses one GPU per inference process.
- Model inference is streaming, while the current reader and writer buffer the
  entire clip in CPU memory.
- Output preserves the input frame count but does not contain audio.
- The script writes an MP4 and a JSON run report, and refuses to overwrite
  either existing output.

## Citation

If ReCaVSR is useful in your research, please cite the paper:

```bibtex
@misc{wang2026recavsronestepstreamingdiffusion,
  title={ReCaVSR: One-Step Streaming Diffusion Video Super-Resolution with Recycled Latents and Learned Cache Routing},
  author={Xijun Wang and Xin Li and Suhang Yao and Zirui Lang and Bingchen Li and Zhibo Chen},
  year={2026},
  eprint={2609.37831},
  archivePrefix={arXiv},
  primaryClass={cs.CV},
  url={https://arxiv.org/abs/2609.37831}
}
```

## Acknowledgements

This project builds on Wan and Hugging Face Diffusers. Color correction follows
[StableSR](https://github.com/IceClear/StableSR).

## License

The code is released under the [Apache License 2.0](LICENSE). Model weights and
demo footage remain subject to their respective licenses.
