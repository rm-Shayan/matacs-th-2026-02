# Prototype: AI-Assisted Proposal Triage

This directory contains an executable Python prototype demonstrating the triage and deterministic scoring engine for Reference `MATACS-TH-2026-02`.

## Overview
- Demonstrates deterministic pass/fail rule gates (pure code).
- Demonstrates prompt injection detection quarantine (Proposal C).
- Computes mathematical weighted scoring without LLM arithmetic errors.
- Generates JSON matching the exact schema required for `ranking.json`.

## How to Run
```bash
python triage.py
```

## Security & API Configuration
Live API keys are never committed. See `.env.example` for environment variable configuration when connecting live LLM inference providers.

