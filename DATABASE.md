# 📊 PostgreSQL JSONB Query Cheat Sheet: Admin Monitoring

This document provides a comprehensive collection of advanced PostgreSQL queries for managing and auditing the **Multi-User Dynamic Form Builder** system. Use these queries for deep forensics, security monitoring, and custom reporting.

---

## 🕒 Section 1: System Activity & Audit Trail
*Focus: Monitoring `system_logs` for high-level events.*

1. **Find all exports by a specific user in the last 7 days:**
   ```sql
   SELECT timestamp, details->>'form_name' as form, details->>'format' as format 
   FROM system_logs 
   WHERE action_type = 'export' 
     AND user_id = [USER_ID] 
     AND timestamp > NOW() - INTERVAL '7 days'
   ORDER BY timestamp DESC;
   ```

2. **Count total PDF vs Excel exports system-wide:**
   ```sql
   SELECT details->>'format' as format, COUNT(*) 
   FROM system_logs 
   WHERE action_type = 'export' 
   GROUP BY details->>'format';
   ```

3. **Identify users who deleted more than 5 submissions today:**
   ```sql
   SELECT user_id, COUNT(*) 
   FROM system_logs 
   WHERE action_type = 'delete_submission' 
     AND timestamp::date = CURRENT_DATE
   GROUP BY user_id 
   HAVING COUNT(*) > 5;
   ```

4. **Find all form renaming history:**
   ```sql
   SELECT timestamp, details->>'old_name' as old, details->>'new_name' as new 
   FROM system_logs 
   WHERE action_type = 'rename_form' 
   ORDER BY timestamp DESC;
   ```

5. **List all restorations made by Admins:**
   ```sql
   SELECT l.timestamp, u.username, l.details->>'form_name' as form 
   FROM system_logs l
   JOIN users u ON l.user_id = u.id
   WHERE action_type = 'restore_submission'
   ORDER BY l.timestamp DESC;
   ```

---

## 📝 Section 2: Data Integrity & Form Submissions
*Focus: Querying inside the `submissions.data_json` bucket.*

6. **Find submissions containing a specific university (dot-agnostic):**
   ```sql
   SELECT id, data_json->>'University Name' 
   FROM submissions 
   WHERE REPLACE(data_json->>'University Name', '.', '') ILIKE '%AMU%';
   ```

7. **List submissions with CGPA greater than 9.0:**
   ```sql
   SELECT id, data_json->>'Full Name' as student, data_json->>'CGPA' as cgpa
   FROM submissions
   WHERE (NULLIF(substring(data_json->>'CGPA' from '^[0-9.]+'), '')::numeric) > 9.0;
   ```

8. **Group submissions by State and count them:**
   ```sql
   SELECT data_json->>'State' as state, COUNT(*)
   FROM submissions
   WHERE data_json->>'State' IS NOT NULL
   GROUP BY data_json->>'State'
   ORDER BY COUNT(*) DESC;
   ```

9. **Find all submissions made within a specific time range (e.g., 9 PM to 6 AM):**
   ```sql
   SELECT id, submitted_at 
   FROM submissions 
   WHERE EXTRACT(HOUR FROM submitted_at) NOT BETWEEN 9 AND 21;
   ```

10. **Extract only the 'Phone Number' from all submissions of Form ID 5:**
    ```sql
    SELECT s.id, s.data_json->>'Phone' as phone
    FROM submissions s
    JOIN form_versions fv ON s.form_version_id = fv.id
    WHERE fv.form_id = 5;
    ```

---

## 🏥 Section 3: Data Health (Missing Entries)
*Focus: Forensics on the 'Remarks' column and missing data.*

11. **Identify forms with the most "Missing Entries":**
    ```sql
    SELECT f.name, COUNT(s.id) as missing_count
    FROM forms f
    JOIN form_versions fv ON f.id = fv.form_id
    JOIN submissions s ON fv.id = s.form_version_id
    WHERE s.data_json->>'Remarks' LIKE 'Missing Entries%'
    GROUP BY f.name
    ORDER BY missing_count DESC;
    ```

12. **Find all submissions for "Branch: CS" that have remarks:**
    ```sql
    SELECT id, data_json->>'Full Name' as name, remarks 
    FROM submissions 
    WHERE data_json->>'Branch' = 'Computer Science' 
      AND (remarks IS NOT NULL OR data_json->>'Remarks' != '');
    ```

13. **List students whose Account Number is less than 12 digits (Potential error):**
    ```sql
    SELECT id, data_json->>'Full Name', data_json->>'Account Number'
    FROM submissions
    WHERE LENGTH(data_json->>'Account Number') < 12;
    ```

---

