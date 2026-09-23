




TEXT
g_api_key varchar2(50 char) := null; -- AOP API Key; only needed when AOP Cloud is used (http(s)://www.apexofficeprint.com/api)
function is_valid_template_hash(p_aop_url in varchar2 default g_aop_url,
* @Param: p_aop_url Description: URL where the AOP Server is running. For the AOP Cloud use c_aop_url
* @Param: p_api_key Description: API Key which can be found when you login at https://www.apexofficeprint.com
* p_aop_url => 'http://localhost:8010');
* p_aop_url => 'http://api.apexofficeprint.com',
* p_api_key => '<your API key>', -- change the API key if you use the AOP Cloud
p_aop_url in varchar2 default null,
p_api_key in varchar2 default null,
p_api_key in varchar2 default null,
p_aop_url in varchar2,




c_aop_version constant varchar2(6 char) := '24.3.0';
-- The default url for the AOP Server
c_aop_url constant varchar2(50 char) := 'http://api.apexofficeprint.com/';
-- The default url for the AOP Fallback Server in case the c_aop_url would fail
c_aop_url_fallback constant varchar2(50 char) := 'https://api-eu.apexofficeprint.com/';
-- The default secure url for the AOP Server
c_aop_url_secure constant varchar2(50 char) := 'https://api.apexofficeprint.com/';
-- The default secure url for the AOP Fallback Server
c_aop_url_secure_fallback constant varchar2(50 char) := 'https://api-eu.apexofficeprint.com/';
c_aop_url_oci_us constant varchar2(50 char) := 'https://api-us.apexofficeprint.com/';
c_aop_url_oci_eu constant varchar2(50 char) := 'https://api-eu.apexofficeprint.com/';
c_aop_url_oci_apac constant varchar2(50 char) := 'https://api-apac.apexofficeprint.com/';
c_aopreport constant varchar2(9 char) := 'aopreport';
g_logger_enabled boolean := true; -- In case you use Logger (https://github.com/OraOpenSource/Logger), you can compile this package to enable Logger output:
g_https_host varchar2(300 char) := null; -- The host name to be matched against the common name (CN) of the remote server's certificate for an HTTPS request.
g_rpt_header_font_name varchar2(50 char) := ''; -- Arial - see https://www.microsoft.com/typography/Fonts/product.aspx?PID=163
g_rpt_data_font_name varchar2(50 char) := ''; -- Arial - see https://www.microsoft.com/typography/Fonts/product.aspx?PID=163
g_url_https_host varchar2(300 char) := null; -- parameter for apex_web_service, not used, please apply APEX patch if issues
function is_valid_template_hash(p_aop_url in varchar2 default g_aop_url,
* - c_debug_remote: store the JSON in your dashboard on https://www.apexofficeprint.com
* @Param: p_aop_url Description: URL where the AOP Server is running. For the AOP Cloud use c_aop_url
* @Param: p_api_key Description: API Key which can be found when you login at https://www.apexofficeprint.com
* @Param: p_failover_aop_url: URL where the AOP Failover Server is running. For the AOP Cloud use c_aop_url_fallback
* @Param: p_media_files_sql: Coming soon (!); use AME API via https://www.apexmediaextension.com
