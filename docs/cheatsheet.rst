******************
Postman Cheatsheet
******************

Thank you for downloading this cheat sheet. This guide refers to the Postman App, not the Chrome extension. Please report any problems with it.

Postman Cheat Sheet is based on the official Postman documentation and own experience.

For a detailed documentation on each feature, check out https://www.getpostman.com/docs.


Variables
=========

Code snippets for working with variables in scripts (pre-request, tests).

Setting variables
-----------------

**Global variable**

.. code-block:: javascript

    pm.globals.set('myVariable', MY_VALUE);

**Collection variable**
    
    Can only be set from Postman (at this moment).

**Environment variable** ::

    pm.environment.set('myVariable', MY_VALUE);

**Data variable**
    
    Can only be set from a CSV or a JSON file.

**Local variable** ::

    var myVar = MY_VALUE;


Examples: ::

    pm.globals.set('name', 'John Doe');
    pm.environment.set('age', 29);

Getting variables
-----------------

**Request builder**

Syntax: ``{{myVariable}}``

Request URL: ``http://{{domain}}/users/{{userId}}``

Headers (key:value): X-{{myHeaderName}}:foo

Body: ``{"id": "{{userId}}", "name": "John Doe"}``

**Depending on the matching scope** ::

    pm.variables.get('myVariable');

**Global variable** ::

    pm.globals.get('myVariable');

**Collection variable**
    
    Can only be used in the Request Builder (at this moment). Syntax: ``{{myVariable}}``

**Environment variable** ::

    pm.environment.get('myVariable');

**Data variable** ::

    pm.iterationData.get('myVariable);

**Local variable** ::

    myVar

Removing variables
------------------

**Global variable**

Remove one variable ::

    pm.globals.unset('myVariable');

Remove ALL global variables ::

    pm.globals.clear();

**Environment variable**

Remove one variable ::
    
    pm.environment.unset("myVariable");

Remove ALL environment variables ::

    pm.environment.clear();


DYNAMIC VARIABLES
-----------------

**Experimental feature**. Can only be used in request builder. Only one value is generated per request.

All dynamic variables can be combined with strings, in order to generate dynamic / unique data. 

Example JSON body:

.. code-block:: json

    {"name": "John Doe", "email": "john.doe.{{$timestamp}}@example.com"}


``{{$guid}}`` - global unique identifier. 

Example output: ``d96d398a-b655-4638-a6e5-40c0dc282fb7``

``{{$timestamp}}`` - current timestamp. 

Example output: `1507370977``

``{{$randomInt}}`` - random integer between 0 and 1000. 

Example output: ``567``


LOGGING / DEBUGGING VARIABLES
-----------------------------

Open Postman Console and use `console.log` in your test or pre-request script. 

Example: ::

    var myVar = pm.globals.get("myVar");
    console.log(myVar);

Assertions
==========

Note: You need to add any of the assertions inside a ``pm.test`` callback. 

Example: ::

    pm.test("Your test name", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.value).to.eql(100);
    });

Status code
-----------

Check if status code is 200: ::

    pm.response.to.have.status(200);


Checking multiple status codes: ::

    pm.expect(pm.response.code).to.be.oneOf([201,202]);


Response time
-------------

Response time below 100ms: ::

    pm.expect(pm.response.responseTime).to.be.below(9);

Headers
-------

Header exists: ::

    pm.response.to.have.header(X-Cache');

Header has value: ::

    pm.expect(pm.response.headers.get('X-Cache')).to.eql('HIT');

Cookies
-------

Cookie exists: ::

    pm.expect(pm.cookies.has('sessionId')).to.be.true;

Cookie has value: ::

    pm.expect(pm.cookies.get('sessionId')).to.eql(’ad3se3ss8sg7sg3');


Body
----

**Any content type / HTML responses**

Exact body match: ::

    pm.response.to.have.body("OK");
    pm.response.to.have.body('{"success"=true}');

Partial body match / body contains: ::

    pm.expect(pm.response.text()).to.include('Order placed.');

**JSON responses**

Parse body (need for all assertions): ::

    var jsonData = pm.response.json();

Simple value check: ::

    pm.expect(jsonData.age).to.eql(30);
    pm.expect(jsonData.name).to.eql('John);

Nested value check: ::

    pm.expect(jsonData.products.0.category).to.eql('Detergent');

**XML responses**

Convert XML body to JSON: ::

    var jsonData = xml2Json(responseBody);

Note: see assertions for JSON responses.

Postman Sandbox
===============

pm
---

this is the object containing the script that is running, can access variables and has access to a read-only copy of the request or response.

pm.sendRequest
--------------

Allows to send simple HTTP(S) GET requests from tests and pre-request scripts. Example: ::

    pm.sendRequest('http://example.com', function (err, res) {
        console.log(err ? err : res.json()); 
    });

Full-option HTTP(S) request: ::

    const postRequest = {
        url: 'http://example.com', method: 'POST',
        header: 'X-Foo:foo',
        body: {
            mode: 'raw',
            raw: JSON.stringify({ name: 'John' })
        } 
    };
    pm.sendRequest(postRequest, function (err, res) {
        console.log(err ? err : res.json()); 
    });


Postman Echo
============

Helper API for testing requests. Read more at: https://docs.postman-echo.com.

**Get Current UTC time in pre-request script** ::

    pm.sendRequest('https://postman-echo.com/time/now', function (err, res) {
        if (err) { console.log(err); } 
        else {
            var currentTime = res.stream.toString();
            console.log(currentTime);
            pm.environment.set("currentTime", currentTime);
        }
    }); 


Workflows
=========

Only work with automated collection runs such as with the Collection Runner or Newman. It will not have any effect when using inside the Postman App. 

Additionaly it is important to note that this will only affect the next request being executed. Even if you put this inside the pre-request script, it will not skip the current request.

**Set which will be the next request to be executed**

``postman.setNextRequest(“Request name");``

**Stop executing requests / stop the collection run**

``postman.setNextRequest(null);``
