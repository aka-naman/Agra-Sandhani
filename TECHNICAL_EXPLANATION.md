# 🛠️ Technical Explanation: "The Lifecycle of a Click"

This document provides a detailed technical mapping of how the **Agra Sandhani** platform operates across the Frontend, Backend, and Database layers.

---

## 1. 🏗️ Architectural Overview
- **Client**: React 18 (Vite) + Context API.
- **Server**: Node.js + Express.
- **Database**: PostgreSQL 15+ (JSONB Heavy).
- **AI**: Qwen-2.5-Coder (GGUF via node-llama-cpp).

---

## 2. ⚡ Scenario 1: Creating a New Form
When a user clicks **"+ New Form"** on the Dashboard:

1.  **Frontend**: `DashboardPage.jsx` triggers `handleCreate()`.
2.  **API Call**: `POST /api/forms` with the `{ name }` payload.
3.  **Backend**: `forms.js` route handler:
    - Verifies the user's JWT.
    - Inserts a record into the `forms` table.
    - Creates **Version 1** in the `form_versions` table.
4.  **Database**: Returns the new Form ID and Version ID.
5.  **State Update**: The Dashboard list refreshes via `fetchForms()`.

---

## 3. 🧩 Scenario 2: Submitting Data (Complex Fields)
When a user clicks **"Submit"** on the Fill page:

### Step 1: Pre-Submission (File Upload)
If the form has "File Upload" fields:
- The frontend uploads files to `/api/forms/upload`.
- The server stores files in `/server/uploads/batch_[TIMESTAMP]/`.
- The server returns the **folder path** (e.g., `/uploads/batch_123`).

### Step 2: Main Submission
- **API Call**: `POST /api/forms/:id/submit`.
- **Backend Logic (`submissions.js`)**:
    1.  **Composite Parsing**: The system splits values like CGPA (`9.2 ||| auto ||| 9.5`) or Address (`Delhi ||| Central ||| 110001`) to perform "Learning."
    2.  **Learning Engine**:
        - If `University` is new -> `INSERT INTO universities`.
        - If `Zone/Group` is new -> `INSERT INTO organizational_groups`.
    3.  **JSONB Storage**: The entire response is stored in the `submissions.data_json` column.
    4.  **Indexing**: The PostgreSQL **GIN Index** on `data_json` immediately indexes this new data for the AI Explorer.

---

## 4. 🧠 Scenario 3: The AI Query Execution
When a user types a prompt in the **AI Explorer**:

1.  **Schema Preparation**: The backend fetches all field labels and IDs for the selected forms.
2.  **Context Injection**: A system prompt is generated for the local LLM:
    ```text
    Table: submissions (data_json JSONB)
    Mapping: "Full Name" -> data_json->>'Full Name'
    User: "Find students from Mumbai"
    ```
3.  **Inference**: The `Qwen-2.5-Coder-0.5B` model generates a SQL string:
    ```sql
    SELECT * FROM submissions WHERE data_json->>'City' ILIKE '%Mumbai%';
    ```
4.  **Sandboxing**: The backend ensures the SQL starts with `SELECT` and injects `deleted_at IS NULL` and `form_id IN (...)` for security.
5.  **Execution**: The query runs against the live database, and the results are rendered in a dynamic table on the frontend.

---

## 5. 📊 Scenario 4: Streaming Excel Exports
When a user clicks **"Export"**:

1.  **Backend**: `export.js` is invoked.
2.  **Memory Management**: Instead of loading all data into RAM (which would crash for 100k rows), it uses a **PostgreSQL Cursor**.
3.  **Excel Streaming**: 
    - It uses `ExcelJS.stream.xlsx.WorkbookWriter`.
    - It fetches 2,000 rows, writes them to the file, and repeats.
    - This keeps RAM usage constant at ~50MB regardless of file size.
4.  **Post-Processing**: The backend replaces the internal ` ||| ` separator with a professional `, ` on-the-fly during the stream.

---

## 6. 🔒 Security & Data Isolation
- **Row-Level Security (Application Layer)**: Every query to fetch forms or submissions includes a check: `WHERE owner_id = $1 OR status = 'Approved'`.
- **Audit Logs**: Every `UPDATE` on a submission triggers an `INSERT` into `submission_audit_log` via a database transaction, ensuring no edit is ever "lost."

---
*Technical Manual v1.0 - Engineering Focus*
