That likely means `AOP_API_PKG` isn't the real package name in your schema — it's probably a synonym pointing to something else (the docs I found earlier referenced a package called `AOP_API24_PKG` for some AOP versions). Let's find the real object first.

Run this:
```sql
select object_name, object_type, status
from user_objects
where object_name like '%AOP%'
order by object_name;
```

If nothing comes back, also try:
```sql
select owner, object_name, object_type
from all_objects
where object_name like '%AOP%'
and object_type in ('PACKAGE','SYNONYM')
order by owner, object_name;
```

This will tell us the exact package name (and its owning schema, if it's not yours) so we can pull its real constant list.
