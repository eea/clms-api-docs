# Basics about the CLMS API

The CLMS Website API is based on Plone's REST API. [Plone](https://plone.org) is the CMS on top of which the CLMS Website is built on and its REST API provides a great number of features to access and interact with the portal.

# URL of the API

If the base URL of the portal is `https://land.copernicus.eu/` then the API is available at `https://land.copernicus.eu/api/`.

For all basic operations, such as searching for content or obtaining information about a given DataSet, the API can be used without authentication.

# Basic concepts..

Plone provides the information about the objects under the object's url. If we have an object at the following URL

To consult a product on the CLMS portal, this would be the link structure:
+ `https://land.copernicus.eu/en/products/product/`
+ `https://land.copernicus.eu/en/products/corine-land-cover`

Then its content will be available through the API at:
+ `https://land.copernicus.eu/api/en/products/product` 
+ `https://land.copernicus.eu/api/en/products/corine-land-cover`

To consult a dataset on the CLMS portal, this would be the link structure:
+ `https://land.copernicus.eu/en/products/product/dataset`
+ `https://land.copernicus.eu/en/products/corine-land-cover/clc2018`

Then its content will be available through the API at:

+ `https://land.copernicus.eu/api/en/products/product/dataset` 
+ `https://land.copernicus.eu/api/en/products/corine-land-cover/clc2018`

# API Content-type

Due to the dual nature of the Plone objects (remember, they share the same URL for both the public face and the API), all API queries must be made using a specific `Accept` http-header so that Plone can return the contents in the correct format.

All API results will be returned in `application/json` format. This is not a selectable format by the user, it is the only format supported by the API.

# Authentication

Authentication is only required to perform DataSet download operations.

You will find the specifics of the authentication process in the next section.
