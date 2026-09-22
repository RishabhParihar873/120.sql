The filename matches exactly (`ict_qa_template.docx`) — so it's not a naming/casing issue. This narrows it down to **execution context**: `aop_api_pkg.plsql_call_to_aop` likely needs to be told explicitly *which application's* Static Files to look in via a `p_app_id` parameter — and our current call doesn't pass one.

Here's why this matters specifically for your setup: an AI Agent tool call happens in a slightly different session context than a normal page request. If `plsql_call_to_aop` relies on an implicit `v('APP_ID')` internally instead of an explicit parameter, it may not resolve correctly when triggered from the chat/tool-call context — so it searches the wrong (or no) application and reports "not found," even though the file is sitting right there in app 120.

Let's confirm the function actually has this parameter available. Run:
```sql
select argument_name, data_type, in_out, defaulted
from user_arguments
where object_name = 'AOP_API24_PKG'
and package_name is null  -- adjust if needed
order by position;
```

If that returns nothing (function might be nested under package), try instead:
```sql
select argument_name, data_type, in_out, defaulted
from user_arguments
where object_name = 'PLSQL_CALL_TO_AOP'
order by position;
```

Paste the result — I want to see if `p_app_id` (and possibly `p_page_id`) exist as parameters so I can add them explicitly to the call with `v('APP_ID')`.
