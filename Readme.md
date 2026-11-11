# Equity Derivatives Trade Lifecycle & Control Engine

## Overview
An end-to-end operational risk pipeline simulating Front-to-Back (T+0 to T+2) trade processing for equity options, swaps, and stock loans. This repository is actively under development, featuring automated trade reconciliation to identify booking mismatches and an AI agent for structured root-cause analysis (RCA).

## Architecture & Roadmap
- [x] **Module 1: Synthetic Data Generator** - Object-oriented generation of FO and BO trade feeds with injected operational breaks (accrual mismatches, aged unconfirmed trades, MTM pricing overrides).
- [ ] **Module 2: Automated Trade Reconciler** - Multi-field outer join matching and operational Key Risk Indicator (KRI) calculation (Exposure at Risk).
- [ ] **Module 3: AI RCA Governance Engine** - 5-Why root-cause analysis using LLMs to output audit-ready executive memorandums.
