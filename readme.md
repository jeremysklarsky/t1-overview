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
The top level of the object `reports` corresponds to the route `COMPASS_BASE/#reports`, telling T1 that the to load up the reporting module and its default view type, which will look for tabs.js in that module. The `modes` are the nested routes below reports. In this case there are 2: `campaigns` and `segments`. In both cases, route objects are built designating where the module is found and its default view. Note that while the top level default view will be found in `reports/views`, the campaigns default view will be found in `reports/campaigns/views/`. 


#### Revelant Folders Overview
- `/src/components/`: In addition to Strand, UI developers have built some of their own custom web components that will not be open-sourced as part of strand.
- `/src/images/`: Contains most of the image assets that are used throughout compass. Many of these are being replaced by `<mm-icon>`. See [Strand's mm-icon documentation] (http://wc-docs.mediamath.com/v0.6.3/mm-icon.html) for more information.
- `/src/js/`: Contains all the JS code, HTML templates, and settings files for the main Backbone application. We'll explore this folder more in depth later.
- `/src/js/sass/`: All styling information. More info on [SASS] (http://sass-lang.com/)

### The Backbone Application / Making sense of `src`
#### models and collections
Primary home for models and collections widely used throughout the application. These are almost exclusively classes of `T1Model` or `T1Collection`.

### libs 
Contains the source code for all 3rd party javascript libraries, including T1 itself. 
In order to add a library to the codebase: 
- create a folder
- add the file
- add the path to the require configs in `src/js/main.js`. 

### templates
These are the HTML templates used by the shared libraries. If a T1 library uses HTML, like T1Menu for example, its HTML will most likely be found in this folder.

