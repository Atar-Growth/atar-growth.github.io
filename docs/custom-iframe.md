---
hide:
  - footer
---
# Atar Webview / iFrame Integration

The guide below will walk you through the steps to integrate Atar into your app via an iFrame or Webview inside your Android or iOS app.

## Step 1: Create an account and get your App Key

If you haven't already, create an account on the [Atar Dashboard](https://app.atargrowth.com/). Once you've created an account, you'll be able to create an app and get your App Key from the [settings page](https://app.atargrowth.com/settings).

## Step 2: Create the URL for the offer Webview or iFrame

Offers can be retrieved from Atar by constructing a personalized URL to the user, and loading the URL in your app. The GET request on the URL will return an HTML page that can be rendered and presented directly.

The URL structure is as follows:
```html
https://api.atargrowth.com/offers
    ?   aK=<your app key>

# App metadata
    &   bId=<the bundle ID or package name>
    &   aV=<the app version>
    &   os=<ios or android>
    &   platform=<phone, tablet, web, etc.>
    &   type=<interstitial, banner, etc.>

# User and event specific parameters
    &   userId=<the user ID>
    &   referenceId=<an event reference ID>
    &   event=<the user event>
```

Here are some examples of constructing the URL in different languages:

=== "iOS Swift"
    ```swift
    guard var components = URLComponents(string: "https://api.atargrowth.com/offers") else { return nil }
    components.queryItems = [
        URLQueryItem(name: "aK", value: "your app key"),
        URLQueryItem(name: "bId", value: Bundle.main.bundleIdentifier ?? ""),
        URLQueryItem(name: "aV", value: Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? ""),
        URLQueryItem(name: "os", value: "ios"),
        URLQueryItem(name: "platform", value: UIDevice.current.model),
        URLQueryItem(name: "type", value: "interstitial"),
        URLQueryItem(name: "userId", value: "userId1234"),
        URLQueryItem(name: "referenceId", value: "12345"),
        URLQueryItem(name: "event", value: "purchase")
    ]
    let url = components.url
    print(url)
    ```

=== "iOS Objective C"
    ```objc
    NSURLComponents *components = [NSURLComponents componentsWithString:@"https://api.atargrowth.com/offers"];
    components.queryItems = @[
        [NSURLQueryItem queryItemWithName:@"aK" value:@"your app key"],
        [NSURLQueryItem queryItemWithName:@"bId" value:[[NSBundle mainBundle] bundleIdentifier] ?: @""],
        [NSURLQueryItem queryItemWithName:@"aV" value:[[NSBundle mainBundle] infoDictionary][@"CFBundleShortVersionString"] ?: @""],
        [NSURLQueryItem queryItemWithName:@"os" value:@"ios"],
        [NSURLQueryItem queryItemWithName:@"platform" value:[UIDevice currentDevice].model],
        [NSURLQueryItem queryItemWithName:@"type" value:@"interstitial"],
        [NSURLQueryItem queryItemWithName:@"userId" value:@"userId1234"],
        [NSURLQueryItem queryItemWithName:@"referenceId" value:@"12345"],
        [NSURLQueryItem queryItemWithName:@"event" value:@"purchase"]
    ];
    NSURL *url = components.URL;
    NSLog(@"URL: %@", url);
    ```

=== "Android Kotlin"
    ```kotlin
    val components = Uri.parse("https://api.atargrowth.com/offers").buildUpon()
        .appendQueryParameter("aK", "your app key")
        .appendQueryParameter("bId", context.packageName)
        .appendQueryParameter("aV", context.packageManager.getPackageInfo(context.packageName, 0).versionName)
        .appendQueryParameter("os", "android")
        .appendQueryParameter("platform", "phone")
        .appendQueryParameter("type", "interstitial")
        .appendQueryParameter("userId", "userId1234")
        .appendQueryParameter("referenceId", "12345")
        .appendQueryParameter("event", "purchase")
        .build()
    println(components.toString())
    ```

=== "Android Java"
    ```java
    Uri.Builder builder = Uri.parse("https://api.atargrowth.com/offers").buildUpon()
        .appendQueryParameter("aK", "your
        .appendQueryParameter("bId", context.getPackageName())
        .appendQueryParameter("aV", context.getPackageManager().getPackageInfo(context.getPackageName(), 0).versionName)
        .appendQueryParameter("os", "android")
        .appendQueryParameter("platform", "phone")
        .appendQueryParameter("type", "interstitial")
        .appendQueryParameter("userId", "userId1234")
        .appendQueryParameter("referenceId", "12345")
        .appendQueryParameter("event", "purchase");
    String url = builder.build().toString();
    System.out.println(url);
    ```

