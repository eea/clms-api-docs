# Download API

The CLMS Website provides several endpoints to handle the downloads of the files provided by the portal.

The download is handled by a background process which informs the user by email when the download is ready. Moreover, it also signals the API with the download address so that the application using the API can handle the download by itself.

The download process takes the following steps:

1. Find the items to be downloaded
2. Decide in which format the items should be downloaded
    - Restrict the spatial extent of the files (optional)
    - Restrict the temporal range of the files (optional)
3. Request the download
4. Wait for the download to be ready

## Find the items to be downloaded

As explained in the introduction, the CLMS Website API is based on Plone so the user needs to use the `@search` endpoint to look for the items he wants to download.

If you don’t know the parameters of the desired dataset, or if you want to obtain the list of available datasets to look for the desired dataset, a simple search off all datasets available in the portal would be as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-search-datasets.req

```

By means of this `@search` endpoint, the user is requesting to retrieve the dataset `UID`, the `dataset\_full\_format` and the `dataset\_download\_information`, for all the available datasets. This information is needed in advance of the download itself. 

**Note** that if you don´t request the `UID` , you only will be provided with the `@id`, but this parameter corresponds to the dataset address, and is not the dataset Identifier, which is mandatory to proceed with the download itself. If you don’t include in the `@search` endpoint, the `UID` parameter, you will have to do an additional request to the dataset address to get the `UID`. 

The instruction will show all the available datasets downloadable through the portal. 

```{warning}

Be aware that the `@search` endpoint returns the results in batches of 25 items. You need to check the `batching` key of the resulting JSON in order to go to the `next` batch of items.

An alternative way to this is to do the HTTP request with an additional parameter `b\_size` indicating the number of items you want to have in the batch. This may make the API process slower though.
```
The results will be like the following:

```{literalinclude} ./http-examples/download-search-datasets.resp
   :language: http
```
When the user finds the DataSet that he wants to download, the user will have to take note of its `UID` (to know what to download), its `dataset\_full\_format` (to know which format conversions the user can request) and the desired `@id` inside the `dataset\_download\_information` (to know which file should be processed for download).

For example, this would be the information for the “_Imperviousness Change 2015-2018 (raster 20 m and 100 m), Europe, 3-yearly_” dataset, available in the portal here: `https://land.copernicus.eu/en/products/high-resolution-layer-imperviousness/imperviousness-change-2015-2018`

This means that the dataset has `UID`": `4062d6a3814c426b84218d91ecc733ca`, with two different collections available, at 20 m and 100 m. The data can be requested in Geotiff format because the type of data is Raster. 

Each collection has its own identifier `af69e45f-646c-43f4-b062-b2855b5d6b5f` and `43f68ebb-e498-416d-9bcd-216aa4580c3a` respectively. 



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

When requesting the download you are required to inform which format you want to download it. We suggest you download the files in the original format (the one set in the `full\_format` parameter in the `dataset\_download\_information`), otherwise, the file organisation (naming convention and band number/organisation) may differ considerably from the Product user manual.
```

This means that “vector” datasets can be only ordered only in any of these formats: GDB, GML, GPKG, SHP and Geojson, and the “raster” datasets can be ordered in any of these formats:  Geotiff and NetCDF. No conversions are available from vector to raster or from raster to vector.

## Projections

The output projection of the downloaded file can be selected the same way. To get the list of the available ones, one needs to pass as a parameter the UID of the dataset, like here:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-available-projections.req

```

The response will be the list of available projections for this specific dataset:

```{literalinclude} ./http-examples/download-available-projections.resp
   :language: http
```

## Temporal extent

When requesting a specific time range download the user needs to know the available time range and the period to request the proper download in advance.

A time series dataset is a dataset which observations are provided along a period.  An approximation of the available time range is defined in the dataset name and in the dataset characteristics available in the dataset page. There are daily, 10-daily, monthly and yearly datasets available in the website.

The time series datasets have time restriction to not overload the download process. This means that the user must order a limited time range for the data to be downloaded.

To check which is the maximum period to download for the time series of a given dataset the user should check the \*\*download\_limit\_temporal\_extent\*\* property, where he will get the number of days he can request to download.

**Pending to add example in this line**

The temporal range must be specified with the Start and End dates of the range. The format is in milliseconds since the epoch (1970-01-01 00:00:00).

