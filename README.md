# Lets build advancedMessenger hybrid app
## Lab 2.62
### Objectives
- Cloning the github project
- Creating a node.js application in Bluemix to mock backend server
- Creating MobileFirst Foundation Service in Bluemix
- Bootstrapping ionic 2 project to MobileFirst

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
