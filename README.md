This is a huge find — the error is NOT coming from `apex_page.get_url` at all. Look at the actual call stack, read bottom to top:

```
ICT_UNIVERSAL_DOC_ENGINE.GENERATE_DOCUMENT (line 33)
  → AOP_API24_PKG.PLSQL_CALL_TO_AOP (line 7550)
    → WWV_FLOW_WEBSERVICES_API.MAKE_REST_REQUEST_B
      → ...REST request fails: "Invalid URL"
```

**Line 33 in your package is the AOP call itself** — meaning we already passed that point correctly (template found, data type fine). The failure is happening *inside* AOP's own code, when it tries to make an outbound REST/HTTP call **to the AOP print server** — and that server's URL is invalid or missing. This has nothing to do with our download page or `apex_page.get_url` — we built that correctly, we just never got far enough to use it yet.

This means: **AOP isn't configured with a valid server URL in this workspace/app.** `plsql_call_to_aop` has an optional `p_aop_url` parameter — if it's not set explicitly and there's no valid default configured elsewhere, it tries to call an empty/blank URL and fails exactly like this.

**Step 1.** Check if AOP has a configured URL anywhere. Run:
```sql
select * from AOP_SETTINGS24_PKG$ 
where rownum <= 20;
```
If that table doesn't exist, try:
```sql
select table_name from user_tables where table_name like 'AOP%';
```
Paste whatever comes back — table names only is fine if the first query errors.
