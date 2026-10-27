# AI Contract Analysis

Vakeel is an AI-powered contract analysis backend built with FastAPI, MongoDB, and Google's Gemini API. It accepts PDF and TXT contracts, extracts their text, and generates structured insights about key clauses, potential risks, and recommendations.

## Features

- **Contract Upload:** Upload PDF and TXT contracts with file-extension and 10 MB size validation.
- **Document Processing:** Extract contract text, page counts, and word counts using PyPDF2 and Python file handling.
- **AI-Powered Analysis:** Use Google's Gemini API to analyze contract text and generate structured insights.
- **Key Clause Identification:** Extract important clauses and provide plain-language explanations.
- **Risk Assessment:** Identify potential contractual risks, assign risk levels, and generate recommendations.
- **Contract Management:** Store uploaded contract metadata and extracted text in MongoDB.
- **Analysis History:** Persist analysis results and retrieve previous analyses by contract or analysis ID.
- **REST API:** Expose contract and analysis operations through FastAPI endpoints.
- **Containerized Database:** Run MongoDB locally using Docker Compose with persistent database storage.
- **Configuration Management:** Load database connection settings and API credentials from environment variables.

## Tech Stack

| Category | Technologies |
|---|---|
| Language | Python |
| Backend | FastAPI, Uvicorn |
| Database | MongoDB, PyMongo |
| Generative AI | Google Gemini API |
| Document Processing | PyPDF2 |
| Data Models | Pydantic |
| Configuration | python-dotenv, environment variables |
| DevOps | Docker Compose |

## Architecture

```text
                  Client
                    |
                    v
              FastAPI Backend
                    |
          +---------+----------+
          |                    |
          v                    v
   Contract Routes       Analysis Routes
          |                    |
          v                    v
   File Validation       Retrieve Contract
          |                    |
          v                    v
   Text Extraction       Gemini API
          |                    |
          v                    v
   MongoDB Storage       Structured Analysis
                               |
                               v
                        MongoDB Storage
                               |
                               v
                          API Response
```

## Project Structure

```text
AI-Contract-analysis---1/
├── README.md
└── vaken-contract-api/
    ├── app/
    │   ├── main.py
    │   ├── config.py
    │   ├── database.py
    │   ├── models.py
    │   ├── routes/
    │   │   ├── contracts.py
    │   │   └── analysis.py
    │   ├── service/
    │   │   ├── document_parser.py
    │   │   ├── gemini_analyse.py
    │   │   └── prompt.py
    │   └── uploads/
    ├── samples/
    │   └── sample_nda.txt
    ├── docker-compose.yml
    └── requirements.txt
```

## Prerequisites

- Python compatible with the project's pinned dependencies
- Docker Desktop or Docker Engine with the Compose plugin
- A Google Gemini API key
- Git

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/awasthypranshu/AI-Contract-analysis---1.git

cd AI-Contract-analysis---1/vaken-contract-api
```

### 2. Create a virtual environment

**Windows PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Linux / macOS**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
pip install google-genai
```

> **Dependency note:** The current source imports `google-genai`, but the repository's `requirements.txt` does not list it. Add `google-genai` to that file so the dependency is installed reproducibly.

### 4. Configure environment variables

Create a `.env` file inside the `vaken-contract-api/` directory:

```dotenv
MONGODB_URI=mongodb://root:mypassword@localhost:27017/?authSource=admin
GEMINI_API_KEY=your_gemini_api_key
```

Replace the example credentials with your own local development values. Never commit API keys or production credentials to GitHub.

### 5. Start MongoDB using Docker Compose

From the `vaken-contract-api/` directory, run:

```bash
docker compose up -d
```

This starts MongoDB on port `27017` and uses a named Docker volume to persist database files.

Check the container status:

```bash
docker compose ps
```

The provided Compose configuration currently uses example root credentials. Replace these before using the setup beyond local development and make sure `MONGODB_URI` matches.

### 6. Start the FastAPI server

From the `vaken-contract-api/` directory:

```bash
cd app
uvicorn main:app --reload
```

The API runs at:

- **API root:** http://127.0.0.1:8000/
- **Swagger UI:** http://127.0.0.1:8000/docs
- **ReDoc:** http://127.0.0.1:8000/redoc

Swagger UI lets you explore the available endpoints and submit requests interactively.

## API Endpoints

