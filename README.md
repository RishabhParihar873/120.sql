DECLARE
    l_blob BLOB; l_mime VARCHAR2(255); l_filename VARCHAR2(255);
BEGIN
    SELECT file_path1, mime_type, file_name
    INTO   l_blob, l_mime, l_filename
    FROM   cgct_ai_document_library
    WHERE  id = :P1095_ID;

    sys.htp.init;
    sys.owa_util.mime_header(l_mime, FALSE);
    sys.htp.p('Content-Length: ' || sys.dbms_lob.getlength(l_blob));
    sys.htp.p('Content-Disposition: attachment; filename="' || l_filename || '"');
    sys.owa_util.http_header_close;
    sys.wpg_docload.download_file(l_blob);
    apex_application.stop_apex_engine;
END;