### Full parameter list with explanation

The more information that you provide, the more relevant we can tailor the offers to your users. Here is a full list of parameters that you can include in the URL:

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

## Step 3: Handle click or cancel callbacks from the webview

When the user interacts with the offer, the webview will send a message to the app. You can handle these messages by listening for specific callback events in the webview.

### iOS Callbacks

The way it works is:
1. You register your webview with a script message handler of a specific label
2. You implement the delegate method to handle the message

#### Step 3.a. Declare your webview delegate

In this example, we're using the `WKScriptMessageHandler` protocol to handle the messages from the webview, and the webview is initialized inside the `WebViewController`.

=== "Swift"
    ```swift
    class WebViewController: UIViewController, WKScriptMessageHandler {
    ```

=== "Objective C"
    ```objc
    @interface WebViewController : UIViewController <WKScriptMessageHandler>
    ```

#### Step 3.b. Register the script message handler

The script message handler is registered with a specific label that the webview will use to send messages to the app. We use a poorly named label `cb` in our JS.

=== "Swift"
    ```swift
    webView.configuration.userContentController.add(self, name: "cb")
    ```

=== "Objective C"
    ```objc
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"cb"];
    ```

#### Step 3.c. Implement the delegate method

The delegate method `userContentController(_:didReceive:)` is called when the webview sends a message to the app. This example will show you how to load the URL in the main browser once received.

=== "Swift"
    ```swift
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        if message.name == "cb" {
            if let jsonString = message.body as? String,
               let jsonData = jsonString.data(using: .utf8) {
                do {
                    let clickObj = try JSONSerialization.jsonObject(with: jsonData) as? [String: String]
                    if let clickObj = clickObj {
                        UIApplication.shared.open(URL(string: clickObj["clickUrl"]!)!, options: [:], completionHandler: nil)
                    }
                } catch {
                    // This catch will handle when offers are not present, or any other errors
                    cancelAndDismiss();
                }
            }
        }
        // If reached this point, dismiss the webview
        dismiss()
    }
    ```

=== "Objective C"
    ```objc
    - (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
        if ([message.name isEqualToString:@"cb"]) {
            // Attempt to parse the JSON string into a dictionary
            NSError *error;
            NSDictionary *clickObj = [NSJSONSerialization JSONObjectWithData:[message.body dataUsingEncoding:NSUTF8StringEncoding] options:0 error:&error];
            if (error) {
                // This catch will handle when offers are not present, or any other errors
                [self cancelAndDismiss];
            } else {
                [[UIApplication sharedApplication] openURL:[NSURL URLWithString:clickObj[@"clickUrl"]] options:@{} completionHandler:nil];
            }
        }
        // If reached this point, dismiss the webview
        [self dismiss];
    }
    ```

### Android Callbacks

Similar to iOS, the way Android works is to register the Javascript handler on your webview with a specific label, and then handle the message in the app.

#### Step 3.a. Register the Javascript interface and implement handling

In this example, we're using the `addJavascriptInterface` method to register the Javascript interface with the webview. The Javascript interface is a class that has methods that can be called from the webview.

=== "Java"
    ```java
    webView.getSettings().setJavaScriptEnabled(true);
    webView.addJavascriptInterface(new Object() {
        @JavascriptInterface
        public void postMessage(String data) {
            JSONObject messageData;
            try {
                messageData = new JSONObject(data);
            } catch (Exception e) {
                // This should never happen, but just in case, handle parsing errors here
                cancelAndDismiss();
                return;
            }

            String clickUrl = messageData.optString("clickUrl");

            if (clickUrl != null && !clickUrl.isEmpty()) {
                try {
                    Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse(clickUrl));
                    browserIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                    getContext().startActivity(browserIntent);
                } catch (Exception e) {
                    // This will trigger in case there are no offers available for whatever reason
                    cancelAndDismiss();
                    return;
                }
            }

            // Dismiss the webview
            dismiss()
        }
    }, "AtarInterface");
    ```

