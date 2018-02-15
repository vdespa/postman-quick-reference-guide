*********
Workflows
*********

How to extract value of an authentication token from a login response body and pass in subsequent request as 'Bearer Token'?
-----------------------------------------------------------------------------------------------------------------------------

Given the response from the authentication server:

.. code-block:: javascript

    {
        "accessToken": "foo",
        "refreshToken": "bar"
        "expires": "1234"
    }

Extract the value of the token from the response in the **Tests** tab: ::

    var jsonData = pm.response.json();
    var token = jsonData.accessToken;

Set the token as a variable (global, environment, etc) so that it can used in later requests: ::

    pm.globals.set('token', token);

To use the token in the next request, in the headers part the following needs to be added (key:value example below): ::

    Authorization:Bearer {‌{token}}


How to read links from response and execute a request for each of them?
-----------------------------------------------------------------------

Given the following response: ::

    {
        "links": [
            "http://example.com/1",
            "http://example.com/2",
            "http://example.com/3",
            "http://example.com/4",
            "http://example.com/5"
        ]
    }

With the following code we will read the response, iterate over the links array and for each link will submit a request, using ``pm.sendRequest``. For each response we will assert the status code. ::

    // Parse the response 
    var jsonData = pm.response.json();

    // Check the response
    pm.test("Response contains links", function () {
        pm.response.to.have.status(200);
        pm.expect(jsonData.links).to.be.an('array').that.is.not.empty;
    });


    // Iterate over the response
    var links = jsonData.links;

    links.forEach(function(link) {
        pm.test("Status code is 404", function () {
            pm.sendRequest(link, function (err, res) {
                pm.expect(res).to.have.property('code', 404);
            });
        });
    });


How to create request parameterization from Excel or JSON file?
----------------------------------------------------------------

TODO

