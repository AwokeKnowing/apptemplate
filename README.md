# Welcome to AppTemplate!


## About
AppTemplate is a starting point for a web app based on modern native JavaScript.  It is based on years of research on how to structure applications to be simple to begin and understand, yet grow as needed while maintaining separation of concerns, reusability, and other key application design concerns.

AppTemplate provides the 'structure' of a framework without forcing you into any code patterns or runtimes or build steps.  There are just a few lines of code to get you started in one way of structuring a web app.  It is designed just like those 'templates' in your favorite IDE, which give you somewhere to start.  In other words, instead of giving you a black box you must build against, you get a basic structure to modify and extend to suit your needs.

The reason this works is that modern browsers have now built in the libraries and languages features that JavaScript was missing, making modern application development be possible without resorting to 3rd party frameworks.  In the past, not using a framework or some kind of build step meant you had to give up modern concepts like classes and modules and async/await, getters/setters, query selectors, etc.  That is no longer the case.  However, we still need to carefully structure our apps.

## AppTemplate Logical Structure
The key to AppTemplate is an application structure, which if carefully adhered to, will go a long way toward ensuring clean, maintainable code.  



## AppTemplate File Structure
In order to implement the application structure, it's important to know what the folders are for, so that all new code is added in the right place, and the dependency guarantees are maintained, to keep the parts testable.

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



# FAQ

## Why not a 'framework'?

Writing modern web applications has been a challenging, constantly evolving domain of software engineering.  Due to the core JavaScript libraries and the language itself being "different" depending upon which browser was being used, many companies turned to frameworks which had their own flavor of html and their own library of built in methods for web applications.

The benefit of that model was 3-fold:
 - They could ensure a unified html/javascript platform which was the same across browsers
 - They could control the evolution, changes, by using build-steps to generate compatible code
 - They could add their own ideas to html/JavaScript (i.e. features they 'wished' html/js had)

The negatives of using a classic framework are present as well:
- ~~redacted to avoid flame war~~
- ~~redacted to avoid flame war~~
- ~~redacted to avoid flame war~~

We say the benefit 'was' because as of the later part of 2020, Microsoft moving to a Chromium based browser (Edge) has changed the game entirely.  All the main browsers, Safari, FireFox, Chrome, Edge have adopted modern JavaScript libraries and features.  

There is no more need to depend on a whole platform and library of code that is just 'abstracting the browser' and making a 'different version of html/js/css'.  The primary benefit of frameworks is a historical issue.  Frameworks can still help because they come with 'boilerplate code' that organizes your work in a modular way, but that is a heavy price to pay for building your app on something you have no control over, and accepting a whole bunch of code that you don't need but is there because 'other apps' need it and have got it added to the framework, and accepting that your html/js is a 'flavor' of html/js that is getting rewritten by some 'engine' into 'real html/js'.

It's now possible to build cross-browser apps in clean, modular, modern, native code, with no buildstep, and your code won't be filled with cross-browser issues, and it will work as is for decades to come, and you won't miss out on all the modern language features and libraries built into browsers.  You can include 3rd party libraries as needed for special features, but you don't need to build your app 'on top of' a big framework/toolchain.

## Doesn't 'vanilla' JavaScript lead to low-quality code?
Writing your own code without a framework has gotten bad publicity, mostly because people who are writing fairly small applications knew they definitely did not 'need' a framework, and didn't want the bulk, but they failed to use good architecture. Often they made a quick few-day prototype, and just added more and more code without structure.

Here is classic vanilla folder organization, which has major flaws:

```
/main.js      // a large file which has tricky callback/promise 'loading code' and many 'shared' functions
/js/          // some topic-specific javascript files, all loaded, and referencing eachother
/js/pages/    // some possibly lazy-loaded code that runs for each page
/js/vendor/   // a bunch of 3rd party code, which may or may not be modified for the app
/css/         // typically one giant css file with stuff for all the different html to use
/img/         // all the images
/templates/   // a bunch of html snippets included by any of the files in js/ 
/node_modules // at some point, node is installed for some helpers to do some task.
```

Typically, there's just a folder per file type, and everything else is free-for-all.  Any file in js can reference methods in other files.  Sometimes, half-baked lazy loading schemes are implemented to load js per page, but whenevern a method needs to be used on more than one page, it gets put in something like a main.js.

Fundamentally, the flaw is that there is no modularity.  Pieces cannot be tested on their own typically, because all the code/css has self-references and cannot run on it's own.  The lack of modularity also means changes in one place have unpredictable effects in other places.  The app will be easy to write at first, but it will become very hard to maintain and extend as it grows.

## Why is Framework code usually 'nicer' than 'vannilla' projects?
We can learn a lot from the various frameworks that have come out.  But what are the real reasons it seems cleaner?
 - application folder stucture
 - modularity concepts (classes, dependency management, modules, plugins)
 - clear way to separate and combine html/js/data
 - 'framework extensions' that already do the hard part
 - pre-made ui components

What's not apparent at first site is that the real reason is because Software Engineers who have to take on very large complex projects typically are very experienced, and know how to write clean, well-structured code.  A framework represents talented Engineers figuring out an application structure that will support the maintainability and testability that they need for their large project.  Further, since they have built something that other devs must use, typically they need to show a lot of examples and usecases to communicate the structure, and they need to solve a lot of classic problems (how to get data, do templates/components).  And futher, they tend to be on teams of other highly talented engineers who take the structure and create a great app, hopefully open source, and so other people see this structure, and nice clean code, and they copy it.  

There is not much inherent in the framework that 'ensures' nice code.  Rather, frameworks are used by experienced engineers who are building any size app because they know they will need a good structure.  If you give the experienced devs no framework, they will still create some kind of good structure for their application.  

The lesson is that nice code involves clear, modular folder stuctures, carefully separated concerns, and modular components that can be tested and which have scopes that don't intertwine with many other parts of the application.

Why not just use it then? Again, the reason is that most of the code in the framework is not really solving your specific problems, and it's forcing you into an alternate world of 'fake' html/js (which doesn't 'work' without some kind of interpretation, maintained by someone else.)  We can simply borrow the good (application structure) and transfer that to our projects.

Now that browser vendors have agreed on built-in ways to accomplish many of the technical concerns of frameworks, the main things we need to get clean code without a framework is solid folder structure, sample code in that structure, some key (but very small) technical concerns (page switching), and documentation of the sample code to clarify how the code can be extended. 

Bottom line, we need a documented 'template' and code patterns that we can start and extend.




