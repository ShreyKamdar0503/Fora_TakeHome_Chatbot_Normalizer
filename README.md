# Fora Chatbot Normalizer Service

A FastAPI-based microservice that classifies, extracts, and enriches advisor messages before they reach the main chatbot.
Implements the **AI Innovator** take-home challenge requirements.

---

## Features

* **Classification**: `urgent`, `high_risk`, `base`

  * `high_risk` (e.g., lost passport, medical emergency) overrides `urgent`
* **Contact extraction**: First/last name, phone, email, ZIP code
* **Entity extraction**: Cities, hotels, restaurants
* **Deterministic enrichment**:

  * `local_emergency_numbers` from [emergencynumberapi.com](https://emergencynumberapi.com)
  * `creative_language`: official language of the country for any extracted cities
* **Partial output**: Omits fields if data is unavailable
* **Comprehensive tests** with `pytest`

---

## Requirements

* Python 3.10+
* `pip`
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

---

## Setup

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>/normalize_service

# 2. Create a virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

---

## Run locally

```bash
uvicorn app:app --reload --port 8000
```

Local server will be available at:

```
http://localhost:8000
```

---

## Public Endpoint (ngrok)

This instance is currently accessible at:

```
https://605ecb13da58.ngrok-free.app/normalize
```

---

## Usage

### Example 1 — High Risk

```bash
curl -s -X POST https://605ecb13da58.ngrok-free.app/normalize \
  -H "Content-Type: application/json" \
  -d '{"message_id":"88e998","text":"Hi Fora, I'\''m Shrey Kamdar (516-322-0440) in 11233. I have misplaced my passport in Brooklyn, Please Help!"}' | jq
```

**Expected JSON Output:**

```json
{
  "message_id": "88e998",
  "category": "high_risk",
  "contact": {
    "first_name": "Shrey",
    "last_name": "Kamdar",
    "phone": "5163220440",
    "zip": "11233"
  },
  "entities": [
    { "type": "city", "value": "new york city" }
  ],
  "enrichment": {
    "local_emergency_numbers": ["911"],
    "creative_language": "English"
  }
}
```

---

### Example 2 — Urgent

```bash
curl -s -X POST https://605ecb13da58.ngrok-free.app/normalize \
  -H "Content-Type: application/json" \
  -d '{"message_id":"test_urgent","text":"My flight is in 3 hours, can you confirm my booking?"}' | jq
```

---

### Example 3 — Base

```bash
curl -s -X POST https://605ecb13da58.ngrok-free.app/normalize \
  -H "Content-Type: application/json" \
  -d '{"message_id":"test_base","text":"Planning Rome in October with a stay at Chapter Roma and maybe NYC"}' | jq
```

---

### Example 4 — Hybrid (High Risk + Urgent)

```bash
curl -s -X POST https://605ecb13da58.ngrok-free.app/normalize \
  -H "Content-Type: application/json" \
  -d '{"message_id":"test_hybrid","text":"I lost my passport and my flight is in 2 hours, please help immediately!"}' | jq
```

**Note:** `high_risk` takes precedence over `urgent`.

---

## Tests

```bash
pytest -q
```

---

## Deployment

To deploy on **Render**:

1. Push the repo to GitHub.
2. Create a new **Web Service** in Render.
3. Environment: Python 3.x
   Start Command:

   ```bash
   uvicorn app:app --host 0.0.0.0 --port $PORT
   ```
4. Deploy — you’ll get a public HTTPS endpoint.

---

## Project Structure

```
normalize_service/
  app.py               # FastAPI app + /normalize endpoint
  classify.py          # Classification rules
  extract.py           # Contact + entity extraction
  enrich.py            # Enrichment logic
  schemas.py           # Request/response models
  data/cities.csv      # City-country mapping
  lib/                 # Utility modules
  tests/               # pytest tests
  requirements.txt
  Dockerfile
```

---

# Fora_TakeHome_Chatbot_Normalizer
