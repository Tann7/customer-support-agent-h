#  AppleSupport AI Support Agent

> An AI-powered customer support agent designed to classify Apple customer issues, identify safety and escalation risks, retrieve relevant historical resolutions, and generate grounded support responses.

**Hiver SDE Intern Take-Home Assignment**

---

##  Overview

This project implements an end-to-end AI customer support pipeline for **AppleSupport Twitter/X conversations**.

The system takes a customer's support message and processes it through three major stages:

1. **Intent Classification** — Identifies the customer's primary support category and confidence.
2. **Risk-Aware Policy & Escalation** — Determines whether the issue can be handled automatically or should be escalated to a human.
3. **Grounded Response Generation** — Retrieves similar historical AppleSupport resolutions and generates a concise, grounded response.

The system is designed with a strong emphasis on **safety, explainability, low latency, and minimizing dangerous false auto-handling of customer issues**.

---

##  Key Features

* Hybrid intent classification
* Risk-aware escalation and safety rules
* TF-IDF-based historical knowledge retrieval
* Grounded customer response generation
* Automated evaluation against labeled benchmark data
* LLM-as-a-Judge evaluation framework
* Human-judge calibration and agreement analysis
* Unit and integration tests
* Lightweight, local-first implementation
* CLI demo, single-query mode, and interactive mode
* Structured Pydantic output models

---

## System Architecture

```text
                 Customer Tweet
                       │
                       ▼
        ┌──────────────────────────┐
        │   Intent Classification  │
        │                          │
        │  • 6 support intents     │
        │  • Confidence scoring    │
        └────────────┬─────────────┘
                     │
                     ▼
        ┌──────────────────────────┐
        │ Policy & Risk Engine     │
        │                          │
        │ • Safety rules           │
        │ • PII/account risks      │
        │ • Billing risks          │
        │ • Hardware hazards       │
        │ • Confidence threshold   │
        └────────────┬─────────────┘
                     │
             ┌───────┴────────┐
             │                │
        Auto Handle       Human Escalation
             │                │
             └───────┬────────┘
                     ▼
        ┌──────────────────────────┐
        │ Historical Retrieval      │
        │                          │
        │ TF-IDF + Top-K Retrieval │
        └────────────┬─────────────┘
                     │
                     ▼
        ┌──────────────────────────┐
        │ Grounded Response        │
        │ Generator                │
        └────────────┬─────────────┘
                     │
                     ▼
              Structured Output
```

---

## Intent Taxonomy

The classifier supports six major customer-support intents:

| Intent                         | Description                                                      |
| ------------------------------ | ---------------------------------------------------------------- |
| `software_os_issue`            | iOS/macOS bugs, crashes, update failures, UI issues              |
| `battery_power_hardware`       | Battery, charging, overheating, and hardware problems            |
| `connectivity_sync`            | Wi-Fi, Bluetooth, AirDrop, iCloud, cellular, CarPlay             |
| `account_billing_subscription` | Apple ID, subscriptions, refunds, payments and billing           |
| `device_setup_how_to`          | Setup, migration, backups, configuration and how-to questions    |
| `complaint_feedback`           | General complaints, dissatisfaction and product/service feedback |

---

## Safety & Escalation

A major focus of the project is preventing unsafe automatic responses.

The policy engine evaluates signals such as:

* Battery swelling, smoke, sparks, overheating and hardware hazards
* Apple ID compromise and account-security issues
* Unauthorized charges and billing disputes
* Legal threats and sensitive complaints
* Complex or multi-symptom failures
* Low-confidence intent predictions

When the system detects sufficient risk, it returns:

```text
escalate_to_human
```

along with a structured category and explanation.

The system also uses **confidence gating**, where sufficiently uncertain predictions can be routed to human support instead of being automatically handled.

---

## Historical Knowledge Retrieval

The response generator uses a local historical AppleSupport knowledge base containing **2,500 historical support resolution pairs**.

The retrieval pipeline:

1. Converts the customer query into a TF-IDF representation.
2. Searches the historical knowledge base.
3. Retrieves the top-3 relevant resolutions.
4. Uses the retrieved context to generate a grounded response.

This approach helps reduce unsupported responses and keeps generated answers connected to previously observed support resolutions.

---

## Evaluation

The project includes a dedicated evaluation framework with:

* Intent accuracy
* Macro F1
* Escalation accuracy
* Escalation precision and recall
* False Auto-Handle Rate (FAHR)
* ROUGE-L
* BLEU-2
* LLM-as-a-Judge scoring
* Human-judge agreement analysis
* Latency measurements

The benchmark uses a **200-example golden evaluation set** covering the supported intent categories.

A separate **50-example human benchmark** is used to evaluate the reliability of the automated judge.

### Headline Results

| Metric                 | Proposed Agent |
| ---------------------- | -------------: |
| Intent Accuracy        |      **94.5%** |
| Intent Macro F1        |      **0.951** |
| Escalation Accuracy    |      **72.5%** |
| Escalation Recall      |      **87.5%** |
| False Auto-Handle Rate |      **12.5%** |
| ROUGE-L F1             |      **0.210** |
| BLEU-2                 |      **0.118** |
| Composite Judge Score  |   **4.07 / 5** |
| Average Latency        |     **2.9 ms** |

