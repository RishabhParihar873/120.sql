CREATE OR REPLACE PACKAGE ict_universal_doc_engine AS
    -- Main function called by the AI Agent Tool
    FUNCTION generate_document (
        p_target_format    IN VARCHAR2, -- 'PDF', 'DOCX', 'XLSX', 'PPTX'
        p_title            IN VARCHAR2,
        p_payload_json     IN CLOB
    ) RETURN CLOB;
END ict_universal_doc_engine;
/

CREATE OR REPLACE PACKAGE BODY ict_universal_doc_engine AS

    FUNCTION generate_document (
        p_target_format    IN VARCHAR2,
        p_title            IN VARCHAR2,
        p_payload_json     IN CLOB
    ) RETURN CLOB
    IS
        l_template_file    VARCHAR2(100);
        l_output_type      VARCHAR2(10);
        l_mime_type        VARCHAR2(100);
        l_file_ext         VARCHAR2(10);
        l_return_blob      BLOB;
        l_new_doc_id       NUMBER;
        l_file_name        VARCHAR2(255);
        l_download_url     VARCHAR2(1000);
    BEGIN
        -- 1. Route template and output type based on format requested by AI
        CASE UPPER(p_target_format)
            WHEN 'PDF' THEN
                l_template_file := 'STATIC_FILE:ict_qa_template.docx';
                l_output_type   := 'pdf'; 
                l_file_ext      := '.pdf';
                l_mime_type     := 'application/pdf';

            WHEN 'DOCX' THEN
                l_template_file := 'STATIC_FILE:ict_qa_template.docx';
                l_output_type   := 'docx';
                l_file_ext      := '.docx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.wordprocessingml.document';

            WHEN 'XLSX' THEN
                l_template_file := 'STATIC_FILE:ict_data_template.xlsx';
                l_output_type   := 'xlsx';
                l_file_ext      := '.xlsx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';

            WHEN 'PPTX' THEN
                l_template_file := 'STATIC_FILE:ict_presentation_template.pptx';
                l_output_type   := 'pptx';
                l_file_ext      := '.pptx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.presentationml.presentation';

            ELSE
                RAISE_APPLICATION_ERROR(-20001, 'Unsupported format.');
        END CASE;

        -- 2. Call AOP (Merging JSON with Template)
        aop_api_pkg.plsql_call_to_aop(
            p_data_type       => 'json',
            p_data_source     => p_payload_json,
            p_template_type   => CASE 
                                   WHEN UPPER(p_target_format) IN ('PDF','DOCX') THEN 'docx'
                                   WHEN UPPER(p_target_format) = 'XLSX' THEN 'xlsx'
                                   WHEN UPPER(p_target_format) = 'PPTX' THEN 'pptx'
                                 END,
            p_template_source => l_template_file,
            p_output_type     => l_output_type,
            p_output_blob     => l_return_blob
        );

        -- 3. File Name Generation
        l_file_name := REGEXP_REPLACE(p_title, '[^a-zA-Z0-9_]', '_') || '_' || TO_CHAR(SYSDATE, 'YYYYMMDD_HH24MI') || l_file_ext;

        -- 4. Store the generated document in your existing Library table
        INSERT INTO cgct_ai_document_library (
            name,
            file_name,
            file_path1,
            mime_type,
            file_folder,
            created_by,
            creation_date,
            package_id
        ) VALUES (
            p_title,
            l_file_name,
            l_return_blob,
            l_mime_type,
            'N',
            v('APP_USER'),
            SYSDATE,
            v('P0_APPLICATION_ID')
        ) RETURNING id INTO l_new_doc_id;

        -- Create download link (using your existing download setup, usually Page 920 or similar)
        l_download_url := 'f?p=' || v('APP_ID') || ':920:' || v('APP_SESSION') || ':::P920_ID:' || l_new_doc_id;

        -- 5. Return success JSON for the AI Agent
        RETURN json_object(
            'status'        VALUE 'SUCCESS',
            'format'        VALUE p_target_format,
            'file_name'     VALUE l_file_name,
            'document_id'   VALUE l_new_doc_id,
            'download_link' VALUE l_download_url
        );

    EXCEPTION
        WHEN OTHERS THEN
            RETURN json_object('status' VALUE 'ERROR', 'message' VALUE SQLERRM);
    END generate_document;

END ict_universal_doc_engine;
/
