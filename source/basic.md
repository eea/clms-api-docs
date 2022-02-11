# Basics about the Copernicus Land Monitoring Service API

The CLMS Portal API is based on Plone's REST API. [Plone](https://plone.org) is the CMS on top of which
the CLMS Portal is built on and its REST API provides great number of features to access and interact
with the portal.

The full documentation about Plone's REST API can be found online [at Read the Docs](https://plonerestapi.readthedocs.io/en/latest/)
and its most important features will be summarized here.

## Address of the API

If the base address of the portal is `https://clms.copernicus.eu/` then the API is available at `https://clms.copernicus.eu/api/`.

## Basic concepts

Plone provides the information about the objects under the object's itself's address. When having an object at the following address `https://clms.copernicus.eu/en/datasets/dataset-1/` then its content will be available through the API at `https://clms.copernicus.eu/api/en/datasets/dataset-1/`

To search all those datasets, the API provides a full-featured `@search` endpoint that can be used to search not only DataSets but also
any other content type that's available in the portal. The full documentation about the `@search` endpoint can be found on the [official documentation](https://plonerestapi.readthedocs.io/en/latest/searching.html) .

## Content-type of the requests

Due the dual nature of the Plone objects (remember, they share the address for both the public face and the API), all API queries must be made
using an specific `Accept` http-header so that Plone can return the contents in the correct format.

All API results will be returned in `application/json` format. This isn't a selectable format by the user, it's the only format supported by the API.

## Authentication

For all basic operations such as searching for content or obtaining information about a given DataSet, the API can be used without authentication.

Authentication is only required to perform DataSet download operations.

You will find the specifics of the authentication process in the next section.
