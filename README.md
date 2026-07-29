# Exploring the Jacobian lens

**Live page: https://isacsahlberg.github.io/jspace-exploration-results/**

An independent replication and extension of Anthropic's
[*Verbalizable Representations Form a Global Workspace in Language Models*](https://transformer-circuits.pub/2026/workspace/index.html)
(Transformer Circuits, 2026). This page is a static, self-contained rendering
of the results we have so far.

The Jacobian lens reads out what a mid-network activation is *disposed to make
the model say*: it transports `h_l` into the final-layer basis with the
corpus-averaged input–output Jacobian `J_l = E[dh_final/dh_l]`, then decodes it
through the model's own unembedding. We set out to test what that "workspace"
readout is actually good for — when it forms during training, whether it
exposes things the model declines to say, how much of it survives changing the
fitting corpus, and which of the paper's claims hold up under matched controls.

Concretely, that meant fitting lenses from scratch across 12 OLMo-3-7B
pretraining checkpoints and on several model families, building a probe harness
to compare in-band readouts against output logits, and designing controls for
the paper's meta-token claim. Some of it replicates cleanly, some of it
doesn't — both are on the page.

## Findings on the page

| | Finding | Status | One-liner |
|---|---|---|---|
| A | Workspace formation | 12-checkpoint sweep done | In OLMo-3-7B the workspace ignites at 25–50k steps — well after LM competence (~5k) — and its readout layer migrates deeper (L9 → L24) |
| B | Lie detection | First pass done (27B strong) | Told to lie, the model says "Toronto" while Ottawa sits at rank 2 in the workspace — a zero-shot polygraph, though it does not hold up as a population statistic |
| C | Censorship | Behavioral solid; lens-level suggestive | Refusals look like suppression, not ignorance — a base model continues with content the aligned model withholds |
| D | Corpus dependence | 3 models done | Workspace-band transport is corpus-invariant; early layers inherit the fitting corpus |
| E | Prompt variance | Main study done + replication | One prompt closes 79% of the logit-lens → n=1000 gap, five close 97% — only early layers need averaging |
| F | Meta-tokens / pun probe | Phase 0 done | The paper's algorithm-naming meta-token does *not* survive matched controls; sense disambiguation and answer anticipation replicate strongly |

Every chart and interactive demo is rendered from cached experiment outputs —
there is no GPU or model behind this page.

## Status: ongoing

This is a snapshot of work in progress, not a finished project — several
threads listed above are still open, and the page will change as they close.
The steering study and the interactive layer × position slice viewers are
omitted here because their source artifacts are not in git; they live in the
main repo's Hugging Face backup. Negative and mixed results are kept on the
page rather than dropped — the meta-token control failure and the lie-detection
scale-up are both reported as they came out.

## Provenance

Research code, data, and full writeups (with caveats) live in the main repo:
**https://github.com/TheUnicat/jspace-exploration** — one folder per finding
under `results/`, each with a summary, a narrative writeup, the underlying
data, and figures. Large artifacts are backed up to
[`isacsahlberg/jspace-lenses`](https://huggingface.co/isacsahlberg/jspace-lenses).

## Rebuilding this page

The page was the `/results` route of a FastAPI demo server that ran on a rented
GPU pod, fetching a JSON bundle from `/api/results`. That pod is gone. Since the
aggregator reads straight out of `results/`, the whole thing bakes down to
static files:

```bash
# from a checkout of TheUnicat/jspace-exploration
python3 build_site.py --out ../jspace-exploration-results
```

That regenerates `index.html`, `results.json`, and `files/` here. No server, no
key, no GPU.