> See [`REPORT.md`](REPORT.md) for the complete benchmark analysis, failure analysis, and discussion of limitations.

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd hiver
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

Or install the package using:

```bash
pip install -e .
```

### 3. Run the Demo

```bash
python scripts/run_pipeline.py --demo
```

### 4. Test a Custom Customer Query

```bash
python scripts/run_pipeline.py \
  --query "My battery is swollen and smells like smoke!"
```

Example output:

```text
Customer Tweet : "My battery is swollen and smells like smoke!"
Intent         : battery_power_hardware
Decision       : ESCALATE_TO_HUMAN
Reason         : Hardware safety hazard detected
Reply          : ...
Latency        : 3.0 ms
```

### 5. Start Interactive Mode

```bash
python scripts/run_pipeline.py --interactive
```

### 6. Run the Evaluation

```bash
python scripts/evaluate_all.py
```

### 7. Verify Automated Judge Agreement

```bash
python scripts/verify_judge.py
```

### 8. Run Tests

```bash
pytest tests/ -v
```

---

## 📁 Project Structure

```text
hiver/
│
├── README.md
├── REPORT.md
├── DECISION_LOG.md
├── pyproject.toml
├── requirements.txt
│
├── data/
│   ├── historical_knowledge_base.jsonl
│   ├── golden_eval_set.json
│   ├── human_judge_benchmark.json
│   ├── benchmark_results.json
│   ├── judge_calibration_results.json
│   ├── build_golden_eval_set.py
│   ├── sample_raw_brand_data.py
│   └── sampling_note.md
│
├── src/
│   ├── agent.py
│   ├── config.py
│   ├── models.py
│   │
│   ├── intent/
│   │   ├── taxonomy.py
│   │   ├── baseline_classifier.py
│   │   └── llm_classifier.py
│   │
│   ├── policy/
│   │   ├── decision_engine.py
│   │   └── escalation_rules.py
│   │
│   ├── response/
│   │   ├── retrieval.py
│   │   ├── grounded_generator.py
│   │   └── baseline_responder.py
│   │
│   └── evaluation/
│       ├── metrics.py
│       ├── agreement.py
│       └── llm_judge.py
│
├── scripts/
│   ├── run_pipeline.py
│   ├── evaluate_all.py
│   └── verify_judge.py
│
└── tests/
    ├── test_agent.py
    ├── test_evaluation.py
    ├── test_intent.py
    ├── test_policy.py
    └── test_retrieval.py
```

---

## Core Components

### `AppleSupportAgent`

The main orchestrator responsible for connecting the complete pipeline:

```python
agent = AppleSupportAgent()

result = agent.process(
    "My iPhone battery is swelling and getting extremely hot."
)
```

The resulting structured object contains:

```text
intent
intent_confidence
retrieved_contexts
draft_reply
escalation_decision
escalation_category
escalation_reason
latency_ms
```

### Intent Classifier

Responsible for determining the customer's support intent and confidence score.

### Risk-Aware Policy Engine

Applies explicit safety and escalation rules before automatic response handling.

### Historical Retriever

Retrieves similar historical AppleSupport resolutions using TF-IDF similarity.

### Grounded Response Generator

Creates concise support responses based on the customer query and retrieved historical context.

### Evaluation Framework

Provides reproducible benchmarking and comparison against baseline approaches.

---

## Testing

The project includes unit and integration tests covering:

* Intent classification
* Policy decisions
* Escalation rules
* Historical retrieval
* End-to-end agent behavior
* Evaluation metrics

Run the complete suite with:

```bash
pytest tests/ -v
```

---

## Documentation

Additional project documentation is available in:

* [`REPORT.md`](REPORT.md) — Detailed technical report, benchmark analysis, failure analysis, limitations, and future roadmap.
* [`DECISION_LOG.md`](DECISION_LOG.md) — Engineering and product decisions with their rationale.
* [`data/sampling_note.md`](data/sampling_note.md) — Dataset sampling and evaluation-set methodology.

---

## Future Improvements

Potential next steps include:

* Multi-turn conversation and thread tracking
* Better handling of compound intents
* Dense retrieval and cross-encoder reranking
* Improved uncertainty calibration
* More sophisticated safety policies
* Production Hiver shared-inbox integration
* Continuous evaluation with newly collected support conversations
* Monitoring and feedback loops for production deployments

---

## Limitations

This project is a take-home assignment / prototype rather than a production deployment.

In particular:

* The knowledge base is a fixed historical snapshot.
* The system does not directly integrate with Apple's internal support systems.
* Responses are evaluated against a curated benchmark rather than live customer traffic.
* Some ambiguous or multi-intent customer messages can still be difficult to classify.
* Historical support data can contain noisy or inconsistent examples.

For a detailed discussion, see [`REPORT.md`](REPORT.md).

---

## Author

**SDE Intern Take-Home Assignment — Hiver**

Built as an end-to-end demonstration of:

**NLP → Retrieval → Policy/Safety → Response Generation → Evaluation**

---

## License

This repository is intended for educational and assessment purposes.
