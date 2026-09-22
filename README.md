This confirms everything. Here's the exact fix — replace two things in the package body:

1. `p_template_type => 'file'` → `p_template_type => aop_api_pkg.c_source_type_apex` (value `'APEX'`, meaning "file uploaded in APEX Static Application Files" — which is exactly where your templates are)
2. `p_data_type => 'json'` → `p_data_type => aop_api_pkg.c_source_type_json` (value `'JSON'`) — using the real constant instead of a hardcoded lowercase string avoids any case-sensitivity surprise
3. Drop the `'APP_FILES:'` prefix from the template filenames — `c_source_type_apex` expects just the plain filename as it appears in Static Application Files, not a prefixed string

**Corrected package body:**

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
            l_template_file := 'ict_qa_template.docx';
            l_output_type   := 'pdf';
            l_file_ext      := '.pdf';
            l_mime_type     := 'application/pdf';
        WHEN 'DOCX' THEN
            l_template_file := 'ict_qa_template.docx';
            l_output_type   := 'docx';
            l_file_ext      := '.docx';
            l_mime_type     := 'application/vnd.openxmlformats-officedocument.wordprocessingml.document';
        WHEN 'XLSX' THEN
            l_template_file := 'ict_data_template.xlsx';
            l_output_type   := 'xlsx';
            l_file_ext      := '.xlsx';
            l_mime_type     := 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
        WHEN 'PPTX' THEN
            l_template_file := 'ict_presentation_template.pptx';
            l_output_type   := 'pptx';
            l_file_ext      := '.pptx';
            l_mime_type     := 'application/vnd.openxmlformats-officedocument.presentationml.presentation';
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
        p_output_filename => l_file_name
    );

    INSERT INTO cgct_ai_document_library (
        name, file_name, file_path1, mime_type, file_folder,
        created_by, creation_date, package_id
    ) VALUES (
        p_title, l_file_name, l_return_blob, l_mime_type, 'N',
        v('APP_USER'), SYSDATE, v('APP_ID')
    ) RETURNING id INTO l_new_doc_id;

    l_download_url := 'f?p=' || v('APP_ID') || ':920:' || v('APP_SESSION') || ':::P920_ID:' || l_new_doc_id;

    RETURN json_object(
        'status' VALUE 'SUCCESS',
        'format' VALUE p_target_format,
        'file_name' VALUE l_file_name,
        'document_id' VALUE l_new_doc_id,
        'download_link' VALUE l_download_url
    );

EXCEPTION
    WHEN OTHERS THEN
        RETURN json_object(
            'status' VALUE 'ERROR',
            'message' VALUE SQLERRM
        );
END generate_document;

END ict_universal_doc_engine;
/
```

**Important — I also removed `p_output_to => 'directory'`.** That was the risk I flagged earlier: it tells AOP to save the file to a server directory instead of handing the BLOB back to your PL/SQL. Since your code needs `l_return_blob` to actually contain the file (to insert into `file_path1` and serve it later), leaving that parameter out lets AOP default to returning the BLOB directly — which is what you need.

**Steps:**
1. Go to SQL Workshop → SQL Commands (or a SQL Script), paste the full corrected package body above, run it.
2. Confirm it compiles: `select status from user_objects where object_name = 'ICT_UNIVERSAL_DOC_ENGINE';` → should say `VALID`.
3. Go back to chat, try "generate me a PDF on car engines" again.

Paste whatever happens next — we're very close now.
