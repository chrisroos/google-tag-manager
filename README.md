Demonstrating how Google Tag Manager works.

## Examples

* [GTM](gtm.html)
* [GTM using event callback](gtm-using-event-callback.html)

## Google Tag Manager configuration

### Variables

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

### Triggers

* Checkout step 1
  * Trigger type: Custom Event
  * Event name: checkout
  * This trigger fires on: Some Custom Events
    * Ecommerce checkout step
    * equals
    * 1

### Tags

* GA Checkout step 1
  * Tag Configuration
    * Tag Type: Google Analytics: Universal Analytics
    * Track Type: Pageview
    * Google Analytics Settings: {{GA Settings}}
  * Triggering
    * Checkout step 1
