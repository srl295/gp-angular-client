AngularJS for the Globalization Pipeline on IBM's Bluemix
===

<!--
/*  
 * Copyright IBM Corp. 2015
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
-->

#  What is this? 
This project provides an SDK for AngularJS developers to dynamically utilize the [Globalization Pipeline](https://www.stage1.ng.bluemix.net/docs/services/GlobalizationPipeline/index.html) running on [Bluemix](http://www.ng.bluemix.net) IBM's next-generation cloud app development platform. 



The SDK provides familiar AngularJS constructs, in the form of a Directive and a Service, that encapsulate usage of the restful API of the Globalization Pipeline to enable globalization of your application.


## Licensing
This project is licensed under the [Apache License](./LICENSE.txt).


## Installation

_Bower_ release installation:

    $ bower install gp-angular-client
    
_Manaual_ installation:

    $ git clone git://github.com/IBM-Bluemix/gp-angular-client    

# Usage

### Overview
The designed workflow for using this SDK is as follows. Application bundle data is uploaded onto a Globalization Pipeline instance running on Bluemix and any desired language translations are selected for that bundle. A bundle user of type READER is defined for that service. Once those pieces are configured, data is pulled from Bluemix and used to complete the SDK configuration in the client app. (see Configuration below) 

The fields needed to configure this SDK are:

* uri
* instanceId
* userId
* password

The uri & instanceId fields are taken from the credentials of your service instance of the Globalization Pipleline

![Create new Globalization Pipeline service instance](https://ibm.box.com/shared/static/v59b5a19qjkfhxqaiwauz37nd9d8o8m2.gif)


The userId & password come from the READER user defined to have access to the bundle(s) under the [Users] tab. IMPORTANT! The password is only available when the user is first defined. It can NOT be obtained later.

![Create new bundle](https://ibm.box.com/shared/static/8p2ytfm28smh29rl50c581gcfb4hsz8z.gif)

Once the GlobalizationPipelineService configuration is complete and you have set your application configuration to those values, you can use the custom `gp-translate` directive or service functions to handle the dynamic globalization needs of your app.

### Details 
The SDK service API consists of two main functions. A Translate [`translate(key, lang)`] function and a function to obtain a list of the available languages [`getAvailableLanguages()`]. A general purpose custom directive [`gp-translation`] is supplied to handle simple cases of calling the translation service. The translation service function can be incorporated into more complex user defined directives should your application require more extensive directives.

In addition to the `translate` & `getAvailableLanguages` functions, other useful functions exposed from the service are:
* `setBundleId(newBundleId)` - enables switching of the default bundle ID used for translations.
* `getBundleId()` - returns the currently configured default bundle ID.
* `getLoadingText()` - obtain the configured loading text value.
* `setTargetLang(newLang)` - enables the application to override the browser default target language for translations.
* `removeTargetLang()` - enables clearing of a configured `targetLang`.


If a language is used for a translation that is not available for a given bundle on the Globalization Pipeline service instance, the source language for that bundle will be used for translations. If a key is not defined in a bundle, the key value will be returned as the "translated" value.

#### - _bundle specification_
For a simple application a single bundle may suffice and maybe configured in the SDK configuration [`bundleId`]. For more granular control, the configured bundleId can be dynamically set in controllers by calling `setBundleId(newBundleId)` for a particular view. This allows a complex application to split up it's G11n application data into smaller more manageable bundles. 

```javascript
.controller('View1Ctrl', ['$routeParams', 'GlobalizationPipelineService', function($routeParams, gp) {
  gp.setBundleId("oneOfYourBundleIds");

```

#### - _handling options for awaiting text translation_
While your application is waiting on the promise from the rest API, this SDK provides 2 facilities to handle the not yet available text. Support of `ng-show` or the specification of loading text.

##### -- _loading text_
The optional configuration parameter of `loadingText` provides a means to display some form of text to indicate that the app is waiting for something to complete. For example, if `loadingText` is set to `loading...` then text elements waiting on the translation will display `loading...` until the promise is resolved. Otherwise the elements will be blank.

##### -- _ng-show support_
`ng-show` is an AngularJS convention used to hold off on making elements visible until such time as everything is ready. This SDK supports `ng-show` thru the global var `gpTranslationComplete`. 

Example:
```html
<div ng-show="gpTranslationComplete">
    <h1 gp-translate="Title"></h1>
    <gp-translate key="Hello"></gp-translate>
</div>
```


#### - _target language specification_
NOTE: While it is assumed that typical usage will be to use the language the user has set in the browser and not have a need to configure or change the `targetLang` configuration parameter, `setTargetLang(newLang)` & `removeTargetLang()` are provided to control overriding the browser setting should that be desirable. Along with being able to specify it on the function/directive.


The target language for the translation is determined in 1 of 3 ways (in order of precedence). 

1. It can be supplied on the translation function call (or on the custom directive).
2. A target language can be set in the configuration (`targetLang:`) to eliminate the need to override it on each translation call. 
3. Or use the language configured in the browser.

language override on the directive example:
```html
    <gp-translation key="DATA-KEY" lang="es"></gp-translation>
```

### custom directive
This SDK provides a custom directive that handles straightforward use of translations and can be specified in two ways.

Example:

```html

    <H1 gp-translation="TITLE"></H1>
    <gp-translation key="ANOTHER-DATA-KEY"></gp-translation>

```

In addition to specifying the key to be translated the directive also supports overriding the target language and/or the bundleId. 

Example:

```html

    <H1 gp-translation="TITLE" lang="es"></H1>
    <span gp-translation="DATA-KEY" bundleId="specialBundleId"></span>
    <gp-translation key="ANOTHER-DATA-KEY" lang="de" bundleId="someOtherBundle"></gp-translation>

```



## Configuration
This service needs to be configured in order to talk to the GP server. On Bluemix you will need to configure the IBM Globalization Pipeline service, define bundle(s), and define a bundle reader. Once that is completed you can take that information and fill in your AngularJS config data to talk to the service.

`bundleId` is required to be present at the time of translation but does NOT have to be present at .config time.

Optional configuration for target language (`targetLang`) & text to display while waiting for the results from the rest call (`loadingText`) can be configure here as well.

Example of configuration

```javascript

.config(['GlobalizationPipelineServiceProvider', function(GlobalizationPipelineService) {

    GlobalizationPipelineServiceProvider.setGpConfig({
        bundleId: "YOUR-BUNDLE-NAME",                   // recommended unless dynamically set
        targetLang: "es",                               // optional default language
        loadingText: "loading...",                      // optional text displayed while waiting on a GP promise
        credentials: {                                  
            uri: "https://YOUR-SERVICE-INSTANCE-URL",   // required
            instanceId: "YOUR-INSTANCE-ID",             // required
            userId: "YOUR-BUNDLES-READER-ID",           // required
            password: "YOUR-BUNDLES-READER-PASSWORD"    // required
        }});
}])
```