## 🗑️ Section 4: Trash Bin & Soft Deletes
*Focus: Managing `deleted_at` records.*

14. **Count total deleted records per form:**
    ```sql
    SELECT f.name, COUNT(s.id) 
    FROM forms f
    JOIN form_versions fv ON f.id = fv.form_id
    JOIN submissions s ON fv.id = s.form_version_id
    WHERE s.deleted_at IS NOT NULL
    GROUP BY f.name;
    ```

15. **Find submissions deleted by a specific user (via log correlation):**
    ```sql
    SELECT s.* 
    FROM submissions s
    JOIN system_logs l ON (l.details->>'submission_id')::int = s.id
    WHERE l.action_type = 'delete_submission' AND l.user_id = [USER_ID];
    ```

16. **Bulk restore all submissions deleted in the last hour:**
    ```sql
    UPDATE submissions 
    SET deleted_at = NULL 
    WHERE deleted_at > NOW() - INTERVAL '1 hour';
    ```

---

## 🧪 Section 5: Advanced JSONB & Pattern Matching

17. **Find any submission where ANY field contains the word "Urgent":**
    ```sql
    SELECT * FROM submissions WHERE data_json::text ILIKE '%Urgent%';
    ```

18. **Find submissions where the Address contains "Mumbai" OR "Pune":**
    ```sql
    SELECT id FROM submissions 
    WHERE data_json->>'Address' ~* '(Mumbai|Pune)';
    ```

19. **List all unique keys currently present in ALL submissions of Form 10:**
    ```sql
    SELECT DISTINCT jsonb_object_keys(data_json) 
    FROM submissions s
    JOIN form_versions fv ON s.form_version_id = fv.id
    WHERE fv.form_id = 10;
    ```

20. **Find submissions that have a 'Pincode' but NO 'State' (Incomplete Address):**
    ```sql
    SELECT id FROM submissions 
    WHERE data_json ? 'Pincode' AND NOT (data_json ? 'State');
    ```

21. **Calculate the average CGPA for "Civil Engineering":**
    ```sql
    SELECT AVG((NULLIF(substring(data_json->>'CGPA' from '^[0-9.]+'), '')::numeric))
    FROM submissions
    WHERE data_json->>'Branch' = 'Civil Engineering';
    ```

22. **Check for Duplicate Names in Form 1:**
    ```sql
    SELECT data_json->>'Full Name', COUNT(*)
    FROM submissions s
    JOIN form_versions fv ON s.form_version_id = fv.id
    WHERE fv.form_id = 1
    GROUP BY data_json->>'Full Name'
    HAVING COUNT(*) > 1;
    ```

23. **List submissions by 'Zone' then 'Group' alphabetically:**
    ```sql
    SELECT data_json->>'Zone' as zone, data_json->>'Group' as grp, id
    FROM submissions
    ORDER BY zone ASC, grp ASC;
    ```

24. **Find exports that included more than 10 fields:**
    ```sql
    SELECT timestamp, jsonb_array_length(details->'selected_fields') as field_count
    FROM system_logs
    WHERE action_type = 'export' AND details ? 'selected_fields'
      AND jsonb_array_length(details->'selected_fields') > 10;
    ```

25. **Identify the most active time of day for submissions:**
    ```sql
    SELECT EXTRACT(HOUR FROM submitted_at) as hour, COUNT(*)
    FROM submissions
    GROUP BY hour
    ORDER BY count DESC;
    ```

26. **Find submissions where a specific bank 'SBI' was used:**
    ```sql
    SELECT id, data_json->>'Bank Name' 
    FROM submissions 
    WHERE data_json->>'Bank Name' ILIKE '%State Bank%';
    ```

27. **Search for submissions containing invalid characters in phone numbers:**
    ```sql
    SELECT id, data_json->>'Phone'
    FROM submissions
    WHERE data_json->>'Phone' ~ '[^0-9 +-]';
    ```

28. **Correlate Submission with Form Owner (Audit):**
    ```sql
    SELECT s.id, f.name as form, u.username as owner
    FROM submissions s
    JOIN form_versions fv ON s.form_version_id = fv.id
    JOIN forms f ON fv.form_id = f.id
    JOIN users u ON f.user_id = u.id;
    ```

29. **Find all "Other" entries that were learned by the system:**
    ```sql
    -- This assumes we logged the "learning" event (which we should)
    SELECT timestamp, details->>'new_entry' as entry, details->>'category' as type
    FROM system_logs
    WHERE action_type = 'learn_entry'
    ORDER BY timestamp DESC;
    ```

30. **Clean Up: Delete logs older than 1 year to save storage:**
    ```sql
    DELETE FROM system_logs WHERE timestamp < NOW() - INTERVAL '1 year';
    ```
