## Installation

Install the Rollbar <a href="https://github.com/rollbar/rollbar.js" target="_blank" rel="noopener">rollbar.js</a> package using npm:

```bash
$ npm install --save rollbar
```

## Configuration

Import rollbar into your app and configure it to catch uncaught exceptions:

```
import * as Rollbar from 'rollbar';
import { BrowserModule } from '@angular/platform-browser';
import { NgModule, ErrorHandler } from '@angular/core';
import { AppComponent } from './app.component';

const rollbarConfig = {
  accessToken: 'POST_CLIENT_ITEM_ACCESS_TOKEN',
  captureUncaught: true,
  captureUnhandledRejections: true,
};

export function rollbarFactory() {
  return new Rollbar(rollbarConfig);
}

@Injectable()
export class RollbarErrorHandler implements ErrorHandler {
  rollbar: any;
  constructor(private injector: Injector) {
    this.rollbar = injector.get(Rollbar);
  }

  handleError(err: any ): void {
    console.log(err);
    this.rollbar.error(err.originalError || err);
  }
}


@NgModule({
  imports: [ BrowserModule ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ],
  providers: [
    { provide: ErrorHandler, useClass: RollbarErrorHandler },
    { provide: Rollbar,  useFactory: rollbarFactory }
  ]
})
export class AppModule { }

```

## Send a test error

Run the following command in your Javascript console.  Rollbar should catch the error and send it to your account:

```bash
window.onerror("TestError: Hello world", window.location.href)
```


## Troubleshooting

If your system consists of the following

```
@angular/cli: 1.4.3
node: 6.11.3
os: darwin x64
@angular/animations: 4.4.3
@angular/cli: 1.4.3
@angular/common: 4.4.3
@angular/compiler: 4.4.3
@angular/compiler-cli: 4.4.3
@angular/core: 4.4.3
@angular/forms: 4.4.3
@angular/http: 4.4.3
@angular/platform-browser: 4.4.3
@angular/platform-browser-dynamic: 4.4.3
@angular/router: 4.4.3
@angular/language-service: 4.4.3
typescript: 2.3.4
```

there are some further steps you may need to implement in order to get rollbar.js working for you. 

When compiling, if you get the error `Error encountered resolving symbol values statically. Function calls are not supported. Consider replacing the function or lambda with a reference to an exported function`, then the inline factory function in providers should be an exported function.

Another error you may encounter when compiling is `Property 'error' does not exist on type '{}'.` In this case, the RollbarErrorHandler `var rollbar` needs to have a type explicitly defined, i.e. `var rollbar: Rollbar`.

