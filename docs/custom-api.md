---
hide:
  - footer
---
# Atar Custom API Integration

The guide below will walk you through the steps to integrate Atar into your app via a custom integration

## Step 1: Create an account and get your App Key

If you haven't already, create an account on the [Atar Dashboard](https://app.atargrowth.com/). Once you've created an account, you'll be able to create an app and get your App Key from the [settings page](https://app.atargrowth.com/settings).

## Step 2: Retrieve an offer via API

The process to retrieve an offer via the API is relatively simple.

### 2.1 First build the request object

In order to retrieve an offer, you need to build a request object. The request object is a JSON object that contains the following fields:

| Req. | Field name  | Description                                    | Data type |
|------|-------------|------------------------------------------------|-----------|
| Req  | event       | User event that preceded the offers            | string    |
| Req  | referenceId | Some ID for reference for deduplication.       | string    |
| Req  | userId      | Some ID for the user.                          | string    |
| Req  | email       | User email.                                    | string    |
| Req  | phone       | User phone number.                             | string    |
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

Here are some examples of building the minimum request object in different languages:

=== "Swift"
    ```swift
    var req = [String: Any]()
    req["event"] = "purchase"
    req["referenceId"] = "12345"
    req["userId"] = "userId1234"
    ```
=== "Objective C"
    ```objc
    NSMutableDictionary *req = [NSMutableDictionary new];
    req[@"event"] = @"purchase";
    req[@"referenceId"] = @"12345";
    req[@"userId"] = @"userId1234";
    ```

=== "Kotlin"
    ```kotlin
    val req = mutableMapOf<String, Any>()
    req["event"] = "purchase"
    req["referenceId"] = "12345"
    req["userId"] = "userId1234"
    ```

=== "Java"
    ```java
    Map<String, Object> req = new HashMap<>();
    req.put("event", "purchase");
    req.put("referenceId", "12345");
    req.put("userId", "userId1234");
    ```

### 2.2 Prepare the API body to POST

The post body includes important information for reporting segmentation and targeting. The body is a JSON object that contains the following fields:

| Field name  | Description                                    | Data type |
|-------------|------------------------------------------------|-----------|
| bId         | Bundle or package name                         | string    |
| aV          | App version                                    | string    |
| aId         | Some user ID for the app                       | string    |
| os          | Operating system (ios or android)              | string    |
| type        | Some string tag for your ad layout             | string    |
| platform    | Platform (phone, tablet, web, etc.)            | string    |
| ip          | The user IP, optional for client side calls    | string    |
| count       | The number of offers you want to display       | string    |
| request     | The request object built above in JSON         | object    |

Here are some examples of building the minimum request object in different languages:

=== "Swift"
    ```swift
    var body = [String: Any]()
    body["bId"] =  Bundle.main.bundleIdentifier ?? "default"
    body["aV"] = Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "unknown"
    body["aId"] = "userId1234"
    body["os"] = "ios"
    body["type"] = "interstitial"
    body["platform"] = "phone"
    body["request"] = req
    ```

=== "Objective C"
    ```objc
    NSMutableDictionary *body = [NSMutableDictionary new];
    body[@"bId"] = [[NSBundle mainBundle] bundleIdentifier] ?: @"default";
    body[@"aV"] = [[NSBundle mainBundle] infoDictionary][@"CFBundleShortVersionString"] ?: @"unknown";
    body[@"aId"] = @"userId1234";
    body[@"os"] = @"ios";
    body[@"type"] = @"interstitial";
    body[@"platform"] = @"phone";
    body[@"request"] = req;
    ```

=== "Kotlin"
    ```kotlin
    val body = mutableMapOf<String, Any>()
    body["bId"] = context.packageName
    body["aV"] = context.packageManager.getPackageInfo(context.packageName, 0).versionName
    body["aId"] = "userId1234"
    body["os"] = "android"
    body["platform"] = "phone"
    body["type"] = "interstitial"
    body["request"] = req
    ```

=== "Java"
    ```java
    JSONObject body = new JSONObject();
    body.put("bId", context.getPackageName());
    body.put("aV", context.getPackageManager().getPackageInfo(context.getPackageName(), 0).versionName);
    body.put("aId", "userId1234");
    body.put("os", "android");
    body.put("platform", "phone");
    body.put("type", "interstitial");
    body.put("request", req);
    ```
    
### 2.3 Make the API call and handle response

Once you have built the request object, you can make the API call to retrieve an offer. The API endpoint is `https://api.atargrowth.com/offers`. You need to pass the request object as the body of the request and the App Key as a header.

Here are some examples of making the API call in different languages:

