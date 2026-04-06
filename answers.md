# api-ethics-assignment
#----------Task1----------
Task 1 — Classify and Handle PII Fields The dataset contains the following fields: full_name, email, date_of_birth, zip_code, job_title, diagnosis_notes Classify each field as either Direct PII or Indirect PII. For each field, state whether you would drop it, mask it, or pseudonymize it before sharing, and briefly justify your choice.
1. Direct PII (Personally Identifiable Information)
These fields can identify a specific individual on their own.
•	full_name
o	Action: Drop or Pseudonymize.
o	Justification: Names are the most direct identifiers. If the partner doesn't need to track individuals over time, drop it. If they do, replace names with a unique ID (e.g., Patient_001).
•	email
o	Action: Drop.
o	Justification: Emails are unique to individuals and provide a direct contact method. They are rarely necessary for health research.
2. Indirect PII (Quasi-Identifiers)
These don't identify someone alone but can be combined with other public data (like voter records) to "re-identify" a patient.
•	date_of_birth
o	Action: Mask (Generalize).
o	Justification: Transform this into Age or Year of Birth. Exact birth dates are highly identifying, but the patient's age is usually critical for medical research.
•	zip_code
o	Action: Mask (Truncate).
o	Justification: Use only the first 3 digits or convert to a Region/State. This preserves geographic trends while protecting residents of low-population areas.
•	job_title
o	Action: Mask (Generalize) or Keep.
o	Justification: If a title is rare (e.g., "CEO of [Specific Local Company]"), it becomes a direct identifier. Group specific roles into broader categories like "Healthcare" or "Management."
•	diagnosis_notes
o	Action: Mask (Scrub).

#----------Task2----------
Task 2 — Audit the API Script for Ethical Compliance
Your team's data collection script is shown below:

**Given code:**
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])

# Store all records permanently in company database
save_to_database(records)

Identify two ethical or TOS violations present in this script. For each violation, explain what the problem is and suggest a corrected version of the relevant code.

**Here are two major ethical and compliance violations:**
**1. Hardcoded API Key (Security Risk)**
•	The Problem: Storing the API_KEY directly in the source code is a major security vulnerability. If this script is pushed to a version control system (like GitHub), anyone with access can steal the key, leading to unauthorized data access or billing charges.
•	The Correction: Need to use Environment Variables to keep sensitive credentials out of the codebase.

**Corrected code:**
import os
# Corrected: Fetch the key from the system environment
API_KEY = os.getenv("HEALTH_STATS_API_KEY") 

**2. Aggressive Rate Limiting (TOS/Ethical Violation)**
•	The Problem: The for loop makes 100 consecutive requests as fast as the CPU allows. This "hammering" of the server can be flagged as a Denial of Service (DoS) attack, likely violating the API's Terms of Service (TOS) for a "free tier" key and potentially crashing the provider's service.
•	The Correction: Need to implement Request Throttling by adding a small delay between calls.

**Corrected code:**
import time
# Corrected: Add a 1-second pause between page requests
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    # ... logic ...
    time.sleep(1) 

**Corrected code(complete code):**
import os
import time
import requests

# 1. Security: Use Environment Variables for sensitive API keys
API_URL = "https://example.com"
API_KEY = os.getenv("HEALTH_STATS_API_KEY")

def save_to_database(data):
    # Placeholder for database logic
    pass

def fetch_records():
    records = []
    try:
        # Loop through pages 1 to 100
        for page in range(1, 101):
            # Include a timeout to prevent the script from hanging indefinitely
            response = requests.get(API_URL, params={"page": page, "key": API_KEY}, timeout=10)
            
            # Check for HTTP errors (e.g., 404, 500)
            response.raise_for_status()
            
            data = response.json()
            records.extend(data.get("results", []))
            
            # 2. Ethics/TOS: Throttling to avoid overwhelming the API server
            time.sleep(1) 
            
    except requests.exceptions.RequestException as e:
        print(f"Error during API collection: {e}")
    
    return records

# Execute the collection and save
all_records = fetch_records()
if all_records:
    save_to_database(all_records)

o	Justification: These often contain "accidental PII" (e.g., "Patient's wife, Jane, noted..."). Need to use text-processing tools to remove names or specific dates from these free-text fields.