=== "Kotlin"
    ```kotlin
    webView.settings.javaScriptEnabled = true
    webView.addJavascriptInterface(object {
        @JavascriptInterface
        fun postMessage(data: String) {
            val messageData = try {
                JSONObject(data)
            } catch (e: Exception) {
                // This should never happen, but just in case, handle parsing errors here
                cancelAndDismiss()
                return
            }

            val clickUrl = messageData.optString("clickUrl")

            if (clickUrl != null && clickUrl.isNotEmpty()) {
                try {
                    val browserIntent = Intent(Intent.ACTION_VIEW, Uri.parse(clickUrl))
                    browserIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
                    context.startActivity(browserIntent)
                } catch (e: Exception) {
                    // This will trigger in case there are no offers available for whatever reason
                    cancelAndDismiss()
                    return
                }
            }

            // Dismiss the webview
            dismiss()
        }
    }, "AtarInterface")
    ```

## Step 4: Implement your custom cancel button

In the event that the user wants to hide the interstitial, you should provide a way for them to do so. This can be a simple button that dismisses the webview. Here's an example of how you can implement a cancel button in your app.

### iOS Cancel Button

In this example, we use a UIButton with an X label that will dismiss the webview when clicked.

=== "Swift"
    ```swift
    let cancelButton = UIButton()
    cancelButton.translatesAutoresizingMaskIntoConstraints = false
    cancelButton.setTitle("x", for: .normal)
    cancelButton.titleLabel?.font = UIFont.monospacedSystemFont(ofSize: 20, weight: .regular)
    cancelButton.setTitleColor(.gray, for: .normal)
    cancelButton.addTarget(self, action: #selector(cancelAndDismiss), for: .touchUpInside)
    
    // Make sure to add this last, so it appears on top
    addSubview(cancelButton)

    cancelButton.topAnchor.constraint(equalTo: yourContentView.topAnchor, constant: 8),
    cancelButton.trailingAnchor.constraint(equalTo: yourContentView.trailingAnchor, constant: -8),
    cancelButton.widthAnchor.constraint(equalToConstant: 30),
    cancelButton.heightAnchor.constraint(equalToConstant: 30)
    ```

=== "Objective C"
    ```objc
    UIButton *cancelButton = [UIButton buttonWithType:UIButtonTypeCustom];
    cancelButton.translatesAutoresizingMaskIntoConstraints = NO;
    [cancelButton setTitle:@"x" forState:UIControlStateNormal];
    cancelButton.titleLabel.font = [UIFont monospacedSystemFontOfSize:20 weight:UIFontWeightRegular];
    [cancelButton setTitleColor:UIColor.grayColor forState:UIControlStateNormal];
    [cancelButton addTarget:self action:@selector(cancelAndDismiss) forControlEvents:UIControlEventTouchUpInside];

    // Make sure to add this last, so it appears on top
    [self.view addSubview:cancelButton];

    [cancelButton.topAnchor constraintEqualToAnchor:yourContentView.topAnchor constant:8].active = YES;
    [cancelButton.trailingAnchor constraintEqualToAnchor:yourContentView.trailingAnchor constant:-8].active = YES;
    [cancelButton.widthAnchor constraintEqualToConstant:30].active = YES;
    [cancelButton.heightAnchor constraintEqualToConstant:30].active = YES;
    ```

#### Android Cancel Button

In this example, we use a Button with an X label that will dismiss the webview when clicked.

=== "Java"
    ```java
    Button closeButton = new Button(context);
    closeButton.setText("X");
    closeButton.setTextColor(Color.BLACK);
    closeButton.setBackground(null);
    int buttonSize = (int) (25 * getResources().getDisplayMetrics().density);
    LayoutParams closeButtonParams = new LayoutParams(buttonSize, buttonSize);
    closeButtonParams.gravity = Gravity.TOP | Gravity.END;
    closeButton.setLayoutParams(closeButtonParams);
    closeButton.setPadding(10, 10, 10, 10); // Padding in pixels
    closeButton.setOnClickListener(v -> cancelAndDismiss());

    yourContentView.addView(closeButton);
    ```

=== "Kotlin"
    ```kotlin
    val closeButton = Button(context)
    closeButton.text = "X"
    closeButton.setTextColor(Color.BLACK)
    closeButton.background = null
    val buttonSize = (25 * resources.displayMetrics.density).toInt()
    val closeButtonParams = LayoutParams(buttonSize, buttonSize)
    closeButtonParams.gravity = Gravity.TOP or Gravity.END
    closeButton.layoutParams = closeButtonParams
    closeButton.setPadding(10, 10, 10, 10) // Padding in pixels
    closeButton.setOnClickListener { cancelAndDismiss() }
    yourContentView.addView(closeButton)
    ```
    
## Appendix: Full example webview interstitial presentation

