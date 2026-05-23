# Dockase Developer REST API & Python SDK Guide

Welcome to the **Dockase Developer REST API** repository. This guide provides a clean, lightweight documentation overview and a ready-to-use Python client wrapper to interact with the Dockase Legal Intelligence Gateway.

With this API, developers and law firms can integrate Nigerian and Commonwealth legal research, semantic vector search (RAG), senior-counsel-level contract risk audits, citation validation, and legal CRM/matter management directly into their own applications and internal scripts.

---

## Prerequisites

1. **Python 3.8+** installed.
2. A **Dockase Developer Account** and API key.
   * Sign up at the [Dockase Developer Portal](https://dockase.com/developer/docs/) to obtain your secret token (`dk_...`).

---

## Authentication

All API requests must carry your secret token in the HTTP Authorization header:

```http
Authorization: Bearer your_secret_api_key_here
Content-Type: application/json
```

---

## Setup & Quickstart

### 1. Clone & Install Dependencies
First, create your workspace and install the required dependencies (only `requests` is needed):

```bash
pip install -r requirements.txt
```

### 2. Quickstart Script (`client.py`)
Create a file named `client.py` and paste the following Python wrapper to begin querying the gateway:

```python
import os
import requests

class DockaseClient:
    def __init__(self, api_key: str, base_url: str = "https://api.dockase.com"):
        self.api_key = api_key
        self.base_url = base_url.rstrip('/')
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }

    def search_judgments(self, query: str, limit: int = 5):
        """Perform semantic vector search (RAG) against the Nigerian case law index."""
        url = f"{self.base_url}/v1/judgments/search"
        payload = {"query": query, "limit": limit}
        response = requests.post(url, json=payload, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def analyze_contract(self, contract_text: str, focus: str = "liability cap"):
        """Run a risk assessment review on contract clauses."""
        url = f"{self.base_url}/v1/contract/analyze"
        payload = {"contract_text": contract_text, "analysis_focus": focus}
        response = requests.post(url, json=payload, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def validate_citations(self, text: str):
        """Scan text and validate legal citations (e.g. LPELR, NWLR) to flag hallucinations."""
        url = f"{self.base_url}/v1/citations/validate"
        payload = {"text": text}
        response = requests.post(url, json=payload, headers=self.headers)
        response.raise_for_status()
        return response.json()

    def create_client(self, name: str, email: str, phone: str, address: str):
        """Create a client in the firm CRM."""
        url = f"{self.base_url}/v1/clients"
        payload = {
            "full_name": name,
            "email": email,
            "phone_number": phone,
            "address": address
        }
        response = requests.post(url, json=payload, headers=self.headers)
        response.raise_for_status()
        return response.json()

# Example Usage:
if __name__ == "__main__":
    # Load token from environment variable
    token = os.environ.get("DOCKASE_API_KEY", "dk_your_secret_key_here")
    client = DockaseClient(api_key=token)

    # 1. Search Precedents
    print("Searching precedents...")
    results = client.search_judgments("elements of fair hearing")
    print(f"Confidence: {results.get('confidence_score')}%")
    for doc in results.get("results", []):
        print(f" - {doc.get('title')} ({doc.get('citation')})")

    # 2. Validate Citations
    print("\nValidating citation text...")
    brief_text = "As held in APC v. Nduul (2017) LPELR-42415(SC) and an invalid case like SC/123/2026."
    val_results = client.validate_citations(brief_text)
    print(val_results.get("validated_text"))
```

---

## Active API Endpoints Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/v1/judgments/search` | Semantic search against the legal database. |
| `POST` | `/v1/contract/analyze` | Assess clauses for risks (indemnity, liability, etc.). |
| `POST` | `/v1/citations/validate` | Detect and flag unverified/hallucinated citations in drafts. |
| `POST` | `/v1/cases/tag` | Auto-tag cases with practice areas based on descriptions. |
| `POST` | `/v1/draft/multi` | Generate motion packs (motions, affidavits, written addresses). |
| `POST` | `/v1/clients` | Add a client to the CRM database. |
| `GET` | `/v1/clients` | List all client folders. |
| `POST` | `/v1/cases` | Create a new litigation matter folder. |
| `GET` | `/v1/cases/<id>/freshness` | Scan for stale or expiring filing deadlines. |
| `POST` | `/v1/cases/<id>/updates` | Log court proceeding logs and hearing dates. |

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.
