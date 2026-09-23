Found it — simple fix. Recall from way back when we listed all the parameters: `P_OUTPUT_FILENAME` is `IN/OUT`, not just `IN`. That means it can't accept a literal string directly — it needs a variable, since the function may modify it. This is exactly what "cannot be used as an assignment target" means.

**Step 1.** Run this corrected version — same test, just using a variable for the filename:
```sql
DECLARE
    l_blob            BLOB;
    l_output_filename VARCHAR2(255) := 'test_output.pdf';
BEGIN
    l_blob := aop_api_pkg.plsql_call_to_aop(
        p_data_type       => aop_api_pkg.c_source_type_json,
        p_data_source     => '{"title":"Test","sections":[{"heading":"Overview","body":"Test body"}]}',
        p_template_type   => aop_api_pkg.c_source_type_apex,
        p_template_source => 'ict_qa_template.docx',
        p_output_type     => 'pdf',
        p_output_filename => l_output_filename,
        p_app_id          => 120,
        p_aop_url         => aop_api_pkg.c_aop_url_secure,
        p_api_key         => '5C23AC68C7438289E0637203000AFBD2'
    );

    dbms_output.put_line('SUCCESS - blob length: ' || dbms_lob.getlength(l_blob));
EXCEPTION
    WHEN OTHERS THEN
        dbms_output.put_line('ERROR: ' || SQLERRM);
END;
/
```

**Step 2.** Make sure DBMS Output is enabled/visible so the printed line shows up.

**Step 3.** Screenshot the result.

This also means our actual package body has had this same bug hiding in it the whole time — `l_file_name` is a plain `VARCHAR2` variable (not the issue there, that part's fine since we already declared it as a variable, not a literal) — so the package itself should be okay on this specific point. This test will confirm whether the real blocker was purely the missing app_id/URL/key, or if there's still something else.
