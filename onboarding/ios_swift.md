## Installation With <a href="http://cocoapods.org/" target="_blank" rel="noopener">Cocoapods</a> 
_**NOTE:** This is our recommended installation method._

In your Podfile:

```ruby
platform :ios

target 'YOUR_APP_NAME' do
  use_frameworks!
  pod "Rollbar", "~> 1.4.2"
end

```
**NOTE:** The `plaftform:ios` and `use_frameworks!` statements are required in your Podfile


### Without Cocoapods

1. Download the `Rollbar.zip` file from the <a href="https://github.com/rollbar/rollbar-ios/releases/latest/" target="_blank" rel="noopener">latest release of rollbar-ios</a>.

2. Extract the Rollbar directory in the zip file to your Xcode project directory.

3. In Xcode, select _File_ -> _Add Files to "[your project name]"_ and choose the Rollbar directory from step 2.

## Configuration and Testing

In your `AppDelegate.swift` file, add the following import statement:

```swift
import Rollbar
```

Then add the following to the`application` function:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        Rollbar.initWithAccessToken("{{client_access_token}}")
        Rollbar.info(withMessage: "Test message to Rollbar")
        //Rest of your code here...
        return true
    }
```

Once you've confirmed that a test message was sent to your Rollbar project, remove the call to `Rollbar.info()`.

That's all that's necessary for Rollbar to capture crashes and report them next time your app starts!

## Further Configuration

For additional configuration instructions, including instructions for symbolication, see the documentation for <a href="https://rollbar.com/docs/notifier/rollbar-ios" target="_blank" rel="noopener">rollbar-ios</a>.
