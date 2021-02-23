*****************
Schema validation
*****************

This section contains different examples of validating JSON responses using the Ajv schema validator. I do not recommend using the tv4 (Tiny Validator for JSON Schema v4).

Response is an object
---------------------

This is the JSON response: ::

    {}

This is the JSON schema: ::

    const schema = {
        "type": "object"
    };

And here is the test: ::

    pm.test("Validate schema", () => {
        pm.response.to.have.jsonSchema(schema);
    });


Object has optional property
----------------------------

This is the JSON response: ::

    {
        "code": "FX002"
    }

This is the JSON schema with a property named code of type String: ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" }
        }
    };

Possible types are:
    - string
    - number
    - boolean
    - null
    - object
    - array

Object has required property
----------------------------

Given this JSON response: ::

    {
        "code": "FX002"
    }

This is the JSON schema with a property named "code" of type String that is mandatory: ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" }
        },
        "required": ["code"]
    };

Nested objects
--------------

Given this JSON response: ::

    {
        "code": "2",
        "error": {
            "message": "Not permitted."
        }
    }

This is the JSON schema with the an nested object named "error" that has a property named "message" that is a string. ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" },
            "error": { 
                "type": "object",
                "properties": {
                    "message": { type: "string" }
                },
                "required": ["message"]
            }
        },
        "required": ["code", "error"]
    };