## Restrict the spatial extent of the files

### Restriction by Country

If you want to crop the dataset to a given world country, you need to know the list of ISO codes that the portal admits: `https://www.iso.org/obp/ui/#search`

If you´d like to download the following dataset for Ecuador country, specify the country code in the NUTS parameter.

```{http:example} curl wget python-requests

:request: ./http-examples/download-request-download-country.req
```

### Restriction by Nomenclature of Territorial Units for Statistics

The \_Nomenclature of Territorial Units for Statistics\_ (NUTS) is a standard used to define the boundaries of the countries and sub-country divisions in the European Union.

If you want to crop the Dataset to a given NUTS region, you need to know that the CLMS Website allows the use of NUTS 0, NUTS 1, NUTS 2 and NUTS 3 codes, which you need to pass when requesting the download.

To know which is the NUTS name you can consult this list of ISO NUTS codes. Note that the NUTS codes that the portal accepts are the 2021 codes: `https://ec.europa.eu/eurostat/web/nuts/maps`

```{http:example} curl wget python-requests

:request: ./http-examples/download-request-download-nuts-restriction.req
```

### Restriction by bounding box

You can also restrict the download to a given bounding box. This is done by passing an array of 4 values in the signaling the 4 coordinates. The values of the bounding box must be the coordinates of a rectangle in the `EPSG:4326` projection.

The order of the 4 coordinates is as follows: 

**BoundingBox**: [`max.lat`,`max.lon`,`min.lat`,`min.lon`] which is the same as [`N`,`E`,`S`,`W`]

```{http:example} curl wget python-requests

:request: ./http-examples/download-request-download-bbox-restriction.req
```

## Restrict the temporal extent of the files

Some available datasets offer time-series information and the user can choose to download only a subset of the available time-series.

```{http:example} curl wget python-requests

:request: ./http-examples/download-request-download-temporal-extent-restriction.req
```

### Combining restrictions

Those temporal and spatial restrictions can be combined to produce an output file that is restricted to a given NUTS code and a given temporal range or to a given bounding box and a given temporal range.

To do so, the user needs to add the required restrictions in the request.

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

:request: ./http-examples/download-request-download-layers.req
```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-nuts-timeseries.resp

:language: http
```

## Full dataset downloads

Full dataset downloads of EEA datasets are available through the prepackaged files of each datasets. In case of requesting the full dataset without definition of the corresponding prepackage, you´ll receive an error:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-full_dataset_ok.req
```

The response will include the task id like in the responses of other requests

```{literalinclude} ./http-examples/download-request-download-full_dataset_ok.resp
   :language: http
```
Check from the list of available prepackages, which corresponds to the full dataset (pan-European dataset). See Download prepackaged files section to know how to see the list of available prepackages

However, when requesting the full dataset of a non-EEA dataset, it makes little sense to first download the whole dataset to the CLMS Website and offer it later to the user.

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

To request datasets not hosted at EEA premises and hosted in platforms such as WEKEO or Amazon, the CLMS website offers directs download links to these datasets though an auxiliary API.

In the request to this auxiliary API the user needs to pass the dataset id and the download information id he wants to download, also a time range to download (in case of time-series datasets) and the API will return the direct download links for the requested dataset.

The available parameters when requesting this downloads are the following:

- date_from: start date of the requested download in ISO format: YYYY-MM-DD
- date_to: end date of the requested download in ISO format: YYYY-MM-DD
- x_max: requested bounding-box coordinates
- y_max: requested bounding-box coordinates
- x_min: requested bounding-box coordinates
- y_min: requested bounding-box coordinates

date_from and date_to only for time series datasets.

If no coordinates are entered for WEkEO or Landcover datasets, the used bounding box will be the following:

x\_max = 32.871094

y\_max = 70.289117

x\_min = -12.480469

y\_min = 35.603719

If no start and end date are entered for WEkEO and Legacy datasets, the used ones will be the following:

start\_date: request date

end\_date: request date - 30 days

This is an example with a WEkEO dataset:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-auxiliary_api_wekeo.req
```

And the response obtained from the server. In this case, the API will not return the direct links to download the
dataset, but the parameters you need to use in the WEkEO API to download the dataset files.

Check The `WEkEO documentation`\\_ to get further information:

```{literalinclude} ./http-examples/download-auxiliary\_api\_wekeo.resp

:language: http
```

