If it already exists, then good — you're not starting from zero. Here's what to check, in order:

**Check 1 — What does it actually return?**
Run this in SQL Workshop → SQL Commands:
```sql
select text 
from user_source 
where name = 'ICT_UNIVERSAL_DOC_ENGINE' 
and type = 'PACKAGE'
order by line;
```
This shows the package spec — look specifically for the `generate_document` function signature. I need to see:
- Its return type (`clob`? `varchar2`? `blob`?)
- What it actually does with `p_target_format`, `p_title`, `p_payload_json` internally

Paste that output here.

**Check 2 — Does it return a finished file, or just a link/ID?**
There are two possible designs, and it changes what our tool code needs to do:
- **Design A:** `generate_document` builds the PDF right there and returns a download URL (like `f?p=&APP_ID.:80:&SESSION.::::P_DOC_ID:abc123`) — in this case, Step 1's fixed code is *already enough*, nothing more needed.
- **Design B:** `generate_document` returns raw file content or just an ID, and something else (a separate download page/process) is needed to actually serve the file to the browser — in this case we still need Steps 6-7.

**Check 3 — Does a download page exist?**
Go to App Builder → search for any page with "download" or "doc" in the name, or check Shared Components → Application Processes for one handling file download. Tell me what you find (or if nothing turns up).

Send me the package source from Check 1 first — that alone will tell me most of what I need, and I can then tell you exactly whether you're done or what's missing.
