The error is not actually in your package code.

The problem is that you've copied SQL from a web page or Word document and it contains HTML tags:

<br>


Oracle is trying to parse:

END ict_universal_doc_engine;
/
<br>
CREATE OR REPLACE PACKAGE BODY ...


and the parser encounters unexpected text after the /, resulting in:

PLS-00103: Encountered the symbol "/"

Fix

Remove all <br> tags completely.

Your script should look like:

CREATE OR REPLACE PACKAGE ict_universal_doc_engine AS
    FUNCTION generate_document (
        p_target_format IN VARCHAR2,
        p_title         IN VARCHAR2,
        p_payload_json  IN CLOB
    ) RETURN CLOB;
END ict_universal_doc_engine;
/

CREATE OR REPLACE PACKAGE BODY ict_universal_doc_engine AS

    FUNCTION generate_document (
        p_target_format IN VARCHAR2,
        p_title         IN VARCHAR2,
        p_payload_json  IN CLOB
    ) RETURN CLOB
    IS
        l_template_file VARCHAR2(100);
        l_output_type   VARCHAR2(10);
        l_mime_type     VARCHAR2(100);
        l_file_ext      VARCHAR2(10);
        l_return_blob   BLOB;
        l_new_doc_id    NUMBER;
        l_file_name     VARCHAR2(255);
        l_download_url  VARCHAR2(1000);
    BEGIN
        ...
    END generate_document;

END ict_universal_doc_engine;
/

One more issue I spotted

This insert looks suspicious:

INSERT INTO cgct_ai_document_library (
    name,
    file_name,
    file_path1,
    mime_type,
    ...
) VALUES (
    p_title,
    l_file_name,
    l_return_blob,
    l_mime_type,
    ...
);


You're inserting:

l_return_blob


into:

file_path1


If FILE_PATH1 is a VARCHAR2 column, you'll get:

ORA-00932: inconsistent datatypes


Check:

DESC cgct_ai_document_library;


and verify the datatype of:

FILE_PATH1


If the actual BLOB column has a different name (for example FILE_CONTENT, DOCUMENT_BLOB, etc.), use that column instead.

Another potential issue

You're using:

v('P0_APPLICATION_ID')


inside a package.

If this package is executed outside an APEX session, you'll get null values.

Safer:

NVL(v('P0_APPLICATION_ID'), 0)


or pass the application ID as a parameter.

First thing I'd check

Run:

DESC cgct_ai_document_library;


and share the output. I can then verify that the INSERT statement and datatypes are correct before you compile the package again.