Below is a full example of how to create a popup interstital in iOS and Android. At a high level, this implementation will create a an Interstital view class that will insert the webview frame on whatever window or Activity is currently present at the top. The webview has some rounded corners and a close button that will dismiss the webview.

The webview sits on top of the current view, and is styled as follows:

- Width: 90% of the screen width
- Height: 80% of the screen height
- Has an activity indicator in the center of the screen while loading the URL
- Gray, translucent background surrounding the offer itself
- Animates in and out from the top of the screen

### iOS Interstitial View Example

#### Using the below iOS interstitial

To use the below iOS interstitial, you can create an instance of the `InterstitialView` class, configure it with the URL you want to load, and then present it on the screen. As mentioned in the code, we recommend that you separate the initialization of the webview from the actual loading of the URL and presentation. This is because iOS has some multi-second initialization latency with the webview sometimes.

=== "Swift"
    ```swift
    let interstitialView = InterstitialView()
    interstitialView.configure(withUrl: url)
    interstitialView.show()
    ```

=== "Objective C"
    ```objc
    InterstitialView *interstitialView = [[InterstitialView alloc] initWithFrame:CGRectZero];
    [interstitialView configureWithUrl:url];
    [interstitialView show];
    ```

#### Webview interstitial view in iOS

=== "Swift"
    ```swift
    class InterstitialView: UIView, WKNavigationDelegate, WKScriptMessageHandler {
        private let contentView = UIView()
        private let webView: WKWebView
        private let activityIndicator = UIActivityIndicatorView(style: .large)
        private let closeButton = UIButton()
        private var lastOfferRequest: OfferRequest?

        // WKWebView has multi-second initialization issues sometimes, 
        // so we separate the initialization of the webview from the actual 
        // loading of the URL and presentation.
        override init(frame: CGRect) {
            let config = WKWebViewConfiguration()
            self.webView = WKWebView(frame: .zero, configuration: config)
            self.webView.loadHTMLString("", baseURL: nil)
            
            super.init(frame: frame)

            let contentController = self.webView.configuration.userContentController
            contentController.add(self, name: "cb")

            setupViews()
        }
        
        required init?(coder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
        
        private func setupViews() {
            backgroundColor = UIColor.black.withAlphaComponent(0.6)
            
            // Content view configuration
            contentView.backgroundColor = .white
            contentView.layer.cornerRadius = 12.0
            contentView.clipsToBounds = true
            contentView.translatesAutoresizingMaskIntoConstraints = false
            addSubview(contentView)

            webView.translatesAutoresizingMaskIntoConstraints = false
            webView.scrollView.contentInset = UIEdgeInsets.zero
            webView.scrollView.bounces = false
            webView.navigationDelegate = self
            contentView.addSubview(webView)

            // Activity Indicator configuration
            activityIndicator.translatesAutoresizingMaskIntoConstraints = false
            contentView.addSubview(activityIndicator)
            activityIndicator.startAnimating()
            
            closeButton.translatesAutoresizingMaskIntoConstraints = false
            closeButton.setTitle("x", for: .normal)
            closeButton.titleLabel?.font = UIFont.monospacedSystemFont(ofSize: 20, weight: .regular)
            closeButton.setTitleColor(.gray, for: .normal)
            closeButton.addTarget(self, action: #selector(cancelAndDismiss), for: .touchUpInside)
            addSubview(closeButton)

            setupConstraints()
        }
        
        private func setupConstraints() {
            NSLayoutConstraint.activate([
                contentView.centerYAnchor.constraint(equalTo: centerYAnchor),
                contentView.centerXAnchor.constraint(equalTo: centerXAnchor),
                contentView.widthAnchor.constraint(equalTo: widthAnchor, multiplier: 0.9),
                contentView.heightAnchor.constraint(equalTo: heightAnchor, multiplier:  0.8),
                
                webView.topAnchor.constraint(equalTo: contentView.topAnchor),
                webView.leftAnchor.constraint(equalTo: contentView.leftAnchor),
                webView.rightAnchor.constraint(equalTo: contentView.rightAnchor),
                webView.bottomAnchor.constraint(equalTo: contentView.bottomAnchor),

                activityIndicator.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
                activityIndicator.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
                
                closeButton.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 8),
                closeButton.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -8),
                closeButton.widthAnchor.constraint(equalToConstant: 30),
                closeButton.heightAnchor.constraint(equalToConstant: 30)
            ])
        }

        // This function will configure and load the URL in the webview
        func configure(withUrl url: URL) {
            let webRequest = URLRequest(url: url!)
            webView.load(webRequest)
        }

        // This function will present the webview on the screen
        func show() {
            guard let window = UIApplication.shared.windows.filter({ $0.isKeyWindow }).first else { return }

            self.alpha = 0.6
            self.frame = window.bounds
            window.addSubview(self)

            UIView.animate(withDuration: 0.3) {
                self.alpha = 1
            }
        }

        @objc func cancelAndDismiss() {
            // Any custom logic you want to run when the user cancels
            dismiss()
        }
        
        func dismiss() {
            UIView.animate(withDuration: 0.3) {
                self.alpha = 0
            } completion: { _ in
                self.removeFromSuperview()
            }
        }
        
        func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
            activityIndicator.stopAnimating()
        }
        
        func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
            if message.name == "cb" {
                if let jsonString = message.body as? String,
                let jsonData = jsonString.data(using: .utf8) {
                    do {
                        let clickObj = try JSONSerialization.jsonObject(with: jsonData) as? [String: String]
                        if let clickObj = clickObj {
                            UIApplication.shared.open(URL(string: clickObj["clickUrl"]!)!, options: [:], completionHandler: nil)
                        }
                    } catch {
                        // This catch will handle when offers are not present, or any other errors
                        cancelAndDismiss();
                    }
                }
            }
            dismiss()
        }
    }
    ```
