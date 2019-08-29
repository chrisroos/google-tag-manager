# GTM Enhanced Ecommerce

## Using Data Layer

* [GTM](gtm.html)
  * Pro: No impact of user experience
  * Con: Possibility of dropped events if page unloads before event sent to GTM

* [GTM using event callback](gtm-using-event-callback.html)
  * Pro: Guarantee event is sent to GTM before user leaves the page
  * Con: Breaks the use of cmd+click to open the link in a new window
  * Con: Possible bad user experience - a delay communicating with GTM causing a delay for the user opening the link

### GTM configuration

#### Variables

* GA Settings
  * Variable type: Google Analytics settings
  * Tracking ID: &lt;Google Analytics tracking ID&gt;
  * More Settings
    * Ecommerce
      * Enabled Enhanced Ecommerce Features
        * Use data layer
* Ecommerce checkout step
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: ecommerce.checkout.actionField.step

#### Triggers

* Checkout step 1
  * Trigger type: Custom Event
  * Event name: checkout
  * This trigger fires on: Some Custom Events
    * Ecommerce checkout step
    * equals
    * 1

#### Tags

* GA Checkout step 1
  * Tag Configuration
    * Tag Type: Google Analytics: Universal Analytics
    * Track Type: Pageview
    * Google Analytics Settings: {{GA Settings}}
  * Triggering
    * Checkout step 1

---

## Using Custom JavaScript Macro

* [GTM using Custom JavaScript Macro](gtm-using-custom-js-macro.html)
  * Pro: Reduces developer effort required to that of adding data-\* attributes containing information required for the checkout
  * Con: Might become hard to manage in GTM
  * Con: It's not clear how, or even if, this approach will continue to work for all Ecommerce functionality we want to support
  * Con: It's not the recommended way of integrating with GTM (using the Data Layer is).

### How it works

* GTM listens for click events on elements that have the 'js-gtm-product' class.
* The `gtm.click` event populates the `gtm.element` with the object that's been clicked (the product link in our case).
* The `GTM Checkout XXX` variables use the `gtm.element` to read the various data-\* attributes.
* The `Ecommerce checkout step` variable returns a JavaScript object in the correct shape to represent a Checkout event. This object contains the values in the various `GTM Checkout XXX` variables.
* The `GA Ecommerce Checkout` Tag sends the data from `Ecommerce checkout step` to GA when the `Product link click` event is fired.

### GTM Configuration

#### Variables

* GA Settings
  * Variable type: Google Analytics settings
  * Tracking ID: &lt;Google Analytics tracking ID&gt;
* GTM Checkout Currency
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmCurrencyCode
* GTM Checkout Step
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmStep
* GTM Checkout Product Brand
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmProductBrand
* GTM Checkout Product Category
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmProductCategory
* GTM Checkout Product ID
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmProductId
* GTM Checkout Product Name
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmProductName
* GTM Checkout Product Price
  * Variable type: Data Layer Variable
  * Data Layer Variable Name: gtm.element.data.gtmProductPrice
* Ecommerce checkout step
  * Variable Type: Custom JavaScript
  * Custom JavaScript:
    ```
    function() {
      var ecommerceData = {
        'event': 'checkout',
        'ecommerce': {
          'currencyCode': {{GTM Checkout Currency}},
          'checkout': {
            'actionField': { 'step': {{GTM Checkout Step}} },
            'products': [
              {
                'name': {{GTM Checkout Product Name}},
                'id': {{GTM Checkout Product ID}},
                'price': {{GTM Checkout Product Price}},
                'brand': {{GTM Checkout Product Brand}},
                'category': {{GTM Checkout Product Category}},
                'variant': ''
              }
            ]
          }
        }
      };
      return ecommerceData;
    }
    ```

#### Triggers

* Product link click
  * Trigger type: Click - All elements
  * This trigger fires on: Some Clicks
    * Click Classes
    * contains
    * js-gtm-product

#### Tags

* GA Ecommerce checkout
  * Tag type: Google Analytics: Universal Analytics
  * Track Type: Event
  * Event Category: Ecommerce
  * Event Action: Checkout
  * Google Analytics Settings: {{GA Settings}}
  * Enable overriding settings in this tag
  * Ecommerce > Enable Enhanced Ecommerce features: True
  * Read Data from Variable: {{Ecommerce checkout step}}

