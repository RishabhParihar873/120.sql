Here is the step-by-step guide to fix both problems in Oracle APEX.

---

### Step 1: Fix the PL/SQL Code in the AI Agent Tool

The error `ERR-1002 UNABLE TO FIND Item id for "return_value"` occurs because APEX thinks `:RETURN_VALUE` is a page item.

1. Navigate to **App Builder** and open **App 120**.
2. Go to **Shared Components** ➔ **AI Agents**.
3. Click on your agent (**`ict-ai-assistant`**).
4. In the **Tools** section, click on the **`generate_custom_document`** tool.
5. In the **PL/SQL Code** text area, replace whatever is currently there with this exact code:

```sql
return ict_universal_doc_engine.generate_document(
    p_target_format => :p_target_format,
    p_title         => :p_title,
    p_payload_json  => :p_payload_json
);
```

*(Note: Use `return`, not `:RETURN_VALUE :=`).*

6. Click **Apply Changes** or **Save**.

---

### Step 2: Update Tool Parameter Descriptions

Make sure each parameter explicitly tells the AI model that *it* must generate the value rather than asking the user.

In the same **`generate_custom_document`** tool:

1. Click on parameter **`p_target_format`**:
   * **Required**: Yes
   * **Description**: `The document type requested by the user. Must be one of: 'PDF', 'DOCX', 'XLSX', or 'PPTX'.`

2. Click on parameter **`p_title`**:
   * **Required**: Yes
   * **Description**: `Generate a concise, professional title based on the topic requested by the user. Do not ask the user for this.`

3. Click on parameter **`p_payload_json`**:
   * **Required**: Yes
   * **Description**: `You must generate and construct this JSON payload autonomously based on the user's prompt. Do not ask the user for content or slide counts. For PPTX use {"document_title":"...", "slides": [{"slide_title":"...", "bullets":"..."}]}. For PDF/DOCX use {"document_title":"...", "qa_items": [{"question":"...", "answer":"..."}]}. For XLSX use {"tables":[{"rows":[["Col1","Col2"], ["Val1","Val2"]]}]}.`

4. Click **Apply Changes** or **Save**.

---

### Step 3: Update the Agent System Prompt

This ensures the AI takes full initiative to write all content on its own.

1. In **Shared Components** ➔ **AI Agents**, click on **`ict-ai-assistant`**.
2. Scroll to the **System Prompt** section.
3. Add or replace the document generation instruction with the following:

```text
DOCUMENT GENERATION INSTRUCTIONS:
When a user asks to create, make, build, or export any document (such as a PDF, Word document, Excel spreadsheet, or PowerPoint presentation):

1. AUTONOMOUS CREATION: You are fully responsible for generating all the contents, outline, slides, questions, answers, and data. NEVER ask the user for titles, slide counts, or content details unless they specifically asked you to clarify.
2. COMPOSE PAYLOAD: Choose an appropriate format ('PDF', 'DOCX', 'XLSX', or 'PPTX'), create a title, and build the required JSON payload with detailed, high-quality content.
3. EXECUTE: Call the `generate_custom_document` tool immediately with the generated parameters.
4. RESPOND: When the tool returns a SUCCESS response, present the document name and provide the clickable download link to the user.
```

4. Click **Apply Changes**.

---

### Step 4: Test in the Chat Window

1. Open your APEX application and go to the AI chat page.
2. Type a simple prompt, for example:
   * `"Build me a PDF explaining machine learning basics"`
   * `"Create a PowerPoint presentation on quarterly project milestones"`
3. The AI should generate the full document outline in the background and return the download link directly.
