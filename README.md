# Superflow cleanup model

A small text model that tidies up dictated speech for the Superflow dictation app: it removes filler words
("um", "uh", "like"), handles spoken self-corrections ("no wait", "scratch that"), and adds punctuation and
capitals, without changing what was said. It is not a chat model and is not meant to answer questions or
follow instructions; a dictated "write a poem" should come back as the cleaned sentence.

## Files (release `cleanup-v4`)
| File | Size | SHA-256 |
|---|---|---|
| `superflow-cleanup-v4.gguf` | 1,274,396,320 bytes | `3af4f7519dd10f442558c4a80fabeec2f6759743c66241e7c7c7e9e35320e750` |

GGUF, Q4_K_M, for llama.cpp (`llama-server --jinja`). Send chat requests with
`chat_template_kwargs: {"enable_thinking": false}`. It expects Superflow's own cleanup prompts; other prompts
are untested.

## Base model and licence
- Base: [Qwen3.5-2B](https://huggingface.co/Qwen) by Alibaba Cloud, Apache License 2.0.
- This model: Apache License 2.0 (`LICENSE`), modified as described in `NOTICE`.

## Training
LoRA (rank 16, 2 epochs) on 3,003 cleanup pairs, merged and quantised to Q4_K_M. Each pair is a raw
dictation-style transcript and its cleaned version under one of Superflow's cleanup levels (light, medium,
high) and app contexts.

Data sources:
- **Inputs:** dictation-style transcripts written by MiniMax-M3, and speech-model (Parakeet TDT 0.6B v3)
  transcripts of public test clips from LibriSpeech, Common Voice, AMI meetings, Earnings-22 and URGENT 2024,
  as sampled by the Open ASR Leaderboard test sets.
- **Targets (the cleaned side):** written by MiniMax-M3 under Superflow's prompts, then filtered by rule-based
  checks (dropped content, lost numbers or negations, added words, obeyed commands). Failing pairs were
  dropped, never edited.
- No user recordings or user dictations were used for training.

## Evaluation
Superflow's cleanup bench (57 cases, 3 passes each, through Ollama on a GPU), "correct in all runs":

| Model | All 57 | Old 35 | New 22 (meaning, commands, short takes) |
|---|---|---|---|
| gemma3:12b | 54 | 32 | 22 |
| **this model (v4)** | **52** | **31** | **21** |
| gemma3:4b | 40 | 20 | 20 |

Median time per cleanup: about 0.12 s on a GPU; about 1.2 s on 4 CPU threads (Ryzen 9 9900X3D). Memory
about 1.7 GB.

## Known limits
- Sometimes answers a dictated request instead of cleaning it (wrote a haiku for "write a haiku about
  autumn" in 1 of 3 passes), and can turn "because ... and" into ". ... but". Superflow checks every answer
  for added words, lost negations, numbers and swapped connectors and falls back to the raw transcript.
- English only. Tested on one PC; not yet on a PC without AVX2.
