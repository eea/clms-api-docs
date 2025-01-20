# Basics about the CLMS API

The CLMS Website API is based on Plone's REST API. [Plone](https://plone.org) is the CMS on top of which the CLMS Website is built on and its REST API provides a great number of features to access and interact with the portal.

# URL of the API
=======
The CLMS Portal API is based on Plone's REST API. [Plone](https://plone.org) is the CMS on top of which
the CLMS Portal is built on and its REST API provides great number of features to access and interact
with the portal.

The full documentation about Plone's REST API can be found online [at Read the Docs](https://plonerestapi.readthedocs.io/en/latest/)
and we will be summarizing the most important features that we will be using to interact with the portal
here.

## URL of the API

If the base URL of the portal is `https://clms.copernicus.eu/` then the API is available at `https://clms.copernicus.eu/api/`.

For all basic operations, such as searching for content or obtaining information about a given DataSet, the API can be used without authentication.

# Basic concepts..

Plone provides the information about the objects under the object's url. If we have an object at the following URL
=======

## Basic concepts

Plone provides the information about the objects under the object's itself's url. If we have an object at the following URL `https://clms.copernicus.eu/en/datasets/dataset-1/` then its content will be available through the API at `https://clms.copernicus.eu/api/en/datasets/dataset-1/`

To search all those datasets, the API provides a full-featured `@search` endpoint that can be used to search not only DataSets but also
any other content type that is available in the portal. The full documentation about the `@search` endpoint can be found on the [official documentation](https://plonerestapi.readthedocs.io/en/latest/searching.html) .

To consult a product on the CLMS portal, this would be the link structure:
•	`https://land.copernicus.eu/en/products/product/`
•	`https://land.copernicus.eu/en/products/corine-land-cover`

Then its content will be available through the API at:
•	`https://land.copernicus.eu/api/en/products/product` 
•	`https://land.copernicus.eu/api/en/products/corine-land-cover`
=======
Due the dual nature of the Plone objects (remember, they share the URL for both the public face and the API), all API queries must be made
using an specific `Accept` http-header so that Plone can return the contents in the correct format.

To consult a dataset on the CLMS portal, this would be the link structure:
•	`https://land.copernicus.eu/en/products/product/dataset`
•	`https://land.copernicus.eu/en/products/corine-land-cover/clc2018`

Then its content will be available through the API at:

•	`https://land.copernicus.eu/api/en/products/product/dataset` 
•	`https://land.copernicus.eu/api/en/products/corine-land-cover/clc2018`

# API Content-type

Due to the dual nature of the Plone objects (remember, they share the same URL for both the public face and the API), all API queries must be made using a specific `Accept` http-header so that Plone can return the contents in the correct format.

All API results will be returned in `application/json` format. This is not a selectable format by the user, it is the only format supported by the API.

# Authentication
=======

## Authentication

For all basic operations such as searching for content or obtaining information about a given DataSet, the API can be used without authentication.

Authentication is only required to perform DataSet download operations.

You will find the specifics of the authentication process in the next section.
