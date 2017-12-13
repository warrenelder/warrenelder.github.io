---
title: 'Build a micro-service in NodeJS to connect to a third-party api'
author: "Warren Elder"
date: 2017-12-13 15:24:49
category:
- Applications
tags:
- NodeJs
- Microservice
- Loopback
- Javascript
clearReading: true
metaAlignment: center
coverImage: api.jpg
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---

As part of a project I'm currently working on, a phone application requires data supplied by a third party API. While it would be possible to make requests directly to the API's end-points, I have decided to employ an alternative approach.
<!-- more -->
I have decided to build a server side API to act as a proxy between the application and third-party API. In practice this means that the phone application makes requests to the proxy API endpoints which in turn makes a request to the third-party API. The response is then sent back through the proxy API and on to the phone application. You may be thinking that this is a needlessly complicated, or unnecessary engineering, but there are several good reasons to utilise this architecture;

* We avoid storing any API keys or sensitive information within the distributed phone application source code. Sensitive information can be stored server side, hidden within the NodeJS code.
* Performance can be improved by adding a cache layer in the proxy API, reducing the total number requests to the third-party API.
* The data provided by the third-party API may not be formatted as required. The proxy can do the heavy lifting, preparing the data to avoid client side processing in the phone application.
* If the endpoints of the third-party API change in a way that breaks the phone application we need only make updates to the server side proxy API. This avoids us having to make a costly app version update across all distributed platforms.
* Creating a proxy API can actually be achieved very quickly.

While this list is by no means exhaustive, it should demonstrate why such an approach can be of use. In the rest of this post I shall describe how I built the proxy API in NodeJS and demonstrate how you can get a microservice up and running in minutes.

I used the {% link Loopback https://loopback.io/ [https://loopback.io/] [Loopback framework] %} framework as the starting point because it has a very helpful CLI that allows you to define the high level utility of your server in minutes, and it has a very useful package that does much of the scaffolding to make the connection with the third-party api.

From here I assume you already have NodeJS installed. We first get the server up and running.
{% codeblock %}
// Install the LoopBack CLI tool
$ npm install -g loopback-cli
// Initialise project
$ lb
// Answer the following prompts;
// ? What's the name of your application?
proxy-api
// ? Enter name of the directory to contain the project:
proxy-api
// ? Which version of LoopBack would you like to use?
3.x (current)
// ? What kind of application do you have in mind?
api-server (A LoopBack API server with local User auth)
{% endcodeblock %}

Now that is done, we define the REST connector that forms the connection with the third-party API.
{% codeblock %}
// Change directory to app
$ cd proxy-api
// Install REST connector
$ npm install --save loopback-connector-rest
// Set up connector
$ lb datasource
// Answer the following prompts;
// ? Enter the datasource name:
third-party-api
// ? Select the connector for third-party-api:
REST services (supported by StrongLoop)
// ? Base URL for the REST service:
https://api.postcodes.io
// ? Default options for the request:
// ? An array of operation templates:
// ? Use default CRUD mapping:
No
{% endcodeblock %}

In the set-up above, I have chosen an open API providing information on UK postcodes to act as the third-party API. The next step in providing a bare-bones, working example of the proxy API is to define an endpoint to make a request to. For simplicity, we shall make a GET request to the proxy API to return data on a single postcode. For this step we must make some small changes to some code. So head to */server/datasource.json* where you should see,
```json
{
  "db": {
    "name": "db",
    "connector": "memory"
  },
  "third-party-api": {
    "name": "third-party-api",
    "baseURL": "https://api.postcodes.io",
    "crud": false,
    "connector": "rest"
  }
}
```
and change to,
```json
{
  "db": {
    "name": "db",
    "connector": "memory"
  },
  "third-party-api": {
    "name": "third-party-api",
    "baseURL": "https://api.postcodes.io",
    "crud": false,
    "connector": "rest",
    "operations": [
      {
        "functions": {
          "getpostcodes": [
            "postcode"
          ]
        },
        "template": {
          "method": "GET",
          "url": "https://api.postcodes.io/postcodes/{postcode}",
          "headers": {
            "accepts": "application/json",
            "content-type": "application/json"
          }
        }
      }
    ]
  }
}
```
This code defines a set of operations for digesting or mapping requests to the third-party API. We must define our named *functions* which provides a url at the proxy API where we make our request to. Meta information, such as the http *method* is stored in the *template*. The final step is to add a *model* which collects the data from the REST connector.
{% codeblock %}
// Create model
$ lb model
// ? Enter the model name:
postcode
// ? Select the datasource to attach postcode to:
third-party-api (rest)
// ? Select model's base class
Model
// ? Expose postcode via the REST API?
Yes
// ? Custom plural form (used to build REST URL):
// ? Common model or server only?
common
// Let's add some postcode properties now.
// Enter an empty property name when done.
// ? Property name:
{% endcodeblock %}
Now we can start our server,
{% codeblock %}
$ node .
{% endcodeblock %}
Lets view our proxy API endpoints at *http://localhost:3000/explorer*. This page also provides a useful interface listing all available endpoints and allows us to make test requests. Trying it out with the postcode *SW1A0AA* we get the following response.
```json
{
  "status": 200,
  "result": {
    "postcode": "SW1A 0AA",
    "quality": 1,
    "eastings": 530268,
    "northings": 179545,
    "country": "England",
    "nhs_ha": "London",
    "longitude": -0.12466272904588,
    "latitude": 51.4998404539774,
    "european_electoral_region": "London",
    "primary_care_trust": "Westminster",
    "region": "London",
    "lsoa": "Westminster 020C",
    "msoa": "Westminster 020",
    "incode": "0AA",
    "outcode": "SW1A",
    "parliamentary_constituency": "Cities of London and Westminster",
    "admin_district": "Westminster",
    "parish": "Westminster, unparished area",
    "admin_county": null,
    "admin_ward": "St James's",
    "ccg": "NHS Central London (Westminster)",
    "nuts": "Westminster",
    "codes": {
      "admin_district": "E09000033",
      "admin_county": "E99999999",
      "admin_ward": "E05000644",
      "parish": "E43000236",
      "parliamentary_constituency": "E14000639",
      "ccg": "E38000031",
      "nuts": "UKI32"
    }
  }
}
```
To confirm this response at the third-party API just visit http://api.postcodes.io/postcodes/SW1A0AA.

And that is it, we have a microservice up and running in minutes. Further upgrades can be made to cache requests and add authorisation keys if required. You can also get the full code at https://github.com/warrenelder/proxy-api.
