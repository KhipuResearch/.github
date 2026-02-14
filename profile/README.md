<div align="center">
  <img src="https://raw.githubusercontent.com/KR-Labs/.github/main/profile/KRLabs_WebLogo.png?v=3" alt="Khipu Research Labs" width="400">
</div>

# Khipu Research Labs

## Open-Source Causal Inference for Policy Analysis

Policy analysis takes weeks because the infrastructure has to be built from scratch every time. KRL provides the evidence infrastructure so analysts can focus on the analysis.

Analysts spend the bulk of that time hunting data across federal agencies, cleaning inconsistent formats, debugging API authentication, and stitching together tools that were never designed to work together. The actual analysis—the part that answers the question—takes a fraction of the time. We built KRL to fix that ratio.

Khipu Research Labs develops evidence infrastructure for organizations that need defensible causal inference without assembling it from scratch. Our open-source notebooks and community data connectors give policy analysts, researchers, and students production-grade methods backed by authoritative federal data sources.

This is not a general-purpose data science platform. Every component exists to serve one discipline: rigorous policy evaluation using quasi-experimental and econometric methods that survive peer review, Inspector General audits, and legislative scrutiny.

---

## KASS: Knowledge & Analytics for Social Science

KASS is our open-source collection of Jupyter notebooks. Each one implements a specific causal inference or policy analysis method using real federal data, with full documentation of assumptions, limitations, and methodological choices.

**What makes these different from textbook examples:** real data from FRED, BLS, and Census. Explicit limitation tables. Sensitivity analysis treated as a first-class requirement, not an appendix. Every notebook opens with a data provenance badge telling you exactly what you're working with—whether the data is real, the treatment is simulated, or both.

### Featured Notebooks

| Notebook | Method | Domain | Type |
|----------|--------|--------|------|
| **NB07** — Labor Market Intelligence | Descriptive analytics, anomaly detection | Employment, wages, economic indicators | Descriptive |
| **NB14** — Synthetic Control Policy Lab | Synthetic control method (Abadie et al.) | State-level policy evaluation | Causal |
| **NB15** — Regression Discontinuity Toolkit | Sharp & fuzzy RDD with bandwidth sensitivity | Program eligibility thresholds | Causal |
| **NB20** — Opportunity Zone Evaluation | DiD + synthetic control | Place-based economic development | Causal |
| **NB22** — Workforce Development ROI | AIPW, cost-benefit per OMB Circular A-4 | Workforce program evaluation | Cost-Benefit |

Each causal notebook includes a **"Does / Doesn't Show"** table—a forcing function for honest interpretation. If the analysis can't establish a causal claim, the table says so before you reach the results.

### Quick Start

```bash
# Clone the repository
git clone https://github.com/KR-Labs/kass.git

# Install dependencies
pip install -r requirements.txt

# Get a free FRED API key at https://fred.stlouisfed.org/docs/api/api_key.html
# Add it to your .env file

# Run any notebook
jupyter notebook
```

**Requirements:** Python 3.9+, free FRED API key. Some notebooks require additional data access — check individual notebook headers for specific requirements.

---

## Who This Is For

**Policy analysts** who need methods that hold up under scrutiny—not toy examples from a textbook chapter. If you've been asked to evaluate a state program and your current toolkit is Excel and good intentions, start here.

**Researchers** building reproducible analysis pipelines. KASS notebooks log every data source, every parameter choice, every assumption. Fork them. Adapt them. Cite them.

**Students** learning applied econometrics and causal inference. These notebooks bridge the gap between theory (Angrist & Pischke, Cunningham) and the messy reality of federal administrative data.

**Government teams** under Foundations for Evidence-Based Policymaking Act mandates who need evaluation infrastructure but lack the staff to build it from zero.

---

## What's Free

Everything in this repository. No signup. No trial period. No credit card.

Community-tier data connectors ship with the notebooks and provide authenticated access to core federal economic data:

- **FRED Basic** — Federal Reserve Economic Data (core series)
- **BLS Basic** — Bureau of Labor Statistics (headline indicators)
- **Custom CSV upload** for your own datasets

