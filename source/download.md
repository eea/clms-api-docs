# Download API

The CLMS portal provides several endpoints to handle the downloads of the files provided by the portal.

Due to the nature of the files present in the portal, the download is handled by a background process which informs the user by email when the download is ready. Moreover it also signals the API with the download address so that the application using the API can handle by itself the download.

The download process takes the following steps:

1. Find the items to be downloaded
2. Decide in which format the items should be downloaded
   3.a. Restrict the spatial extent of the files (optional)
   3.b. Restrict the temporal range of the files (optional)
3. Request the download
4. Wait for the download to be ready

## Find the items to be downloaded

As explained in the introduction, the CLMS portal API is based on Plone, so the user needs to use the `@search` endpoint to look for the items he wants to download.

A simple search off all datasets available in the portal would be as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-search-datasets.req

```

The important bits here are in the parameters passed to the `@search` endpoint where the user is requesting to include the `UID`, the `dataset_full_format` and the `dataset_download_information` of each DataSet. If the user doesn't request the `UID` he will only obtain the `@id` of each datasets which corresponds to its address, and then he would have to do an additional request to that address to get the `UID` of the DataSet.

And the results will be similar to the following:

```{literalinclude} ./http-examples/download-search-datasets.resp
   :language: http
```

When the user find the DataSet that he wants to download, the user will have to take note of its `UID` (to know what to download), its `dataset_full_format` (to know which format conversions the user can request) and the desired `@id` inside the `dataset_download_information` (to know which file should be processed for download).

```{warning}
Be aware that the `@search` endpoint returns the results in batches of 25 items. You need to check the `batching` key of the resulting JSON in order to go to the `next` batch of items.

An alternative way to this is to do the HTTP request with an additional parameter `b_size` saying the number of items you want to have in the batch. This may make the API process slower though.

```

## File formats

The CLMS API provides an endpoint where the user can check which conversions are available for the available formats. To get the list of available conversions the user needs to do the following request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-available-conversions.req

```

The response will be a JSON object where each of the format has a `true/false` value indicating if the conversion is available or not.

```{literalinclude} ./http-examples/download-available-conversions.resp
   :language: http
```

```{warning}
When requesting the download you are required to inform in which format you want to download it. We suggest to download the files in the original format (the one set in the `full_format` parameter in the `dataset_download_information`), otherwise the file organisation (naming convention and band number/organisation) may differ considerably from the Product user manual.
```


## Projections

In the same way the output projection of the downloaded file can be selected. This selection is open on all available projections. To get the list of the available ones, one needs to do the following request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-available-projections.req

```

The response will be the list of available projections:

```{literalinclude} ./http-examples/download-available-projections.resp
   :language: http
```

## Restrict the spatial extent of the files

### Restriction by Nomenclature of Territorial Units for Statistics

The _Nomenclature of Territorial Units for Statistics_ (NUTS) is a standard used to define the boundaries of the countries and sub-country divisions in the European Union.

If you want to crop the DataSet to a given NUTS region, you need to know that the CLMS Portal allows the use of NUTS 0, NUTS 1, NUTS 2 and NUTS 3 codes, which you need to pass when requesting the download.

### Restriction by bounding box

In addition to the NUTS way, you can also restrict the download to a given bounding box. This is done by passing an array of 4 values in the signaling the 4 points that parameter. The values of the bounding box must be the coordinates of a rectangle in the `EPSG:4326` projection.

## Restrict the temporal extent of the files

Some available datasets offer time-series information and the user can choose to download only a subset of the available time-series.


## Full dataset downloads

The standard download API allows downloading the full dataset without any spatial or temporal restrictions. To do so, the user needs to do a request like this one:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-full_dataset_ok.req
```

The response will include the task id like in the responses of other requests

```{literalinclude} ./http-examples/download-request-download-full_dataset_ok.resp
   :language: http
```

These downloads will only work for datasets coming from EEA sources.

Due to the way those dataset files are stored in the servers, sometimes the CLMS portal needs to go to the original source (such as Wekeo) to download the files and provide them to the end-user.

This process is OK when the user requests to download the dataset with a time or spatial restriction because the CLMS portal needs to transform and process the files.

But when requesting the full dataset of a non-EEA dataset, it makes little sense to download first the whole dataset to the CLMS portal and offer it later to the user.

That's why, in such a case, the download API will issue an error informing the user to use some other API endpoint.

In such endpoint the user will be offered direct download links for the datasets he is requesting.

This is an example request requesting a full download of an external dataset:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-full_dataset_nok.req
```

And the response obtained from the server:

```{literalinclude} ./http-examples/download-request-download-full_dataset_nok.resp
   :language: http
```

### Auxiliary API to get direct download links for non-EEA datasets

To request the direct download links of full datasets for non-EEA datasets, the CLMS portal offers an auxiliary API.

In the request to this auxiliary API the user needs to pass the dataset id and the collection name he wants to download, and the API will return the direct download links for the requested dataset:


```{http:example} curl wget python-requests
    :request: ./http-examples/download-auxiliary_api.req
```

And the response obtained from the server:

```{literalinclude} ./http-examples/download-auxiliary_api.resp
   :language: http
```


## Request the download

When the user has all the data he needs to download the DataSet, he can request the download by doing the following request.

### Restrict using a Nomenclature of Territorial Units for Statistics code

The request must include a correct NUTS code in the request:

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

### Combining restrictions

Those temporal and spatial restrictions can be combined to produce an output file that's restricted to a given NUTS code and a given temporal range or to a given bounding box and a given temporal range.

To do so, the user need to add the required restrictions in the request.

This is the example with a bounding-box and a temporal range:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-bbox-timeseries.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-bbox-timeseries.resp
   :language: http
```

This is the example with a NUTS code and a temporal range:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-nuts-timeseries.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-nuts-timeseries.resp
   :language: http
```

## Wait for the download to be ready

As said, the download tool will inform the user by email when the download is ready. But the user can always request the status of all his requests using the `@datarequest_search` endpoint as follows:

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

In this case, it will include 2 additional parameters: the date and time of the finalization of the download process and the address where the download will be available for the next _72 hours_.

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

## Download prepackaged files

CLMS portal also provides a way to download some prepackaged files. Those prepackaged files offer a way to easily download already processed and redacted data.

To inspect which prepackaged files are available for download, one can request that information to the dataset search endpoint:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-search-dataset-prepackaged.req

```

The result will include all the information regarding the prepackaged files in the `downloadable_files` attribute:

```{literalinclude} ./http-examples/download-search-dataset-prepackaged.resp
   :language: http
```

To request the download of one or several files, one has to note their `@id` and request their download as follows. In this case trying to request a format and/or projection change will fail because all prepackaged files
are process as they are. CLMS portal can't handle their transformation:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-prepackaged.req

```

The response will contain the task id of the download request, like in earlier options:

```{literalinclude} ./http-examples/download-request-download-prepackaged.resp
   :language: http
```

## Combined download of datasets and prepackaged files

One can also request in a single request to download a prepackaged file and a full dataset. To do so, one only has to include the required information in the request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-combined.req

```

In this case the request will be splitted in two download packages, one will contain all the prepackaged files and the other one the rest of the downloads.

This is done this way to speed up the preparing of the packages, because all the prepackaged files are already available and their download process is much faster.

```{literalinclude} ./http-examples/download-request-combined.resp
   :language: http
```
