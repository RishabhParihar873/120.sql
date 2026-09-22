select text 
from user_source 
where name = 'AOP_API_PKG' 
and type = 'PACKAGE' 
and text like '%c_source_type%' 
order by line;
