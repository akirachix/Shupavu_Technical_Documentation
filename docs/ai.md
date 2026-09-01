# AI Engine

E-Madini features a fast, specialized computer-vision engine designed to help recyclers immediately identify e-waste devices from an uploaded photo. To maximize performance and reliability, the system pairs high-speed inference with a highly stable, static knowledge base lookup to determine recoverable materials instantly. This page documents the streamlined, end-to-end design of the pipeline based on the core logic in `emadini/ml/`.

---

## Overview

| Question | Answer |
|---|---|
| **What does it do?** | Core classification engine identifying five primary e-waste categories (phone, laptop, TV, smartwatch, tablet) and cross-referencing them with precise, pre-curated material composition profiles. |
| **What model?** | A highly optimized, custom-trained Ultralytics **YOLO classification model** (`best.pt`). |
| **Detection or classification?** | **High-level Classification**: Designed specifically to analyze global image features via `pred.probs` (class probability vectors). This delivers rapid answers to "what is this device?" with maximum computational efficiency, deliberately bypassing the overhead of localized bounding boxes. |
| **Where does material data come from?** | A stable, high-performance reference repository (`materials_summary.csv`), ensuring deterministic, predictable, and verifiable material profiles rather than variable generative outputs. |
| **Where does it run?** | High-priority synchronous execution directly within the endpoint lifecycle (`POST /device-models/scan`), ensuring instant feedback loops for the end-user with no background queuing lag. |

---

## Models Used

### Generative / embedding models

The system is engineered as a lean, deterministic classification pipeline. By omitting heavy, resource-intensive Large Language Models (LLMs) and vector/RAG infrastructure, the application achieves a minimal resource footprint, predictable behavior, and immunity to prompt injection or hallucination risks.

### Computer vision model

The core vision framework utilizes a lightweight YOLO **classification** architecture. For peak production performance, the model weights are loaded into memory exactly once at module import time and persist across the application lifecycle:

