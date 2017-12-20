# ngx-sails-socketio
[![npm version](https://badge.fury.io/js/ngx-sails-socketio.svg)](https://badge.fury.io/js/ngx-sails-socketio)
[![Build Status](https://travis-ci.org/burntblark/ngx-sails-socketio.svg?branch=master)](https://travis-ci.org/burntblark/ngx-sails-socketio)
[![Coverage Status](https://coveralls.io/repos/github/burntblark/ngx-sails-socketio/badge.svg?branch=master&cacheBuster=1)](https://coveralls.io/github/burntblark/ngx-sails-socketio?branch=master)

An Angular module for connecting to your SailsJs Backend/API through SocketIO.

## Installation

 ```bash
 npm i ngx-sails-socketio
 ```

## Usage

Add `SailsModule` to your application module.

 ```ts
 // Override options to configure connection to your sailsjs backend
 const options: SailsOptions = {
    url: "ws://127.0.0.1:1337",
    prefix: "/api", // Optional uri prefix
    environment: SailsEnvironment.DEV,
    query: "__sails_io_sdk_version=0.11.0&__sails_io_sdk_platform=windows&__sails_io_sdk_language=javascript",
    reconnection: true,
    autoConnect: false
};

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    SailsModule.forRoot(options, [AuthInterceptor,])
    ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})

export class AppModule { }
 ```

Inject the `Sails` into your components/services and instatiate classes specific to your objective.

 ```ts
class ExampleComponent implements OnInit {

  data: any;

  constructor(private sails: SailsClient) { }

  ngOnInit() {
    const req = new SailsRequest(this.sails);
    req.get('/model/action').subscribe(res => {
      this.data = res.getBody();
    });
  }

}
 ```

## API

### class SailsRequest

Makes a request for a resource to the configured sails server.

* get(url: `string`): `Observable<SailsResponse>`
* post(url: `string`, data: `any`): `Observable<SailsResponse>`
* put(url: `string`, data: `any`): `Observable<SailsResponse>`
* patch(url: `string`, body: `any`): `Observable<SailsResponse>`
* delete(url: `string`): `Observable<SailsResponse>`
* on(event: `string`): `Observable<SailsEvent>`
* off(event: `string`): `Observable<SailsEvent>`

### class SailsEvent

A class representing an event on a model from the server. This follows the information as describe by the sailsjs Pub-Sub event.

### class SailsQuery

A class for querying records from the server. Similar to `SailsRequests` but has methods mapping actions as detail on waterline api used by sailsjs.

### interface SailsInterceptorInterface

An interface to construct an iterceptor to use for requests.

#### Example

An authentication interceptor to set the *Authorization* header on every request.

```ts
@Injectable()
export class AuthInterceptor implements SailsInterceptorInterface {

    constructor(private router: Router) {
    }

    intercept(request: SailsRequestOptions, next: SailsInterceptorHandlerInterface): Promise<SailsResponse> {
        request.clone({
            headers: request.headers.set("Authorization", token)
        });
        
        const response = next.handle(request);

        return response.then(res => {
            if (res.getStatusCode() === 401) {
                this.router.navigateByUrl("login");
            }
            return res;
        });
    }
}
```
