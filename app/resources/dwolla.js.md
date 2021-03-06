---
layout: twoColumn
section: resources
type: article
title:  "Dwolla.js"
description: "Quickly integrate instant bank verification for developers using the Dwolla ACH API."
---

# Dwolla.js

Dwolla.js is a client-side JavaScript library with the primary function of securely transmitting sensitive data (bank account and routing number) from your application's front-end to Dwolla without the data passing through your server. When you attach a bank account to a Dwolla account or white label Customer, use dwolla.js and let Dwolla reduce your risk of handling sensitive data. You will generate a funding sources token, collect the user's bank account information, and call a JavaScript function to send this data to Dwolla.

When you collect and submit the user's bank account information, dwolla.js has built-in validation that will trigger an error if any of the required fields are invalid.

### Testing Dwolla.js and Customization
Test Dwolla.js and customization functionality <a href="https://www.dwolla.com/dwollajs-bank-verification">here.</a>

### Generate a funding sources token
Before utilizing dwolla.js to add a new funding source, you need to generate a funding sources token. Your server initiates a POST request to Dwolla, specifying for which Dwolla account or white label Customer you want to add a bank account. Dwolla will respond with a funding sources `token` that expires in an hour. This token will be sent to the client and used to authenticate the HTTP request asking Dwolla to add a new funding source. 

```raw
curl -X POST 
\ -H "Content-Type: application/vnd.dwolla.v1.hal+json"
\ -H "Accept: application/vnd.dwolla.v1.hal+json"
\ -H "Authorization: Bearer qe634nV7dIYpYDf3VGZPciziPU2BCboUZ7G7EG8XEyGswKkBV5"
\ "https://api-uat.dwolla.com/customers/28138609-30FF-4607-B28C-4A3872F8FD4A/funding-sources-token"

HTTP/1.1 200 OK
{
  "_links": {
    "self": {
      "href": "https://api-uat.dwolla.com/customers/28138609-30ff-4607-b28c-4a3872f8fd4a/funding-sources-token"
    }
  },
  "token": "Z9BvpNuSrsI7Ke1mcGmTT0EpwW34GSmDaYP09frCpeWdq46JUg"
}
```
```ruby
# coming soon
```
```javascript
dwolla.then(function(dwolla) {
    dwolla.customers.createFundingSourcesTokenForCustomer({id: '247B1BD8-F5A0-4B71-A898-F62F67B8AE1C'})
    .then(function(data) {
       console.log(data.obj.token);
    });
});
```
```python
# coming soon
```
```php
// coming soon
```

### Include dwolla.js and add form to collect bank account information
Begin the client-side implementation by including dwolla.js in the HEAD of your HTML page. Please note that the example below uses development version 1 of dwolla.js. You could also use the minified version of `<script src="https://cdn.dwolla.com/1/dwolla.min.js"></script>`.

```htmlnoselect
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
  <script src="https://cdn.dwolla.com/1/dwolla.js"></script>
</head>
```

Next, add the form to the body of the page where you want to collect the user's bank account information.

```htmlnoselect
<form>
  <div>
    <label>Routing number</label>
    <input type="text" id="routingNumber" placeholder="273222226" />
  </div>
  <div>
    <label>Account number</label>
    <input type="text" id="accountNumber" placeholder="account number" />
  </div>
  <div>
    <label>Bank account name</label>
    <input type="text" id="name" placeholder="name" />
  </div>
  <div>
    <select name="type" id="type">
      <option value="checking">checking</option>
      <option value="savings">savings</option>
    </select>
  </div>
  <div>
    <input type="submit" value="Add Bank">
  </div>
</form>

<div id="logs">
</div>
```


### Configure and call JavaScript function to create a new funding source
Once you initialize `dwolla.js` on your page, you can configure and create the function that will call `dwolla.fundingSources.create()`. For this example, jQuery is used to call the function that creates a funding source when the user clicks the "Add Bank" button on your page. To test in the sandbox environment, use the `dwolla.configure()` helper function and pass in the value of `uat`. Additional configuration options are available below. 

