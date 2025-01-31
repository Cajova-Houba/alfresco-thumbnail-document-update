# About

This repository contains the configuration used to replicate a (possible) bug in the Alfresco's thumbnail generation process.

# Steps to reproduce

1. docker-compose up
2. Wait for the Alfresco to be up and running
3. Open the browser and go to `http://localhost:8080/share`
4. Login using credentials `admin:admin`
3. Upload the test image (hackerman.jpg) to the repository
4. Get the document's id, e.g. if the document's nodeRef is `workspace://SpacesStore/81503dd2-3a60-4e71-9b62-716aa7068422`, then the id is `81503dd2-3a60-4e71-9b62-716aa7068422`
5. Get the node using REST API: `GET http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{{documentId}}`
6. The document should have an aspect called `rn:renditioned`
7. Attempt to update the document's content using the REST API: `PUT http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{{documentId}}/content?majorVersion=true` with the file `hackerman.jpg`
8. The request times out and the reverse proxy returns HTTP 504 Gateway Timeout
9. Open the browser again, navigate to the document, you'll see that the version has incremented
10. Remove the `rn:renditioned` aspect using REST API: `PUT http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{{documentId}}` with json body

```json
{
"aspectNames": [
      "cm:versionable",
      "cm:titled",
      "cm:auditable",
      "cm:author",
      "cm:thumbnailModification",
      "exif:exif"
    ]
}
```

11. Call the request from step 7 again, this time the server returns HTTP 201 Created

## Notes

- Step 7 can be done in the UI as well. Open the document in the browser, click on the `Upload new version`, select the file and confirm the upload. A window with progress bar is shown but never finishes and eventually fails with 504 Gateway Timeout error. The document version is incremented.
- The `rn:renditioned` aspect is added to the document every time it's displayed in the UI. The aspect can be added to the document by calling renditions api: `http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{{documentId}}/renditions`.
- Updating just the document properties (not the conent) works fine.   