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
#### Revelant Folders Overview
- `/src/components/`: In addition to Strand, UI developers have built some of their own custom web components that will not be open-sourced as part of strand.
- `/src/images/`: Contains most of the image assets that are used throughout compass. Many of these are being replaced by `<mm-icon>`. See [Strand's mm-icon documentation] (http://wc-docs.mediamath.com/v0.6.3/mm-icon.html) for more information.
- `/src/js/`: Contains all the JS code, HTML templates, and settings files for the main Backbone application. We'll explore this folder more in depth later.
- `/src/js/sass/`: All styling information. More info on [SASS] (http://sass-lang.com/)

#### The Backbone Application / Making sense of `src`