In our example, `dwolla.fundingSources.create()` takes three arguments: a string value of the funding sources token, JavaScript object containing bank account information entered by the user, and a callback function that will handle any error or response.

#### Additional configuration options
There are 2 config values you can manually set:

- `dwolla.config.dwollaUrl` (default: `https://www.dwolla.com`)
- `dwolla.config.apiUrl` (default: `https://api.dwolla.com`)

Or you can use the `dwolla.configure` helper for preset configurations:

```javascriptnoselect
//UAT
// dwolla.config.dwollaUrl = 'https://uat.dwolla.com'
// dwolla.config.apiUrl = 'https://api-uat.dwolla.com'
dwolla.configure('uat')
//Production
// dwolla.config.dwollaUrl = 'https://www.dwolla.com'
// dwolla.config.apiUrl = 'https://api.dwolla.com'
dwolla.configure('prod') // default
```

```javascriptnoselect
<script type="text/javascript">
$('form').on('submit', function() {
  dwolla.configure('uat');
  var token = 'Z9BvpNuSrsI7Ke1mcGmTT0EpwW34GSmDaYP09frCpeWdq46JUg';
  var bankInfo = {
    routingNumber: $('routingNumber').val(),
    accountNumber: $('accountNumber').val(),
    type: $('type').val(),
    name: $('name').val()
  }
  dwolla.fundingSources.create(token, bankInfo, callback);
  return false
})

function callback(err, res) {
  $div = $('<div />')
  var logValue = {
    error: err,
    response: res
  }
  $div.text(JSON.stringify(logValue))
  console.log(logValue)
  $('#logs').append($div)
}
```

### Callback function - handling the error or response
The callback function (err, res) allows you to determine if there is an error with the request (e.g. `Routing number invalid.`) or if the response was successful. In the example above, we are displaying any error or response within a `logs` container on our page.

* If there is an error: Display the error to the user to have them correct any fields, and have them re-attempt to add their bank.
  * Example: `{"error":{"code":"ValidationError","message":"Validation error(s) present. See embedded errors list for more details.","_embedded":{"errors":[{"code":"Invalid","message":"Routing number invalid.","path":"/routingNumber"}]}}}`
* If successful: You will receive a JSON response that includes a link to the newly attached funding source. 
  * Example:  `{"error":null,"response":{"_links":{"funding-source":{"href":"https://api-uat.dwolla.com/funding-sources/746d5c93-acb9-4826-a9c1-78ecf16401a6"}}}}`

## Using Dwolla.js for Instant Account Verification (IAV)
For white label partners, `dwolla.js` has the added function of facilitating Instant Account Verification (IAV) on their Customer's bank or credit union account. By calling a separate function, `dwolla.iav.start()` the white label partner application can render the IAV flow within a specified container. Read more about [funding source verification](/resources/funding-source-verification.html) and how to use dwolla.js to quickly add and verify a white label Customer’s bank account.

## Using Dwolla.js for On-demand bank transfers
On-demand bank transfers allow Dwolla white label partners to receive variable totals from a payer’s bank account via an ACH transaction—perfect for companies with usage-based business models, such as utilities or advertising platforms. No other third-party gateways or merchant accounts required, just your Dwolla integration and payer authorization.

This is an account level setting for a white label partner. When enabled, the end user is presented with text on the bank selection screen within the IAV flow giving authorization to Dwolla for future variable payments.

![Screenshot of On-demand](/images/on-demand-iav.png "On-demand bank transfers")

Once you have collected all of the authorizations required for a bank transfer, including the additional authorization from the Customer for on-demand transfers, you will be able to <a href="https://docsv2.dwolla.com/#initiate-transfer">initiate transfers</a> for variable amounts and dates.