Base URL: `http://127.0.0.1:8000`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Get API information and available endpoints |
| POST | `/contracts/upload` | Upload a PDF or TXT contract |
| GET | `/contracts/` | List uploaded contracts |
| GET | `/contracts/{contract_id}` | Retrieve a contract by ID |
| POST | `/analysis/analyse/{contract_id}` | Analyze a contract using Gemini |
| GET | `/analysis/{analysis_id}` | Retrieve a specific analysis |
| GET | `/analysis/` | List stored analyses |
| GET | `/analysis/contract/{contract_id}` | Retrieve analyses for a contract |

## Usage

### 1. Upload a contract

Open a second terminal in the `vaken-contract-api/` directory and run:

```bash
curl -X POST "http://127.0.0.1:8000/contracts/upload" \
  -F "file=@samples/sample_nda.txt"
```

For a PDF, replace the sample path with your PDF file.

The API checks the file extension and rejects files larger than 10 MB. It then extracts the text, saves the uploaded file locally, and stores the contract record in MongoDB.

The response includes a contract ID that you can use for analysis.

### 2. Analyze the contract

Replace `<contract_id>` with the ID returned by the upload endpoint:

```bash
curl -X POST \
  "http://127.0.0.1:8000/analysis/analyse/<contract_id>"
```

The API retrieves the contract text from MongoDB, sends it to Gemini, parses the model response, and stores the resulting analysis.

The current implementation sends up to the first 15,000 characters of the extracted contract text to Gemini.

### 3. Retrieve the analysis

Replace `<analysis_id>` with the ID returned by the analysis endpoint:

```bash
curl "http://127.0.0.1:8000/analysis/<analysis_id>"
```

### 4. List uploaded contracts

```bash
curl "http://127.0.0.1:8000/contracts/"
```

### 5. Retrieve all analyses for a contract

```bash
curl "http://127.0.0.1:8000/analysis/contract/<contract_id>"
```

## Analysis Output

The analysis response is modeled around the following fields:

```json
{
  "contract_id": "contract_id",
  "summary": "A concise summary of the contract",
  "contract_type": "NDA",
  "key_clauses": [
    {
      "clause_title": "Confidentiality",
      "clause_text": "Relevant clause text",
      "explanation": "Plain-language explanation",
      "is_standard": true
    }
  ],
  "risk_flags": [
    {
      "risk_title": "Potential contractual risk",
      "description": "Explanation of the identified risk",
      "risk_level": "medium",
      "recommendation": "Review the relevant clause",
      "clause_reference": "Clause 4"
    }
  ],
  "overall_risk_level": "medium",
  "recommendations": [
    "Review ambiguous contractual obligations"
  ]
}
```

This is an illustrative example based on the application's Pydantic models, not a guarantee that Gemini will return these exact values.

## Database Design

MongoDB stores data in two collections:

- **`contracts`** — Stores uploaded contract metadata, extracted text, filenames, document statistics, and status information.
- **`analysis`** — Stores generated summaries, contract types, key clauses, risk flags, overall risk levels, and recommendations.

The application creates indexes for contract filenames and analysis contract IDs during startup.

## Configuration Reference

| Variable | Purpose |
|---|---|
| `MONGODB_URI` | MongoDB connection string |
| `GEMINI_API_KEY` | Gemini API credential |

Uploaded files are stored in the local `uploads/` directory. The MongoDB volume persists database files, but it does not automatically persist the API's uploaded files.

## Current Limitations

- Authentication and per-user authorization are not implemented in the current API routes.
- File validation currently relies on file extensions and file size; content-signature validation should be added.
- Gemini responses may contain invalid JSON or API errors, which require more robust error handling.
- The analysis route calls Gemini before returning its response; it does not currently implement a background job queue.
- Contract status handling should be unified between the model and analysis workflow.
- MongoDB IDs should be validated before database queries, and returned database objects should be normalized consistently.
- The API service itself does not currently have a Dockerfile; Docker Compose is used for MongoDB.
- AI-generated legal insights may be incomplete or inaccurate and should be reviewed by a qualified legal professional.

## Roadmap

- [ ] Implement JWT authentication and per-user authorization
- [ ] Improve file validation and upload security
- [ ] Strengthen error handling for MongoDB and Gemini failures
- [ ] Add automated tests for upload, parsing, and analysis endpoints
- [ ] Standardize contract and analysis status tracking
- [ ] Add a Dockerfile for the FastAPI application
- [ ] Extend Docker Compose to run both the API and MongoDB
- [ ] Add secure deployment configuration and secret management
- [ ] Evaluate analysis quality using a reviewed collection of sample contracts

## Disclaimer

Vakeel is an AI-assisted contract review tool intended for informational purposes. It does not provide professional legal advice and should not be used as a substitute for review by a qualified legal professional.

## License

No license is currently specified. Add a `LICENSE` file if you intend to permit others to use, modify, or distribute this project.