=== "Swift"
    ```swift
    let url = URL(string: "https://api.atargrowth.com/offers")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("Bearer YOUR_APP_KEY", forHTTPHeaderField: "Authorization")
    request.httpBody = try? JSONSerialization.data(withJSONObject: body)
    
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        guard let dict = data as? [String: Any] else {
            print("Invalid response format")
            return
        }
        print("Offer dict: \(dict)")
        
        if let successString = dict["success"] as? String, successString == "false" {
            let errorMessage = dict["message"] as? String ?? "Unknown error"
            print("Error fetching offer: \(errorMessage)")
            return
        } else {
            if dict["offers"] != nil {
                let offer = dict["offers"] as? [String: Any] ?? [:]
            } else {
                return
            }
        }
    }
    task.resume()
    ```

=== "Objective C"
    ```objc
    NSURL *url = [NSURL URLWithString:@"https://api.atargrowth.com/offers"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"Bearer YOUR_APP_KEY"];
    NSData *postData = [NSJSONSerialization dataWithJSONObject:body options:0 error:nil];
    [request setHTTPBody:postData];

    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
        NSLog(@"Offer dict: %@", dict);
        
        if ([dict[@"success"] isEqualToString:@"false"]) {
            NSString *errorMessage = dict[@"message"] ?: @"Unknown error";
            NSLog(@"Error fetching offer: %@", errorMessage);
            return;
        } else {
            NSDictionary *offer = dict[@"offers"] ?: @{};
        }
    }];
    [task resume];
    ```

=== "Kotlin"
    ```kotlin
    Thread({
        try {
            val url = URL("https://api.atargrowth.com/offers")
            val connection = url.openConnection() as HttpURLConnection
            connection.requestMethod = "POST"
            connection.setRequestProperty("Content-Type", "application/json")
            connection.setRequestProperty("Authorization", "Bearer YOUR_APP_KEY")
            connection.doOutput = true
            val output = DataOutputStream(connection.outputStream)
            output.writeBytes(body.toString())
            output.flush()
            output.close()
            val responseCode = connection.responseCode
            val response = connection.inputStream.bufferedReader().use { it.readText() }

            val dict = JSONObject(response)
            println("Offer dict: $dict")

            if (dict.getString("success") == "false") {
                val errorMessage = dict.getString("message") ?: "Unknown error"
                println("Error fetching offer: $errorMessage")
            } else {
                val offers = dict.getJSONArray("offers")
                println("Offers: $offers")
            }
        } catch (Exception e) {
            e.printStackTrace()
            Log.e("AtarTestBed", "Error fetching offer: " + e.toString())
        }
    }).start()
    ```
    
=== "Java"
    ```java
    new Thread(() -> {
            try {
                URL url = new URL("https://api.atargrowth.com/offers");
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                connection.setRequestMethod("POST");
                connection.setRequestProperty("Content-Type", "application/json");
                connection.setRequestProperty("Authorization", "Bearer YOUR_APP_KEY");
                connection.setDoOutput(true);
                DataOutputStream output = new DataOutputStream(connection.getOutputStream());
                output.writeBytes(body.toString());
                output.flush();
                output.close();

                int responseCode = connection.getResponseCode();
                BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
                String line;
                StringBuffer response = new StringBuffer();
                while ((line = reader.readLine()) != null) {
                    response.append(line);
                }
                reader.close();

                JSONObject dict = new JSONObject(response.toString());
                Log.i("AtarTestBed", "Offer dict: " + dict);
                
               if (resp.getString("success").equals("false")) {
                    String errorMessage = resp.getString("message") != null ? resp.getString("message") : "Unknown error";
                    Log.e("AtarTestBed","Error fetching offer: " + errorMessage);
                } else {
                    JSONArray offers = resp.getJSONArray("offers");
                    for (int i = 0; i < offers.length(); i++) {
                        JSONObject offer = offers.getJSONObject(i);
                        Log.i("AtarTestBed", "Offer: " + offer.toString());
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
                Log.e("AtarTestBed", "Error fetching offer: " + e.toString());
            }
    }).start();
    ```

## Step 3: Show the offer to the user

The contents of the Offer object returned from the API call can be used to show the offer to the user. The offer object contains the following fields:

| Field          | Type    | Description                          |
|----------------|---------|--------------------------------------|
| id             | String  | Unique identifier for the offer      |
| title          | String  | Title of the offer                   |
| description    | String  | Description of the offer             |
| cta            | String  | Call to action button text           |
| iconUrl        | String  | URL to the offer's icon image        |
| bannerUrl      | String  | URL to the offer's banner image     |
| clickUrl       | String  | URL to handle click actions          |
| destinationUrl | String  | URL where the user is redirected     |

You can now construct the offer in the appropriate native format that makes sense for your app. For example, you can show a notification, a message, or a dialog with the offer details. Make sure to route the user to the clickUrl when the user interacts with the offer.


## Need help?

If you have any questions or need help, please contact us at [support@atargrowth.com](mailto:support@atargrowth.com).
