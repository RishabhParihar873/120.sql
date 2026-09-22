Found the likely break point. Two things stand out:

**Suspect #1 — `p_output_to => 'directory'` combined with capturing a BLOB.**
When you tell AOP (Apex Office Print) to write output `'directory'`, it typically means "save the file to a database DIRECTORY object on the server" — not "hand me the file bytes back." In that mode, `aop_api_pkg.plsql_call_to_aop` usually returns `NULL` (or nothing meaningful) for the BLOB, because the actual file content never comes back to PL/SQL — it's sitting on disk. So `l_return_blob` is likely **empty**.

**Suspect #2 — inserting `l_return_blob` (a BLOB) into `file_path1`.**
The column name `file_path1` strongly suggests it's meant to hold a **text path/filename** (varchar2), not raw file bytes. If that column is actually varchar2 and you're inserting a BLOB variable into it, PL/SQL won't implicitly convert that — this would be a **compile-time type mismatch**, which would make the whole package body invalid.

Since your `WHEN OTHERS` catches *runtime* errors and returns a clean JSON — but a *compile* error wouldn't even let the function run at all — I need to know which one it is.

Run these two and paste results:

**Query 1 — is the package actually valid?**
```sql
select object_name, status 
from user_objects 
where object_name = 'ICT_UNIVERSAL_DOC_ENGINE';
```

**Query 2 — what type is `file_path1` really?**
```sql
select column_name, data_type 
from user_tab_columns 
where table_name = 'CGCT_AI_DOCUMENT_LIBRARY' 
order by column_id;
```

Once I see these, I'll know exactly whether it's the type mismatch, the `'directory'` vs `'blob'` output mode, or something else — and give you the exact corrected package body.