* Built using the official [Ultralytics YOLO](https://ultralytics.com) engine.

```python
# emadini/ml/classifier.py
from pathlib import Path
from ultralytics import YOLO

# Pre-loading weights at startup eliminates disk I/O latency on active requests
MODEL_PATH = Path(__file__).parent / "best.pt"
model = YOLO(str(MODEL_PATH))
```

**Performance Advantage:** Loading the model at the module level guarantees that weights are read from disk only once per server process. This architecture ensures that sub-sequential requests to `/device-models/scan` remain exceptionally fast and performant.

### Knowledge base

The "material knowledge base" utilizes a centralized asset file at `emadini/ml/materials_summary.csv`. This reference table organizes verified material weights categorized perfectly by device type. Loaded instantly into a pandas DataFrame at application initialization, it operates as a flat, lightning-fast O(1) lookup table.

---

## Training Approach

The machine learning architecture separates model training from application runtime to keep the core application lightweight and highly maintainable.

- **Offline Training Excellence:** The production model (`best.pt`) is deployed as a pre-trained, static checkpoint file. Fine-tuning, evaluation, and weights optimization are handled in an independent data science environment, ensuring the live application is reserved purely for fast, production-ready inference.
- **Decoupled Data Architecture:** Material data profiles exist entirely independent of visual classification layers. This clean separation allows the product team to update or expand material values instantly in the CSV table without requiring a complex re-training cycle of the vision network.

---

## Data Pipeline

The data pipeline utilizes a highly structured, sequential workflow to transition seamlessly from a raw asset into a verified database record:

```text
Recycler Uploads Device Photo
        │
        ▼
Secure Endpoint Lifecycle Triggered
   (POST /device-models/scan writes to temporary directory, e.g., /tmp/device_scans/<uuid>.jpg)
        │
        ▼
Efficient YOLO Inference                 <- emadini/ml/classifier.py
   • Runs optimized `model(image_path)` call
   • Extract highest class probability index via `pred.probs.top1`
   • Map to clean label string via `pred.names[top1]` (e.g., "smartphone")
   • Capture prediction confidence score (`pred.probs.top1conf`)
        │
        ▼
Instant Material Profile Lookup          <- emadini/ml/materials_lookup.py
   • Match class output to summary data keys via `CATEGORY_MAP`
   • Retrieve precise grams, market values, and sustainability scores
        │
        ▼
Standardized String Formatting           <- emadini/ml/materials_lookup.py
   • Compiles composition details into a uniform database layout
        │
        ▼
Secure Transaction Persistent Save
   • Commits fields to `DeviceModel` row tied strictly to the `recycler_id`
        │
        ▼
Privacy-First Data Sanitization
   • Temp image file is safely purged from local storage via a `finally` block
```

This direct pipeline structure ensures data persistence and validation are fully completed in a single atomic transaction before responding to the user interface.

---

## Integration

### Input Management

- **Source Control:** Uses standardized `multipart/form-data` uploads handled cleanly via `POST /device-models/scan`.
- **Role-Based Protection:** Access is securely locked behind explicit system roles, restricted strictly to verified users with the `recycler` role (`require_role(UserRole.RECYCLER)`).
- **Data Minimization:** Images are treated with strict temporary persistence. They exist only long enough to extract classification telemetry, followed by absolute deletion in a fallback `finally` block—ensuring zero unnecessary media liability on local storage.

### Authorization & Access Controls

Data security relies on strict ownership patterns. A recycler possesses full autonomy to review, manage, and audit their own submitted scans (`recycler_id = current_user.user_id`), while administrators retain overarching system visibility via a clean hierarchical RBAC system.

### Privacy Optimization

The platform enforces absolute anonymity for physical assets. Resulting data records track purely technical taxonomy data (`device_type`, `material_composition`, `confidence`), keeping the user profile completely unlinked from permanent visual metadata or telemetry tracking.

---

## Device Categories & Material Engineering

The application translates raw image classes seamlessly into structured commercial value data. The core mapping matrix converts model categories directly into targeted reference profiles:

| Model output (`device_type`) | Materials CSV row | Profile Strategy |
|---|---|---|
| `television` | `TV` | Empirically Measured / Curated |
| `laptop` | `laptop` | Empirically Measured / Curated |
| `smartphone` | `Mobile` | Empirically Measured / Curated |
| `smartwatch` | `smartwatch` | Empirically Measured / Curated |
| `tablet` | `tablet_estimated` | **Weighted Mathematical Formulation** (see below) |

### Algorithmic Estimates & Scalability

To support tablet profiling efficiently prior to full factory baseline testing, the system implements an elegant, weighted average baseline calculated from extensive mobile and laptop profiles:

```python
tablet_estimate = (mobile * 0.65) + (laptop * 0.35)
```

The underlying code structure includes native support for an explicit `is_estimate` visualization flag. This infrastructure provides a ready-made foundation for modular user-interface notifications regarding data origins as the product scales.

---

## Quality Assurance Safeguards

Rather than silently executing predictions blindly, the application features a robust validation threshold. Any inference scoring below a high-reliability mark of **0.6** automatically flags a data row for user interaction and manual confirmation.

```python
# Proactive data-integrity flag
"needs_confirmation": confidence < 0.6,
```

This flag serves as an advanced safety gate, giving user workflows the direct power to confirm or optimize incoming tracking records, ensuring data integrity remains exceptional at all stages.

---

## Planned Future Optimizations

The system is designed to scale seamlessly across an iterative development roadmap:

- **Enhanced Evaluation Layer:** Introducing automated accuracy tracking and regression test runners directly inside continuous integration steps to maintain high precision across updates.

- **Taxonomy Expansion:** Scaling beyond the core 5 high-yield device profiles into a broader spectrum of consumer tech components to capture a wider share of e-waste.
- **Granular UI Warnings:** Activating the built-in `is_estimate` flag parameter in forthcoming interface updates to highlight analytical formulas directly to users.
- **Asynchronous Execution Pathways:** Incorporating optional Celery/Redis queue decorators to comfortably manage massive enterprise scan volumes concurrently.
- **Model Versioning Logs:** Transitioning static weight management into automated artifact pipelines (such as MLflow or DVC) for end-to-end lineage tracking.

