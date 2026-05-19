# 🗺️ Project Database Context: Multi-User Dynamic Form Builder

This document serves as a technical map for LLMs to understand the database schema and data storage patterns of this application. Use this context to generate accurate PostgreSQL queries for reporting, auditing, and data extraction.

---

## 🏗️ Architectural Overview
The system uses a **Hybrid Relational-JSONB Architecture**. 
*   **Static Data:** User accounts, form metadata, and versioning are stored in standard relational tables.
*   **Dynamic Data:** Form submissions are stored in a **Single-Table Multi-Tenant** pattern using a PostgreSQL `JSONB` column. Every form's data lives in the same table, but with a unique JSON key structure.

---

## 📊 Core Tables & Schema

### 1. `users`
Stores system accounts and roles.
*   `id` (SERIAL, PK)
*   `username` (TEXT, UNIQUE)
*   `password` (TEXT, Hashed)
*   `role` (TEXT) — either `'admin'` or `'user'`.
*   `created_at` (TIMESTAMP)

### 2. `forms`
Metadata for the forms created by users.
*   `id` (SERIAL, PK)
*   `name` (TEXT) — The human-readable title.
*   `user_id` (INT, FK) — Reference to the owner in `users`.
*   `is_locked` (BOOLEAN) — If true, fields cannot be edited.
*   `created_at` (TIMESTAMP)

### 3. `form_versions`
Tracks changes to form structures.
*   `id` (SERIAL, PK)
*   `form_id` (INT, FK) — Reference to `forms`.
*   `version_number` (INT) — Increments on every field change.
*   `created_at` (TIMESTAMP)

### 4. `form_fields`
Defines the individual inputs for a specific form version.
*   `id` (SERIAL, PK)
*   `form_version_id` (INT, FK) — Reference to `form_versions`.
*   `label` (TEXT) — **CRITICAL:** This label is used as the key in the submission JSON.
*   `type` (TEXT) — e.g., `'text'`, `'number'`, `'cgpa_converter'`, `'address_composite'`, `'bank_details'`, `'file_upload'`.
*   `required` (BOOLEAN)
*   `field_order` (INT)

### 5. `submissions` (The Core Data Bucket)
Stores all responses to all forms.
*   `id` (SERIAL, PK)
*   `form_version_id` (INT, FK) — Reference to `form_versions`.
*   `data_json` (JSONB) — **The Data Bucket.** Keys are the `label` from `form_fields`.
*   `remarks` (TEXT) — Internal notes/missing entry flags.
*   `submitted_at` (TIMESTAMP)
*   `updated_by` (INT, FK) — Reference to `users`.
*   `deleted_at` (TIMESTAMP) — Soft-delete flag (if NOT NULL, it's "deleted").

### 6. `system_logs`
Unified event tracking.
*   `id` (SERIAL, PK)
*   `action_type` (TEXT) — e.g., `'export'`, `'delete_submission'`, `'rename_form'`.
*   `user_id` (INT, FK)
*   `timestamp` (TIMESTAMP)
*   `details` (JSONB) — Event-specific metadata (e.g., `{"format": "pdf", "form_name": "Survey"}`).

---

## 🔍 Data Storage Patterns (JSONB)

### The `data_json` Structure
Inside `submissions.data_json`, data is stored as key-value pairs.
*   **Simple Fields:** `{"Full Name": "John Doe", "Age": 25}`
*   **Composite Fields:** Values are joined by ` ||| ` (triple pipe).
    *   *Example Address:* `{"Home Address": "Flat 402 ||| MG Road ||| Mumbai ||| Maharashtra ||| 400001"}`
    *   *Example Bank:* `{"Bank Info": "SBI ||| 123456789012 ||| SBIN0001234"}`
    *   *Example CGPA:* `{"Academics": "9.2 / 10.0 (87.4%)"}`

### PostgreSQL Query Tips for LLMs
*   `->>` : Returns a value as **Text** (Use this for filtering/display).
*   `->` : Returns a value as **JSON/Object** (Use for arrays).
*   `?` : Checks if a key **exists** in the JSON.
*   `::numeric` : Casts a JSON string to a number for math/sorting.

---

## 🛠️ Representative Query Examples

### Join Submissions to Form Name
```sql
SELECT f.name as form_title, s.data_json
FROM submissions s
JOIN form_versions fv ON s.form_version_id = fv.id
JOIN forms f ON fv.form_id = f.id
WHERE s.deleted_at IS NULL;
```

### Search within a Composite Field
```sql
-- Find submissions where State (part of Address) is 'Punjab'
SELECT * FROM submissions 
WHERE data_json->>'Address' LIKE '%||| Punjab |||%';
```

### Sorting by a JSON Number
```sql
-- Sort by CGPA field (Handling potential string extraction)
SELECT id, data_json->>'CGPA'
FROM submissions
ORDER BY (NULLIF(substring(data_json->>'CGPA' from '^[0-9.]+'), '')::numeric) DESC NULLS LAST;
```

### Filtering System Logs
```sql
-- Find all PDF exports for Form ID 15
SELECT timestamp, details->>'form_name'
FROM system_logs
WHERE action_type = 'export' AND details->>'format' = 'pdf' AND (details->>'form_id')::int = 15;
```

---

## 📑 Mapping Context Summary
To query data correctly, an LLM must:
1.  **Identify Form ID** from `forms`.
2.  **Identify Field Labels** from `form_fields` (these are the keys in `data_json`).
3.  **Filter `deleted_at IS NULL`** for active data.
4.  **Use `->>`** to extract keys from `data_json`.
5.  **Use `LIKE` or `ILIKE`** for partial matches in composite fields (separated by ` ||| `).
