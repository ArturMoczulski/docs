## Installation

To send errors to Rollbar from your Java application, you should use our <a href="https://github.com/rollbar/rollbar-java" target="_blank" rel="noopener">rollbar-java</a> package. 

Add the following to your `build.gradle`:

``` java
dependencies {
    compile('com.rollbar:rollbar-java:1.2.1')
    compile('com.rollbar:rollbar-android:1.2.1@aar')
    //Rest of your dependencies...
}
```

For an Android app, we have some more specific pieces which allow you to capture more information about the Android environment automatically than what you would have to do with `rollbar-java` directly.

Set your access token in your `AndroidManifest.xml` file:

``` java
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
    <application ...>
        ...
        <meta-data android:name="com.rollbar.android.ACCESS_TOKEN"
        android:value="{{ client_access_token }}" />
    </application>
</manifest>
```

Then initialize Rollbar in your `MainActivity.java`:

``` java
import com.rollbar.android.Rollbar

public class MainActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Rollbar.init(this);
    Rollbar.instance().error(new Exception("This is a test error")); //remove this after initial testing
  }
}
```

You can make direct calls to Rollbar via the managed instance:

``` java
void clickAction() {
  Rollbar.instance().log("Some button was clicked");
}
```

All uncaught exceptions which cause a crash will be logged by Rollbar, but will not be sent until the next time the app runs.

You can see a rollbar-android example <a href="https://github.com/rollbar/rollbar-java/tree/master/examples/rollbar-android" target="_blank" rel="noopener">on GitHub</a>.


## Configuration

All configuration is done via the Config object in `rollbar-java`. You can see the interface <a href="https://github.com/rollbar/rollbar-java/blob/master/rollbar-java/src/main/java/com/rollbar/notifier/config/Config.java" target="_blank" rel="noopener">here</a>.
