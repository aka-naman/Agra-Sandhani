# 📗 Hyper-Detailed User Manual: Agra Sandhani (Multi-User Form Builder)

---

## 📌 Table of Contents
1.  [**Introduction**](#introduction)
2.  [**Dashboard: Your Command Center**](#dashboard)
    *   [Main Action Buttons](#main-buttons)
    *   [Understanding Form Cards](#form-cards)
3.  [**The Form Builder: Designing for Data**](#builder)
    *   [The Interface](#builder-interface)
    *   [**The Field Encyclopedia (20 Field Types)**](#field-types)
4.  [**Submissions & Data Analytics**](#submissions)
    *   [Searching & Sorting](#search-sort)
    *   [Audit History (The Time Machine)](#audit-history)
    *   [Exporting (Excel vs PDF)](#export-options)
5.  [**🧠 AI Explorer: Natural Language Queries**](#ai-explorer)
6.  [**🔑 Permissions & Access Management**](#permissions)
7.  [**👑 Admin Dashboard (Superusers Only)**](#admin)
    *   [System Health Monitor](#health)
    *   [Trash Bin & Recovery](#trash)
8.  [**Technical Constraints & Best Practices**](#constraints)

---

<a name="introduction"></a>
## 1. Introduction
Welcome to **Agra Sandhani**, a professional-grade form builder designed for complex data collection and institutional analysis. This manual covers every aspect of the application, from simple form creation to advanced AI-powered data forensics.

---

<a name="dashboard"></a>
## 2. Dashboard: Your Command Center
The Dashboard is the first page you see. It organizes all forms you have access to.

<a name="main-buttons"></a>
### 2.1 Main Action Buttons
*   **➕ New Form**: Opens a window to name and create a fresh form structure.
*   **🧠 AI Explorer**: Opens the advanced search interface (See Section 5).
*   **🔑 Manage Access**: Opens the "Delegation" panel to grant your total data access to another user temporarily.
*   **📊 Admin Dashboard**: (Visible to Admins only) Direct link to system-wide management.
*   **🌙/☀️ Theme Toggle**: Switches between Dark and Light visual modes.
*   **🔔 Notification Bell**: Shows pending requests from other users wanting to see your forms.

<a name="form-cards"></a>
### 2.2 Understanding Form Cards
Each form is represented by a "Card." Depending on your permissions, you will see different buttons:
*   **✏️ Build**: Enter the designer mode to add or change questions. (Only visible if you are the owner or admin).
*   **📝 Fill**: Opens the public-facing link to submit data.
*   **📊 View**: Opens the spreadsheet view of all entries received.
*   **📥 Export**: Instantly downloads an Excel file of all data.
*   **📋 Duplicate**: Creates a copy of the form. You can choose to copy "Template Only" or "With All Records."
*   **🗑️ Delete**: Soft-deletes the form. It goes to the Trash Bin (See Section 7.2).

---

<a name="builder"></a>
## 3. The Form Builder: Designing for Data
<a name="builder-interface"></a>
### 3.1 The Interface
*   **💾 Save Fields**: Always click this after making changes! The system doesn't auto-save to the database (only to your browser draft).
*   **▲/▼ Arrows**: Change the order of questions.
*   **✕ Remove**: Deletes a specific question.
*   **Required Toggle**: If enabled, the user *cannot* submit the form without answering this question.

<a name="field-types"></a>
### 3.2 The Field Encyclopedia (20 Field Types)
This is the core of your form. Choose the right type for the best data quality.

1.  **📝 Short Answer**: A single line for names, titles, or short text.
2.  **📄 Paragraph**: A large box for descriptions, feedback, or long stories.
3.  **📧 Email**: Validates that the input looks like an email address (e.g., user@example.com).
4.  **📱 Phone**: Strictly enforces a 10-digit number.
5.  **☑️ Multiple Choice (MCQ)**: The user can select **multiple** options from a list.
6.  **🔘 Single Choice (Checkbox)**: Despite the name, this acts as a "Radio Button"—the user picks **exactly one** option.
7.  **📋 Dropdown**: A compact list where the user selects one option.
8.  **📊 Linear Scale**: A rating from 1 to 5 (or 0 to 10). You can label the ends (e.g., "Poor" to "Excellent").
9.  **⭐ Rating**: A visual "Star" selector.
10. **📅 Date**: A calendar picker.
11. **🕐 Time**: A clock picker.
12. **🔢 Number**: Strictly enforces numbers. You can set a **Min** and **Max** (e.g., Age between 18 and 60).
13. **🎯 Branch / Stream**: A specialized dropdown pre-filled with engineering branches.
14. **⏱️ Duration**: Pre-filled options for institutional timeframes (e.g., "January to June").
15. **🎓 University Autocomplete**: A powerful search box linked to a database of thousands of universities. 
    *   *Feature*: If a university is missing, users can add it, and the system "learns" it for the next person!
16. **🏠 Residential Address**: A composite field. Captures State, District, and Pincode.
    *   *Constraint*: Strictly enforces a **6-digit Pincode**.
17. **🏦 Bank Details**: Captures Bank Name, Account Number (12-digit validation), and IFSC.
18. **🏢 Group (Zone-based)**: A two-step selector. First pick a "Zone" (I-VIII), then a "Group" within that zone.
19. **🧮 CGPA to Percentage**: Automatically calculates percentage based on a factor (default is 9.5). Supporting "Auto" and "Manual" modes.
20. **📂 Upload Document**: Allows users to upload files (PDF/Images) up to **1GB**.

---

<a name="submissions"></a>
## 4. Submissions & Data Analytics
<a name="search-sort"></a>
### 4.1 Searching & Sorting
*   **🔍 Server-side Search**: Type anything (name, ID, university) and the system searches the entire database instantly.
*   **Advanced Filter**: Group your view by "Branch" or "CGPA" (High to Low) to find top performers quickly.

<a name="audit-history"></a>
### 4.2 Audit History (The Time Machine)
If an entry is edited, a **🕒 History** button appears.
*   Clicking this shows a "Snapshot" of the data **before** every single edit.
*   It tells you **Who** changed it and **When**.

<a name="export-options"></a>
### 4.3 Exporting Options
*   **Excel Standard**: Just the data.
*   **Excel Custom**: Allows you to toggle "Submitted At", "Submitted By", and "Missing Entries Remarks."
*   **PDF Custom**: Generates a professional landscape report. You can pick which fields to include and even start a new page for every "Branch."

---

<a name="ai-explorer"></a>
## 5. 🧠 AI Explorer: Natural Language Queries
The "Agra Sandhani" AI allows you to talk to your data.
1.  **Select Scope**: Choose which forms the AI should look into.
2.  **Type your question**: *"Show me all students from 'IIT Delhi' with CGPA above 9."*
3.  **Autocomplete**: As you type, the AI suggests field names and sample values from your database to help you be precise.
4.  **Export Results**: You can download the AI's findings as a separate Excel file.

---

<a name="permissions"></a>
## 6. 🔑 Permissions & Access Management
*   **Requesting Access**: If a form is owned by someone else, you must click "Request Access." They must approve it before you can see the data.
*   **Delegation**: Use the "Manage Access" button to grant another user "Full Viewing Rights" for a set amount of time (e.g., 2 hours for an audit). After the time expires, their access is automatically revoked.

---

<a name="admin"></a>
## 7. 👑 Admin Dashboard (Superusers Only)
<a name="health"></a>
### 7.1 System Health Monitor
*   Shows a visual "Pie Chart" for every form.
*   **Complete vs Missing**: Tells you what percentage of submissions have missing values (remarks).

<a name="trash"></a>
### 7.2 Trash Bin & Recovery
*   Items deleted on the dashboard are NOT gone forever. They stay in the **Trash Bin** for 30 days.
*   **Restore**: Moves the form/submission back to the live dashboard.
*   **Purge**: Permanently deletes the item. This cannot be undone.

---

<a name="constraints"></a>
## 8. Technical Constraints & Best Practices
*   **Browser Memory**: Form drafts are saved to *your* computer. If you switch computers before saving, the draft won't follow you.
*   **Concurrent Editing**: Avoid having two people build the same form at the same time; the last person to click "Save" will overwrite previous changes.
*   **Excel Limit**: Exports are optimized for speed, but files with 50,000+ rows may take up to a minute to prepare. Please be patient.
*   **Pincode Integrity**: Users *must* enter exactly 6 digits. Entering more or fewer will prevent the "Submit" button from working.

---
*Manual Version 3.1 - May 2026*