=== "Objective C"
    ```objc
    @interface InterstitialView : UIView <WKNavigationDelegate, WKScriptMessageHandler>
    @property (nonatomic, strong) UIView *contentView;
    @property (nonatomic, strong) WKWebView *webView;
    @property (nonatomic, strong) UIActivityIndicatorView *activityIndicator;
    @property (nonatomic, strong) UIButton *closeButton;
    @property (nonatomic, strong) OfferRequest *lastOfferRequest;
    @end

    @implementation InterstitialView

    - (instancetype)initWithFrame:(CGRect)frame {
        WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
        self.webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:config];
        [self.webView loadHTMLString:@"" baseURL:nil];
        
        self = [super initWithFrame:frame];
        if (self) {
            WKUserContentController *contentController = self.webView.configuration.userContentController;
            [contentController addScriptMessageHandler:self name:@"cb"];
            
            [self setupViews];
        }
        return self;
    }

    - (void)setupViews {
        self.backgroundColor = [[UIColor blackColor] colorWithAlphaComponent:0.6];
        
        // Content view configuration
        self.contentView = [[UIView alloc] init];
        self.contentView.backgroundColor = [UIColor whiteColor];
        self.contentView.layer.cornerRadius = 12.0;
        self.contentView.clipsToBounds = YES;
        self.contentView.translatesAutoresizingMaskIntoConstraints = NO;
        [self addSubview:self.contentView];
        
        self.webView.translatesAutoresizingMaskIntoConstraints = NO;
        self.webView.scrollView.contentInset = UIEdgeInsetsZero;
        self.webView.scrollView.bounces = NO;
        self.webView.navigationDelegate = self;
        [self.contentView addSubview:self.webView];
        
        // Activity Indicator configuration
        self.activityIndicator = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleLarge];
        self.activityIndicator.translatesAutoresizingMaskIntoConstraints = NO;
        [self.contentView addSubview:self.activityIndicator];
        [self.activityIndicator startAnimating];
        
        self.closeButton = [UIButton buttonWithType:UIButtonTypeCustom];
        self.closeButton.translatesAutoresizingMaskIntoConstraints = NO;
        [self.closeButton setTitle:@"x" forState:UIControlStateNormal];
        self.closeButton.titleLabel.font = [UIFont monospacedSystemFontOfSize:20 weight:UIFontWeightRegular];
        [self.closeButton setTitleColor:UIColor.grayColor forState:UIControlStateNormal];
        [self.closeButton addTarget:self action:@selector(cancelAndDismiss) forControlEvents:UIControlEventTouchUpInside];
        [self addSubview:self.closeButton];
        
        [self setupConstraints];
    }

    - (void)setupConstraints {
        [NSLayoutConstraint activateConstraints:@[
            [self.contentView.centerYAnchor constraintEqualToAnchor:self.centerYAnchor],
            [self.contentView.centerXAnchor constraintEqualToAnchor:self.centerXAnchor],
            [self.contentView.widthAnchor constraintEqualToAnchor:self.widthAnchor multiplier:0.9],
            [self.contentView.heightAnchor constraintEqualToAnchor:self.heightAnchor multiplier:0.8],
            
            [self.webView.topAnchor constraintEqualToAnchor:self.contentView.topAnchor],
            [self.webView.leftAnchor constraintEqualToAnchor:self.contentView.leftAnchor],
            [self.webView.rightAnchor constraintEqualToAnchor:self.contentView.rightAnchor],
            [self.webView.bottomAnchor constraintEqualToAnchor:self.contentView.bottomAnchor],
            
            [self.activityIndicator.centerXAnchor constraintEqualToAnchor:self.contentView.centerXAnchor],
            [self.activityIndicator.centerYAnchor constraintEqualToAnchor:self.contentView.centerYAnchor],
            
            [self.closeButton.topAnchor constraintEqualToAnchor:self.contentView.topAnchor constant:8],
            [self.closeButton.trailingAnchor constraintEqualToAnchor:self.contentView.trailingAnchor constant:-8],
            [self.closeButton.widthAnchor constraintEqualToConstant:30],
            [self.closeButton.heightAnchor constraintEqualToConstant:30]
        ]];
    }

    - (void)configureWithUrl:(NSURL *)url {
        NSURLRequest *webRequest = [NSURLRequest requestWithURL:url];
        [self.webView loadRequest:webRequest];
    }

    - (void)show {
        UIWindow *window = [[UIApplication sharedApplication].windows filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"isKeyWindow == YES"]].firstObject;
        self.alpha = 0.6;
        self.frame = window.bounds;
        [window addSubview:self];
        
        [UIView animateWithDuration:0.3 animations:^{
            self.alpha = 1;
        }];
    }

    - (void)cancelAndDismiss {
        // Any custom logic you want to run when the user cancels
        [self dismiss];
    }

    - (void)dismiss {
        [UIView animateWithDuration:0.3 animations:^{
            self.alpha = 0;
        } completion:^(BOOL finished) {
            [self removeFromSuperview];
        }];
    }

    - (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
        [self.activityIndicator stopAnimating];
    }
    
    - (void)userContentController:(WKUserContentController *)userContentController didReceiveMessage:(WKScriptMessage *)message {
        if ([message.name isEqualToString:@"cb"]) {
            // Attempt to parse the JSON string into a dictionary
            NSError *error;
            NSDictionary *clickObj = [NSJSONSerialization JSONObjectWithData:[message.body dataUsingEncoding:NSUTF8StringEncoding] options:0 error:&error];
            if (error) {
                // This catch will handle when offers are not present, or any other errors
                [self cancelAndDismiss];
            } else {
                [[UIApplication sharedApplication] openURL:[NSURL URLWithString:clickObj[@"clickUrl"]] options:@{} completionHandler:nil];
            }
        }
        // If reached this point, dismiss the webview
        [self dismiss];
    }

    @end
    ```

