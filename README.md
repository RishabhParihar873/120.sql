Use this as the first version of `get_current_page_info`.

### SQL Query

```sql
SELECT
    p.page_id       AS "Page ID",
    p.page_name     AS "Page Name",
    p.page_title    AS "Page Title",
    r.region_name   AS "Region Name",
    r.source_type   AS "Region Type"
FROM apex_application_pages p
LEFT JOIN apex_application_page_regions r
       ON r.application_id = p.application_id
      AND r.page_id        = p.page_id
WHERE p.application_id = :APP_ID
  AND p.page_id        = :APP_PAGE_ID
ORDER BY r.display_sequence
```

### Data Description

```text
Returns information about the currently displayed APEX page.

Use the returned information to explain the current page when the user asks questions such as:
- What is this page?
- What does this page do?
- What is this page used for?
- What sections are on this page?

The result contains the page ID, page name, page title, and the regions available on the current page.

Use the page name, title, and region names to explain the purpose and functionality of the page.

Do not invent functionality that is not supported by the returned page information.

If no page information is returned, state that information about the current page could not be found.
```

### Settings

- **Tool:** `get_current_page_info`
- **Type:** `Retrieve Data`
- **Execution Point:** `Augment System Prompt`
- **Server-side Condition:** **Blank**
- **Type:** `SQL Query`
- **Max Tokens:** `500`

One thing before testing: **don't create the tool and test yet.** There is a potential issue with using `:APP_PAGE_ID` inside an AI Agent SQL query, so after you create it, we'll verify that the agent actually receives the current page context rather than assuming it does.
