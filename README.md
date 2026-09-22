TEXT
c_source_type_apex constant varchar2(4 char) := 'APEX'; -- Template Type
c_source_type_workspace constant varchar2(9 char) := 'WORKSPACE'; -- Template Type
c_source_type_sql constant varchar2(3 char) := 'SQL'; -- Template and Data Type
c_source_type_plsql_sql constant varchar2(9 char) := 'PLSQL_SQL'; -- Template and Data Type
c_source_type_plsql constant varchar2(5 char) := 'PLSQL'; -- Template and Data Type
c_source_type_url constant varchar2(3 char) := 'URL'; -- Template and Data Type
c_source_type_url_aop constant varchar2(7 char) := 'URL_AOP'; -- Template Type
c_source_type_rpt constant varchar2(6 char) := 'IR'; -- Data Type
c_source_type_xml constant varchar2(3 char) := 'XML'; -- Data Type
c_source_type_json constant varchar2(4 char) := 'JSON'; -- Template and Data Type
c_source_type_json_files constant varchar2(10 char) := 'JSON_FILES'; -- Data Type
c_source_type_refcursor constant varchar2(9 char) := 'REFCURSOR'; -- Data Type
c_source_type_sql_array constant varchar2(9 char) := 'SQL_ARRAY'; -- Data Type
c_source_type_filename constant varchar2(8 char) := 'FILENAME'; -- Template Type
c_source_type_db_directory constant varchar2(12 char) := 'DB_DIRECTORY'; -- Template Type
c_source_type_aop_report constant varchar2(10 char) := 'AOP_REPORT'; -- Template Type
c_source_type_apex_report constant varchar2(11 char) := 'APEX_REPORT'; -- Template Type
c_source_type_apex_report_do constant varchar2(14 char):= 'APEX_REPORT_DO'; -- Template Type
c_source_type_layouts constant varchar2(14 char) := 'REPORT_LAYOUTS'; -- Template Type
c_source_type_aop_template constant varchar2(1 char) := null; -- Template Type
c_source_type_clob_base64 constant varchar2(11 char) := 'CLOB_BASE64'; -- Template Type
c_source_type_oci_objs constant varchar2(8 char) := 'OCI_OBJS'; -- Template Type
c_source_type_none constant varchar2(4 char) := 'NONE'; -- Template and Data Type
c_source_type_converter constant varchar2(9 char) := 'CONVERTER';
* Following constants exists in aop_api_pkg: c_source_type_sql, c_source_type_plsql_sql, c_source_type_plsql, c_source_type_url, c_source_type_rpt, c_source_type_refcursor, c_source_type_sql_array, c_source_type_xml, c_source_type_json, c_source_type_json_files, c_source_type_none
* - c_source_type_sql: SQL statement with cursor syntax or returning JSON
* - c_source_type_plsql_sql: PL/SQL function returning SQL statement with mime type and blob
* - c_source_type_plsql: PL/SQL function returning JSON with the template file base64 encoded
* - c_source_type_url: URL which contains the file
* - c_source_type_rpt: static id(s) or region id(s) of the APEX regions
* - c_source_type_refcursor: REF Cursor
* - c_source_type_sql_array: Array of SQL statements
* - c_source_type_xml: XML
* - c_source_type_json: JSON data part
* - c_source_type_json_files: JSON including files
* - c_source_type_none: leave the source blank
* Following constants exists in aop_api_pkg: c_source_type_apex, c_source_type_workspace, c_source_type_sql, c_source_type_plsql_sql, c_source_type_plsql,
* c_source_type_url, c_source_type_filename, c_source_type_url_aop, c_source_type_json, c_source_type_db_directory, c_source_type_oci_objs,
* c_source_type_aop_report, c_source_type_apex_report, c_source_type_aop_template, c_source_type_clob_base64, c_source_type_none
* - c_source_type_apex: file uploaded in APEX Static Application Files
* - c_source_type_workspace: file uploaded in APEX Workspace Files
* - c_source_type_sql: SQL statement returning mime type and blob
* - c_source_type_plsql_sql: PL/SQL function returning SQL statement with mime type and blob
* - c_source_type_plsql: PL/SQL function returning JSON with the template file base64 encoded
* - c_source_type_url: URL which contains the file (will be read from DB server)
* - c_source_type_url_aop: URL which contains the file (will be read from AOP server)
* - c_source_type_filename: file specified in a directory on the AOP Server
* - c_source_type_db_directory: file specified in a directory on the Database Server, use DIRECTORY:filename
* - c_source_type_json: JSON with the template file base64 encoded
* - c_source_type_clob_base64: BLOB in CLOB base64 encoded (user apex_web_service.blob2clobbase64)
* - c_source_type_aop_template: AOP will generate a starter template
* - c_source_type_aop_report: AOP will use it's own template, used to generate one or more APEX regions
* - c_source_type_apex_report: APEX will generate one region (native functionality)
* - c_source_type_oci_objs: Oracle Cloud Infrastructure - Object Storage
* - c_source_type_none: leave the source blank
* @Param: p_ref_cursor: when data type is c_source_type_refcursor, we will read the ref cursor specified here
* @Param: p_sql_array: when data type is c_source_type_sql_arrea, different SQL statements can be passed by using t_query_list
* p_data_type => aop_api_pkg.c_source_type_json,
* p_template_type => aop_api_pkg.c_source_type_aop_template,
* p_data_type => aop_api_pkg.c_source_type_rpt,
p_data_type in varchar2 default c_source_type_sql,
p_template_type in varchar2 default c_source_type_apex,
