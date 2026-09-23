No worries — that view name was wrong for this version. Let's go straight to the source instead: check the package itself for where the AOP server URL is supposed to come from — either a hardcoded default, or a parameter default pointing at something.

**Step 1.** Run this:
```sql
select text 
from user_source 
where name = 'AOP_API24_PKG' 
and type = 'PACKAGE' 
and (upper(text) like '%P_AOP_URL%' or upper(text) like '%P_API_KEY%')
order by line;
```

**Step 2.** Also run this — it'll show the actual default values, if any, baked into the package spec:
```sql
select text 
from user_source 
where name = 'AOP_API24_PKG' 
and type = 'PACKAGE' 
and (upper(text) like '%DEFAULT%URL%' or upper(text) like '%HTTPS%' or upper(text) like '%C_AOP%')
order by line;
```

Paste both — this should tell us whether there's a default AOP server URL baked in (and possibly broken/blank) or whether every call is required to explicitly pass `p_aop_url` and we've simply been missing that parameter entirely.
