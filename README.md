No AOP tables at all — so its configuration isn't stored in a settings table. It's likely passed via explicit parameters (`p_aop_url`, `p_api_key`) on each call, or stored as an APEX **Application Item** or **Preference**. Let's check both.

**Step 1.** Run this:
```sql
select item_name 
from apex_application_items 
where application_id = 120 
and upper(item_name) like '%AOP%';
```

**Step 2.** Also run this:
```sql
select preference_name 
from apex_application_preferences 
where application_id = 120 
and upper(preference_name) like '%AOP%';
```

Paste both results — one of these should reveal where the AOP server URL and API key are meant to be stored, so we can either reference them correctly in our package call or realize they were never set up at all.