### Android Interstitial View Example

#### Using the below Android interstitial

To use the below Android interstitial, you can create an instance of the `InterstitialView` class, configure it with the URL you want to load, and then present it on the screen.

=== "Java"
    ```java
    InterstitialView interstitialView = new InterstitialView(context);
    interstitialView.loadUrl(url);
    interstitialView.show(activity);
    ```

=== "Kotlin"
    ```kotlin
    val interstitialView = InterstitialView(context)
    interstitialView.loadUrl(url)
    interstitialView.show(activity)
    ```

#### Webview interstitial view in Android

=== "Java"
    ```java
    import android.app.Activity;
    import android.content.Context;
    import android.content.Intent;
    import android.graphics.drawable.ShapeDrawable;
    import android.graphics.drawable.shapes.RoundRectShape;
    import android.net.Uri;
    import android.view.ViewOutlineProvider;
    import android.webkit.JavascriptInterface;
    import android.webkit.WebView;
    import android.webkit.WebViewClient;
    import android.widget.FrameLayout;
    import android.widget.ProgressBar;
    import android.widget.Button;
    import android.view.ViewGroup;
    import android.view.Gravity;
    import android.graphics.Color;

    import org.json.JSONObject;

    public class InterstitialView extends FrameLayout {
        private FrameLayout contentView;
        private WebView webView;
        private ProgressBar activityIndicator;
        private Button closeButton;

        private Activity contextActivity;

        public InterstitialView(Context context) {
            super(context);
            initializeViews(context);
            setupWebView();
        }

        private void initializeViews(Context context) {
            setBackgroundColor(Color.parseColor("#99000000")); // Semi-transparent
            setLayoutParams(new FrameLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));

            // Content view
            contentView = new FrameLayout(context);
            contentView.setBackgroundColor(Color.TRANSPARENT);
            contentView.setElevation(10f);
            int contentViewWidth = (int) (getResources().getDisplayMetrics().widthPixels * 0.9);
            int contentViewHeight = (int) (getResources().getDisplayMetrics().heightPixels * 0.8);
            LayoutParams contentViewParams = new LayoutParams(contentViewWidth, contentViewHeight);
            contentViewParams.gravity = Gravity.CENTER;
            contentView.setLayoutParams(contentViewParams);
            addView(contentView);

            // Setting rounded corners for the contentView
            contentView.setClipToOutline(true);
            contentView.setOutlineProvider(ViewOutlineProvider.BACKGROUND);
            float[] radii = new float[]{25, 25, 25, 25, 25, 25, 25, 25}; // radii for each corner in dp
            ShapeDrawable background = new ShapeDrawable(new RoundRectShape(radii, null, null));
            background.getPaint().setColor(Color.WHITE);
            contentView.setBackground(background);

            // WebView setup
            webView = new WebView(context);
            LayoutParams webViewParams = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT, Gravity.CENTER);
            webView.setLayoutParams(webViewParams);
            contentView.addView(webView);

            // Activity Indicator
            activityIndicator = new ProgressBar(context);
            activityIndicator.setLayoutParams(new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT, Gravity.CENTER));
            contentView.addView(activityIndicator);

            // Close button
            closeButton = new Button(context);
            closeButton.setText("X");
            closeButton.setTextColor(Color.BLACK);
            closeButton.setBackground(null); // Removing default background to make it less obtrusive
            int buttonSize = (int) (25 * getResources().getDisplayMetrics().density); // 25dp to pixels
            LayoutParams closeButtonParams = new LayoutParams(buttonSize, buttonSize);
            closeButtonParams.gravity = Gravity.TOP | Gravity.END;
            closeButton.setLayoutParams(closeButtonParams);
            closeButton.setPadding(10, 10, 10, 10); // Padding in pixels
            closeButton.setOnClickListener(v -> cancelAndDismiss());
            contentView.addView(closeButton); // Adding the button to contentView so it's inside the WebView
        }

        private void setupWebView() {
            webView.setWebViewClient(new WebViewClient() {
                @Override
                public void onPageFinished(WebView view, String url) {
                    activityIndicator.setVisibility(GONE);
                }
            });

            // Enable JavaScript
            webView.getSettings().setJavaScriptEnabled(true);
            webView.addJavascriptInterface(new Object() {
                @JavascriptInterface
                public void postMessage(String data) {
                    JSONObject messageData;
                    try {
                        messageData = new JSONObject(data);
                    } catch (Exception e) {
                        // This should never happen, but just in case, handle parsing errors here
                        cancelAndDismiss();
                        return;
                    }

                    String clickUrl = messageData.optString("clickUrl");

                    if (clickUrl != null && !clickUrl.isEmpty()) {
                        try {
                            Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse(clickUrl));
                            browserIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                            getContext().startActivity(browserIntent);
                        } catch (Exception e) {
                            // This will trigger in case there are no offers available for whatever reason
                            cancelAndDismiss();
                            return;
                        }
                    }

                    // Dismiss the webview
                    dismiss();
                }
            }, "AtarInterface");
        }

        public void loadUrl(String url) {
            Logger.getInstance().i("InterstitialView: Loading URL: " + url);
            webView.loadUrl(url);
        }

        public void show(Activity activity) {
            ViewGroup decorView = (ViewGroup) activity.getWindow().getDecorView();
            decorView.addView(this);
            setAlpha(0f);
            animate().alpha(1f).setDuration(300).start();
            contextActivity = activity;
        }

        public void cancelAndDismiss() {
            // Implement any custom logic for when the user cancels
            dismiss();
        }

        public void dismiss() {
            if (contextActivity == null) {
                return;
            }
            contextActivity.runOnUiThread(() -> animate().alpha(0f).setDuration(300).withEndAction(() -> ((ViewGroup) getParent()).removeView(this)).start());
        }
    }
    ```

