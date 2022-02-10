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

A simple search off all datasets available in the portal would be as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-search-datasets.req

```

The important bits here are in the parameters passed to the `@search` endpoint where we are requesting to include the `UID` and the `dataset_full_format` of each DataSet. If we don't request the `UID` we will only obtain the `@id` of each datasets which corresponds to its URL, and then we would have to do an additional request to that URL to get the `UID` of the DataSet.

And the results will be similar to the following:

```{literalinclude} ./http-examples/download-search-datasets.resp
   :language: http
```

When we find the DataSet that we want to download we will have to take note of its `UID` (in order to know what to download) and its `dataset_full_format` (in order to know which format conversions can we request).

## Decide in which format the items should be downloaded

The CLMS API provides an endpoint where we can check which coversions are available for the available formats. To get the list of available conversions we need to do the following request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-available-conversions.req

```

The response will be a JSON object where each of the format has a `true/false` value indicating if the conversion is available or not.

```{literalinclude} ./http-examples/download-available-conversions.resp
   :language: http
```

In the same way the output projection of the downloaded file can be selected. This selection is open on all available projections. To get the list of the available ones, one needs to do the followint request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-available-projections.req

```

The response will be the list of available projections:

```{literalinclude} ./http-examples/download-available-projections.resp
   :language: http
```

## Restrict the spatial extent of the files

### Restriction by NUTS

NUTS is the _Nomenclature of Territorial Units for Statistics_ which is a standard used to define the boundaries of the countries and sub-country divisions in the European Union.

If you want to crop the DataSet to a given NUTS region, you need to know that the CLMS Portal allows the use of NUTS 0, NUTS 1, NUTS 2 and NUTS 3 codes, which you need to pass when requesting the download.

### Restriction by bounding box

In addition to the NUTS way, you can also restrict the download to a given bounding box. This is done by passing an array of 4 values in the signaling the 4 points that parameter. The values of the bounding box are the coordinates of a rectangle in EPSG:4326 projection.

## Restrict the temporal extent of the files

Some of the available datasets provide time series information and the user can choose to download only a subset of the available time serie.

## Request the download

When the user has all the data he needs to download the DataSet, he can request the download by doing the following request.

### Restrict using a NUTS code

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-nuts.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-nuts.resp
   :language: http
```

### Restrict using a bounding box

The bounding box must be specified passing four coordinates that represent
the 4 points of the rectangle to be downloaded:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-bbox.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-bbox.resp
   :language: http
```

### Restrict using a temporal range

The temporal range must be specified with the Start and End dates of the range.
The format is in milliseconds since the epoch (1970-01-01 00:00:00).

The endpoint will do several validations, such as providing both values, the start date is before the end date and the dates are valid.

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-timeseries.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-timeseries.resp
   :language: http
```

## Wait for the download to be ready

As said, the download tool will inform the user by email when the download is ready. But the user can always request the status of all of his requests using the `@datarequest_search` endpoint as follows:

Request all in progress downloads:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-in-progress.req

```

The response contains a JSON object where each of the keys represent a download task. It also includes the UIDs of the datasets that are being prepared, the download metadata (NUTS, Bounding Box, etc.), and also the date and time of the request.

```{literalinclude} ./http-examples/download-request-download-in-progress.resp
   :language: http
```

Similarly a user can request all the finished downloads:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-finished.req

```

The response contains a JSON object where each of the keys represent a download task. Like in the previous case, it also includes the UIDs of the datasets that are being prepared, the download metadata (NUTS, Bounding Box, etc.), and also the date and time of the request.

In this case, it will include 2 additional parameters: the date and time of the finalization of the download process and the URL where the download will be available for the next _10 days_.

```{literalinclude} ./http-examples/download-request-download-finished.resp
   :language: http
```

Similarly, instead of using the search, the user can request the status of a specific download by using the `@datarequest_status_get` endpoint as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-status.req

```

The result will be similar to the previous ones but will contain only the information of the requested dataset.

```{literalinclude} ./http-examples/download-request-download-status.resp
   :language: http
```

## Cancelling a download

A download can also be cancelled by using the `@datarequest_delete` endpoint as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-delete.req

```

The result will have no content if the request is deleted correctly

```{literalinclude} ./http-examples/download-request-delete.resp
   :language: http
```