Additional connectors — including Census detailed tables, international sources (World Bank, IMF, OECD), and domain-specific feeds — are available through Professional and Enterprise tiers. See [krlabs.dev](https://krlabs.dev/subscriptions/) for the full connector inventory.

The Community connectors handle rate limiting, caching, and error recovery so you can focus on the analysis. Notebooks that reference data beyond Community-tier access will note this in their requirements section.

---

## Methodological Standards

We follow the methods literature closely and cite it explicitly. Notebook implementations reference:

- Imbens & Lemieux (2008) on regression discontinuity
- Abadie, Diamond & Hainmueller (2010, 2015) on synthetic control
- Cinelli & Hazlett (2020) on sensitivity analysis for unobserved confounding
- Cattaneo, Idrobo & Titiunik (2020) on RDD inference
- OMB Circular A-4 on cost-benefit methodology

When a notebook uses simulated treatment assignment on real data, it says so. Line one. Not buried in a footnote. The data provenance badge system exists because we've seen too many analyses where simulated results get treated as empirical findings. That practice ends here.

---

## The KRL Suite

Notebooks use components from the KRL package ecosystem:

| Package | Purpose | Availability |
|---------|---------|-------------|
| `krl-open-core` | Logging, configuration, shared utilities | Public PyPI |
| `krl-data-connectors` | Authenticated access to federal statistical APIs | Public PyPI |
| `krl-causal-policy-toolkit` | Causal inference implementations (DiD, RDD, synthetic control, PSM, AIPW) | Platform access required |

`krl-open-core` and `krl-data-connectors` install via pip from public PyPI. The causal policy toolkit is available through the KRL platform and requires a Professional or Enterprise subscription. Notebooks that depend on it will note this in their setup cells — where possible, they include standalone implementations that run without platform access.

---

## Contributing

We want contributions that sharpen methodological rigor, expand data coverage, or improve documentation clarity. Bug reports, notebook suggestions, and method critiques all belong in GitHub Discussions or Issues.

**Discussion categories:**
- **Q&A** — Method questions, implementation help
- **Ideas** — Notebook proposals, feature requests
- **Show and Tell** — Your adaptations, derived analyses, results

Ground rules: be constructive, share context, give credit. If you build on someone's work, say so.

---

## Go Further

KASS notebooks and Community-tier connectors are free and always will be. If your work demands expanded data access, production-scale automation, or enterprise compliance infrastructure — the platform scales with you.

[![Subscribe to KRL Community](https://img.shields.io/badge/Community_Tier-Subscribe-0A2540?style=for-the-badge&logo=stripe&logoColor=white)](https://checkout.krlabs.dev/b/6oUdRa4Rp3lhbZv8939sk02)

Full tier comparison and Enterprise options at [krlabs.dev](https://krlabs.dev/subscriptions/).

**Not a Python user?** The KRL web platform provides the same analytical capabilities through a browser-based interface — no installation, no environment configuration. Details at [krlabs.dev](https://krlabs.dev).

---

## What This Is Not

This is not a consulting firm's website. We don't claim to solve every analytics problem. KASS notebooks cover a specific and well-defined set of quasi-experimental methods for policy evaluation. They require statistical literacy to interpret correctly. The limitation tables help—but they don't replace judgment.

KRL's platform capabilities extend beyond what's in this open-source repository. If your organization needs production-scale automation, enterprise data access, or compliance infrastructure, visit [krlabs.dev](https://krlabs.dev) for details on Professional and Enterprise tiers.

**Current status:** Beta. We ship verified capabilities and document what isn't ready yet. If that level of transparency is unusual in your experience with analytics vendors, we understand. It's deliberate.

---

## Connect

- **Platform:** [krlabs.dev](https://krlabs.dev)
- **GitHub:** [@KhipuResearch](https://github.com/KhipuResearch)
- **Email:** info@krlabs.dev
- **Support:** support@krlabs.dev

---

<div align="center">

**Evidence infrastructure for policy analysis.**

*Rigorous methods. Real data. Honest limitations.*

</div>