=== "Kotlin"
    ```kotlin
    import android.app.Activity
    import android.content.Context
    import android.content.Intent
    import android.graphics.Color
    import android.graphics.Outline
    import android.graphics.Rect
    import android.os.Build
    import android.view.View
    import android.view.ViewGroup
    import android.view.ViewOutlineProvider
    import android.webkit.JavascriptInterface
    import android.webkit.WebView
    import android.webkit.WebViewClient
    import android.widget.Button
    import android.widget.FrameLayout
    import android.widget.ProgressBar
    import androidx.annotation.RequiresApi

    import org.json.JSONObject

    class InterstitialView(context: Context) : FrameLayout(context) {
        private val contentView: FrameLayout
        private val webView: WebView
        private val activityIndicator: ProgressBar
        private val closeButton: Button

        private var contextActivity: Activity? = null

        init {
            setBackgroundColor(Color.parseColor("#99000000")) // Semi-transparent
            layoutParams = LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT)

            // Content view
            contentView = FrameLayout(context)
            contentView.setBackgroundColor(Color.TRANSPARENT)
            contentView.elevation = 10f
            val contentViewWidth = (resources.displayMetrics.widthPixels * 0.9).toInt()
            val contentViewHeight = (resources.displayMetrics.heightPixels * 0.8).toInt()
            val contentViewParams = LayoutParams(contentViewWidth, contentViewHeight)
            contentViewParams.gravity = Gravity.CENTER
            contentView.layoutParams = contentViewParams
            addView(contentView)

            // Setting rounded corners for the contentView
            contentView.clipToOutline = true
            contentView.outlineProvider = object : ViewOutlineProvider() {
                @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
                override fun getOutline(view: View?, outline: Outline?) {
                    val radii = floatArrayOf(25f, 25f, 25f, 25f, 25f, 25f, 25f, 25f) // radii for each corner in dp
                    outline?.setRoundRect(Rect(0, 0, view?.width ?: 0, view?.height ?: 0), radii)
                }
            }

            // WebView setup
            webView = WebView(context)
            val webViewParams = LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT, Gravity.CENTER)
            webView.layoutParams = webViewParams
            contentView.addView(webView)

            // Activity Indicator
            activityIndicator = ProgressBar(context)
            activityIndicator.layoutParams = LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT, Gravity.CENTER)
            contentView.addView(activityIndicator)

            // Close button
            closeButton = Button(context)
            closeButton.text = "X"
            closeButton.setTextColor(Color.BLACK)
            closeButton.background = null // Removing default background to make it less obtrusive
            val buttonSize = (25 * resources.displayMetrics.density).toInt() // 25dp to pixels
            val closeButtonParams = LayoutParams(buttonSize, buttonSize)
            closeButtonParams.gravity = Gravity.TOP or Gravity.END
            closeButton.layoutParams = closeButtonParams
            closeButton.setPadding(10, 10, 10, 10) // Padding in pixels
            closeButton.setOnClickListener { cancelAndDismiss() }
            contentView.addView(closeButton) // Adding the button to contentView so it's inside the WebView
        }

        private fun setupWebView() {
            webView.webViewClient = object : WebViewClient() {
                override fun onPageFinished(view: WebView?, url: String?) {
                    activityIndicator.visibility = GONE
                }
            }

            // Enable JavaScript
            webView.settings.javaScriptEnabled = true
            webView.addJavascriptInterface(object {
                @JavascriptInterface
                fun postMessage(data: String) {
                    val messageData = try {
                        JSONObject(data)
                    } catch (e: Exception) {
                        // This should never happen, but just in case, handle parsing errors here
                        cancelAndDismiss()
                        return
                    }

                    val clickUrl = messageData.optString("clickUrl")

                    if (clickUrl != null && clickUrl.isNotEmpty()) {
                        try {
                            val browserIntent = Intent(Intent.ACTION_VIEW, Uri.parse(clickUrl))
                            browserIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
                            context.startActivity(browserIntent)
                        } catch (e: Exception) {
                            // This will trigger in case there are no offers available for whatever reason
                            cancelAndDismiss()
                            return
                        }
                    }

                    // Dismiss the webview
                    dismiss()
                }
            }, "AtarInterface")
        }

        public fun loadUrl(url: String) {
            Logger.getInstance().i("InterstitialView: Loading URL: " + url)
            webView.loadUrl(url)
        }

        public fun show(activity: Activity) {
            val decorView = activity.window.decorView as ViewGroup
            decorView.addView(this)
            setAlpha(0f)
            animate().alpha(1f).setDuration(300).start()
            contextActivity = activity
        }

        public fun cancelAndDismiss() {
            // Implement any custom logic for when the user cancels
            dismiss()
        }

        public fun dismiss() {
            contextActivity?.runOnUiThread {
                animate().alpha(0f).setDuration(300).withEndAction { (parent as ViewGroup).removeView(this) }.start()
            }
        }
    }
    ```

## Need help?

If you have any questions or need help, please contact us at [support@atargrowth.com](mailto:support@atargrowth.com).
