# 🗣️ SpeechT5 Text-to-Speech Fine-Tuning on VoxPopuli (Dutch)

This project fine-tunes Microsoft's [SpeechT5](https://huggingface.co/microsoft/speecht5_tts) model for Dutch Text-to-Speech (TTS) synthesis using the [VoxPopuli](https://huggingface.co/datasets/facebook/voxpopuli) dataset. The fine-tuned model is published to the Hugging Face Hub and can be used to synthesize natural-sounding Dutch speech from text.

---

## Overview

This notebook implements an end-to-end pipeline for fine-tuning a pre-trained TTS model on a language-specific dataset. While VoxPopuli is primarily an ASR (Automatic Speech Recognition) dataset, it is used here as a practical exercise for TTS fine-tuning, leveraging speaker embeddings extracted via SpeechBrain's x-vector model to condition the synthesis on specific speaker identities.

The fine-tuned model is available on the Hugging Face Hub at:
**[`Satvik-ai/speecht5_finetuned_voxpopuli_nl`](https://huggingface.co/Satvik-ai/speecht5_finetuned_voxpopuli_nl)**

---

## Dataset

**[Facebook VoxPopuli](https://huggingface.co/datasets/facebook/voxpopuli)** — Dutch (`nl`) subset

- A large-scale multilingual speech corpus sourced from 2009–2020 European Parliament event recordings.
- Contains labelled audio-transcription data for 15 European languages.
- The Dutch subset is used for this project.

**Data Selection Strategy:**
- A random 25% subset of the full Dutch training split is used to reduce training time.
- Audio is resampled to **16,000 Hz**.
- Speakers with fewer than 100 or more than 400 examples are filtered out to balance the dataset and improve training stability.
- Sequences longer than 200 tokens are removed to prevent memory issues.
- Final split: **90% train / 10% test**.

---

## Model Architecture

| Component | Model |
|---|---|
| TTS Backbone | [`microsoft/speecht5_tts`](https://huggingface.co/microsoft/speecht5_tts) |
| Vocoder | [`microsoft/speecht5_hifigan`](https://huggingface.co/microsoft/speecht5_hifigan) |
| Speaker Encoder | [`speechbrain/spkrec-xvect-voxceleb`](https://huggingface.co/speechbrain/spkrec-xvect-voxceleb) |

**SpeechT5** uses a unified encoder-decoder framework for speech tasks. For TTS:
- Text is tokenized and encoded via the **encoder**.
- The **decoder** generates log-mel spectrograms conditioned on both the encoded text and a **speaker embedding** (x-vector).
- The **HiFi-GAN vocoder** converts the generated spectrogram back into a raw audio waveform.

---

## Project Pipeline

```
Raw Text (Dutch)
      │
      ▼
 SpeechT5Processor  ───────────────────────────────────────┐
 (Tokenization)                                            │
      │                                                    │
      ▼                                                    │
 SpeechT5ForTextToSpeech  ◄── Speaker Embeddings (x-vec)   │
 (Fine-tuned Encoder-Decoder)                              │
      │                                                    │
      ▼                                                    │
 Log-Mel Spectrogram                                       │
      │                                                    │
      ▼                                                    │
 SpeechT5HiFiGan (Vocoder)                                 │
      │                                                    │
      ▼                                                    │
 Audio Waveform (.wav / IPython Audio)  ◄──────────────────┘
```

---

## Results

The fine-tuned model is capable of synthesizing intelligible Dutch speech conditioned on a target speaker embedding. Given that VoxPopuli is an ASR dataset (not optimized for TTS), the output quality is reasonable for a fine-tuning exercise and serves as a solid baseline.

The model is hosted on the Hugging Face Hub:
🔗 **[`Satvik-ai/speecht5_finetuned_voxpopuli_nl`](https://huggingface.co/Satvik-ai/speecht5_finetuned_voxpopuli_nl)**

---

## Acknowledgements

- [Microsoft SpeechT5](https://arxiv.org/abs/2110.07205) — The pre-trained TTS model.
- [Facebook VoxPopuli](https://arxiv.org/abs/2101.00390) — The multilingual speech dataset.
- [SpeechBrain](https://speechbrain.github.io/) — Speaker embedding (x-vector) extraction.
- [Hugging Face](https://huggingface.co/) — Model hub, `transformers`, and `datasets` libraries.
- This project is inspired by the [Hugging Face Audio Course](https://huggingface.co/learn/audio-course).