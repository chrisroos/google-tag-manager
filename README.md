# GTM Enhanced Ecommerce

This README and the associated HTML pages demonstrate different ways of enabling the Enhanced ecommerce in Google Tag Manager.

To use the examples you'll need to:

1. Create a Google Tag Manager account
2. Configure a Container in GTM
3. Update the GTM snippets in the HTML pages to use your Container
4. Configure GTM by following the instructions below
5. Enable Preview mode in GTM to test the effect of the configuration in GTM

[View this on GitHub pages](https://chrisroos.github.io/google-tag-manager/) for the links to work as expected.

---

## Using Data Layer

* Pro: Recommended way of integrating with GTM

* [GTM without waiting for GTM](gtm.html)
  * Pro: No impact of user experience
  * Con: Possibility of dropped events if page unloads before event sent to GTM

* [GTM using event callback](gtm-using-event-callback.html)
  * Pro: Guarantee event is sent to GTM before user leaves the page
  * Con: Breaks the use of cmd+click to open the link in a new window
  * Con: Possible bad user experience - a delay communicating with GTM causing a delay for the user opening the link

### How it works

* We use jQuery to attach an event handler to the click event of elements with the `js-gtm-track` class
* This event handler constructs an `ecommerce` shaped checkout event object using the values from the data-\* attributes on the element that was clicked, and pushes this checkout event onto the Data Layer.
* The `Checkout step 1` trigger is fired when the `checkout` step 1 event is seen in the Data Layer
* The `GA Checkout step 1` Tag sends the content of `ecommerce` in the Data Layer to GA when `Checkout step 1` fires
* In the event callback variant, we prevent the default link behaviour (using `e.preventDefault()`) and instead pass an `eventCallback` function along with the event which GTM invokes once the data has been received.

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
  * Tag Configuration
    * Tag type: Google Analytics: Universal Analytics
    * Track Type: Event
    * Event Category: Ecommerce
    * Event Action: Checkout
    * Google Analytics Settings: {{GA Settings}}
    * Enable overriding settings in this tag
    * Ecommerce > Enable Enhanced Ecommerce features: True
    * Read Data from Variable: {{Ecommerce checkout step}}
  * Triggering
    * Product link click
