Got the key. Let's wire it in.

**Step 1.** Run this in SQL Commands — this adds the AOP URL and API key explicitly to the call:

```sql
CREATE OR REPLACE PACKAGE BODY ict_universal_doc_engine AS

FUNCTION generate_document (
    p_target_format IN VARCHAR2,
    p_title         IN VARCHAR2,
    p_payload_json  IN CLOB
) RETURN CLOB
IS
    l_template_file  VARCHAR2(100);
    l_output_type    VARCHAR2(10);
    l_mime_type      VARCHAR2(100);
    l_file_ext       VARCHAR2(10);
    l_return_blob    BLOB;
    l_new_doc_id     NUMBER;
    l_file_name      VARCHAR2(255);
    l_download_url   VARCHAR2(1000);
BEGIN
    CASE UPPER(p_target_format)
        WHEN 'PDF' THEN
            l_template_file := 'ict_qa_template.docx'; l_output_type := 'pdf'; l_file_ext := '.pdf'; l_mime_type := 'application/pdf';
        WHEN 'DOCX' THEN
            l_template_file := 'ict_qa_template.docx'; l_output_type := 'docx'; l_file_ext := '.docx'; l_mime_type := 'application/vnd.openxmlformats-officedocument.wordprocessingml.document';
        WHEN 'XLSX' THEN
            l_template_file := 'ict_data_template.xlsx'; l_output_type := 'xlsx'; l_file_ext := '.xlsx'; l_mime_type := 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
        WHEN 'PPTX' THEN
            l_template_file := 'ict_presentation_template.pptx'; l_output_type := 'pptx'; l_file_ext := '.pptx'; l_mime_type := 'application/vnd.openxmlformats-officedocument.presentationml.presentation';
        ELSE
            RAISE_APPLICATION_ERROR(-20001, 'Unsupported document format requested.');
    END CASE;

    l_file_name := REGEXP_REPLACE(p_title, '[^a-zA-Z0-9_]', '_') || '_' || TO_CHAR(SYSDATE, 'YYYYMMDD_HH24MI') || l_file_ext;

    l_return_blob := aop_api_pkg.plsql_call_to_aop(
        p_data_type       => aop_api_pkg.c_source_type_json,
        p_data_source     => p_payload_json,
        p_template_type   => aop_api_pkg.c_source_type_apex,
        p_template_source => l_template_file,
        p_output_type     => l_output_type,
        p_output_filename => l_file_name,
        p_app_id          => v('APP_ID'),
        p_aop_url         => aop_api_pkg.c_aop_url_secure,
        p_api_key         => '5C23AC68C7438289E0637203000AFBD2'
    );

    INSERT INTO cgct_ai_document_library (
        name, file_name, file_path1, mime_type, file_folder, created_by, creation_date, package_id
    ) VALUES (
        p_title, l_file_name, l_return_blob, l_mime_type, 'N', v('APP_USER'), SYSDATE, v('APP_ID')
    ) RETURNING id INTO l_new_doc_id;

    l_download_url := apex_page.get_url(
        p_application  => v('APP_ID'),
        p_page         => 1095,
        p_items        => 'P1095_ID',
        p_values       => l_new_doc_id,
        p_absolute_url => true
    );

    RETURN json_object(
        'status' VALUE 'SUCCESS', 'format' VALUE p_target_format, 'file_name' VALUE l_file_name,
        'document_id' VALUE l_new_doc_id, 'download_link' VALUE l_download_url
    );

EXCEPTION
    WHEN OTHERS THEN
        RETURN json_object('status' VALUE 'ERROR', 'message' VALUE SQLERRM);
END generate_document;

END ict_universal_doc_engine;
/
```

**Step 2.** Confirm valid:
```sql
select status from user_objects where object_name = 'ICT_UNIVERSAL_DOC_ENGINE';
```

**Step 3.** Go to chat, test: "generate me a pdf explaining ICT"

**Step 4.** Screenshot the result. This time, if you get a download link, actually click it and confirm a real PDF opens.

One heads up for later: that API key is now sitting in plain text inside your package source, visible to anyone who can view the code. Fine for testing right now — but before this goes anywhere real, it should move into a proper secured place (an APEX credential store or app setting) instead of hardcoded in the package. We'll deal with that once we know the core flow actually works.
