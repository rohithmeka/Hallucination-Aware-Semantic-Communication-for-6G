# Hallucination Detection and Semantic Drift Verification for Reliable 6G Semantic Communication

**A receiver-side Semantic Consistency Verifier (SCV) that flags when a semantic decoder outputs a message whose meaning has drifted from what was sent (a changed number, a flipped negation, an antonym) without the receiver ever seeing the original text.**

## The problem

Semantic communication systems, a key idea in 6G research, transmit the **meaning** of a message (an embedding) instead of its exact bits. The receiver reconstructs text from that meaning.

This creates a failure mode that classical bit-level error checking cannot catch: the decoder can output a sentence that is **fluent, close in embedding space, and wrong**.

| Sent | Decoded | Embedding similarity | Meaning |
|---|---|---|---|
| The meeting is at 5 pm | The meeting is at 7 pm | High | Wrong |
| The valve is open | The valve is not open | High | Opposite |
| Temperature is rising | Temperature is falling | High | Opposite |

We call these **semantic hallucinations**. A CRC would pass them, and a plain similarity score often would too.

## Our contribution

A **Semantic Consistency Verifier (SCV)** at the receiver that decides, for each decoded message, whether to **accept** it or **flag** it as a likely hallucination. The decision uses only what a real receiver has: the received (noisy) embedding and the decoder's candidates. The receiver never has the sender's original text.

---

## System pipeline

```text
 TRANSMITTER                    CHANNEL                    RECEIVER
┌──────────────┐          ┌──────────────────┐     ┌──────────────────────┐
│ 1. Data      │          │ 3. 6G channel    │     │ 4. Semantic decoder  │
│    source    │          │    model         │     │    (candidate-based) │
│      │       │          │                  │     │          │           │
│      ▼       │ embedding│                  │noisy│          ▼           │
│ 2. Semantic  │─────────►│                  │────►│ 5. Semantic          │
│    encoder   │  (384-d) │                  │     │    Consistency       │
│ (MiniLM-L6)  │          │                  │     │    Verifier (SCV)    │
└──────────────┘          └──────────────────┘     │          │           │
                                                   │   ACCEPT / FLAG      │
                                                   └──────────────────────┘
```

### 1. Data source
Arbitrary user text messages. <!-- name the dataset / sentence set and its size -->

### 2. Semantic encoder
Each message is encoded into a 384-dimensional sentence embedding using `all-MiniLM-L6-v2` (sentence-transformers).

### 3. 6G channel model
The embedding is transmitted over a simulated channel.
<!-- document exactly: fading model (e.g. Rayleigh / Rician), AWGN, any quantization or modulation, and how SNR is defined (per symbol? per embedding dimension? in dB) -->

SNR definition used throughout: <!-- e.g. SNR = 10·log10(signal power / noise power) per embedding dimension -->

### 4. Semantic decoder
The receiver reconstructs text by choosing the best match from a candidate set. Candidates include controlled semantic corruptions: number substitutions, antonym swaps and negation flips. This mimics the near-miss outputs a real generative decoder can produce.

<!-- state clearly how candidates are generated WITHOUT using the original sentence at the receiver -->

### 5. Semantic Consistency Verifier (SCV)
For the chosen decoded message, the SCV computes:

| Signal | What it measures |
|---|---|
| Similarity score | Cosine similarity between the received embedding and the decoded text's embedding |
| Margin | Gap between the best and second-best candidate; a small margin means the decoder was unsure |
| Fact consistency | Whether numbers, negations and antonym-sensitive words are stable across the top candidates |

These are combined into an accept/flag decision. Thresholds are calibrated on a training split and evaluated on a held-out test split.

---

## Evaluation

Four systems are compared on the same channel conditions:

| System | Description |
|---|---|
| S1: No verifier | Accept every decoded message |
| S2: Similarity only | Flag if similarity < threshold |
| S3: Similarity + margin | Add decoder-confidence margin |
| S4: Full SCV | Similarity + margin + fact consistency |

### Results
<!-- fill from your test split; report mean ± 95% CI over repeated runs -->

| System | Hallucination detection rate | False-alarm rate | F1 |
|---|---|---|---|
| S1 | | | |
| S2 | | | |
| S3 | | | |
| S4 (SCV) | | | |

<!-- add a plot: detection rate vs SNR for S1–S4 -->

---

## Installation

```bash
git clone https://github.com/rohithmeka/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
```

**Requirements:** Python 3.x, NumPy, scikit-learn, sentence-transformers

## Usage

<!-- e.g. open notebooks/pipeline.ipynb in Colab/Jupyter and run all cells, or: python main.py -->

## Project structure

```text
<repo-name>/
├── notebooks/          # Colab / Jupyter notebooks
├── src/                # encoder, channel, decoder, verifier modules
├── results/            # metrics and plots
├── requirements.txt
└── README.md
```
<!-- match your actual layout -->

---

## Limitations

- Corruptions are **controlled** (numbers, negations, antonyms). Real generative decoders can hallucinate in other ways.
- The decoder is candidate-based, not a trained generative model.
- The channel is a simulation; results are not measured over real 6G hardware.
- Evaluation is on English sentences only.

## Future scope

- A trained generative semantic decoder in place of candidate retrieval
- Richer channel models (multipath fading, mobility, quantized transmission)
- Detection of broader hallucination types (entities, dates, units)
- Feedback from SCV to the transmitter to request retransmission of flagged messages

---

- Rohith Meka
GitHub: @rohithmeka

Course: Computer Networks · Instructor: Dr. Sundaresan

## License

MIT
