********************
JavaScript libraries
********************

Postman comes with a few built-in libraries. If you prefer to add additioal JavaScript libraries, please take a look at the Custom libraries section.

Built-in JavaScript libraries
-----------------------------

**cheerio**

Simple library for working with the DOM model. Useful if you are getting back HTML. ::

    responseHTML = cheerio(pm.response.text());
    console.log(responseHTML.find('[name="firstName"]').val());

Example fetching a CSRF code form the meta tag: ::

    const $ = cheerio.load('<meta name="csrf" Content="the code">');
    console.log($("meta[name='csrf']").attr("content"));

Read more: https://github.com/cheeriojs/cheerio

**Moment.js**

Moment.js is already built into Postman and can make your life easier if you are working with dates. 

Here is a basic usage example: ::

    const moment = require('moment');
    moment('2021-08-15').add(1, 'days').format('DD.MM.YYYY')

This script will parse a date, will add one day and will reformat the date.

Read more: https://momentjs.com/docs

**crypto-js**

Library which implements different crypto functions.

Hash string using SHA256 ::

    CryptoJS.SHA256("some string").toString()

HMAC-SHA1 encryption ::

    CryptoJS.HmacSHA1("Message", "Key").toString()

AES Encryption ::

    const encryptedText = CryptoJS.AES.encrypt('message', 'secret').toString();

AES Decryption ::

    const plainText = CryptoJS.AES.decrypt(encryptedText, 'secret').toString(CryptoJS.enc.Utf8);

Read more: https://www.npmjs.com/package/crypto-js

Node.js libraries
-----------------

NodeJS modules that are available inside Postman:

- path
- assert
- buffer
- util
- url
- punycode
- querystring
- string_decoder
- stream
- timers
- events


Custom libraries
----------------

There is no standard way of including 3rd party JavaScript libraries. 

Currently the only way is to fetch (and optionally store) the content of the JavaScript library and to use the JavaScript `eval` function to execute the code.

Template: ::

    pm.sendRequest("https://example.com/your-script.js", (error, response) => {
        if (error || response.code !== 200) {
            pm.expect.fail('Could not load external library');
        }
    
        eval(response.text());
        
        // YOUR CODE HERE
    });

Example loading a library using unpkg as a CDN: ::

    pm.sendRequest("https://unpkg.com/papaparse@5.1.0/papaparse.min.js", (error, response) => {
        if (error || response.code !== 200) {
            pm.expect.fail('Could not load external library');
        }
    
        eval(response.text());

        const csv = `id,name\n1,John`;
        const data = this.Papa.parse(csv); // notice the this
        console.log(data);
    });

Notice: in order to load a library and use it in Postman, the JavaScript code needs to be "compiled" and ready for distribution. Usually the code will be available as a \*.min.js file or within the dist or umd folder.