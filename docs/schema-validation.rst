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
        "type": "object",
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
    - integer
    - number
    - boolean
    - null
    - Object
    - array

Object has required property
----------------------------

This is the JSON response: ::

    {
        "code": "FX002"
    }

This is the JSON schema with a property named code of type String that is mandatory: ::

    const schema = {
        "type": "object",
        "properties": {
            "code": { "type": "string" }
        },
        "required": ["code"]
    };