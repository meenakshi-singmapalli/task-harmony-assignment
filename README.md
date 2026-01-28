# Task Harmony – LLM Email Extraction System

**Author:** [Your Name]  
**Role:** Backend / AI Engineer Assessment  

---

## Project Overview

This project implements an **LLM-powered email extraction system** for freight forwarding pricing enquiries. My goal was to build a Python-based system that can accurately extract structured shipment details from unstructured emails, including:

- Product line (import/export LCL)
- Origin and destination ports (with UN/LOCODE mapping)
- Incoterm
- Cargo weight (kg) and CBM
- Dangerous goods detection

The system leverages **Groq LLaMA 3.1/3.3 70B Versatile** models and Python automation to process, normalize, and validate email data.

---

## How I Worked on This Project

### 1. Python-based Extraction Pipeline
I built the extraction pipeline in Python to:

1. **Load and preprocess emails**: Read JSON files containing email subject/body.  
2. **Map ports**: Use `port_codes_reference.json` to normalize port names to UN/LOCODEs.  
3. **LLM integration**: Use Groq Python SDK to send each email to the model with a structured prompt.  
4. **Business rule enforcement**: Post-process LLM output in Python to handle:
   - Default incoterms (FOB)
   - Dangerous goods detection with negation logic
   - Numeric conversions and rounding
   - Subject vs body conflicts (body wins)
5. **Validation**: Use **Pydantic models** to ensure output matches the required schema, with proper types and null handling.

This pipeline ensures that every email is processed consistently and errors are handled gracefully. If an email fails extraction, all fields are set to `null` but the pipeline continues.

---

### 2. LLM Prompt Engineering

I iteratively designed and refined prompts to maximize extraction accuracy:

- **v1 – Basic Extraction**: Ask LLM to extract origin, destination, and CBM.  
- **v2 – Port Normalization**: Added UN/LOCODE examples in the prompt so LLM outputs standardized codes.  
- **v3 – Business Rules Integration**: Incorporated:
  - India detection for product lines
  - Dangerous goods detection rules
  - Defaulting incoterm to FOB
  - Handling multiple shipments (first only)
  - Numeric conversions (lbs → kg, tonnes → kg)
  
Each iteration was tested against the 50 sample emails, and results were evaluated using `evaluate.py`. Metrics were recorded, and errors were analyzed to guide the next iteration.

---

### 3. Handling Edge Cases

While building the system, I encountered several challenges:

1. **Multiple shipments in one email**: I implemented logic to extract the first shipment and ignore others.  
2. **Ambiguous dangerous goods mentions**: Used Python regex to detect both positive and negative indications.  
3. **Port name variations**: Created a lookup dictionary from `port_codes_reference.json` to normalize abbreviations like "HK" → "HKHKG".  
4. **Units conversion**: Ensured cargo weight in lbs and tonnes are converted to kg, rounded to 2 decimals.  

These edge cases were documented in the README and tested against specific emails.

---

### 4. Python Libraries and Tools Used

- **Groq SDK** – to query LLaMA 3.1/3.3 LLMs  
- **Pydantic** – output validation and schema enforcement  
- **JSON / OS / Time** – file handling, retries, and backoff logic  
- **Regex (`re`)** – parsing dangerous goods, weights, and CBM values  
- **dotenv** – managing API keys securely  

I implemented retry logic with exponential backoff to handle Groq API rate limits and timeouts.

---

### 5. Accuracy Evaluation

I created `evaluate.py` to compare the LLM extraction results (`output.json`) against the provided ground truth (`ground_truth.json`).  

- Field-level metrics: product_line, origin/destination codes and names, incoterm, cargo weight, CBM, dangerous goods  
- Overall accuracy calculation: `(total correct fields) / (total fields)`  
- Floating point values rounded to 2 decimals  
- String comparison: case-insensitive and trimmed  

After three iterations of prompt improvements, the overall extraction accuracy reached **~88–89%** on the sample dataset.

---

### 6. Summary of Work

Through this project, I demonstrated:

- Full **Python development** of an LLM-powered extraction pipeline  
- **Prompt engineering** with iterative improvements to increase accuracy  
- Handling of **edge cases** and business rules in code  
- **Validation and evaluation** with structured schemas  
- Practical integration of **LLM API calls** with retries and error handling  

This repository reflects not just the final outputs, but also the thought process, prompt evolution, and systematic approach to building a reliable AI-powered email extraction system.

---

### 7. Usage Instructions

```bash
pip install -r requirements.txt
cp .env.example .env
# Add your Groq API key to .env

python extract.py       # Generate output.json
python evaluate.py      # Evaluate accuracy
