# Lets build advancedMessenger hybrid app
## Lab 2.62
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

Goto `employee-provider.ts` and replace the url with http://<your-mock-server>.mybluemix.net/employees

Goto `news-provider.ts` and replace the url with
http://<your-mock-server>.mybluemix.net/news

Goto `schedule-provider.ts` and replace the url with http://<your-mock-server>.mybluemix.net/schedule

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
? Enter the context root of the MobileFirst administrative services: Enter key
? Enter the MobileFirst Server connection timeout in seconds: Enter key
```








