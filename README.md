# Lets build advancedMessenger hybrid app
## Lab 2.62 - Adding MobileFirst SDK and registering app on server
### Objectives
- Cloning the github project
- Creating a node.js application in Bluemix to mock backend server
- Creating MobileFirst Foundation Service in Bluemix
- Preview the ionic2 sample
- Bootstrapping ionic2 sample project to MobileFirst

#### Cloning the github project
Open `Terminal`

```sh
$ cd dev/workspaces
$ mkdir am
$ cd am/
$ git init
$ git remote add origin https://github.com/prem911/advancedMessenger
$ git pull origin initial
$ ls
$ cd mockServer/
$ ls 'Look at the files of this node.js application'
```
#### Creating a node.js application in Bluemix to mock backend server
```sh
$ cf login -a api.ng.bluemix.net
Email > 'write your Bluemix id'
Password > 'write your Bluemix password'
$ cf push 'give-a-unique-name-to-your-app' -m 512M
```
Open Browser and point to http://<give-a-unique-name-to-your-app>.mybluemix.net/api

It should return **Mock APIs are running**

#### Creating MobileFirst Foundation Service in Bluemix
Open http://bluemix.net and login

Goto Catalog / Mobile

Click Mobile Foundation to create the service. Give a unique name to the service. Choose Developer Edition

Click the link "Start Basic Server". It will take 3-4 minutes to deploy this server in Bluemix.

Lookup the admin password by clicking the "eye".

Copy the Server Route and 

Click "Launch Console" and provide admin and the above password in the credentials window.

#### Preview the ionic sample
```sh
$ cd ../advancedMessenger
$ code . 'This opens the project in Visual Studio Code'
```
click the Search icon in the left side toolbar and search for 'localhost'

Goto `employee-provider.ts` and replace the url with http://{your-mock-server}.mybluemix.net/employees

Goto `news-provider.ts` and replace the url with
http://{your-mock-server}.mybluemix.net/news

Goto `schedule-provider.ts` and replace the url with http://{your-mock-server}.mybluemix.net/schedule

Save all files

```sh
$ npm install 'this loads all the dependencies'
$ ionic serve 'Starts serving the ionic project and opens a browser'
```
Review the app and then quit ionic live reload by pressing 'q' in terminal. Close the browser tab.

#### Bootstrapping ionic2 project to MobileFirst
```sh
$ cordova platform add android 'do this from the am/advancedMessenger folder'
$ cordova plugin add cordova-plugin-mfp 'adds the core mfp plugin'
$ cordova plugin add cordova-plugin-mfp-push 'adds the push capability'
$ cordova plugin add cordova-plugin-mfp-jsonstore 'adds the json store capability'
```

