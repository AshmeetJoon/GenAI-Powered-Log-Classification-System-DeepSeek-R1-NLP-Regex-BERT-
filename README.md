# NLP Log Classification System

An intelligent log classification system that uses multiple NLP techniques (regex patterns, BERT embeddings, and Large Language Models) to categorize system logs into predefined classes. The system provides both programmatic interfaces and a REST API for batch processing of log files.

## Features

- **Multi-Modal Classification**: Combines regex patterns, BERT-based embeddings, and LLM reasoning for robust log categorization
- **Source-Aware Classification**: Different classification strategies based on log source (e.g., LegacyCRM uses LLM, others use regex + BERT)
- **REST API**: FastAPI-based web service for CSV file uploads and classification
- **Predefined Categories**:
  - Workflow Error
  - Deprecation Warning
  - User Action
  - System Notification
  - Unclassified
- **Batch Processing**: Process multiple logs from CSV files efficiently
- **Extensible Architecture**: Easy to add new classification methods or categories

## Installation

### Prerequisites
- Python 3.8+
- Groq API key (for LLM classification)

### Setup
1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd project-nlp-log-classification-main
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Set up environment variables:
   Create a `.env` file in the root directory with:
   ```
   GROQ_API_KEY=your_groq_api_key_here
   ```

4. Ensure the BERT model is available:
   The pre-trained model should be located at `models/log_classifier.joblib`

## Usage

### Local Classification

#### Classify Individual Logs
```python
from classify import classify_log

# Classify a single log
label = classify_log("LegacyCRM", "Case escalation for ticket ID 7324 failed because the assigned support agent is no longer active.")
print(label)  # Output: Workflow Error
```

#### Classify CSV File
```python
from classify import classify_csv

# Process a CSV file with 'source' and 'log_message' columns
output_file = classify_csv("input.csv")
print(f"Classified logs saved to {output_file}")
```

### API Usage

#### Start the Server
```bash
python server.py
```
The API will be available at `http://localhost:8000`

#### Classify Logs via API
Upload a CSV file containing `source` and `log_message` columns:

```bash
curl -X POST "http://localhost:8000/classify/" \
     -H "accept: application/json" \
     -H "Content-Type: multipart/form-data" \
     -F "file=@test.csv"
```

Or using Python requests:
```python
import requests

with open('test.csv', 'rb') as f:
    response = requests.post('http://localhost:8000/classify/', files={'file': f})
    with open('classified_logs.csv', 'wb') as out:
        out.write(response.content)
```

## API Documentation

### POST /classify/
Classify logs from an uploaded CSV file.

**Parameters:**
- `file` (multipart/form-data): CSV file with columns `source` and `log_message`

**Response:**
- CSV file with additional `target_label` column containing classifications

**Error Responses:**
- 400: Invalid file format or missing required columns
- 500: Internal server error during processing

## Project Structure

```
├── classify.py              # Main classification orchestrator
├── processor_llm.py         # LLM-based classifier (Groq)
├── processor_bert.py        # BERT embedding + ML classifier
├── processor_regex.py       # Regex pattern-based classifier
├── server.py                # FastAPI web server
├── requirements.txt         # Python dependencies
├── .env                     # Environment variables (not in repo)
├── models/
│   └── log_classifier.joblib  # Trained BERT classifier
├── resources/
│   ├── test.csv             # Sample test data
│   └── output.csv           # Classification results
├── training/
│   ├── log_classification.ipynb  # Training notebook
│   └── dataset/
│       └── synthetic_logs.csv   # Training data
└── README.md                # This file
```

## Classification Logic

The system uses a hierarchical classification approach:

1. **LegacyCRM Source**: Uses LLM (Groq) to classify into Workflow Error, Deprecation Warning, or Unclassified
2. **Other Sources**:
   - First attempts regex pattern matching
   - Falls back to BERT-based classification if regex fails
   - Returns "Unclassified" if BERT confidence < 50%

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-classifier`
3. Make your changes and add tests
4. Commit your changes: `git commit -am 'Add new classifier'`
5. Push to the branch: `git push origin feature/new-classifier`
6. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
