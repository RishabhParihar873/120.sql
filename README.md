Progress — but with an important caveat. This test ran through **SQL Commands**, which is its own separate mini-app, completely outside app 120's session. So `v('APP_ID')` inside the function almost certainly returned **NULL** here — meaning this specific "template not found" result might just be an artifact of testing this way, not necessarily the real chat failure.

But it also might be the *exact same* root cause hiding behind the chat's silent error — since we never actually confirmed `v('APP_ID')` resolves correctly even during a real AI tool call (this was my original suspicion, several steps back). Let's settle it directly: hardcode the app ID instead of relying on `v('APP_ID')`.

**Step 1.** Run this direct test — same as before, but explicitly confirms app_id resolution isn't the issue by never depending on `v()`:
```sql
select aop_api_pkg.plsql_call_to_aop(
    p_data_type       => aop_api_pkg.c_source_type_json,
    p_data_source     => '{"title":"Test","sections":[{"heading":"Overview","body":"Test body"}]}',
    p_template_type   => aop_api_pkg.c_source_type_apex,
    p_template_source => 'ict_qa_template.docx',
    p_output_type     => 'pdf',
    p_output_filename => 'test_output.pdf',
    p_app_id          => 120,
    p_aop_url         => aop_api_pkg.c_aop_url_secure,
    p_api_key         => '5C23AC68C7438289E0637203000AFBD2'
) as result
from dual;
```

**Step 2.** This returns a BLOB, so SQL Commands won't display it directly as text — it'll likely show something like `(BLOB)` or offer a download link. That's fine. What matters is: does it **error**, or does it run successfully?

**Step 3.** Screenshot whatever result/error appears.

If this succeeds (no error), we know the real fix is: always hardcode `p_app_id => 120` in the package instead of `v('APP_ID')`. If it *still* errors the same way, the problem isn't app_id resolution at all — it's something about how AOP is matching the template itself, and we dig further from there.
