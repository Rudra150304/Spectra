# SPECTRA — Full Project Overview

## What SPECTRA Is

SPECTRA is a multimodal assistive AI system designed primarily to help visually impaired users understand image content through structured language and speech output.

Core idea:

```text
Image → Vision Model → Structured Metadata → Language Model → TTS → Spoken Output
```

Instead of making a vision model directly generate captions, the system separates perception from reasoning.

* Vision model detects what is present.
* Language model decides what matters and how to describe it.
* TTS speaks the result naturally.

This modular design improves control, explainability, and future extensibility.

---

# Main Components

## 1. SPECTRA-VLM

Current direction: ViT-based (Vision Transformer based vision module)

Purpose:
Analyze uploaded images and extract structured approximate metadata.

Expected outputs include:

### Scene-level context

* indoor / outdoor
* room / road / mountain / river / forest / office etc.
* lighting (day / night / low light)

### Large objects / structures

* bed
* road
* building
* tree
* table
* mountain
* sofa

### Mid-size objects

* person
* car
* dog
* chair
* laptop
* bicycle

### Fine details (optional if detectable)

* keys
* charger
* glasses
* bag
* rims
* phone

### Approximate attributes

* color
* size
* position
* relative position
* confidence score

Example output:

```json
{
  "scene": "outdoor",
  "lighting": "day",
  "objects": [
    {
      "id": 1,
      "type": "person",
      "size": "medium",
      "color": "dark",
      "pos": "center-left"
    },
    {
      "id": 2,
      "type": "car",
      "size": "large",
      "color": "white",
      "pos": "right"
    }
  ],
  "relations": [
    { "from": 1, "to": 2, "type": "near" }
  ]
}
```

Important principle:

The VLM does NOT narrate.
It only perceives and structures observations.

---

## 2. SPECTRA-LLM

Custom decoder-only Transformer language model.

Planned architecture (GPT-2 Small inspired):

* ~124M parameters
* 12 layers
* 12 attention heads
* hidden size 768
* FFN size 3072
* context length 1024
* Pre-LayerNorm
* GELU activation
* Absolute positional embeddings
* Weight tying
* PyTorch implementation

Purpose:

Convert structured visual metadata + user intent into fluent natural language.

Example:

Input:

```text
<VISION>
SCENE outdoor
OBJ person medium center-left
OBJ car large right
REL person near car
</VISION>

<INTENT>
Describe the scene for a visually impaired user.
Focus on obstacles and important nearby objects.
</INTENT>
```

Output:

```text
A person is standing slightly to your left near a large white car on the right side of the scene. The area appears to be outdoors in daylight.
```

Capabilities intended:

* fluent generation
* controllable tone
* concise or detailed modes
* humorous / dramatic / neutral style
* prioritize hazards / nearby objects
* user-adaptive narration

Important principle:

The LLM reasons and narrates.

---

## 3. TTS Layer

Speech synthesis layer.

Plan:
Use external API or offline model (Hugging Face / Ollama / open-source TTS).

No custom TTS training planned.

Purpose:
Convert generated description into spoken audio.

---

# Why This Architecture

Instead of:

```text
Image → Caption directly
```

SPECTRA uses:

```text
Image → Structured facts → Reasoned description
```

Advantages:

* more controllable outputs
* easier debugging
* modular upgrades
* can swap vision model later
* can add OCR / audio / depth later
* better user-specific narration

---

# Development Strategy

## Phase 1 — Build SPECTRA-LLM First

Train standalone language model first.

Goals:

* fluency
* clean responses
* tone following
* intent handling
* stable generation

No vision conditioning initially.

---

## Phase 2 — Build SPECTRA-VLM

Use ViT-based vision model to produce structured metadata.

Evaluate real output quality:

* what it detects reliably
* what confidence scores mean
* relation accuracy
* attribute quality

---

## Phase 3 — Freeze Metadata Contract

After seeing real VLM outputs, finalize structured schema.

Then convert metadata into tokenized prefixes for LLM.

---

## Phase 4 — Fine-Tune LLM

Train:

```text
[VISION TOKENS] + [INTENT] → DESCRIPTION
```

This aligns the language model to visual metadata.

---

## Phase 5 — Deploy

Targets:

### Web App

Upload image → choose narration mode → receive text/audio

### Mobile App

Same pipeline, later possibly partial on-device inference.

---

# Tech Stack

## Core ML

* Python
* PyTorch
* NumPy
* Pandas
* Hugging Face Transformers
* Hugging Face Datasets

## Tokenization

* GPT-2 tokenizer initially

## Vision

* ViT-based architecture

## Deployment

* FastAPI backend likely
* React / Next.js frontend possible
* ONNX export planned
* Torch export / inference optimization planned

## Compute

* Local laptop GPU: RTX 3050 4GB
* Kaggle free GPUs
* Google Colab free GPUs
* Optional paid GPU credits if needed

---

# Design Philosophy

This is not meant to be just a notebook model.

Goals include learning:

* transformer internals
* training custom LLMs
* multimodal architecture design
* model export (ONNX)
* deployment to web/mobile
* practical ML engineering

---

# Current Priority

Immediate focus is SPECTRA-LLM.

The VLM is already conceptually decided (ViT-based), but not the active coding priority yet.

---

# One-Line Summary

SPECTRA is a modular multimodal AI assistant that converts images into structured perception, transforms that into intelligent narration, and delivers it through speech.

Spectra is currentlt in the production phase with a custom 124m LLM and a ViT training in progress.
