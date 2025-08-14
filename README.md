
# File save in local directory in oracle apex

**Here's how to save a file to a directory on the Oracle database server from APEX:**

## Create a Physical and DB Directory Object in the Database: 

- Create a physical directory/folder.
- This is a logical database object that points to a physical directory on the server's file system.
- The user must have read and write permissions to this physical directory.
  
 ## Create Directory and give permission:

 ```create directory and give permission

     CREATE DIRECTORY STU_PHOTO AS 'E:\ORDS\images\STU_PHOTO';
     GRANT READ, WRITE ON DIRECTORY STU_PHOTO TO your_schema_user;

```

## Create a procedure for Save file to the Directory

```Create a procedure

create or replace PROCEDURE blob_to_file(p_blob      IN OUT NOCOPY BLOB,
						p_dir   IN  VARCHAR2,
						p_filename  IN  VARCHAR2)
						AS
						  l_file      UTL_FILE.FILE_TYPE;
						  l_buffer    RAW(32767);
						  l_amount    BINARY_INTEGER := 32767;
						  l_pos       INTEGER := 1;
						  l_blob_len  INTEGER;
						BEGIN
						  l_blob_len := DBMS_LOB.getlength(p_blob);
						  l_file := UTL_FILE.fopen(p_dir, p_filename,'WB', 32767);
						WHILE l_pos <= l_blob_len LOOP
							DBMS_LOB.read(p_blob, l_amount, l_pos, l_buffer);
							UTL_FILE.put_raw(l_file, l_buffer, TRUE);
							l_pos := l_pos + l_amount;
						  END LOOP;
						  UTL_FILE.fclose(l_file);
						EXCEPTION
						  WHEN OTHERS THEN
							IF UTL_FILE.is_open(l_file) THEN
							   UTL_FILE.fclose(l_file);
							END IF;
							UTL_FILE.fclose(l_file);
							RAISE;
						END blob_to_file;

```


## Upload File from Oracle APEX Page Blob Item (apex_application_temp_files)

```pl/sql process for File save 

BEGIN
   IF :P28_FILE_DATA IS NOT NULL THEN
      DECLARE
         l_blob BLOB;
      BEGIN
         SELECT BLOB_CONTENT
           INTO l_blob
           FROM apex_application_temp_files
          WHERE NAME = :P28_FILE_DATA;

         blob_to_file(
            p_blob     => l_blob,
            p_dir      => 'STU_PHOTO',
            p_filename => :P28_ADMIT_CODE || '.pdf'
         );


         UPDATE ADMISSION_INFO
            SET FILE_URL = '\i\STU_PHOTO\'||ADMIT_CODE||'.pdf'
          WHERE ADMIT_CODE = :P28_ADMIT_CODE;

      EXCEPTION
         WHEN OTHERS THEN
            NULL;
      END;
   END IF;
END;

```

## Show image from Local directory

``` image URL
<Img src="\i\STU_PHOTO\'||ADMIT_CODE||'.pdf" alt=" "height="42" width="42">

```



**If the DB server and application server are separate and this URL (\i\STU_PHOTO\'||ADMIT_CODE||'.pdf) does not work, then the image can be sent through the REST API.**


 # Thank you
 ## Sanjay Sikder
