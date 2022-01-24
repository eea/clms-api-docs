# Introduction

Copernicus Land Monitoring Service portal has an extensive API that allows not only
portal users to access and download the data, but also allows unattended machine-to-machine
communication between external services and the portal.

In this documentation you will find a list of all the main options, explained with useful
examples.

## Reading add-ons records

Reading a single record:

```{http:example} curl wget python-requests
    :request: ./http-examples/addons_get.req
```    

Example Response:

```{literalinclude} ./http-examples/addons_get.resp
   :language: http
```   
