None of these rows are from your test — this table just has old, unrelated data (Finance, OICS, etc.), nothing recent. **This confirms the function's `INSERT` step never ran at all** — meaning it's failing earlier, most likely right at the AOP call itself, and our `WHEN OTHERS` is quietly catching it and returning a JSON error that the chat is just paraphrasing instead of showing you word-for-word.

Let's stop going through the slow chat→AI→tool round-trip entirely. We can call the function directly ourselves and see the exact error text immediately.

**Step 1.** Go to SQL Commands, paste this exactly, and run it:
```sql
select ict_universal_doc_engine.generate_document(
    p_target_format => 'PDF',
    p_title         => 'Test ICT Document',
    p_payload_json  => '{"title":"Test ICT Document","sections":[{"heading":"Overview","body":"This is a test."}]}'
) as result
from dual;
```

**Step 2.** Screenshot the result — this will show the exact JSON our function returns, including the real `SQLERRM` message if it's still failing, with zero AI paraphrasing in the way.

This single query should finally show us the precise, unfiltered error text.
