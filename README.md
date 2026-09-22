Here's the full path to get there, step by step:

**Step 1 — Fix the tool's PL/SQL** (as discussed) so it actually runs without erroring:
```sql
declare
    l_result clob;
begin
    l_result := ict_universal_doc_engine.generate_document(
        p_target_format => :p_target_format,
        p_title         => :p_title,
        p_payload_json  => :p_payload_json
    );
    apex_ai.set_tool_result(l_result);
end;
```

**Step 2 — Make P_TARGET_FORMAT foolproof.** In the Parameters tab, set its **Allowed Values** to `PDF, DOCX, XLSX, PPTX`. This stops the model from guessing a wrong string. Also in the tool's **Description** (Identification tab), add: *"If the user doesn't mention a format, default to PDF."* Right now nothing tells the model what to do when format is unspecified — that's likely why it kept asking you.

**Step 3 — Make P_PAYLOAD_JSON self-explanatory to the model.** This is the actual fix for your "just say the topic and boom" requirement. In that parameter's Description field, define the exact JSON shape you expect, e.g.:
```
Generate and pass a JSON object with this structure:
{
  "title": "string",
  "sections": [
    { "heading": "string", "body": "string (detailed paragraph content)" }
  ]
}
You must write complete, detailed content yourself based on the user's topic.
Do not ask the user for outline, audience, or structure — infer and generate it.
```
The model writes the actual PDF content itself — your job is just telling it not to ask permission first.

**Step 4 — Update the tool's top-level Description** (what tells the model *when* to call this tool) to something like:
```
Call this tool immediately whenever the user asks for a document, PDF, report, 
or export on any topic. Do not ask clarifying questions about title, audience, 
or content — generate reasonable, complete content yourself and call the tool.
```
This is the real fix for the behavior in your first screenshot, where it kept interrogating you instead of just doing it.

**Step 5 — Check `ict_universal_doc_engine.generate_document` itself** actually produces a file, not just text. It needs to:
- Parse `p_payload_json`
- Build a real PDF (BLOB) — via a PDF-generation utility (AS_PDF3, PL/PDF, APEX Office Print, or similar — tell me what you're using and I'll help with that code specifically)
- Store the BLOB in a table (e.g. `ICT_GENERATED_DOCS` with a BLOB column + generated GUID)
- Return a **downloadable URL**, not raw content

**Step 6 — Serve the file.** You need an APEX page/application process (type "Download BLOB") that, given the doc ID/GUID, streams the file back. `generate_document` should return something like:
```
f?p=&APP_ID.:DOWNLOAD_PAGE:&SESSION.::::P_DOC_ID:<guid>
```

**Step 7 — Return a clickable link in chat.** `l_result` passed to `apex_ai.set_tool_result` should be markdown like `Here's your PDF: [Download](f?p=...)` — the AI Agent chat renders markdown links, so the user gets a clickable download right there.

**Step 8 — Test end-to-end:** type only *"give me a PDF on X"* with nothing else. If it still asks a question, the gap is in Step 3/4 (the model doesn't feel authorized to assume) — tighten that wording further.

Want to start with Step 1 (get it compiling) and confirm that works before we build out Steps 5–7 (actual file generation)? That's the safer order — no point wiring up markdown links if the tool call itself still errors.
