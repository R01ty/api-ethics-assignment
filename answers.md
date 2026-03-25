Task 1 — Classify and Handle PII Fields
| Field             | Type of PII           | Recommended Handling | Justification                                                                                                                     |
| ----------------- | --------------------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `full_name`       | Direct PII            | Pseudonymize         | Full names uniquely identify individuals. Replace with a unique identifier or hash to allow analysis without exposing identities. |
| `email`           | Direct PII            | Drop                 | Emails directly identify someone. Unless absolutely required, dropping is safest to prevent re-identification.                    |
| `date_of_birth`   | Indirect PII          | Mask                 | Birthdates combined with other info (like ZIP) can identify individuals. Mask to only the year or age range.                      |
| `zip_code`        | Indirect PII          | Mask                 | ZIP codes are quasi-identifiers. Mask to the first 3 digits or region level to reduce re-identification risk.                     |
| `job_title`       | Indirect PII          | Keep                 | Job title alone is unlikely to directly identify someone, safe to retain for analysis.                                            |
| `diagnosis_notes` | Not PII but sensitive | Mask / Redact        | May contain sensitive health info. Mask or anonymize sensitive content before sharing.                                            |
Notes:

Direct PII: Fields that can identify a person on their own (full name, email).
Indirect PII: Fields that could identify someone when combined with other info (DOB, ZIP).
Sensitive but not PII: Health notes are highly sensitive under HIPAA even if not direct identifiers.

Task 2 — Audit the API Script for Ethical Compliance

```import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])```

# Store all records permanently in company database
save_to_database(records)

Identified Ethical / TOS Violations:

Violation: Using a free-tier API key to collect large volumes of data
Problem: Using the free_tier_key_abc123 to collect 100 pages may exceed the free-tier usage limits, violating the API’s terms of service.
Correction: Implement proper authentication and respect rate limits.

```import requests
import time

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "paid_tier_key_xyz789"  # Use authorized paid-tier API key

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    response.raise_for_status()  # Ensure request is successful
    data = response.json()
    records.extend(data["results"])
    time.sleep(1)```  # Respect API rate limits

Violation: Storing raw patient-level data without anonymization

Problem: Storing sensitive health records with direct identifiers in a database violates privacy regulations (HIPAA, GDPR) and ethical standards.
Correction: Clean or pseudonymize PII before storage.

```def pseudonymize(record):
    record['full_name'] = hash(record['full_name'])
    record['email'] = None
    record['date_of_birth'] = record['date_of_birth'][:4]  # Keep only year
    record['zip_code'] = record['zip_code'][:3]  # Mask ZIP
    # diagnosis_notes can be redacted if necessary
    return record

cleaned_records = [pseudonymize(r) for r in records]
save_to_database(cleaned_records)```
