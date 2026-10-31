# orbit-models

Models that [Orbit](https://github.com/DragonQLF) runs on your own machine. Nothing here is a service: each model is a
fixed file in a release, downloaded at build time, pinned by size and SHA-256, and run locally. No data leaves the
computer to use them.

## action-model-v2 — Orbit's consequence check

A small text classifier that reads an action an AI agent is about to take — a button it is about to click, a shell
command, a tool call — and says what kind of consequence it has:

| Label | Means |
|---|---|
| `money` | pays, orders, subscribes, transfers |
| `account` | changes an account, its security or who has access |
| `destructive` | deletes something that can't be undone |
| `send` | sends or publishes something to other people |
| `benign` | none of the above |

Orbit combines it with fixed rules (which always win), its confidence, and checks for reversibility, prompt
injection and whether the action matches what the person asked. Whatever the model is unsure of is asked, never
allowed.

**Input format:** `"<verb> <target>"`, optionally `" | <context>"` — e.g. `click Pay now | Order total 42,90 €`,
`run git push --force origin main`.

**Model:** [`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2)
fine-tuned with [SetFit](https://github.com/huggingface/setfit) on 2,830 generated examples in English, Portuguese,
Spanish, French, German and Italian. The training data is synthetic; it contains no personal data. The transformer
is exported to ONNX and dynamically quantized to INT8 (per channel); the classification head is a logistic
regression stored as JSON. Inference: tokenize → transformer → mean pooling → `head.json`.

**Measured** on 48 held-out actions never used in training (including Spanish and French): 46/48 correct, and no
consequential action predicted as benign.

### Files

| File | Size | SHA-256 |
|---|---|---|
| `model.int8.onnx` | 118,478,634 | `485d09625dbf8d3fdbd8b5a37a53a1333790a40d84cfa69a0ef0dd3c8880f400` |
| `tokenizer.json` | 17,082,987 | `cad551d5600a84242d0973327029452a1e3672ba6313c2a3c3d69c4310e12719` |
| `tokenizer_config.json` | 1,457 | `f469534f4662884f6a1f5ff2a5fe70cbc6dc4a8c34c1fa765d2f022ea9a0ac12` |
| `head.json` | 41,560 | `27a8740391916903252e1395a95c8aa9773486153cf8a6e4fd526ac2252d7c37` |

A file that doesn't match is never used.

## License

Apache License 2.0 (see `LICENSE`). The model is a derivative of
`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`, © the sentence-transformers authors, also Apache 2.0;
the tokenizer files come from that model unchanged.
