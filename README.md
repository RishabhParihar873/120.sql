Confirmed — `P_APP_ID` exists as an optional parameter, and it's very likely not being passed correctly right now since AI Agent tool calls run outside a normal page session context (where `v('APP_ID')` usually resolves automatically). Let's pass it explicitly.

**Update the package body's AOP call to this:**

```sql
l_return_blob := aop_api_pkg.plsql_call_to_aop(
    p_data_type       => aop_api_pkg.c_source_type_json,
    p_data_source     => p_payload_json,
    p_template_type   => aop_api_pkg.c_source_type_apex,
    p_template_source => l_template_file,
    p_output_type     => l_output_type,
    p_output_filename => l_file_name,
    p_app_id          => v('APP_ID')
);
```

That's the only change — just adding `p_app_id => v('APP_ID')` as an explicit parameter.

**Do this:**
1. Recompile the package body with just that one addition (I can give you the full body again if you want to paste-replace the whole thing — just say so).
2. Confirm valid: `select status from user_objects where object_name = 'ICT_UNIVERSAL_DOC_ENGINE';`
3. Test in chat again: "generate me a PDF on car engines"

If `v('APP_ID')` itself returns NULL in the tool's execution context (which is possible — AI tool calls might not carry full page session state), we'll need to hardcode `120` instead as a fallback test, just to isolate whether that's the actual problem. But let's try the clean version first and see what error (if any) comes back.
