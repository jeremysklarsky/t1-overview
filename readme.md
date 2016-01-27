## UI development in T1 Overview

#### Purposes
The goal of this document is not to be exhaustive documentation, but to give a high level overview of the framework, libraries, and general best practices. It will focus on routing, views, and the T1 Library functions.

#### Table of Contents
1. [What Is T1?] (#what-is-t1)
2. [Compass Architecture] (#compass-architecture)
3. [`src`] (#src)
  - [Important Files Worth Noting] (#important-files-worth-noting)
  - [Relevant Folders Overview] (#revelant-folders-overview)
4. [The Backbone Application] (#the-backbone-application--making-sense-of-src)
  - [models and collections] (#models-and-collections)
  - [libs] (#libs)
  - [templates] (#templates)
  - [modules] (#modules)
5. [T1View and T1Layout] (#t1view-and-t1layout)
  - [Differences] (#whats-the-difference)
  - [T1View Lifecycle] (#t1view-lifecycle)
  - [Serialize] (#serialize)
  - [Templating] (#templating)
6. [Additional T1 Features] (#additional-t1-features)
  - [EventHub] (#eventhub)
  - [Data Events] (#data-events)
7. [Feature Flags] (#feature-flags)
8. [Case Study in adding a new view] (#case-study-in-adding-a-new-view-segments-bulk-create)
  - [Routing] (#step-1-routing)
  - [Adding Libraries] (#step-2-adding-libraries)
  - [Creating a module] (#step-3-creating-a-module)
  - [SCSS] (#step-4-sass--css)
  - [Summary] (#step-5-thats-it)
9. [Tips and Tricks] (#tips-and-tricks)
10. [Notes for DMP Users] (#notes-for-dmp-users)

### What is T1?
'T1' is a broadly used term to describe the entire Compass codebase. Specifically, it is a set of Javascript library files that are either: 
- extensions of core Backbone files (e.g. View, Model, Collection) 
- supplemental functionality to Backbone (e.g. T1.Layout, T1.Comm)
- provide additional plug-in type support, similar to jQuery-UI (e.g. T1.Menu, T1.ColumnSortable, T1.Notifier, etc)
- provide classes of utility-type functions (e.g. T1 itself, T1.Timezones, T1.Currencies). 

Many of the plug-ins are slowly being replaced by the Strand web components framework. Some examples include:
- T1.DatePicker -> `<mm-datepicker>`
- T1.Spinner -> `<mm-spinner>` 
- T1.Loader -> `<mm-loader>` & `<mm-progressbar>`

### Compass Architecture
#### Important Folders and Files
- `/contrib/image/sites`: Contains both `compass.conf` and `dmp.conf`. These files are where the API remote URL's are fixed for your local environment.
- `/bower.json`: Dependency management. The most important piece for a UI dev is the `ui-framework` and `webcomponentjs` versions. When changed, a task compiles the specified version's source code to `/.tmp/bower_components`. If you've changed something, make sure that the correct version is listed in `/.tmp/bower_components/ui-framework/.bower.json` and `/.tmp/bower_components/webcomponentsjs/.bower.json`. _NOTE: If you need to run Compass locally in Firefox, you must (until further notice) downgrade your webcomponents and ui-framework versions to 0.5.1. Anything newer causes require.js to fail when compiling the web components framework for Firefox._
- `/src`: contains almost everything else you need as a Compass UI dev.

### ./src

#### Important files worth noting
- `env_local.js`: Where environmental variables and UI feature flags are set for your locally run instance of Compass. 
- `main.js`: Sets the paths, globally, so files, folders, and libraries can be added in the `define` block of your files by using an alias instead of needing to spell out the full path.
- `router.config.js`: When Backbone initializes the application in `app.js`, it spins up an instance of `router.js`. It passes to it the configs found in `router.config.js`, similar in principle to `routes.rb` in a Rails application. This allows us with simple notation to designate the routes and URL's of the various sections of the application as well as the default view file for the route. This is important as it tells the application which module and view file to look for when a route is accessed by the browser. Let's look at an example:

```javascript
'reports': {
  'el': '.app-container',
  'tmplPath': 'templates/overlay_layout.html',
  'layout': {
    '.overlay-view': [{
      'module': 'reporting',
      'viewType': 'tabs',
      'showLoader': true
    }]
  },
  'modes': {
    'campaigns': [{
      'module': 'reporting',
      'viewType': 'tabs',
      'showLoader': true
    }],
    'segments': [{
      'module': 'reporting',
      'viewType': 'tabs',
      'showLoader': true
    }]
  },
  'bindings': {
    'onUserLoginOk': { 'action': 'reloadLocation' },
    'organization:explicitChange': {
      'action': 'changeLocation',
      'params': ['reports', { 'replace': true }]
    }
  }
}

```
The top level of the object `reports` corresponds to the route `COMPASS_BASE/#reports`, telling T1 that the to load up the reporting module and its default view type, which will look for tabs.js in that module. The `modes` are the nested routes below reports (so we'll now have access to the URLs `COMPASS_BASE/#reports/segments` and `COMPASS_BASE/#reports/campaigns`). In this case there are 2: `campaigns` and `segments`. In both cases, route objects are built designating where the module is found and its default view. Note that while the top level default view will be found in `reports/views`, the campaigns default view will be found in `reports/campaigns/views/`. What does `bindings` do? I'm not really sure.

Additional Settings:
- `showLoader` sets whether a loader and spinner will display while the module is loading
- `permissions` can put an entire module behind a feature flag (e.g. `segments`)


#### Revelant Folders Overview
- `/src/components/`: In addition to Strand, UI developers have built some of their own custom web components that will not be open-sourced as part of strand.
- `/src/images/`: Contains most of the image assets that are used throughout compass. Many of these are being replaced by `<mm-icon>`. See [Strand's mm-icon documentation] (http://wc-docs.mediamath.com/v0.6.3/mm-icon.html) for more information.
- `/src/js/`: Contains all the JS code, HTML templates, and settings files for the main Backbone application. We'll explore this folder more in depth later.
- `/src/js/sass/`: All styling information. More info on [SASS] (http://sass-lang.com/)

### The Backbone Application / Making sense of `src`
#### models and collections
Primary home for models and collections widely used throughout the application. These are almost exclusively classes of `T1Model` or `T1Collection`.

#### libs 
Contains the source code for all 3rd party javascript libraries, including T1 itself. 
In order to add a library to the codebase: 
- create a folder
- add the file
- add the path to the require configs in `src/js/main.js`. 
- your library can now be referred to by its alias in the `define` blocks of a file. So for example, we can say...

```javascript
define(['jQuery'], function ($) {
});
```
...instead of 

```javascript
define(['libs/jquery/jquery'], function ($) {
});
```

Additional Resources: [RequireJS] (http://requirejs.org/)

#### templates
These are the HTML templates used by the shared libraries. If a T1 library uses HTML, like T1Menu for example, its HTML will most likely be found in this folder.

#### modules
T1 Code is organized not around its views but around modules. One advantage is that the modules are perpendicular to each other - that is, it is not the main Backbone app or router's job to instantiate views. The router simply points to a module's path and the T1 uses the module's configs to instantiate views. The source code for how T1Module works can be found in `src/js/libs/T1/T1.Module.js`.

When a route is accessed, T1 will look for a `main.js` file in the folder the router has told it to access. Let's look at a simple example: `reporting/segments/main.js`

```javascript
define([
  'T1Module',
  'modules/reporting/segments/dataExport/models/model'
], function (T1Module, DataExportModel) {
  'use strict';

  var models = {
    'setupDataExportModel': function setupDataExportModel() {
      if (!this.DataExport) {
        this.DataExport = new DataExportModel();
      }
    }
  };

  return new T1Module({
    'defaultView': 'segment_reports',
    'name': 'reporting/segments',
    'viewCfgs': {
      'segment_reports': { 'models': models }
    }
  });
});
```
This file returns a new instance of the T1Module class. In its options, we designate the default view that will be loaded up (found in the same folder's `views` folder). The name corresponds to the folder's location - which is important for telling T1 where to find its folders. Finally, for each view we intend to use, an configs object must be instantiated in the `viewCfgs` object. This is typically an empty object (e.g. `'segments': {}`). In this case, the module is creating an instance of the `DataExportModel` and passing this to the view which can then be accessed by the `options` argument in the `initialize(options)` function of the view.

### T1View and T1Layout
#### What's the difference?
T1View is a simple extension of Backbone's `View` object. A T1View can be used either as a standalone view, or it can be used as a wrapper to manage a lot of views contained within it. This is where T1Layout comes into play. T1Layout is essentially a view-loadng wrapper. Because T1View is an extension of Backbone.View, setting up a T1View works in the same fashion, except that we have to require `T1View` in the define block. 

```javascript
/*globals define*/
define([
  'jQuery',
  'Underscore',
  'T1',
  'T1View'
], function ($, _, T1, T1View) {

  return T1View.extend({
  });

});
```

In Segments, which uses vanilla Backbone, we would typically manually create a new instance of a view class and assign it to an element. Using T1Layout, we can simple create an instance of T1Layout and load.

_For an example of how `T1Layout` is used, check out `reporting/campaigns/dataExport/createEdit/createEdit`. To see how it is used in conjunction with `T1TabLayoutV2`, check out `admin/views/tabs` and `reporting/views/tabs`. Or feel free to search for `T1Layout` in `src/js/modules` and see what you can find._

#### T1View Lifecycle
A T1View's lifecycle is very similar to a Backbone View. When it is created, `initialize()` is called implicitly, as is `load()`. One advantage of T1View is `render()` returns a `$.Deferred` promise. The most common pattern is something along the lines of:

```javascript
load: function() {
  this.render().then(function(){
    // do some cool stuff now that your view has rendered!
  })
}
```

This is especially useful when using Layouts. That pattern would look something like this:

1. In initialize, load up the layouts.
2. In load, render that view's template, which should be prepopulated with elements for each of the layouts.
3. After render, load the layouts.

Example:
```javascript
initialize: function() {
  this.layout = new T1Layout({
    'el': function () {
      return self.el.find('.your-layout-identifier');
    },
    'template': '<div class="your-layout-el"></div>' //or an actual HTML template file,
    'layout': {
      '.your-layout-el': [{
        'module': 'pathToYourModule',
        'viewType': 'nameOfView',
        'options': {}
      }]
    } 
  })
},

load: function() {
  var self = this
  this.render().then(function(){
    self.layout.load();
  })
}
```
In these simple examples, the T1View does not need a `render()` function explicitly defined because the parent class defines it for us. In our Backbone views we _always_ override the default `render()` function.

#### Serialize
The T1View default `render()` function will be looking for a `serialize()` function in your view. In Backbone views, you typically need to define variables for your templates at the moment the template is either compiled or rendered to the DOM. `serialize()` is T1's way of handling this. A render function using Hogan might look like this
```javascript
$(this.el).html(this.compiledTemplate.render({
  model: '{{model}}',
  scope: '{{scope}}',
  id: '{{model.id}}',
  icon: "{{model.editable == 'true'}}",
  noicon: "{{model.editable == 'false'}}",
  targeting: "{{model.pixel_targeted == 'true'}}",
  nottargeting: "{{model.pixel_targeted == 'false'}}",
  creator: '{{model.creator}}',
  readOnly: READ_ONLY,
  readOnlyMessage: READ_ONLY_MESSAGE
}));
```

Using T1View, we need only to define `serialize()` on our view to return an object containing all our of our defined variables, and T1View's render function will render our templates with values inserted.

Example:

```javascript
serialize: function () {
  return {
    'toggleClass': this.emailSettingsDrawerState,
    'userEmail': USER_EMAIL,
    'toEmails': DataExportModel.get('emailToAddresses'),
    'subject': DataExportModel.get('emailSubject'),
    'frequencyGrain': T1.Utils.toInitialCaps(frequencyGrain),
    'frequencyValue': DataExportModel.getEmailFrequencyValue(true) || '32',
    'endDate': endDate.toString(displayFormat),
    'preferredTime': DataExportModel.getEmailTime(),
    'preferredAMPM': DataExportModel.getEmailAMPM(),
    'message': DataExportModel.getEmailMessage()
  };
}
```
If you do _not_ include a `serialize` function on your view, T1View will call `toJSON` on your view's model (assuming it has been assigned one) and send all those attributes as variables to your template. If you do write a `serialize` method, however, this will be overwritten and you would be responsible to include _all_ variables you want in your `serialize` method.

If you are working in a non T1View (e.g. the entire segments module), you templating and rendering must take place manually. At present, segments uses [Hogan] (http://twitter.github.io/hogan.js/), a variation of Mustache for rendering templates.

#### Templating
By default, `T1View` uses [Mustache] (https://mustache.github.io/). You only need a few simple pieces to get started:

- Interpolating literal values:
``` html
<mm-button type="secondary" id="browse-bulk-segments" unresolved>
  <label>{{label}}</label>
</mm-button>
```

- If / else (i.e. if `create === true`)
``` html
{{#create}}
  <mm-button type="secondary" id="save" unresolved>
    <label>Save</label>
  </mm-button>
{{/create}}

{{^create}}
  <mm-button type="secondary" id="save" unresolved>
    <label>Save</label>
  </mm-button>
{{/create}}
```

Interpolated data is sent to the template when `render` as served to it from `serialize`. So if you want to access a variable in your template, include it in the `serialize` function on your view (if you've written one - see above discussion of `T1View` for more detail).

### Additional T1 Features

#### EventHub
While a typical backbone event only has an `events: {}` object to listen to jQuery events on that particular view, T1 has a powerful feature to allow different views to communicate with each other. This is implemented using EventHub. It is used similarly to the jQuery `events` object in a Backbone view.

The `EventHub` is, essentially, a wrapper for (jQuery pub/sub) [https://api.jquery.com/jQuery.Callbacks/]. It makes public three functions: `publish`, `subscribe`, and `unsubscribe`.

The basic game of catch works like this. In the sending view, we dispatch the event by calling `T1.EventHub.publish('eventName')`. In the receiving view, we set up the eventHub:

```javascript
eventhubEvents: {
  'eventName': 'callBackFunction'
}
```

When the sender view publishes the `eventHubEvent`, the receiver view fires off the callback function. Like the jQuery events object, functions can also be defined in line, and arguments passed in when the event is published are sent implicitly to the callback functions without needing to be passed in on the receiving end of the eventHubEvents object.

When creating eventHubEvents, there is a particular naming convention that developers try to keep to. While functionally these don't matter, they are useful to help locating where an event is coming from.

If a particular file sends an event, then it would be `'myFile.functionWhereEventPublished: 'callbackFunction'`. If multiple files can send the same type of event, then a more descriptive action is described: `'select:dropdown': 'callBackFunction'`

Finally, if you are overriding the default T1View `unload` function (or are using regular Backbone views), it is strongly recommended that in your view's `unload` you manually unsubscribe from any EventHub events to which your view is subscribed.

_IMPORTANT! `eventHubEvents: {}` can only be used in T1Views. If you are using a vanilla Backbone view, the receiver view must manually subscribe in order to listen for a published event (e.g. `T1.EventHub.subscribe('eventName':'callBack')`_

#### Data Events
In a traditional Backbone app, views can listen to change events on models through the `object.on({'change:property': callback})` pattern. In T1, `dataEvents` adds another jQuery events-like object to the view to handle these events.

`dataEvents` work very similarly to the eventHubEvents, though they are used far less frequently.  Like `eventHubEvents`, `dataEvents: {}` can only be employed on a T1View. In this example, the view will listen for two events on its collection: a `reset` event, and when there is a change to the `status` property on the collection. Each listener then points to a callback.
```javascript
dataEvents: {
  collection: {
    'reset': 'reload',
    'change:status': 'updateActive'
  }
}
```

### Feature Flags
Occasionally, product and/or engineering has a need to restrict access to various sections or features of T1. The feature may be in Beta, or there may be a need to treat users differently based on their permissions and access. There are two different types of permissions to be aware of:
- Permissions:
`COMPASS_BASE/api/v2.0/users/{user_id}/permissions`
This returns the permissions that are set on the user. These are high level permissions generally organized around the type of user (ADMIN, REPORTER, MANAGER)

- Settings:
`COMPASS_BASE/api/v2.0/users/{user_id}/settings`
This is used to persist a User's preferences (housed in the `UserPreferences`) model, but it is also where feature flag data is stored.

In most cases, a feature flag can be checked directly against the name of the flag by requiring the `T1.Permissions` module and using the `check` function: `T1Permissions.check('feature', 'name_of_flag'))`. This will return a true / false value that can be used in view logic.

If you need a flag based on a _permission_ that has to be gotten slightly differently. Since these will be stored on the `User` itself, the permission will be stored as an atrribute. So if we wanted to check a User's role, for example:

```javascript
var user = User.getUser();
var role = user.get('role')
```

`T1.Permissions` wraps a lot of this into functions that can then be by calling the same function. This module also allows UI developers to bypass the permissions by looking to the `env_local.js` files as a way of streamlining permissions in the local environment. Stay tuned as this may be changing.


### Case study in adding a new view: Segments Bulk Create
#### Step 1: Routing
We want the route `COMPASS_BASE/#segments/bulkCreate` so we add the following to the `modes` object contained within `'segments'` found in `router.config.js`:
```javascript
'bulkCreate': [
  {
    'module': 'segments/bulkCreate',
    'viewType': 'bulkCreate',
    'showLoader': true
  }
]
```
#### Step 2: Adding Libraries
Since we'll be using the XLSX and JSZip libraries, we need to add them to `src/js/libs`. Then we must add their paths to `src/js/main.js` so that they can be easily required in our view files and models. Since `xlsx.js` is not designed for AMD (Asynchronous module definition), we have to do some additional configurations so that it will load properly with RequireJS.

We add the following to the that `paths` object:
```javascript
  shim: 'libs/js-xlsx/shim',
  JSZip: 'libs/jszip/jszip',
  xlsx: 'libs/js-xlsx/xlsx',
```
And the following to the `shim` object:
```javascript
xlsx: {
  deps: ['JSZip'],
  exports: 'XLSX'
},
```
This tells our application that `xlsx.js` is dependant on JSZip, and to manually export the defined `XLSX` variable from the module. Fortunately, most of the libraries in T1 have been built for AMD do not require this manual configuration.

#### Step 3: Creating a module
Since we're creating a brand new section of the segments module, we'll create a new module. Were we adding a new view to an existing page, this would not be necessary. Within the `src/js/modules/segments`, we'll add the new folder `bulkCreate`. At minimum that folder will need 3 things:

- A `views` folder containing the default view designated in the routing object property `viewType`: `bulkSegments.js`.
- A `templates` folder which will contain our HTML templates.
- A `main.js` file that will serve as the `T1Module` instance governing the module's configurations. Whenever we introduce a new module, it must have a `main.js` T1Module because that is what the T1 Framework will be looking for in order to navigate the folder and serve up the views to Backbone. 

The folder structure would appear like this.
```
./src/js/modules/segments
└── bulkCreate
   ├── main.js
   ├── templates
   │   └── bulkCreate.html
   └── views
       └── bulkCreate.js
```

Our module is very simple, containing, for now just the one view. So our module will look like this:

```javascript
define([
  'jQuery',
  'T1',
  'T1Module'
], function ($, T1, T1Module) {
  'use strict';

  var instance;

  if (instance === undefined) {
    instance = new T1Module({
      name: 'segments/bulkCreate',
      viewCfgs: {
        'bulkCreate': {}
      },
      defaultView: 'bulkCreate'
    });
  }

  return instance;
});
```

#### Step 4: Sass / CSS
- If we're adding a new module, we may want to separate the CSS files for the new views from existing modules. In the main template for bulk create, we'll wrap the entire template in a div that contains, among others, the class `"bulk-create-wrapper"`. 

- Create `_segments_bulk_create.scss` file in the `src/sass` folder.

- In the new sass file just created, anything you want to apply to that template will go inside 
```scss
.bulk-create-wrapper {}
```

- Include the new sass file in the application. In `src/sass/compass.scss` add the line `@import "segments-bulk-create";`. This tells the application to import the file you just created so its CSS will be loaded when the application compiles.

#### Step 5: That's it...
Now that our routing is set, our libraries are loaded, our is created and module is defined, and our view object is configured within the module, we can then move forward creating HTML templates and our view file (in this case, `bulkCreate.js`). Because T1 is just providing the framework and routing, the view file itself can either be an instance of `T1View` or (like in the rest of the segments module), a boilerplate Backbone view. Should you work in any other module besides segments, you will be required to work using T1Views and T1Layouts. 

### Tips and Tricks
- To source a view responsible for a particular interaction, one of the easiest ways to is identify the element's CSS class or ID. Search globally for that class and you will return hits on either the HTML template file or some views itself. 
- To find live instances / examples of T1 Component files, search by the aliased name of the library class as set in `main.js` (e.g. to find an example of T1.Menu, search for `T1Menu`.) For strand web components, searching by name should return some hits (e.g. 'mm-grid').
- When first working with web components (particularly `<mm-grid>`), the variable rendering may seem confusing. Because `<mm-grid>` employs its internal templating, we must go through two layers of interpolation.
  - In our template: `<mm-grid-item model="{{model}}" scope="{{scope}}" style="font-size: 11px; line-height:18px">`
  - In our view's `serialize`: `model: '{{model}}'`
  - At its most basic, this is to ensure that when `"{{model}}"` is interprated by the templating engine, it is replaced with what is in our `serialize` (or `render`) function: literally `'{{model}}'`. 
- Use your developer tools to set break points (or use `debugger;` or `console.log()` in your code) to be able to step through and into functions, check variables at different times.
- When possible, keep your code changes contained to the modules and views in which you are working. Changes to core T1 files and functions should be done with extreme prejudice and only when absolutely necessary. Dozens of files are potentially dependent on these files. Potentially breaking changes could be introduced that would be very difficult to identify and test. It may become necessary to do at some point, but especially when new to the codebase it is advised to check in with other developers if you find yourself in a T1 library file when implementing a fix.
- Use a [delinter] (https://github.com/MediaMath/compass/blob/develop/docs/code-style-checking.md) and follow the [Compass Style Guide] (https://github.com/MediaMath/compass/wiki/Compass-JavaScript-Style-Guide). 

### Notes for DMP Users
There are a few key differences to be aware of as a DMP developer.
- Follow the instructions in the Compass main repo to make sure your hosts file is edited to point dmp.compass.dev to the proper port. To see your local version of compass, direct your browser to `dmp.compass.dev` instead of just `compass.dev`. In addition, the API endpoints will be governed by what is in `dmp.conf` instead of `compass.conf`. 
- DMP / Segments module does _not_ use T1View or T1Layout. You must still adhere to the module folder system and setup modules properly. T1Models and T1Collections can still integrate into these views, but note that these are regular Backbone views.
- As is noted throughout this document but summarized here, there are a few important pieces to note: 
  - While you can still publish EventHub events, your view will not recognize the EventHub object, so you must subscribe manually to EventHub events in order to listen for them on your views.
  - Since these are not T1Views, follow the normal procedure for creating Backbone views. Segments is currently using Hogan, a variant of Mustache, that can be used in rendering templates.