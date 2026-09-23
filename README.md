That error is specific to running this as a raw `select ... from dual` — Oracle's SQL engine sometimes struggles resolving package constants through a synonym that way, even though the same constant works fine *inside* PL/SQL code (like our package body, which compiled fine). This isn't a sign anything is actually broken — it's just the wrong test format. Let's use a proper PL/SQL block instead, which is how it actually runs for real.

**Step 1.** Run this instead:
```sql
DECLARE
    l_blob BLOB;
BEGIN
    l_blob := aop_api_pkg.plsql_call_to_aop(
        p_data_type       => aop_api_pkg.c_source_type_json,
        p_data_source     => '{"title":"Test","sections":[{"heading":"Overview","body":"Test body"}]}',
        p_template_type   => aop_api_pkg.c_source_type_apex,
        p_template_source => 'ict_qa_template.docx',
        p_output_type     => 'pdf',
        p_output_filename => 'test_output.pdf',
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

**Step 2.** Make sure **DBMS Output** is enabled so you can see the printed line — in SQL Commands there should be an option/toggle for this, or switch to **SQL Scripts** and check "Enable DBMS Output" if using that tool instead.

**Step 3.** Screenshot whatever prints — either "SUCCESS - blob length: [some number]" or "ERROR: [exact message]".

This directly tests whether hardcoding `p_app_id => 120` fixes the original "template not found" error, without the dual-select quirk getting in the way.
