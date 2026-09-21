Here is your complete, end-to-end master guide for adding the AI Document Generation feature to your Oracle APEX application. Follow these steps sequentially.

---

### STEP 1: Compile the PL/SQL Package
Go to **SQL Workshop ➔ SQL Commands**. Copy the code below, paste it into the command window, and click **Run**. 

```sql
CREATE OR REPLACE PACKAGE ict_universal_doc_engine AS
    FUNCTION generate_document (
        p_target_format    IN VARCHAR2, 
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
        -- 1. Route template based on format
        CASE UPPER(p_target_format)
            WHEN 'PDF' THEN
                l_template_file := 'APP_FILES:ict_qa_template.docx';
                l_output_type   := 'pdf'; 
                l_file_ext      := '.pdf';
                l_mime_type     := 'application/pdf';

            WHEN 'DOCX' THEN
                l_template_file := 'APP_FILES:ict_qa_template.docx';
                l_output_type   := 'docx';
                l_file_ext      := '.docx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.wordprocessingml.document';

            WHEN 'XLSX' THEN
                l_template_file := 'APP_FILES:ict_data_template.xlsx';
                l_output_type   := 'xlsx';
                l_file_ext      := '.xlsx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';

            WHEN 'PPTX' THEN
                l_template_file := 'APP_FILES:ict_presentation_template.pptx';
                l_output_type   := 'pptx';
                l_file_ext      := '.pptx';
                l_mime_type     := 'application/vnd.openxmlformats-officedocument.presentationml.presentation';

            ELSE
                RAISE_APPLICATION_ERROR(-20001, 'Unsupported document format requested.');
        END CASE;

        -- 2. Call AOP Function
        l_file_name := REGEXP_REPLACE(p_title, '[^a-zA-Z0-9_]', '_') || '_' || TO_CHAR(SYSDATE, 'YYYYMMDD_HH24MI') || l_file_ext;

        l_return_blob := aop_api_pkg.plsql_call_to_aop(
            p_data_type        => 'json',
            p_data_source      => p_payload_json,
            p_template_type    => 'file',
            p_template_source  => l_template_file,
            p_output_type      => l_output_type,
            p_output_filename  => l_file_name,
            p_output_to        => 'directory'
        );

        -- 3. Save to Library
        INSERT INTO cgct_ai_document_library (
            name, file_name, file_path1, mime_type, file_folder, created_by, creation_date, package_id 
        ) VALUES (
            p_title, l_file_name, l_return_blob, l_mime_type, 'N', v('APP_USER'), SYSDATE, v('APP_ID')
        ) RETURNING id INTO l_new_doc_id;

        -- 4. Create response
        l_download_url := 'f?p=' || v('APP_ID') || ':920:' || v('APP_SESSION') || ':::P920_ID:' || l_new_doc_id;

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
```
*(Ensure it says "Package Body Created" before moving on).*

---

### STEP 2: Create and Upload the Templates
1. **Create the files on your computer:**
   * **`ict_qa_template.docx`** (Word): Type `{document_title}` at the top. Below it, type: `{#qa_items} Q: {question} A: {answer} {/qa_items}`
   * **`ict_data_template.xlsx`** (Excel): In cell A1, type: `{#tables}{#rows}{0} {1} {2}{/rows}{/tables}`
   * **`ict_presentation_template.pptx`** (PowerPoint): On Slide 1, type `{document_title}`. On Slide 2, type `{#slides}{slide_title}` as the header, and `{bullets}{/slides}` as the content.
2. **Upload them:**
   * Go to **Shared Components ➔ Static Application Files** ➔ **Create**
   * Leave the **Directory** field blank.
   * Upload all three files. Verify their names in the list are exactly as written above.

---

### STEP 3: Create the AI Agent Tool
1. Go to **Shared Components ➔ AI Agents** and click on your agent (`ict-ai-assistant`).
2. Scroll to **Tools** and click **Create Tool**:
   * **Tool Name:** `generate_custom_document`
   * **Tool Type:** `Execute Code` (Select **Server-Side Code** or **PL/SQL** if it asks)
   * **Execution Point:** `Action` (or `Tool Invocation`)
   * **Description:** 
     > Call this tool when the user asks to generate, export, or create a document (PDF, Excel, PowerPoint, or Word) based on data, questions, or analysis. Pass the target format, document title, and the structured JSON payload.
3. **PL/SQL Code to Execute:**
   ```sql
   :RETURN_VALUE := ict_universal_doc_engine.generate_document(
       p_target_format => :p_target_format,
       p_title         => :p_title,
       p_payload_json  => :p_payload_json
   );
   ```

---

### STEP 4: Add the 3 Tool Parameters
Inside the Tool you just created, scroll down to **Parameters** and add these exactly:

**1. `p_target_format`**
* **Type:** String 
* **Required:** Yes
* **Description:** `The format requested by the user. Must be exactly one of: 'PDF', 'DOCX', 'XLSX', or 'PPTX'`

**2. `p_title`**
* **Type:** String 
* **Required:** Yes
* **Description:** `A short, professional file name or title for the document based on the topic.`

**3. `p_payload_json`**
* **Type:** String (or CLOB)
* **Required:** Yes
* **Description:** Copy and paste the following text exactly:
  > `Construct a single, valid JSON object payload. For PPTX, use {"document_title":"...", "slides": [{"slide_title":"...", "bullets":"..."}]}. For PDF/DOCX, use {"document_title":"...", "qa_items": [{"question":"...", "answer":"..."}]}. For XLSX, use {"tables":[{"rows":[["Col1","Col2"], ["Val1","Val2"]]}]}. Ensure JSON is properly escaped.`

---

### STEP 5: Update the Agent System Prompt
Go back to the main settings page for your AI Agent (`ict-ai-assistant`). Add this rule to the very bottom of the **System Prompt** text box:

> **Document Generation Rule:**
> If the user asks you to create, generate, or export a file (like a PDF, Word document, Excel, or PowerPoint presentation) based on uploaded documents or project data: Intelligently format the answers or data into a JSON structure and call the `generate_custom_document` tool to produce the file. When the tool returns successfully, provide the download URL to the user.

***

**You are done.** Save everything. Go to your app's chat window and type: *"I uploaded a document. Read it and generate a PPT explaining it"* and watch it work!