This is an example with a LEGACY dataset. In this case the `date\_from` and `date\_to` parameters are mandatory:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-auxiliary_api_legacy.req
```

And the response obtained from the server:

```{literalinclude} ./http-examples/download-auxiliary_api_legacy.resp
   :language: http
```

This is an example for a Global Dynamic Land Cover dataset:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-auxiliary_api_landcover.req
```

And the response obtained from the server:

```{literalinclude} ./http-examples/download-auxiliary_api_landcover.resp
   :language: http
```

## Request the download

When the user has all the data he needs to download the DataSet, he can request the download by making the following request.

### Restrict using a Nomenclature of Territorial Units for Statistics code

The request must include a correct NUTS code in it:

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

The endpoint will do several validations, such as checking that both values are provided, that the start date is before the end date and the dates are valid.

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-timeseries.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-timeseries.resp
   :language: http
```
## Layer/band selection

Although the Net CDF files are composed by bands, the user can´t request to download specific bands. 
This parameter has to be left blank and the system will use by default the value `ALL BANDS`, which means that all of them will be downloaded
:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-layers.req

```

The response will include the task id assigned to the download process:

```{literalinclude} ./http-examples/download-request-download-layers.resp
   :language: http
```

## Wait for the download to be ready

The download tool will inform the user by email when the download is ready. The user can always request the status of all his requests using the `@datarequest\_search` endpoint as follows:

Request all in progress downloads:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-in-progress.req

```

The response contains a JSON object where each of the keys represents a download task. It also includes the UIDs of the datasets that are being prepared, the link to the dataset metadata and the parameters of the custom download such as: NUTS, Boundinbox, time range. etc

```{literalinclude} ./http-examples/download-request-download-in-progress.resp
   :language: http
```

Similarly, a user can request all the finished downloads:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-finished.req

```

The response contains a JSON object where each of the keys represent a download task. Like in the previous case, it also includes the UIDs of the datasets that are being prepared, the link to the dataset metadata and the parameters of the custom download such as: NUTS, Boundinbox, time range. etc.

In this case, it will include 2 additional parameters: the date and time of the finalization of the download process and the address where the download will be available for the next \_72 hours\_.

```{literalinclude} ./http-examples/download-request-download-finished.resp
   :language: http
```

Similarly, instead of using the search, the user can request the status of a specific download by using the `@datarequest\_status\_get` endpoint as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-status.req

```

The result will be similar to the previous ones but will only contain the information of the requested dataset.

```{literalinclude} ./http-examples/download-request-download-status.resp
   :language: http
```

## Cancelling a download

A download can also be cancelled by using the `@datarequest\_delete` endpoint as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-delete.req

```

The result will have no content if the request is deleted correctly

```{literalinclude} ./http-examples/download-request-delete.resp
   :language: http
```

## Download prepackaged files

CLMS Website also provides a way to download some prepackaged files. Those prepackaged files offer a way to easily download already processed and redacted data.

To inspect which prepackaged files are available for download, one can request that information to the dataset search endpoint:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-search-dataset-prepackaged.req

```

The result will include all the information regarding the prepackaged files in the `downloadable\_files` attribute:

```{literalinclude} ./http-examples/download-search-dataset-prepackaged.resp
   :language: http
```

When ordering a prepackage it is not possible to define custom parameters as the format and the projection as the file is already prepared. To rder a prepackage it is only needed to request the `@id`, if not, the download will fail.
See the below example

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-download-prepackaged.req

```

The response will contain the task id of the download request, like in earlier options:

```{literalinclude} ./http-examples/download-request-download-prepackaged.resp
   :language: http
```

## Combined download of datasets and prepackaged files

One can also request to download a prepackaged file and a custom download in a single request. To do so, one only has to include the required information in the request:

```{http:example} curl wget python-requests
    :request: ./http-examples/download-request-combined.req

```

In this case the request will be splitted into two download packages, one will contain all the prepackaged files and the other the rest of the downloads.

It is done this way to speed up the preparing of the packages because all the prepackaged files are already available and their download process is much faster.

```{literalinclude} ./http-examples/download-request-combined.resp
   :language: http
```

.. \\_`WEkEO documentation`: https://help.WEkEO.eu/en/collections/3530725-WEkEO-harmonized-data-access ELIMINAR??
