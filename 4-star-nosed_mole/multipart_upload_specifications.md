# Up-/Download Client & Production POC (Star-Nosed Mole) - Multipart Uploads

This is an addition to the Star-Nosed mole epic to adress challenges in multipart uploads

## Scope:
A scope definition can be found here: https://wiki.verbis.dkfz.de/x/fAFzCQ

## User Journeys

The addition to this epic covers the following user journeys:

### Multipart Data Upload:
![Data Upload](./images/multipart_data_upload.png)

A Data Submitter specifies the file ID, the URL to the Upload Contoller API, and the file path on the local file system using the CLI interface of the client (1.0). Internally, the CLI client translates the user-defined data into a request to the Upload Contoller API to obtain an S3 upload id (1.1). The client sends a request to the Upload Controller for each individual part of the uploaded file and recieves a presigned post for each individual part (1.2). The client reads multiple parts from the source file and uploads them as a stream using the pre-signed post (1.3). Once the upload has been completed the client sends a confirmation to the Upload Controller API (1.4).

## CLI:

The CLI commands remain the same.

## UCS:

The following Updates are performed in the Upload Controller Service:

### Database:

A new table "multipart_uploads" is added, containing the following columns:
```
- s3_upload_id (Primary key)
- file_id (not unique)
- upload_status
```

upload_status is an Enum and can have the following values:

```
- pending
- canceled
- failed
- uploaded
- accepted
- rejected
```

The column "upload_status" from then table "files" is removed.
A column "s3_upload_id" (unique) is added to the table "files".


### Endpoints:


Replace:
```
GET: /presigned_post/{file_id}
    gets presigned_post from S3
    sets UploadState to Pending
returns presigned_post
```

with:
```
POST /files/{file_id}/multipart_uploads
    if there is already a multipart_upload with file_id and upload_status == pending, updated or accepted
        returns error
    else:
        gets s3_upload_id from S3
        creates new row in multipart_uploads with file_id, s3_upload_id and upload_status=pending
        put s3_upload_id into database
returns s3_upload_id and the part size in bytes for this download (default: 16 MiB)
```

Add:
```
GET /files/{file_id}/multipart_uploads?status=pending
returns the s3_upload_id in the table multipart_uploads where file_id=file_id and upload_status=pending (There should never be more than one.) or None
```

Add:
```
POST /multipart_uploads/{upload_id}/part/{part_no}/presigned_post
    creates part-specific presigned post
returns presigned post
```

Replace:
```
patch: /confirm_upload/{file_id}
    body: state: UploadState (Uploaded)
    sets upload_status to uploaded
returns: 204
```

with:
```
PATCH /multipart_uploads/{upload_id}/
    body:       status==uploaded/cancelled
                part_no to eTag mapping (only if status==uploaded)
    if status=uploaded && upload_status of upload_id is pending:
        finish multipart upload via s3 api
        sets upload_status of upload_id to uploaded
    if status=cancelled && upload_status of upload_id is pending:
        sets upload_status of upload_id to canceled
returns 204
```

### Async communications:

Adapt async communications where needed.
