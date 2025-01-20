# Basics about the CLMS API

The CLMS Website API is based on Plone's REST API. [Plone](https://plone.org) is the CMS on top of which
the CLMS Website is built on and its REST API provides a great number of features to access and interact
with the portal.

The full documentation about Plone's REST API can be found online [at Read the Docs](https://plonerestapi.readthedocs.io/en/latest/)
and here we will be summarizing the most important features that will de required to interact with the portal.

## URL of the API

If the base URL of the portal is `https://land.copernicus.eu/` then the API is available at `https://land.copernicus.eu/api/`.

## Basic concepts..

Plone provides the information about the objects under the object's itself's url. If we have an object at the following URL `https://land.copernicus.eu/en/datasets/dataset-1/` then its content will be available through the API at `https://land.copernicus.eu/api/en/datasets/dataset-1/`

To search all those datasets, the API provides a full-featured `@search` endpoint that can be used to search not only DataSets but also
any other content type that is available in the portal. The full documentation about the `@search` endpoint can be found on the [official documentation](https://plonerestapi.readthedocs.io/en/latest/endpoints/searching.html) .

## API Content-type

Due to the dual nature of the Plone objects (remember, they share the same URL for both the public face and the API), all API queries must be made
using an specific `Accept` http-header so that Plone can return the contents in the correct format.

All API results will be returned in `application/json` format. This is not a selectable format by the user, it is the only format supported by the API.

## Authentication

For all basic operations, such as searching for content or obtaining information about a given DataSet, the API can be used without authentication.

Authentication is only required to perform DataSet download operations.

You will find the specifics of the authentication process in the next section.
