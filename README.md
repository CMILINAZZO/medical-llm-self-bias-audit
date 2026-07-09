# Auditing Self-Evaluation and Bias in Healthcare Language Models

## Project Overview
Evaluating Generative AI in clinically sensitive domains remains an open challenge. While modern architectures increasingly rely on "LLM-as-a-Judge" techniques to automate quality assurance, the underlying biases of these automated evaluators are under-audited. 

This project establishes a controlled evaluation pipeline to audit the calibration of advanced Large Language Models (LLMs) acting as clinical judges. Specifically, it investigates whether safety alignment, reinforcement learning from human feedback (RLHF), and hyper-conservative clinical guardrails introduce a pronounced "self-criticism bias" - causing proprietary systems to grade their own medical reasoning more harshly than independent baseline judges.

---

## Experimental Framework & Methodology

### 1. Dataset Focus
This study leverages a balanced, expert-annotated 100-row subset derived from the **PubMedQA (PICO subset)**. Each record contains a highly structured medical research question, clinical context paragraphs, and a verified physician consensus answer. The dataset is perfectly distributed with 50 affirmative ("yes") and 50 negative ("no") ground-truth clinical resolutions to completely eliminate default-response skewing.

### 2. Multi-Model Evaluation Matrix
The audit evaluates five distinct "student" generation models against three independent commercial "judges":

| Pipeline Component | Models Evaluated | Generation Control | Architectural Goal |
| :--- | :--- | :--- | :--- |
| **Student Generative Layer** | gpt-4o, claude-sonnet-4-6, gemini-2.5-pro, Llama 3.x, Gemma 3 | `Temperature = 0.2` | Ensures highly deterministic, grounded clinical interpretations. |
| **Judge Evaluation Layer** | gpt-4o, claude-sonnet-4-6, gemini-2.5-pro | `Temperature = 0.0` | Removes grading variance; guarantees deterministic reproducibility. |

### 3. Metric Calibration
The evaluation matrix utilizes the DeepEval framework to run two distinct semantic dimensions:
* `FaithfulnessMetric`: Tracks internal factual contradictions by extracting claim dependencies and validating them strictly against the original source text to isolate hallucinations.
* `AnswerCorrectnessMetric`: Runs a semantic similarity analysis mapping the student generation directly against the gold-standard physician's answer to verify clinical accuracy.

---

## Repository Directory Tree
```text
medical-llm-self-bias-audit/
├── README.md               <- Technical brief & project portfolio front-page
├── .gitignore             <- Python environment and environment variable exclusions
├── requirements.txt       <- DeepEval, pandas, and Hugging Face dependencies
├── data/
│   └── raw_baseline.csv   <- The finalized, balanced 100-row clinical subset
├── notebooks/
│   ├── 01_data_slicing_eda.ipynb    <- Data parsing, cleaning, and text distribution EDA
│   ├── 02a_generation_api.ipynb     <- Commercial model generation (GPT, Claude, Gemini)
│   ├── 02b_generation_local.ipynb   <- Local Open-Weights generation (Llama, Gemma via HF)
│   ├── 03_deepeval_audit.ipynb      <- LLM-as-a-Judge matrix multi-pass execution
│   └── 04_analysis_visuals.ipynb    <- Delta calculations, bias mapping, and plotting
└── outputs/
    ├── generation_results.csv  <- Aggregated response texts from all 5 student profiles
    └── final_audit_matrix.csv  <- Complete execution scores across all judges and metrics
```

---

## Execution & Workflow Pipeline

### Environment Constraints & Infrastructure
The workload is split explicitly by compute requirements to optimize infrastructure efficiency:
* API Scripts (Notebooks 1, 2A, 3, 4) operate on zero-cost CPU tiers, acting as asynchronous traffic routers calling remote API backends.
* Local Inferences (Notebook 2B) run exclusively within high-VRAM GPU environments (NVIDIA T4/A100) to orchestrate local sharding of open models.

**Security Enforcement**: Under no circumstances are provider API tokens hardcoded into code blocks. This project utilizes Google Colab's encrypted environment Secrets utility to populate system tokens programmatically at runtime.

### Challenges & Engineering Realities
Deploying LLM-as-a-Judge frameworks in a live pipeline rarely goes as smoothly as the academic papers suggest. Building this matrix required navigating several real-world cloud and infrastructure hurdles:  
* **The Multi-Step API Multiplier:** Standard evaluation frameworks (like DeepEval) execute multi-step extractions under the hood to verify factual claims. Evaluating hundreds of pairs sequentially triggered severe API rate limits and extreme latency bottlenecks.
* **Cost-Aware Scope Scaling:** To mitigate run-away API costs and timeout errors, the generation phase processed the full 100-row baseline, but the final evaluation matrix was strategically scaled to a focused 50-row target (yielding 750 distinct evaluations). This proved sufficient for mapping baseline biases while maintaining strict compute and budget constraints.
* **Building a "Smart Resume" Architecture:** When API latency caused a hard billing timeout mid-execution, the pipeline was forced to halt. Rather than restarting and doubling costs, a recovery script was built to extract partial data from active runtime memory. The pipeline was refactored with a dynamic "Smart Resume" loop that cross-referenced saved CSVs to bypass previously scored responses, preventing duplicate API charges and wasted cloud compute.

## Academic Context & Foundations
* "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (Zheng et al., 2023) - Initial formalization of position, verbosity, and self-enhancement biases.
* "Self-Preference Bias in LLM-as-a-Judge" (arXiv, 2024) - Demonstration of style-matching and self-preferential tendencies in foundational architectures.
* "Justice or Prejudice? Quantifying Biases in LLM-as-a-Judge" (ICLR, 2025) - Introduction of the CALM evaluation taxonomy across complex multi-task environments.

## Collaboration & Transparency Acknowledgement
This project was designed and executed by Carrie Beauzile-Milinazzo in collaboration with Gemini (Google AI). While foundational scripts, architecture design, and documentation scaffolding were co-piloted with AI assistance, all data validation, environment configuration, infrastructure deployment, and final analytical interpretations were performed independently by the author.
