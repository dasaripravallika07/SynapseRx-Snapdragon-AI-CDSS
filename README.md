# SynapseRx

> Real-time, on-device medication safety checks and drug interaction analysis built for Snapdragon®-powered HP laptops using Qualcomm® AI Hub.

[![Platform](https://img.shields.io/badge/Platform-Snapdragon%20X%20Elite-0052CC?style=flat-square)](https://www.qualcomm.com/products/mobile-processors/snapdragon-x-elite)
[![Acceleration](https://img.shields.io/badge/Hardware-Hexagon%20NPU%20(45%20TOPS)-0096D6?style=flat-square)](#)
[![Framework](https://img.shields.io/badge/AI%20Runtime-Qualcomm%20AI%20Hub%20%7C%20ONNX%20QNN-FF5722?style=flat-square)](https://aihub.qualcomm.com/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## Why We Built This

Doctors and clinical pharmacists are overwhelmed by alert fatigue. Current hospital EHR systems throw flashing pop-up warnings for almost every drug combination. Because over 85% of these alerts are generic or clinically irrelevant to the specific patient, doctors reflexively click "Ignore." When a genuinely fatal interaction comes along, it slips right past.

Cloud-based AI solutions create a separate problem: you cannot legally or safely pipe private patient records (PHI) across public internet APIs under HIPAA and data privacy laws. Furthermore, hospital networks frequently drop connection in thick-walled wards or basement ICUs.

**SynapseRx** solves both problems by running entirely on the edge. Designed for the **HP OmniBook series powered by Snapdragon X Elite**, it evaluates multi-drug prescriptions locally against patient lab work in under 150 milliseconds. It doesn't just flag an alert—it explains the biological mechanism and suggests safe alternatives available in the hospital formulary.

---

## The Core Concept: Zero-Hallucination Hybrid Safety

Large language models cannot be trusted to guess drug interactions from memory—they can hallucinate dosages. Traditional rule engines cannot write human-friendly clinical summaries. SynapseRx pairs both:

1. **Deterministic Ground Truth (Knowledge Graph):** We map drugs to standardized **RxNorm** identifiers and cross-check known cytochrome P450 (CYP450) metabolic pathways, contraindications, and active chemical duplicates through local clinical databases.
2. **Contextual Lab Filtering:** Before raising an alarm, the engine checks the patient's dynamic labs (such as eGFR for kidney function or AST/ALT for liver function). If a minor theoretical interaction exists but the patient's vitals and dosage are well within tolerance, the alert is silenced.
3. **On-Device LLM Synthesis (Hexagon NPU):** When a genuine conflict occurs, verified graph nodes are fed into a quantized model compiled from **Qualcomm AI Hub** running on the 45 TOPS Hexagon NPU. The model formats an actionable, clear warning note with validated substitutes.

---

## Workflow
[ New Prescription + Patient Health Record (FHIR JSON) ]
│
▼
[ 1. Entity Extraction & Normalization ]
• Standardizes trade names to RxNorm Concept Unique IDs
• Maps chronic diagnoses to ICD-10 codes
│
▼
[ 2. Deterministic Knowledge Graph Check ]
• Queries local CYP enzyme pathways & contraindications
• Flags duplicate active ingredients and structural allergens
│
▼
[ 3. Patient Biomarker Evaluation ]
• Assesses organ clearance (e.g., eGFR < 30 mL/min)
• Filters low-confidence noise to avoid alert fatigue
│
▼
[ 4. On-Device Inference on Qualcomm Hexagon NPU ]
• Model: Quantized INT4 LLM via Qualcomm AI Hub
• Runtime: ONNX Runtime with QNN Execution Provider
│
▼
[ Actionable Clinical Alert Card (<150 ms) ]
• Biological mechanism explained in plain English
• Verified formulary-safe alternative suggestions
---

## 5 Specific Risks SynapseRx Catches

* **Drug-Drug Interactions (DDI):** Detects metabolic bottlenecks, such as pairing Warfarin with Fluconazole, which blocks CYP2C9 clearance and triggers severe hemorrhage risks.
* **Drug-Disease Conflicts:** Flags medications that worsen pre-existing conditions, such as giving non-selective beta-blockers (Propranolol) to an asthmatic patient.
* **Duplicate Therapy:** Catches accidental double-dosing across different brand names (e.g., prescribing Percocet alongside over-the-counter Tylenol, causing toxic acetaminophen buildup).
* **Organ-Specific Dosing Errors:** Recalculates safe dosage ceilings when recent lab reports show compromised kidney clearance (e.g., warning against standard Metformin doses when eGFR drops below 30).
* **Structural Cross-Allergies:** Evaluates shared chemical backbones (such as the beta-lactam ring between penicillins and early cephalosporins) instead of relying solely on exact name matches.

---

## Optimized for Snapdragon-Powered HP PCs

SynapseRx is tuned specifically for the **HP OmniBook (Snapdragon X Elite / X Plus)** hardware profile:

| Layer | Implementation Detail |
| :--- | :--- |
| **NPU Acceleration** | Offloads generative token processing and graph inference directly to the **Qualcomm Hexagon NPU (45 TOPS)**. |
| **Model Optimization** | Quantized **INT4/W4A16** models obtained and compiled through **Qualcomm AI Hub**. |
| **Inference Engine** | **ONNX Runtime** configured with the **Qualcomm AI Engine Direct (QNN) Execution Provider**. |
| **Privacy & Security** | Operates air-gapped with zero cloud calls. All medical data stays in local RAM. |
| **System Efficiency** | Near-zero CPU/GPU overhead leaves the laptop cool, silent, and capable of all-day battery life on clinical rounds. |

---

## Project Structure
SynapseRx/
├── data/
│   ├── sample_patients.json        # Mock clinical records (FHIR-compliant)
│   └── interaction_graph.json      # Structured CYP450 & contraindication graph
├── models/
│   └── qnn_config.json             # Qualcomm AI Hub model execution parameters
├── src/
│   ├── parser.py                   # Medical text extraction & RxNorm mapping
│   ├── graph_validator.py          # Deterministic interaction checking logic
│   ├── lab_evaluator.py            # Renal/hepatic biomarker context checks
│   └── npu_engine.py               # Local QNN-accelerated inference runner
├── app.py                          # Local interface and API endpoints
├── requirements.txt                # Python environment dependencies
└── README.md

## Getting Started

### Prerequisites
* A Snapdragon-powered Windows 11 on ARM device (e.g., HP OmniBook Ultra / OmniBook X).
* Python 3.11 installed for ARM64.
* Qualcomm AI Engine Direct SDK (QNN) installed and added to your system path.

### 1. Clone the Repository
```bash
git clone [https://github.com/dasaripravallika07/SynapseRx-Snapdragon-AI-CDSS.git](https://github.com/dasaripravallika07/SynapseRx-Snapdragon-AI-CDSS.git)
cd SynapseRx-Snapdragon-AI-CDSS

python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt

python app.py

{
  "alert_id": "SRX-4029",
  "status": "CRITICAL_INTERACTION",
  "latency_ms": 114,
  "hardware_target": "Qualcomm Hexagon NPU",
  "conflict": {
    "prescribed_medication": "Clarithromycin 500mg BID",
    "existing_medication": "Simvastatin 40mg daily",
    "conflict_type": "Metabolic Enzyme Inhibition (CYP3A4)"
  },
  "clinical_explanation": "Clarithromycin is a potent CYP3A4 inhibitor. Because Simvastatin relies exclusively on this pathway for hepatic clearance, co-administration can spike active statin plasma levels up to 10-fold, significantly elevating the risk of acute rhabdomyolysis.",
  "patient_specific_context": "Patient has Stage 3b Chronic Kidney Disease (baseline eGFR: 32 mL/min). Muscle breakdown byproducts present an immediate risk of acute-on-chronic renal shutdown.",
  "suggested_actions": [
    "Temporarily suspend Simvastatin for the duration of the antibiotic treatment.",
    "Alternative: Switch antibiotic therapy to Azithromycin (minimal CYP3A4 inhibition profile)."
  ]
}
