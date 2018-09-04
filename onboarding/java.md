## Installation

To send errors to Rollbar from your Java application, you should use our <a href="https://github.com/rollbar/rollbar-java" target="_blank" rel="noopener">rollbar-java</a> package.

``` java
compile('com.rollbar:rollbar-java:1.3.1')
```

## rollbar-java

The minimal amount of code you need to use `rollbar-java` is:

``` java
import static com.rollbar.notifier.config.ConfigBuilder.withAccessToken;
import com.rollbar.notifier.Rollbar;
Rollbar rollbar = Rollbar.init(withAccessToken("{{ server_access_token }}").build());
rollbar.log("Hello, Rollbar");
```

A few seconds after you execute this code, the message should appear on your project's "Items" page.

For a more detailed example integration, see below:

We initialize `com.rollbar.notifier.Rollbar` with a `Config` which has the access token and other configuration properties set on it.
We can then this this instance of Rollbar to report messages via the `log/debug/warning/error/critical` methods.

Any uncaught exceptions are automatically reported to Rollbar. This behavior can be turned off via the configuration.

``` java
import static com.rollbar.notifier.config.ConfigBuilder.withAccessToken;
import com.rollbar.notifier.Rollbar;
import com.rollbar.notifier.config.Config;

public class Application {
  private Rollbar rollbar;

  public Application() {
    Config config = withAccessToken("{{ server_access_token }}")
        .environment("development")
        .codeVersion("1.0.0")
        .build();
    this.rollbar = Rollbar.init(config);
  }

  public static void main(String[] args) {
    Application app = new Application();
    app.execute();
  }

  private void execute() {
    try {
      throw new RuntimeException("Some error");
    } catch (Exception e) {
      rollbar.error(e, "Hello, Rollbar");
    }
  }
}
```

A more full-featured example can be found <a href="https://github.com/rollbar/rollbar-java/tree/master/examples/rollbar-java" target="_blank" rel="noopener">on GitHub</a>.


## Configuration

All configuration is done via the Config object in `rollbar-java`. You can see the interface <a href="https://github.com/rollbar/rollbar-java/blob/master/rollbar-java/src/main/java/com/rollbar/notifier/config/Config.java" target="_blank" rel="noopener">here</a>.

## Spring

Check out [this blog post](https://rollbar.com/blog/spring-mvc-exception-handling/) for more information on how to use rollbar-java in your Spring app.
