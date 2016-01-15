## UI development in T1 Overview

#### Purposes
The goal of this document is not to be exhaustive documentation, but to give a high level overview of the framework, libraries, and general best practices. It will focus on routing, views, and the T1 Library functions.

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
The top level of the object `reports` corresponds to the route `COMPASS_BASE/#reports`, telling T1 that the to load up the reporting module and its default view type, which will look for tabs.js in that module. The `modes` are the nested routes below reports. In this case there are 2: `campaigns` and `segments`. In both cases, route objects are built designating where the module is found and its default view. Note that while the top level default view will be found in `reports/views`, the campaigns default view will be found in `reports/campaigns/views/`. What does `bindings` do? I'm not really sure.

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

#### templates
These are the HTML templates used by the shared libraries. If a T1 library uses HTML, like T1Menu for example, its HTML will most likely be found in this folder.

#### modules
See below :smile:

### Modules
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
T1View is a simple extension of Backbone's `View` object. A T1View can be used either as a standalone view, or it can be used as a wrapper to manage a lot of views contained within it. This is where T1Layout comes into play. T1Layout is essentially a view-loadng wrapper. 

In Segments, which uses vanilla Backbone, we would typically manually create a new instance of a view class and assign it to an element. Using T1Layout, we can simple create an instance of T1Layout and load.

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
The T1View default `render()` function will be looking for a `serialize()` function in your view. In Backbone views, you typically need to define variables for your templates at the moment the template is either compiled or rendered to the DOM. `serialize()` is T1's way of handling this. A render function using Hogan might look like this:
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

Using T1View, we need only to define 'serialize()' on our view to return an object containing all our of our defined variables, and T1View's render function will render our templates with values inserted.

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

