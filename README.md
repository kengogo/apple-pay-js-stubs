# apple-pay-js-stubs
The Apple Pay JS Stubs provide a stubbed implementation of the ApplePay JS framework allowing you to acceptance test your Apple Pay for the Web code without requiring Safari, or an iPhone with iOS 10

[![Build Status](https://travis-ci.org/indiegogo/apple-pay-js-stubs.svg?branch=master)](https://travis-ci.org/indiegogo/apple-pay-js-stubs)

This stubbed implementation substitutes the ApplePay JS API normally provided by the Safari browser on iOS 10, and macOS Sierra with a stubbed javascript implementation. 

Unlike the offical API, apple-pay-js-stubs presents no paysheet or other visual feedback when called, instead it can be configured to executes the ApplePay JS callbacks based the test scenario pre-configured by *you* the developer. 

This approach allows you to simulate both the ApplePay paysheet and user behaviour quickly and easily without requiring a physical ApplePay capable device.

# Requirements 
- ECMAScript 6 complient browser 
  - Tested in: 
    - Chrome 51.0.2704.103 (64-bit)
    - Firefox 48.0 
    - Safari Version 9.1 (11601.5.17.1)

# Installation and Usage 

## Step 1: Install apple-pay-js-stubs
### Option a: Install using node
 ```bash
npm install apple-pay-js-stubs
```

### Option b: Manually by downloading

Alternatively you can download the [apple-pay-js-stubs.js file here](https://github.com/indiegogo/apple-pay-js-stubs/blob/master/src/apple-pay-js-stubs.js)
 
## Step 2: Load Javascript file in your acceptance tests

In order for apple-pay-js-stubs.js to be available when your acceptance tests run, you'll need to load the javascript file on the page in your website which normally interacts with the window.ApplePaySession object. 

**Note: You should only do this when you're running your automated acceptance tests to avoid conflicting with Safari in your production environment**

How you go about this will depend on what language you are writing your acceptance tests in.

For acceptance tests written in Ruby's RSpec you can load `apple-pay-js-stubs.js` after you visit your ApplePay supporting webpage as follows: 
```ruby
javascript = <<-JAVASCRIPT
        var js = document.createElement("script");
        js.type = "text/javascript";
        js.src = "/assets/test_support/apple-pay-js-stubs.js";
        document.body.appendChild(js);
      JAVASCRIPT
page.execute_script(javascript)
```
(where `/assets/test_support/` is the path on your website to the apple-pay-js-stubs.js file)

Note: This approach will result in a delay in `window.ApplePaySession` being available which may not be the same as on the real ApplePay supporting browser.

## Step 3: Configure the Stubs for your current test 

Each test using ApplePay/apple-pay-js-stubs, will need to configure the ApplePaySessionStubs class (stored in window.ApplePaySession) before executing the test. This is how you configure the call backs to simulate the users, and ApplePay JS's normal behaviour for a specific scenario. 

You can configure this by running the following javascript code on you page before each test: 

```javascript
// Configure what a call to ApplePaySession.canMakePaymentsWithActiveCard(merchantIdentifier) should result to
ApplePaySession.stubCanMakePaymentsWithActiveCard = true; // ApplePaySession.canMakePaymentsWithActiveCard() returns promise resulting to true

ApplePaySession.stubExecuteAfterMerchantValidation = function(session) {
  // Call callbacks on session to simulate ApplePay JS / user behaviour 
};
```

# Example Configurations 
Some example apple-pay-js-stubs javascript configurations:

### User without ApplePay configured 
```javascript
ApplePaySession.stubCanMakePaymentsWithActiveCard = false;
```

### User cancels payment after opening paysheet 
```javascript
ApplePaySession.stubCanMakePaymentsWithActiveCard = true;
ApplePaySession.stubExecuteAfterMerchantValidation = function(session) {
  session.oncancel({})
};
```

### User selects a shipping address, then authorizes the payment
```javascript
ApplePaySession.stubCanMakePaymentsWithActiveCard = true;
ApplePaySession.stubExecuteAfterMerchantValidation = function (session) {
  var event = {};
  // 1. Stub acts as if the user selects a on a shipping address
  session.onshippingcontactselected(event);
  // 2. Stub acts as if the user authorizes the payment
  session.onpaymentauthorized(event);
};
```


# apple-pay-js-stubs Demo / Example 

You can see an example web page which demonstrates some of the capabilities of the stubs in [/example/index.html](/example/index.html).

A few things to note:
* To get the demo to load, you'll need to clone the repository locally then open example/index.html in the Chrome browser.
* This demonstrates the stubs through an interactive 'Console'. Typically usage will be driven from your acceptance tests instead of interactively. 


## Code of Conduct
Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.
