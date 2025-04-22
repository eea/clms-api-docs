Download API
============

The CLMS Website provides several endpoints to handle the downloads of
the files provided by the portal.

The download is handled by a background process which informs the user
by email when the download is ready. Moreover, it also signals the API
with the download address so that the application using the API can
handle the download by itself.

The download process takes the following steps:

1. Find the items to be downloaded
2. Decide in which format the items should be downloaded

   -  Restrict the spatial extent of the files.
   -  Restrict the temporal range of the files.

3. Request the download
4. Wait for the download to be ready

Temporal extent
---------------

A time series dataset is a dataset which observations are provided along
a period. An approximation of the available time range is defined in the
dataset name and in the dataset characteristics available in the dataset
page. There are daily, 10-daily, monthly and yearly datasets available
in the website.

The time series datasets have time restrictions to not overload the
download process. This means that the user must order a limited time
range for the data to be downloaded.

To check which is the maximum period to download for the time series of
a given dataset the user should check the
**download_limit_temporal_extent** property, where he will get the
number of days he can request to download.

.. http:example:: curl wget python-requests
   :host: land.copernicus.eu

   :request:

     GET /api/@search?portal_type=DataSet&metadata_fields=UID&metadata_fields=download_limit_temporal_extent&b_size=300
     Accept: application/json
