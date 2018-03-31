**********
Assertions
**********

Assertions in Postman are based on the capabilities of the Chai Assertion Library. 
You can read an extensive documentation on Chai by visiting http://chaijs.com/api/bdd/

How find object in array by property value?
-------------------------------------------

Given the following response: ::

    {
        "companyId": 10101,
        "regionId": 36554,
        "filters": [
            {
                "id": 101,
                "name": "VENDOR",
                "isAllowed": false
            },
            {
                "id": 102,
                "name": "COUNTRY",
                "isAllowed": true
            },
            {
                "id": 103,
                "name": "MANUFACTURER",
                "isAllowed": false
            }
        ]
    }

Assert that the property isAllowed is true for the COUNTRY filter. ::

    pm.test("Check the country filter is allowed", function () {
        // Parse response body
        var jsonData = pm.response.json();
        
        // Find the array index for the COUNTRY
        var countryFilterIndex = jsonData.filters.map(
                function(filter) {
                    return filter.name; // <-- HERE is the name of the property
                }
            ).indexOf('COUNTRY'); // <-- HERE is the value we are searching for
        
        // Get the country filter object by using the index calculated above
        var countryFilter = jsonData.filters[countryFilterIndex];
        
        // Check that the country filter exists
        pm.expect(countryFilter).to.exist;
        
        // Check that the country filter is allowed
        pm.expect(countryFilter.isAllowed).to.be.true;
    });


How find nested object by object name?
--------------------------------------

Given the following response: ::

    {
        "id": "5a866bd667ff15546b84ab78",
        "limits": {
            "59974328d59230f9a3f946fe": {
                "lists": {
                    "openPerBoard": {
                        "count": 13,
                        "status": "ok", <-- CHECK ME
                        "disableAt": 950,
                        "warnAt": 900
                    },
                    "totalPerBoard": {
                        "count": 20,
                        "status": "ok",  <-- CHECK ME
                        "disableAt": 950,
                        "warnAt": 900
                    }
                }
            }
        }
    }

You want to check the value of the `status` in both objects (openPerBoard, totalPerBoard). The problem is that in order to reach both objects you need first to reach the lists object, which itself is a property of a randomly named object (59974328d59230f9a3f946fe). 

So we could write the whole path ``limits.59974328d59230f9a3f946fe.lists.openPerBoard.status`` but this will probably work only once.

For that reason it is first needed to search inside the ``limits`` object for the ``lists`` object. In order to make the code more readable, we will create a function for that: ::

    function findObjectContaininsLists(limits) {
        // Iterate over the properties (keys) in the object
        for (var key in limits) {
            // console.log(key, limits[key]);
            // If the property is lists, return the lists object
            if (limits[key].hasOwnProperty('lists')) {
                // console.log(limits[key].lists);
                return limits[key].lists;
            }
        }
    }

The function will iterate over the limits array to see if any object contains a ``lists`` object.

Next all we need to do is to call the function and the assertions will be trivial: ::

    pm.test("Check status", function () {
        // Parse JSON body
        var jsonData = pm.response.json();
        
        // Retrieve the lists object
        var lists = findObjectContaininsLists(jsonData.limits);
        pm.expect(lists.openPerBoard.status).to.eql('ok');
        pm.expect(lists.totalPerBoard.status).to.eql('ok');
    });


How to compare value of a response with an already defined variable?
---------------------------------------------------------------------

Lets presume you have a value from a previous response (or other source) that is saved to a variable. ::

    // Getting values from response
    var jsonData = pm.response.json();
    var username = jsonData.userName;

    // Saving the value for later use
    pm.globals.set("username", username);

How do you compare that variable with values from another API response?

In order to access the variable in the script, you need to use a special method, basically the companion of setting a variable. Curly brackets will not work in this case: ::

    pm.test("Your test name", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.value).to.eql(pm.globals.get("username"));
    });

How to compare value of a response against multiple valid values?
-----------------------------------------------------------------

