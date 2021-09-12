# Welcome to AppTemplate!


## About
AppTemplate is a starting point for a web app based on modern native JavaScript.  It is based on years of research on how to structure applications to be simple to begin and understand, yet grow as needed while maintaining separation of concerns, reusability, and other key application design concerns.

AppTemplate provides the 'structure' of a framework without forcing you into any code patterns or runtimes or build steps.  There are just a few lines of code to get you started in one way of structuring a web app.  It is designed just like those 'templates' in your favorite IDE, which give you somewhere to start.  In other words, instead of giving you a black box you must build against, you get a basic structure to modify and extend to suit your needs.


## Why not a 'framework'

Writing modern web applications has been a challenging, constantly evolving domain of software engineering.  Due to the core JavaScript libraries and the language itself being "different" depending upon which browser was being used, many companies turned to frameworks which had their own flavor of html and their own library of built in methods for web applications.

The benefit of that model was 3-fold:
 - They could ensure a unified html/javascript platform which was the same across browsers
 - They could control the evolution, changes, by using build-steps to generate compatible code
 - They could add their own ideas to html/JavaScript (i.e. features they 'wished' html/js had)

The negatives of using a classic framework are present as well:
- ~~redacted to avoid flame war~~
- ~~redacted to avoid flame war~~
- ~~redacted to avoid flame war~~

I said the benefit 'was' because as of the later part of 2020, Microsoft moving to a Chromium based browser (Edge) has changed the game entirely.  All the main browsers, Safari, FireFox, Chrome, Edge have adopted modern JavaScript libraries and features.  

There is no more need to depend on a whole platform and library of code that is just 'abstracting the browser' and making a 'different version of html/js/css'.  The primary benefit of frameworks is a historical issue.  Frameworks can still help because they come with 'boilerplate code' that organizes your work in a modular way, but that is a heavy price to pay for building your app on something you have no control over, and accepting a whole bunch of code that you don't need but is there because 'other apps' need it and have got it added to the framework.

It's now possible to build cross-browser apps in clean, modular, modern, native code, with no buildstep, and your code won't be filled with cross-browser issues, and it will work as is for decades to come, and you won't miss out on all the modern language features and libraries built into browsers.  You can include 3rd party libraries as needed for special features, but you don't need to build your app 'on top of' a big framework/toolchain.


## File Structure
```
/data/                // all db/json data that exists 'between visits'
/data/data.js         // module for accessing the data. keeps token, session state, config
/data/user/           // data related to the logged in user. specific to this one user
/data/site/           // data related to the site (eg portal or other). same for everyone
/data/text/           // text strings for templates

/shell/               // global concerns common to kc apps
/shell/jsUtil/        // a few methods javascript 'should' have (not app-specific)
/shell/dialogManager/ // manage stack of dialogs. (content template is injected)
/shell/pageManager/   // handle switching between pages, sending events on show/hide
/shell/sessionManger/ // manage log in/out, token refresh, auth code (initialize in app.js)

                      // app initializes the shell and other long-lived parts
app.js                // included in index.html, loads/configures shell and parts
config.js             // all the default config
configOverride.js     // per environment config overrides (.gitignored but expected)
app.css               // global structural css. possibly empty. maybe fonts

theme/                // themes provide optional css variables for overrides
theme/theme.js        // handle swapping themes and setting css values/themes. 
theme/sky/
theme/sky/theme.css   // set variables for use in other css files

parts/                // parts of app which 'stay alive' on multiple pages (setup in app.js)
parts/parts.js        // parts are initialized here, and can be accessed by pages if needed
parts/header/         // header can have 'features' and change look between pages
parts/footer/         // footer is another part that is 'app-level' not page level
parts/sidebar/        // some apps have a persistent sidebar between pages
parts/helpchat/       // app can manage various global components that are not page concerns

pages/                // shown/hide, and receive events from pageManager, build themselves
pages/index/          // pages can depend on parts/shell/data/blocks/tags
pages/a/              // pages can replace their content based on urls, or show/hide parts 
pages/b/              // rarely used or very heavy pages can load 'politely'
pages/c/              // pages share data (/data) but otherwise should not touch other pages

blocks/               // can be instantiated by any page, can use tags, not other blocks
blocks/requestAccess/ // blocks can be tested on their own and don't know about other blocks
blocks/ourClients/    // blocks can have multiple instances, be configured in js, use data/

tags/                 // html custom-elements defined for use in html templates
tags/tags.js          // tags.js defines all the avialble tags. loaded in app.js
tags/fancy-input/     // tags can be tested by themselves. they know nothing outside
tags/combo-box/       // may use shadow dom to mimic built in html tags

/vendors/             // dependencies not maintained by app. is yuck.
```