```sh
$ mfpdev server add 'to point to the MobileFirst Server created in Bluemix'
? Enter the name of the new server profile: myMFPServer
? Enter the fully qualified URL of this server: http://<hostname-mfp-server>.mybluemix.net:80
? Enter the MobileFirst Server administrator login ID: admin
? Enter the MobileFirst Server administrator password: <your-admin-pwd>
? Save the administrator password for this server?: Y
? Enter the context root of the MobileFirst administrative services: Use Default
? Enter the MobileFirst Server connection timeout in seconds: Use Default
```
```sh
$ mfpdev app register 'ensure that working directory is am/advancedMessenger'
$ cordova prepare
```
Open mfpconsole in browser ( http://{your-mfp-server}.mybluemix.net/mfpconsole ) or refresh if already open
advancedMessenger is now shown in the list of Applications
```sh
mfpdev app preview 'preview using mfp commands, select simple browser'
```
Open Developer Console in the browser and look at console. You will find messages from worklight.js.
`Note that these calls have happened after the data load`

##### Live reload using mfpdev - Use gulp watch
gulp watch checks for changes in source files and prepares the www contents.
Any change in source will regenerate the www and the browser preview will change.
```sh
$ gulp watch 'working directory should be am/advancedMessenger'
```

##### Catching mfpjsloaded event
We should do all backend invokations only after MFP API is initialised. To do this we need to catch an event called mfpjsloaded. This is defined in `bootstrap.js` in plugins/cordova-plugin-mfp
Changes required in `app.ts`

```javascript
import {Component, Renderer} from '@angular/core';
import {Platform, ionicBootstrap} from 'ionic-angular';
import {StatusBar} from 'ionic-native';
import {TabsPage} from './pages/tabs/tabs';

@Component({
  template: '<ion-nav [root]="rootPage"></ion-nav>'
})
export class MyApp {
  private rootPage:any;

  constructor(private platform:Platform, renderer: Renderer ) {
    console.log('constructor done');
    renderer.listenGlobal('document', 'mfpjsloaded', () => {
      console.log('--> MFP API init complete');
      this.MFPInitComplete();
    })
    platform.ready().then(() => {
      StatusBar.styleDefault();
    });
  }
  MFPInitComplete(){
    console.log('--> MFPInitComplete function called')
    this.rootPage = TabsPage; 
  }
}
ionicBootstrap(MyApp)
```
Save `app.ts`. Browser preview refreshes. Look at the sequence of calls in the console.
Now all the backend calls are happening only after MFP API has been initialised.

## Lab 2.63 - Using Java and Javascript adapters to perform backend calls
### Objectives
- Create Javascript Adapter - EmployeeAdapter
- Create Java Adapter from a sample
- Invoking the JavaScript Adapter from client code
- Invoking the JavaHttp Adapter from client code

#### Create Javascript Adapter - EmployeeAdapter
Open a new tab in `Terminal`

Set the working directory to am/

Open project in Visual Code from 'am' folder if not already open

```sh
$ mfpdev adapter create
? Enter adapter name: employeeAdapter
? Select adapter type: HTTP
? Enter group ID: com.adapters
```
A new HTTP adapter is created. This adapter will call http://{your-mock-server}.mybluemix.net/employees
> Try this bluemix URL in your browser to see the json output.
> You will see a json data dump of few random users.

In Visual Studio Code, expand employeeAdapter look at `pom.xml` and `adapter.xml`

In `adapter.xml` change protocol to http, domain to {your-mock-server}.mybluemix.net and port to 80

Remove the two procedures and write a new one
<procedure name="getRating"/>

Modify `employeeAdapter-impl.js`
```javascript
function getRating() {
	var input = {
	    method : 'get',
	    returnedContentType : 'json',
	    path : 'employees'
	};
	return MFP.Server.invokeHttp(input);
}
```
In `Terminal`
```sh
$ cd employeeAdapter/
$ mfpdev adapter build 'It should result in Successfully built adapter'
$ mfpdev adapter deploy 'Deploys it to the MobileFirst Server in Bluemix'
```
> Refresh the mfpConsole to find the new adapter

Click employeeAdapter, check Configurations and Resources tab.
In Resources tab, Security defined is DEFAULT_SCOPE

Goto Runtime Settings / Confidential Clients, add a new Test Client ( `this should not be done for production servers` )
Give Display Name as Test Client, ID as test, Secret as test and Allowed Scope as '**' ( double asterix )

Go to EmployeeAdapter / Resources Tab and open Swagger Document.

We can test /getRating.
> Execute without specifying any scope. We will get 401

> Specify the DEFAULT_SCOPE by clicking the toggle button on the right side and Authorize. Give test/test in Challenge window. Now try the api and it will show the same json result. This makes the API calls through adapters scoped and safe.

##### Changing the endpoint url of employeeAdapter
Add a property in `adapter.xml`
```xml
<property name="endpoint" displayName="Endpoint" defaultValue="employees"/>
```

Modify `employeeAdapter-impl.js`
```javascript
function getRating() {
	var endpoint = MFP.Server.getPropertyValue("endpoint");
	var input = {
	    method : 'get',
	    returnedContentType : 'json',
	    path : endpoint
	};
	return MFP.Server.invokeHttp(input);
}
```
Build and deploy these adapter changes
```sh
$ mfpdev adapter build
$ mfpdev adapter deploy
```
> Goto mfpconsole / employeeAdapter / Configurations.
> Change "Endpoint" value to "news" and test the /getRating api in Swagger Docs.
You should see a news json. Reset the "Endpoint" back to "employees"

#### Create Java Adapter from a sample
In the browser, click on Tutorials and Developing Adapters/Java/HTTP Adapter
This describes how to define and invoke Java Adapters
Download the sample adapter from this page.
Unzip the file and copy the JavaHTTP folder to am/ folder
```sh
$ cd && cd Downloads/
$ unzip Adapters-release80.zip
$ cd Adapters-release80
$ cp -R JavaHTTP/ /home/ibm/dev/workspaces/am/
$ cd ~/dev/workspaces/am/JavaHTTP/
```

Look at the JavaHTTP folder's `adapter.xml` in Visual Studio Code
Change the displayName and description to NewsJavaAdapter

Open `JavaHTTPResource.java`

Change to
```javascript
public static void init() {
	client = HttpClientBuilder.create().build();
	host = new HttpHost("{your-mock-server}.mybluemix.net", 80, "http");
}

public void execute(HttpUriRequest req, HttpServletResponse resultResponse)
		throws IOException,
		IllegalStateException, SAXException {
	HttpResponse JSONResponse = client.execute(host, req);
	ServletOutputStream os = resultResponse.getOutputStream();
	if (JSONResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK){
		resultResponse.addHeader("Content-Type", "application/json");
		JSONObject result = JSONObject.parse(JSONResponse.getEntity().getContent());
		String json = result.toString();
		os.write(json.getBytes(Charset.forName("UTF-8")));

	}else{
		resultResponse.setStatus(JSONResponse.getStatusLine().getStatusCode());
		JSONResponse.getEntity().getContent().close();
		os.write(JSONResponse.getStatusLine().getReasonPhrase().getBytes());
	}
	os.flush();
	os.close();
}

@GET
@Produces("application/json")
public void get(@Context HttpServletResponse response)
		throws IOException, IllegalStateException, SAXException {
		execute(new HttpGet("/news"), response);
}
```
```sh
$ mfpdev adapter build
$ mfpdev adapter deploy 'optionally try out deploying an adapter from the mfpconsole, Actions/Deploy Adapter/pick the file from target folder of JavaHTTP'
```

> Test the api using Swagger docs. This too has DEFAULT_SCOPE and can be tested similar to /getRating

#### Invoking the JavaScript Adapter from client code
Take a look at the Swagger Doc for /getRating url
> http://{your-mfp-server}.mybluemix.net/mfp/api/adapters/employeeAdapter/getRating

Close any editors in Visual Studio Code and open `employee-provider.ts`.
This provider is called when Ratings page is opened in the app.

```javascript
import {Injectable} from '@angular/core';
/*
  Generated class for the EmployeeProvider provider.
  See https://angular.io/docs/ts/latest/guide/dependency-injection.html
  for more info on providers and Angular 2 DI.
*/
@Injectable()
export class EmployeeProvider {
  data: any = null;
  constructor() {}
  load() {
    console.log('---> called EmployeeProvider load');  
    if (this.data) {
      // already loaded data
      return Promise.resolve(this.data);
    }
    // don't have the data yet
    return new Promise(resolve => {
      // We're using Angular Http provider to request the data,
      // then on the response it'll map the JSON data to a parsed JS object.
      // Next we process the data and resolve the promise with the new data.
      let dataRequest = new WLResourceRequest("/adapters/employeeAdapter/getRating", WLResourceRequest.GET);
      dataRequest.send().then((response) => {
        console.log('--> data loaded from adapter', response);
        this.data = response.responseJSON.results;
        resolve(this.data);
      }, (failure) => {
        console.log('--> failed to load data', failure);
        resolve('error');
      })
    });
  }
}
```

Gulp shows errors for WLResourceRequest. To resolve this open `typings/main.d.ts` and make the following changes to include mfp core, jsonstore and mfppush

```javascript
/// <reference path="main/ambient/es6-shim/index.d.ts" />
/// <reference path="../plugins/cordova-plugin-mfp/typings/worklight.d.ts" />
/// <reference path="../plugins/cordova-plugin-mfp-jsonstore/typings/jsonstore.d.ts" />
/// <reference path="../plugins/cordova-plugin-mfp-push/typings/mfppush.d.ts" />
```

> Save the changes and preview the app in the browser and look at console for log information on adapter data. With this, the getRating calls are being routed through the adapter

#### Invoking the JavaHttp Adapter from client code
Open `news-provider.ts `

> Look at the Swagger Doc to get the URL for the JavaAdapter Call

```javascript
import {Injectable} from '@angular/core';
/*
  Generated class for the NewsProvider provider.
  See https://angular.io/docs/ts/latest/guide/dependency-injection.html
  for more info on providers and Angular 2 DI.
*/
@Injectable()
export class NewsProvider {
  data: any = null;
  constructor() {}
  load() {
    console.log('---> called NewsProvider load');
    if (this.data) {
      // already loaded data
      return Promise.resolve(this.data);
    }
    // don't have the data yet
    return new Promise(resolve => {
      // We're using Angular Http provider to request the data,
      // then on the response it'll map the JSON data to a parsed JS object.
      // Next we process the data and resolve the promise with the new data.
      let dataRequest = new WLResourceRequest("/adapters/JavaHTTP/", WLResourceRequest.GET);
      dataRequest.send().then((response) => {
        console.log('--> data loaded from adapter', response);
        this.data = response.responseJSON.news;
        resolve(this.data);
      }, (failure) => {
        console.log('--> failed to load data', failure);
        resolve('error');
      })
    });
  }
}
```

## Lab 2.64 - Securing backend calls with user authentication
### Objectives
- User Authentication Sample from Download Center
- Apply Scope element security to employeeAdapter
- Apply Scope element security to Java Adapter
- Client side changes to invoke Security Challenge
- Test it on emulator with cdvlive

#### User Authentication Sample from Download Center
Goto mfpconsole
> http://{your-mfp-server}.mybluemix.net/mfpconsole,
> Click Download Center ( on the left hand menu ), goto Samples tab and download UserAuthentication Sample

Close the live reload command and also close the browser tab. From now on we will test the app on an emulator

```sh
$ cd ~/Downloads/
$ unzip UserLogin.zip
$ mv UserLogin ../dev/workspaces/am/
$ cd dev/workspaces/am/UserLogin/
$ mfpdev adapter build && mfpdev adapter deploy 'To build and deploy this adapter'
```

> Goto mfpconsole and refresh the page. UserLogin adapter appears with a lock icon. It signifies that this is the security check for Authentication

Open `UserLoginSecurityCheck.java`. In the validateCredentials, we have a check for username and password.

#### Apply Scope element security to employeeAdapter

In `adapter.xml` of employeeAdapter :
```xml
<procedure name="getRating" scope="restrictedData"/>
```
```sh
$ cd ../employeeAdapter/
$ mfpdev adapter build && mfpdev adapter deploy
```
> Goto console and click employeeAdapter. Click on Resources tab. There is a change in Security scope. Earlier it was DEFAULT_SCOPE and now it has changed to restrictedData. This is done without a server restart too!

Lets create the mapping to pass this security data.

Goto mfpconsole -> advancedMessenger/Android and click Security tab.
- Scroll down to find Scope-Elements Mapping.
- Click New. 
- Fill restrictedData in the Scope element.
- For Custom Security Checks, select UserLogin and press Add.

#### Apply Scope element security to Java Adapter

Open `JavaHTTPResource.java` from JavaHTTP

Add an import statement for OAuthSecurity
```javascript
import com.ibm.mfp.adapter.api.OAuthSecurity;
```
Use this security to secure the /GET endpoint. Add the decorator @OAuthSecurity

```javascript
@GET
@Produces("application/json")
@OAuthSecurity(scope = "restrictedData")
public void get(@Context HttpServletResponse response)
    throws IOException, IllegalStateException, SAXException {
        execute(new HttpGet("/news"), response);
}
```
Lets build and deploy this adapter
```sh
$ cd ../JavaHTTP/
$ mfpdev adapter build && mfpdev adapter deploy
```
> Goto mfpconsole and click NewsJavaAdapter. Click on Resources tab. There is a change in Security scope. Earlier it was DEFAULT_SCOPe and now it has changed to restrictedData. Note, no server restart required.

#### Client side changes to invoke Security Challenge
Open `app.ts`

Let's create an Auth Handler and an Alert to grab the credentials.
For Alert sample code, lets goto Ionic2 website.
Open your browser, click Ionic v2 -> components -> Alerts -> Prompt Alerts
```javascript
AuthInit(){
    this.AuthHandler = WL.Client.createSecurityCheckChallengeHandler("UserLogin");
    this.AuthHandler.handleChallenge = ((response) => {
        console.log('--> inside handleChallenge');
        this.displayLogin(msg);
    })
}
displayLogin(msg) {
    let prompt = Alert.create({
        title: 'Login',
        message: msg,
        inputs: [
            {
                name: 'username',
                placeholder: 'Username'
            },
            {
                name: 'password',
                placeholder: 'Password',
                type: 'password'
            },
        ],
        buttons: [
            {
                text: 'Login',
                handler: data => {
                    console.log('--> Trying to auth with user', data.username);
                    this.AuthHandler.submitChallengeAnswer(data);
                }
            }
        ]
    });
    this.nav.present(prompt);
}
```
We have added AuthHandler,Alert and nav above. Alert needs to referenced and we need to initialize "nav" based on its lifecycle. NavController is loaded after this view initialises.

Modify an existing import statement to
```javascript
import {Platform, Alert, App, ionicBootstrap} from 'ionic-angular';
```
Add couple of private vars
```javascript
private AuthHandler: any;
private nav: any;
```
Handle the Nav Controller life cycle
```javascript
ngAfterViewInit(){
    this.nav = this.app.getActiveNav();
}
```
In a `Terminal`
```sh
$ android avd
```
#### Test it on emulator with cdvlive
Start "nexus4" ' with "Scale display to real size" option

Once the emulator has launched, open another terminal window. We will launch cdvlive for the client project. cdvlive does live reload similar to the live reload in browser
```sh
$ cdvlive android 'Note that the current working directory should be advancedMessenger/
```
> This takes a while to load. Any changes to the source code is picked up by Gulp watch and is packaged to www folder. cdvlive then picks these changes and updates the app

In Browser, open chrome://inspect/#devices

Review the sequence of calls. There is a CORS error by google apis. Lets fix it.
Open `schedule.ts` in app/pages/schedule
Comment off the distance calculation block in loadSchedule()
```javascript
//   this.schedule.calc(geos).then((results) => {
//       for (var i=0; i < results.length; i++) {
//           this.delivery[i].distance = results[i].distance.text;
//           this.delivery[i].duration = results[i].duration.text;
//       }
//   })
```
> The login credentials alert will be shown. We can experiment with giving wrong credentials. To succeed give the same string for username and password.
Note that we didnot get any error message with the wrong credentials.

Open `UserLoginSecurityCheck.java` from UserLogin Adapter.

Look at createChallenge(). The challenge is returning us errorMsg and remainingAttempts.
These are configured in the `adapter.xml`

Open `adapter.xml`

Lets modify the login window to use the errorMsg and remainingAttempts

Open `app.ts`

Modify AuthInit() to
```javascript
AuthInit(){
    this.AuthHandler = WL.Client.createSecurityCheckChallengeHandler("UserLogin");
    this.AuthHandler.handleChallenge = ((response) => {
        console.log('--> inside handleChallenge');
        if(response.errorMsg){
          var msg = response.errorMsg + '<br>';
          msg += 'Remaining attempts: ' + response.remainingAttempts;
        }
        this.displayLogin(msg);
    })
}
```
Modify the definition of displayLogin() to
```javascript
displayLogin(msg)
```
> Test this login window with wrong credentials to see the error message and the attempts remaining.




