# Explainable AI-Powered Phishing URL Detector — Summary

A content-based phishing detection system that classifies URLs as **phishing** or **benign** and produces a grounded natural-language explanation in the same model output. Built as the ENS 491–492 graduation project at Sabancı University.

## What We Did

- **Collected a custom dataset** from OpenPhish (phishing, live feed via Docker + Scrapling every 5 minutes), Tranco (benign), and urlscan.io (benign-suspicious, validated against VirusTotal and Google Safe Browsing), all stored in MongoDB.
- **Built a feature extractor** that turns each webpage into three layers: raw input (URL, title, text), numerical measurements (form count, redirect count, external anchor ratio, etc.), and detected signals tagged with `direction`, `severity`, and a human-readable statement.
- **Audited features with XGBoost + SHAP + LOFO + a per-feature source check** to drop artifacts and dead-weight features. 23 of 36 features kept.
- **Reformulated detection as text generation**: the model emits four structured sections — suspicious features, benign features, rationale, verdict.
- **Fine-tuned Llama-3.2-3B and Llama-3.1-8B** with LoRA (rank 16, alpha 32) on 4-bit quantized base weights, on a single A100.
- **Used a domain-aware split** with a 30-row-per-domain cap, so no registered domain appears in more than one of train / val / test.
- **Controlled leakage**: explanation targets are built from annotated feature metadata at training time, but those annotations are never given to the model at inference.
- **Evaluated both classification and explanation**: accuracy, precision, recall, F1, parse-failure rate, plus faithfulness (overlap between cited signals and actual signals), unsupported-concept rate as a hallucination indicator, ROUGE-L, and BLEU.
- **Built a Django + Docker proof-of-concept app** where a user submits a URL, a hardened scraper container fetches and analyzes it, and the fine-tuned model returns a verdict and rationale.

## Headline Results (1,500-URL test set, cleaned features)

| Model | Accuracy | F1 | Parse failures | Faithfulness F1 |
|---|---|---|---|---|
| Llama-3.2-3B FT | 0.975 | 0.975 | 0 | 0.648 |
| Llama-3.1-8B FT | 0.987 | 0.987 | 0 | 0.646 |

The 8B model wins on classification by ~1 F1 point but has nearly identical faithfulness, suggesting explanation grounding is bounded by **feature quality**, not model size.

## Main Contribution

Unlike most phishing detectors, classification and explanation are produced **jointly** by the same model, the explanations are **grounded in the same signals the model sees as input**, and explanation faithfulness can be **checked automatically** rather than only judged by humans.

## Known Limitations

- Hallucinated explanations still occur, especially when the model leans on weak generic signals.
- Hosted-platform impersonation (`github.io`, `vercel.app`, `pages.dev`) remains a hard case.
- Text-only; no visual modality (screenshots, logo matching) due to budget.
- Classical ML (XGBoost) still edges out the LLM on raw classification — the LLM's advantage is the grounded rationale, not the label.