Given the following API response: ::

    {
        "SiteId": "aaa-ccc-xxx",
        "ACL": [
            {
                "GroupId": "xxx-xxx-xx-xxx-xx",
                "TargetType": "Subscriber"
            }
        ]
    }

You want to check that ``TargetType`` is *Subscriber* or *Customer*.

The assertion can look like: ::

    pm.test("Should be subscriber or customer", function () {
        var jsonData = pm.response.json();
        pm.expect(.TargetType).to.be.oneOf(["Subscriber", "Customer"]);
    });

where:
    - jsonData.ACL[0] is the first element of the ACL array
    - to.be.oneOf allows an array of possible valid values


How to parse a HTML response to extract a specific value?
---------------------------------------------------------

Presumed you want to get the _csrf hidden field value for assertions or later use from the response below: ::

    <form name="loginForm" action="/login" method="POST">
            <input type="hidden" name="_csrf" value="a0e2d230-9d3e-4066-97ce-f1c3cdc37915" />
            <ul>
                <li>
                    <label for="username">Username:</label>
                    <input required type="text" id="username" name="username" />
                </li>
                <li>
                    <label for="password">Password:</label>
                    <input required type="password" id="password" name="password" />
                </li>
                <li>
                    <input name="submit" type="submit" value="Login" />
                </li>
            </ul>
    </form>

To parse and retrive the value, we will use the cherrio JavaScript library: ::

    // Parse HTML and get the CSRF token
    responseHTML = cheerio(pm.response.text());
    console.log(responseHTML.find('[name="_csrf"]').val());

Cheerio is desinged for non-browser use and implements a subset of the jQuery functionality. Read more about it at https://github.com/cheeriojs/cheerio


How to fix the error "ReferenceError: jsonData is not defined"?
---------------------------------------------------------------

If you have a script like this: ::

    pm.test("Name should be John", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.name).to.eql('John');
    });

    pm.globals.set('name', jsonData.name);


You will get the error ``ReferenceError: jsonData is not defined`` while setting the global variable. 
 
The reason is that ``jsonData`` is only defined inside the scope of the anonymous function (the part with ``function() {...}`` inside ``pm.test``). Where you are trying to set the global variables is outside the function, so ``jsonData`` is not defined. ``jsonData`` can only live within the scope where it was defined. 

So you have multiple ways to deal with this:

1. define ``jsonData`` outside the function, for example before your pm.test function (preferred) ::

    var jsonData = pm.response.json(); <-- defined outside callback

    pm.test("Name should be John", function () {
        pm.expect(jsonData.name).to.eql('John');
    });

    pm.globals.set('name', jsonData.name);


2. set the environment or global variable inside the anonymous function (I would personally avoid mixing test / assertions with setting variables but it would work). ::

    pm.test("Name should be John", function () {
        var jsonData = pm.response.json();
        pm.expect(jsonData.name).to.eql('John');
        pm.globals.set('name', jsonData.name); // <-- usage inside callback
    });

Hope this helps and clarifies a bit the error.

How to do a partial object match assertion?
-------------------------------------------

Given the reponse: ::

    {
        "uid": "12344",
        "pid": "8896",
        "firstName": "Jane",
        "lastName": "Doe",
        "companyName": "ACME"
    }

You want to assert that a part of the reponse has a specific value. For example you are not interested in the dynamic value of uid and pid but you want to assert firstName, lastName and companyName. 

You can do a partial match of the response by using the ``to.include`` expression. Optionally you can check the existence of the additional properties without checking the value. ::

    pm.test("Should include object", function () {
        var jsonData = pm.response.json();
        var expectedObject = {
            "firstName": "Jane",
            "lastName": "Doe",
            "companyName": "ACME"
        }
        pm.expect(jsonData).to.include(expectedObject);

        // Optional check if properties actually exist
        pm.expect(jsonData).to.have.property('uid');
        pm.expect(jsonData).to.have.property('pid');
    });
