# vue-apex

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


## Visual Force Setup

Zip the folowing files and upload as Static Resource on Salesforce dashboard, I'll name it as `VueResources`
```
Resources.zip
├── material-icons.css
├── MaterialIcons-Regular.ttf
├── MaterialIcons-Regular.woff
├── MaterialIcons-Regular.woff2
├── vue.js
├── vuetify.js
└── vuetify.min.css
```

### Front-end setup

Then go to your Developer Console, create a new Visualforce Page, define a name and paste the following code:
``` html
<apex:page showHeader="true" sidebar="false" controller="VueInventoryController">
    <apex:stylesheet value="{!URLFOR($Resource.VueResources, 'material-icons.css')}"/>
    <apex:stylesheet value="{!URLFOR($Resource.VueResources, 'vuetify.min.css')}"/>
    <apex:includeScript value="{!URLFOR($Resource.VueResources, 'vue.js')}"/>
    <apex:includeScript value="{!URLFOR($Resource.VueResources, 'vuetify.js')}"/>
    
    <html xmlns:v-bind="http://vue.org" xmlns:v-on="http://vue.org">
        <body>
            <!-- <v-app></v-app> -->
            <!-- <script></script> -->
        </body>
    </html>
    
</apex:page>
```
In the commented section above, goes the "src/App.vue" `v-app` and `script` contents (DO NOT PUT THE ROOT `<template>` TAG)

Also change here:
``` js
export default {
    name: 'app',
    /* .... */
}
```

To
``` js
var app = new Vue({
    el: '#app',
    /* .... */
)}
```

The final structure should be something like:
``` html
<apex:page showHeader="true" sidebar="false" controller="VueInventoryController">
    <apex:stylesheet value="{!URLFOR($Resource.VueResources, 'material-icons.css')}"/>
    <apex:stylesheet value="{!URLFOR($Resource.VueResources, 'vuetify.min.css')}"/>
    <apex:includeScript value="{!URLFOR($Resource.VueResources, 'vue.js')}"/>
    <apex:includeScript value="{!URLFOR($Resource.VueResources, 'vuetify.js')}"/>
    
    <html xmlns:v-bind="http://vue.org" xmlns:v-on="http://vue.org">
        <body>

            <v-app id="app">
                <v-toolbar ...>
                    ...
                </v-toolbar>
                
                <!-- Items Tab -->
                <v-content ...>
                    ...
                </v-content>
                
                <!-- Cases Tab -->
                <v-content ...>
                    ...
                </v-content>
            </v-app>

            <script>
                var app = new Vue({
                    el: '#app',
                    data () {...},
                    methods: {...},
                    beforeMount () {...}
                )}
            </script>

        </body>
    </html>
    
</apex:page>
```

### Back-end setup

In your Developer Console, create a new Apex Class, I will call `VueInventoryController`, then paste the following code:
```java
public with sharing class VueInventoryController {

    @RemoteAction
    public static List<Inventory__c> getItens() {
        ID userId = UserInfo.getUserId();
        List<Inventory__c> items = [SELECT Id, Name__c, Description__c, Price__c, Quantity__c FROM Inventory__c WHERE CreatedById =: userId];
        return items;
    }
    
    @RemoteAction
    public static List<Case> getCases() {
        List<Case> cases = [SELECT Id, Subject, LastModifiedDate, Status, Description FROM Case];
        return cases;
    }

    @RemoteAction
    public static Inventory__c updateOrInsertItem(String itemSerialized) {
        Inventory__c item = (Inventory__c)JSON.deserialize(itemSerialized, Inventory__c.class);
        try {
            upsert item;
        } catch(DmlException e) {
            System.debug('An unexpected error has occurred: ' + e.getMessage());
        }
        return item;
    }

}
```

Remember to match the class name with your 'src/App.vue', in this case `VueInventoryController`
```js
Visualforce.remoting.Manager.invokeAction(
    '{!$RemoteAction.VueInventoryController.getItens}',
    (result, event) => {},
)
```

After this all shoud be working, to test: `{Visualforce page name}.vfp` > Preview (top left corner)

;)