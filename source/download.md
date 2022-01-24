# Download API

The CLMS portal provides several endpoints to handle the downloads of the files provided by the portal.

Due to the nature of the files present in the portal, the download is handled by a background process which informs
the user by email when the download is ready. Moreover it also signals the API with the download URL so that the
application using the API can handle by itself the download.

The download process takes the following steps:

1. Find the items to be downloaded
2. Decide in which format the items should be downloaded
3. Restrict the spatial extent of the files
4. Request the download
5. Wait for the download to be ready

In the following sections we will explain all of them.

## Find the items to be downloaded

As explained in the introduction, the CLMS portal API is based on Plone, so the user needs to use the `@search` endpoint to look for the items he wants to download.
