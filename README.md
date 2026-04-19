# Architectural Analysis and Mitigation of Software Supply Chain Attacks in Modern DevOps Environments

> **Author:** Laiba Saleem  
> **Roll No:** BITF22M043 | **Session:** F22-OC | **Presentation:** 20 May 2026

---

## Abstract

Software supply chain (SSC) attacks have evolved from isolated incidents into a systemic threat to modern DevOps ecosystems. High-profile breaches such as **SolarWinds**, **Log4Shell**, and the **XZ Utils backdoor** have demonstrated that adversaries increasingly target the development pipeline itself rather than the final application.

This paper synthesizes the state of the art across **ten recent research contributions** (2024–2025) spanning threat modelling, SBOM integrity, framework analysis, and agentic AI defence. Building on this synthesis, it proposes the **Risk-Aware Artifact Propagation Model (RAPM)**: a novel architectural framework introducing the concept of *trust decay* across CI/CD pipeline stages.

---

## The Problem

Modern DevOps pipelines are increasingly targeted by attackers who compromise the **build and distribution process** rather than the application itself. Three critical pain points motivate this work:

| Problem | Scale |
|---|---|
| Malicious open-source packages | ↑ 200% year-on-year (929 in 2020 → 459,070 in 2024) |
| SBOM vulnerability scanner false positives | **97.5%** false positive rate |
| SolarWinds downstream blast radius | ~18,000 organisations compromised |

Existing frameworks (SLSA, SBOM, SSDF) are fragmented and treat trust as a **binary property**: either present or absent at the point of attestation. No framework models how trust in a software artefact *degrades* as it moves through unsigned pipeline stages, leaving organisations with blind spots between attestation checkpoints.

---

## Contributions

### C1. Literature Synthesis
Structured review of 10 recent SSC security papers, identifying convergences, divergences, and open research gaps across five capability dimensions (Threat Modelling, SBOM Integrity, False Positive Reduction, Trust Decay Modelling, Real-time Response).

### C2. Risk-Aware Artifact Propagation Model (RAPM)
A rule-based framework that treats trust as a **continuously diminishing property** across CI/CD stages. Trust is only restored when fresh attestation is performed. Three scoring factors:

- **Initial Source Trust** — derived from branch protection, code review, signed commits, and SLSA level
- **Attestation Quality Factor** — rewards artefact signing, provenance documents, SBOM attachment, and compliance scans at each stage
- **Time-Based Decay** — configurable half-lives per criticality tier (139 hrs / 35 hrs / 7 hrs for internal / customer-facing / critical infrastructure)

### C3. Contextual SBOM (C-SBOM)
An extension to standard CycloneDX 1.5 / SPDX 2.3 documents that embeds a `reachabilityContext` block per component, recording which vulnerable library functions are actually *called* by the application.

- Expected **≥60% reduction** in actionable vulnerability alerts
- Directly defends against SBOM insider manipulation attacks (Ozkan et al.)
- Compatible with existing SBOM toolchains

### C4. DevOps Trust Topology Graph (DTTG)
An automatically constructed, directed map of the entire CI/CD pipeline. Every component is a **node** (with a security rating 0–1) and every artefact handoff is a **connection** (with an attestation weight 0–1).

- Identifies the single highest-priority **trust hotspot** for remediation
- Built automatically from GitHub Actions YAML, Jenkins, or Tekton manifests in **under 1 second**
- Provides a structured data interface for agentic AI defences

---

## RAPM in Action: 5-Stage Pipeline Example

| Stage | Node Rating | Conn. Weight | Gap (hrs) | Trust Score |
|---|---|---|---|---|
| Source Commit | 0.90 | 1.00 | 0.0 | 0.90 |
| CI Build | 0.85 | 0.70 | 0.5 | 0.53 |
| Container Registry | 0.80 | **0.00** | 2.0 | **0.00*** |
| Staging Deploy | 0.75 | 0.80 | 1.0 | 0.43 |
| Production Deploy | 0.95 | 1.00 | 0.5 | 0.41 |

> *\*Connection weight of 0.0 (no signing at registry push) collapses trust to zero, triggering the deployment gate.*

Adding a single `cosign` signing step to the registry push restores production trust to **0.41**, above a typical gate threshold of 0.30.

---

## Comparison with Existing Approaches

| Approach | Scored | Dynamic | FP-Aware | Automated |
|---|:---:|:---:|:---:|:---:|
| SLSA | | | | Partial |
| STRIDE / OWASP | | | | |
| SBOM + VEX | | | Partial | Partial |
| Agentic AI | | ✓ | | ✓ |
| SCM2 Reference Architecture | | | | |
| **RAPM (this work)** | **✓** | **✓** | **✓** | **✓** |

---

## Gap Analysis: Related Work Coverage

| Work | Threat Modelling | SBOM Integrity | False Positives | Trust Decay | Real-time |
|---|:---:|:---:|:---:|:---:|:---:|
| Dhandapani (STRIDE) | ✓ | | | | |
| Chalmers (Framework) | | ✓ | | | |
| Ozkan (SBOM Integrity) | | ✓ | | | |
| Zhou (False Positives) | | | ✓ | | |
| Safronov (UniBOM) | | | ✓ | | |
| Tran (SCM2 RA) | | ✓ | | | |
| Syed (Agentic AI) | ✓ | | | | ✓ |
| Zahan (TOSEM Survey) | ✓ | | | | |
| Hamer (CTI Analysis) | ✓ | | | | |
| Tamanna (SLSA) | | | | | |
| **RAPM (this work)** | **✓** | **✓** | **✓** | **✓** | **✓** |

---

## SolarWinds Attack Simulation

RAPM was validated against a simulated SolarWinds-style build-injection attack. Under conventional SLSA-only monitoring, the attack succeeds because the artefact hash matches the compromised output. Under RAPM, **three independent defences trigger**:

1. **DTTG** detects non-hermetic build isolation, dropping the node security rating from 0.9 → 0.4, pushing the trust score below the deployment gate threshold.
2. **C-SBOM** registry cross-validation detects the injected component (absent from the lock file and public registry), halting SBOM assembly with a tamper alert.
3. **DTTG hotspot report** directs the security team immediately to the point of compromise, eliminating the need for manual investigation.

---

## Repository Contents
📄 paper.pdf          — Full IEEE-format research paper (9 pages)
📄 one-pager.pdf      — Project one-pager / handout summary
