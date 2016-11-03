# Adyen Checkout

### Index
**[About](#about)**

**[UI implementation](#ui-implementation)**

**[Credit Card Form UI implementation](#credit-card-form-ui-implementation)**

**[Library implementation](#library-implementation)**

**[Merchant server](#merchant-server)**

### About

Adyen Checkout is a native client for accepting payments in-app.

By using the token generated by Adyen, you don't have to deal with PCI compliance. 

You can integrate with Adyen library by either:

* Importing our UI library and allowing us to take care of the token generation and data encryption. 
* Integrating Adyen library SDK to have the freedom of creating your own UI model and make calls to our library.

### UI implementation

Include the payment form to make your first payment.

#### Importing the binaries

1. In your `build.gradle` of the root directory add the following line:
    
    ```gradle
    buildscript {
        repositories {
            ...
        }
        dependencies {
            ...
            classpath 'de.undercouch:gradle-download-task:2.1.0'
        }
    }
    ```
    This plugin will allow Gradle to download the Adyen library.
    
2. In your `build.gradle` of the app module add the following task:

    ```gradle
    import de.undercouch.gradle.tasks.download.Download
    
    ...
    
    task downloadAdyenLibrary(type: Download) {
        src 'https://raw.githubusercontent.com/Adyen/AdyenCheckout-android/master/adyenpaysdk/adyenpaysdk-1.0.0.aar'
        src 'https://raw.githubusercontent.com/Adyen/AdyenCheckout-android/master/adyenuisdk/adyenuisdk-1.0.0.aar'
        dest('libs');
    }
    ```
   Once you run this task using `./gradlew downloadAdyenLibrary` command, the `adyenuisdk` and `adyenpaysdk` .aar files will be downloaded in you `libs` folder.
   
3. Next add the following snippet in your `build.gradle` of the app module:

    ```gradle
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
    ```


#### Importing the sources

Follow these steps to edit your `build.gradle` and start using Adyen checkout for your Android application:

1. Run `git pull https://github.com/Adyen/AdyenCheckout-android.git`.
2. Copy `adyenpaysdk` and `adyenuisdk` in your application folder.
3. Add `':adyenuisdk', ':adyenpaysdk'` in your `settings.gradle` file. 
4. Copy `volley-release.aar` from one of the modules added above `lib` folder into your `app` lib folder.
5. Add the following code snippet to your `android` module insinde `build.gradle`"

    ``` gradle
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
    ```

6. Add the following code snippet to your `dependencies` module insinde `build.gradle`:

    ``` gradle
    compile project(':adyenpaysdk')
    compile project(':adyenuisdk')
    ```

#### Code implementation

Add the `PaymentActivity` in your `AndroidManifest.xml`:

``` xml
<activity
    android:name="adyen.com.adyenuisdk.PaymentActivity"
    android:screenOrientation="locked"/>
```

Next, in order to lauch `PaymentActivity` add the code snippet bellow on the click event on your checkout button:

```java
CheckoutRequest checkoutRequest = new CheckoutRequest();
try {
    checkoutRequest.setBrandColor(your_brand_color);
    checkoutRequest.setBrandLogo(your_brand_logo);
    checkoutRequest.setCheckoutAmount(checkout_amount);
    checkoutRequest.setCurrency(Currency.EUR);
    checkoutRequest.setToken(your_token);
    checkoutRequest.setTestBackend(true);//default is set to false. Set it to true if you want to use Adyen's test back-end.

    Intent intent = new PaymentActivity.PaymentActivityBuilder(checkoutRequest).build(this, context);
    startActivity(intent);
} catch (CheckoutRequestException e) {
    Log.e("tag", e.getMessage(), e);
}
```

Make sure your Activity or Fragment `implements AdyenCheckoutListener`. By doing this you will also implement `checkoutAuthorizedPayment(String paymentData, float amount)`
as well as `checkoutFailedWithError(String errorMessage)`. The parameter `paymentData` of `checkoutAuthorizedPayment` represents the encrypted payment
data that needs to be sent to the merchant server.

Check `adyen.com.pay.MainActivity.java` for a complete code example of a UI library implementation.

### Credit Card Form UI implementation

This is a combination of the Adyen checkout UI library and the Adyen checkout SDK implementation. This type of implementation
offers the merchant the option to implement our UI for the credit card form and design their own screen around the credit card form. 
Follow these steps to edit your `build.gradle` and start using Adyen checkout for your Android application:

1. Run `git pull https://github.com/Adyen/AdyenCheckout-android.git`.
2. Copy `adyenpaysdk` and `adyenuisdk` in your application folder.
3. Add the following code snippet to your `dependencies` module insinde build.gradle:

    ```gradle
    compile project(':adyenpaysdk')
    compile project(':adyenuisdk')
    ```
    
Now that both the UI library and the SDK are part of your project you need to add the credit card form inside your activity or fragment xml:

``` xml
<com.adyen.adyencheckout.ui.CreditCardForm
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

As you are implementing your own payment button, you need to call the following API calls to retrieve the encrypted card data, and send it to the your server.

```java
public void initCreditCardPayment(final CardPaymentData cardPaymentData) {
    Adyen.getInstance().fetchPublickKey(new CompletionCallback() {
        @Override
        public void onSuccess(String result) {
            String ecryptedData = cardPaymentData.serialize();
            
            //Send ecryptedData to your merchant server
        }
        
        @Override
        public void onError(String message) {

        }
    });
}
```

Check our `adyen.com.adyenuisdk.PaymentFragment.java` for a complete code example of a UI library implementation.

### Library implementation

Follow these steps to edit your `build.gradle` and start using Adyen checkout for your Android application:

1. Run `git pull https://github.com/Adyen/AdyenCheckout-android.git`.
2. Copy `adyenpaysdk` in your application folder.
3. Add the following code snippet to your `dependencies` module insinde build.gradle:

    ```gradle
    compile project(':adyenpaysdk')
    ```
    
After including the library SDK in your Android application, implement the following API calls to retrieve the encrypted card data, and send it to your server.

```java
public void initCreditCardPayment(final CardPaymentData cardPaymentData) {
    Adyen.getInstance().fetchPublickKey(new CompletionCallback() {
        @Override
        public void onSuccess(String result) {
            String ecryptedData = cardPaymentData.serialize();
            
            //Send ecryptedData to your merchant server
        }
        
        @Override
        public void onError(String message) {

        }
    });
}
```

Check our `adyen.com.adyenpaysdk.Adyen.java` for a complete code example of a library SDK implementation.

### Merchant server

For an example of how you can build your own merchant server, see the <a href="https://github.com/Adyen/AdyenCheckout-ios/blob/master/ServerSideExample/Parse/cloud/app.js" target="_blank"> Merchant Server sample code. </a>
