Found it — `AOP_API_PKG` is just a **synonym** pointing to the real package `AOP_API24_PKG`. That's why `user_source` returned nothing (synonyms aren't in `user_source`, only the real package is).

Run this instead, using the real name:
```sql
select text 
from user_source 
where name = 'AOP_API24_PKG' 
and type = 'PACKAGE' 
and text like '%c_source_type%' 
order by line;
```

This should list the actual constants (like `c_source_type_apex`, `c_source_type_json`, etc.) we need to plug into the package body fix.
