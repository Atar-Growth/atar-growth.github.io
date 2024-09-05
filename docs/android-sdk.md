---
hide:
  - footer
---
# Integrate the Atar Android SDK

The guide below will walk you through the steps to integrate the Atar Android SDK into your app, in order to start showing offers to your users.

## Step 1: Create an account and get your App Key

If you haven't already, create an account on the [Atar Dashboard](https://app.atargrowth.com/). Once you've created an account, you'll be able to create an app and get your App Key from the [settings page](https://app.atargrowth.com/settings).

## Step 2: Add the Atar Android SDK to your app

The Atar SDK is distributed as an AAR file. You can follow these instructions to add it to your app.

### 2.1 Download the AAR

Download the Atar Android SDK AAR file from here: [https://host.atargrowth.com/public/atar-sdk-1.1.0.aar](https://host.atargrowth.com/public/atar-sdk-1.1.0.aar)

### 2.2 Add the AAR to your project

Place the Atar SDK in the `libs` folder of your Android project. If you don’t have a `libs` folder, create one. It should be placed in the same folder as your `src` folder like so:

```bash
project-folder/src/main/java/com/example/project/MainActivity.java
project-folder/libs/atar-sdk-1.1.0.aar
```

### 2.3 Add the AAR to your build.gradle

Open your app's build.gradle file and add the following line to the dependencies block:

```groovy
dependencies {
    implementation files('libs/atar-sdk-1.1.0.aar')
}
```

or in other Gradle versions:

```groovy
dependencies {
  implementation(files('libs/atar-sdk-1.1.0.aar'))
}
```

## Step 3: Initialize the Atar SDK

Initialize the Atar SDK in your **Application** class's **onCreate** method.

=== "Java"
    ```java
    @Override
    public void onCreate() {
      super.onCreate();
      Atar.getInstance(this).onCreate("YOUR APP KEY");
      // any other code you have
    }
    ```

=== "Kotlin"
    ```kotlin
    override fun onCreate() {
      super.onCreate()
      Atar.getInstance(this).onCreate("YOUR APP KEY")
      // any other code you have
    }
    ```

Replace **YOUR APP KEY** with the App Key you got from the Atar Dashboard.

## Step 4: Show Offers after the user has completed a transaction

Once you've initialized the Atar SDK, you can show offers to your users. Here's an example of how you can show an interstitial offer after the user has completed a transaction.

=== "Java"
    ```java
    OfferRequest request = new OfferRequest(new OfferRequestCallback() {
        @Override
        public void onPopupShown(boolean success, String error) {
            if (success) {
                Log.i("YourApp", "Popup shown successfully");
            } else {
                Log.e("YourApp", "Error: " + error);
            }
        }
        @Override
        public void onPopupCanceled() {
            Log.i("YourApp", "Popup was canceled and closed");
        }
        @Override
        public void onClicked() {
            Log.i("YourApp", "Popup was clicked and closed!");
        }
    });

    // Complete the information about the user and transaction

    // Required
    request.event = "purchase";
    request.userId = "userId1234";
    request.referenceId = "12345"; // this is some unique reference ID for the transaction
    
    // Recommended
    request.email = "user1234@email.com";
    request.phone = "1234567890";
    
    request.gender = "M";
    request.dob = "YYYYMMDD";
    request.address1 = "123 Example St";
    request.address2 = "Suite 123";
    request.city = "City";
    request.state = "State";
    request.zip = "12345";
    request.country = "Country";
    request.amount = 29.98;

    Atar.getInstance(this).showOfferPopup(request);
    ```

=== "Kotlin"
    ```kotlin
    val request = OfferRequest(object : OfferRequestCallback {
        override fun onPopupShown(success: Boolean, error: String?) {
            if (success) {
                Log.i("YourApp", "Popup shown successfully")
            } else {
                Log.e("YourApp", "Error: " + error?.message)
            }
        }
        override fun onCanceled() {
            Log.i("YourApp", "Popup was canceled and closed")
        }
        override fun onClicked() {
            Log.i("YourApp", "Popup was clicked and closed!")
        }
    })

    // Complete the information about the user and transaction

    // Required
    request.event = "purchase";
    request.userId = "userId1234";
    request.referenceId = "12345"; // this is some unique reference ID for the transaction

    // Recommended
    request.email = "user1234@email.com";
    request.phone = "1234567890";

    request.gender = "M";
    request.dob = "YYYYMMDD";
    request.address1 = "123 Example St";
    request.address2 = "Suite 123";
    request.city = "City";
    request.state = "State";
    request.zip = "12345";
    request.country = "Country";
    request.amount = 29.98;

    Atar.getInstance(this).showOfferPopup(request);
    ```

Replace the values in the **OfferRequest** object with the appropriate values for your user and transaction. Below you can find the full documented list of fields you can use in the **OfferRequest** object.

Note that the callback is optional and purely in case you would like to tie any additional functionality to the presentation of the offer.

### Offer Request Object Fields

| Req. | Field name  | Description                                    | Data type |
|------|-------------|------------------------------------------------|-----------|
| Req  | event       | User event that preceded the offers            | string    |
| Req  | referenceId | Some ID for reference for deduplication.       | string    |
| Req  | userId      | Some ID for the user.                          | string    |
| Rec  | email       | User email.                                    | string    |
| Rec  | phone       | User phone number.                             | string    |
| Rec  | gender      | User gender: M/F                               | string    |
| Rec  | dob         | Date of birth, YYYYMMDD                        | string    |
| Rec  | firstName   | User first name                                | string    |
| Rec  | lastName    | User last name                                 | string    |
| Opt  | amount      | Transaction amount                             | string    |
| Opt  | quantity    | Quantity of items purchased                    | int       |
| Opt  | paymentType | credit, PayPal, or other                       | string    |
| Opt  | address1    | Street address line 1                          | string    |
| Opt  | address2    | Street address line 2                          | string    |
| Opt  | city        | City                                           | string    |
| Opt  | state       | State                                          | string    |
| Opt  | country     | Country                                        | string    |

## Step 5: Trigger offer notifications on user action

You can also trigger offer notifications on user actions. Here's an example of how you can trigger an offer notification.

=== "Java"
    ```java
    OfferRequest request = new OfferRequest(new OfferRequestCallback() {
        @Override
        public void onNotifScheduled(boolean success, String error) {
            if (success) {
                Log.i("YourApp", "Popup shown successfully");
            } else {
                Log.e("YourApp", "Error: " + error);
            }
        }
        @Override
        public void onClicked() {
            Log.i("YourApp", "Popup was clicked and closed!");
        }
    });
    request.event = "level_completed";
    request.referenceId = UUID.randomUUID().toString();
    request.userId = "userId1234";
    
    Atar.getInstance().triggerOfferNotification(request);
    ```

=== "Kotlin"
    ```kotlin
    val request = OfferRequest(object : OfferRequestCallback {
        override fun onNotifScheduled(success: Boolean, error: String?) {
            if (success) {
                Log.i("YourApp", "Popup shown successfully")
            } else {
                Log.e("YourApp", "Error: " + error?)
            }
        }
        override fun onClicked() {
            Log.i("YourApp", "Popup was clicked and closed!")
        }
    })

    request.event = "level_completed"
    request.referenceId = UUID.randomUUID().toString()
    request.userId = "userId1234"
    
    Atar.getInstance().triggerOfferNotification(request)
    ```

### Customize the prefix for the push message

You can customize the prefix for the push message by passing your custom string to the triggerOffer method as shown below:

=== "Java"
    ```java
    Atar.getInstance().triggerOfferNotification(request, "You're awesome!");
    ```

=== "Kotlin"
    ```kotlin
    Atar.getInstance().triggerOfferNotification(request, "You're awesome!")
    ```

## Step 6: Enable or disable the post session notification

You have full control over whether post session notifications are enabled or disabled. By default, post session notifications are disabled, so you would have to have previously enabled it.

### Disable post session notifications globally

You can head to the [dashboard settings](https://app.atargrowth.com/settings) and disable post session notifications globally. This will disable post session notifications for all users.

### Disable post session notifications for a specific user

To disable for a specific user, you can use the following method in the SDK whenever. This is persisted in user defaults and will be remembered for the user.

=== "Java"
    ```java
    Atar.getInstance().setPostSessionNotifDisabled(true);
    ```

=== "Kotlin"
    ```kotlin
    Atar.getInstance().setPostSessionNotifDisabled(true)
    ```

## Need help?

If you have any questions or need help, please contact us at [support@atargrowth.com](mailto:support@atargrowth.com).
