# Welcome to AppTemplate!


## About
AppTemplate is a starting point for a web app based on modern native JavaScript.  It is based on years of research on how to structure applications to be simple to begin and understand, yet grow as needed while maintaining separation of concerns, reusability, and other key application design concerns.

AppTemplate provides the 'structure' of a framework without forcing you into any code patterns or runtimes or build steps.  There are just a few lines of code to get you started in one way of structuring a web app.  It is designed just like those 'templates' in your favorite IDE, which give you somewhere to start.  In other words, instead of giving you a black box you must build against, you get a basic structure to modify and extend to suit your needs.

The reason this works is that modern browsers have now built in the libraries and languages features that JavaScript was missing, making modern application development be possible without resorting to 3rd party frameworks.  In the past, not using a framework or some kind of build step meant you had to give up modern concepts like classes and modules and async/await, getters/setters, query selectors, etc.  That is no longer the case.  However, we still need to carefully structure our apps.

## AppTemplate Logical Structure
The key to AppTemplate is an application structure, which if carefully adhered to, will go a long way toward ensuring clean, maintainable code.  

![layers-blocks](/app-structure-blocks.jpg "AppTemplate structure")


### Shell, data, and tags
The goal of good app architecture is modularity and testability.  
 - The Shell is a collection of 'ready to use tools' that make wiring together a new application very easy. The shell includes some basic utility methods (js extensions), and several classes to help with managing dialogs, sessions, and page switching (SPA routing).
 - The data component is crucial to the success of the app. Many apps make the unmaintainable mistake of not completely separating the data from the rest of the application. The data represents all the objects/arrays/strings that 'perist' when the app is not running. So it is the single source for all 'external' data (data not defined within the app modules). The data can be on an API, a CDN, or a File or any other kind of service.  Note this is not 'html' or templates/images.  Think of it as a set of async methods to get/set json objects.  A key point here is that it can be fully tested on it's own, without the rest of the app
 - tags are [custom-element definitions](https://googlechromelabs.github.io/howto-components/howto-tabs/#demo) . in html5 we can define custom tags like `<circle-dropdown>`.  On of the great things that framworks provide is this 'custom tag' ability. Here though, we define them in plain js.  Then the app can use them in any of the templates it uses.  A key point is that tags don't know about eachother, and tags can be tested along, and tags cannot access anything 'outside' like data or app (though potentially some 'property' could accept some data source)

So with a well defined, separated set of 'wiring tools' (shell), and a pure data single source for external data state that can be tested apart from the app, and a rich set of custom tags, the work of building the app is off to a good start.

### App
The job of the App is to be the entry point. It loads the settings and initializes custom html tags, data api, any long-lived parts like header, footer, and page manager.  It is intentionally limited in functionality as 'piling everying into main.js global space' is the most critical danger to app maintainability.

Note especially that App is not forced to use anything in Shell.  Things in Shell are utilities which may be used as needed. So shell components can be tested on their own.

There is no default 'app class' to inherit from.  This is very much on purpose again, to avoid global bloat.  Pages generally do not access app.  It is simply the initializer.

### Parts / Page Manager
Parts refer to 'parts of the app'.  They are global instances of some part of the app that stays alive throughout the visit, and doesn't get reloaded every time page navigation happens.  Typical example is header and footer, and sidebar, but there can be other things like a 'smart assistant' or 'help chat' or some 'gamification manager' etc.  These are global objects instantiated by the app, and may have permanent visibility, or may just show on some pages, or may not show at all.

The Page Manager is a special 'part' that stays alive throughout the app visit and handles showing/hiding the various pages.  The page manager can be used to initiate navigation to another url/page, and it handles detecting url changes and activating the corresponding page.  Activating a page involves, hiding other pages, showing the container of the new page, and sending an event to the page so it knows it got visibililty (including 404 page).

The Page Manager is provided out of the box in the Shell, for easy page handling.  However other potential strategies or libraries could be used while still following the same general app structure.

A good way to think of the parts/page manager is:


| header part  |
|--------------|


| page manager |
|--------------|


| footer part  |
|--------------|

Crucially, the parts are all independently testable.  The parts are should be able to be initialized on a blank test page, and they expose methods to manipulate them, as well as listen for events.  They should handle their own style (at least default style, which can be themed/overridden but which is fully functional)


### Pages
Each page is a module with a class the knows how to build the page, within the element that is passed to the constructor.  The minimal valid page is simply:

MyPage.js
```js
export default class MyPage {}
```

Of course, navigating to the above page would result in a blank page because pageManger creates the root element that is displayed, and gives a default minimum size.  A hello world page would be like this:

HelloWorld.js
```js
export default class HelloWorldPage
{
  constructor(page) {
    page.insertAdjacentHTML('beforeend', '<p>Hello World</p>');
  }
}
```

The above results in seeing Hello World when navigating to the HelloWorld page.  Note that navigation is configured in the pageManager.  So page manager is configured to detect a certain url, and activate the HelloWorld class (page).  The page itself can then further inspect the url to show 'subpages' or load different content based on url

Note that the constructor only runs once.  Navigating to other pages and back does not run the constructor again.  Many apps however want to know when the user arrives at the page to load/update things, and they may want to do some action when the user leaves, so that the page will be 'ready' for next visit (ie unload something).

For this purpose, the pageManager sends a custom event 'appshow' on the page when it is showing.  This is much better than a registered callback because the page is not obligated to use it or provide it, and the page manager does not have to be calling methods on the page class (keeps it decoupled).  This allows us to avoid a 'page class' that we would otherwise consider building to 'inherit methods' that the page manager calls (bad coupling, and another place to endlessly stuff logic)

A minimal example of a page would be:

Minimal.js
```js
export default class MinimalPage 
{
  constructor(page) {
    page.insertAdjacentHTML('beforeend', '<p>Hello World</p>');
    page.addEventListener('appshow', this.onAppShow.bind(page));
  }
  
  onAppShow() {
    this.insertAdjacentHTML('beforeend', '<div>Welcome Back</div>');
  }
}
```


### Blocks
In general, a page should try to be self-contained, and any sub-parts can held in the page folder and included dynamically or by imports etc.  However, there are likely to be some blocks of html which appear on different pages, and so we want a way to be able to reuse items in different pages.

Blocks are like a higher level of 'tags'.  Think of them as a rendered 'template' of some subscription of the page, where that template is rendered on several different pages.

Blocks, like tags, should not depend on eachother, and should also be testable by being able to be instantiated by themselves on a test page.  The difference with blocks is that they are not implemented as 'tags', but are instantiated and inserted into the page.  Further, the blocks are allowed to access the data layer, to get data for themselves (or the data can be passed to them).  So each block has a folder with at least a JavaScript module.  Any page that wishes to instantiate the block will import that module, and then using e.g. new SomeBlock(targetDiv) will instantiate 1 or more instances of it, and then be able to set properties and call methods on the class to affect the object.

A final note is that Blocks are specifically for reuse.  A 'block' of html should only be promoted to a Block if it is used on more than one page.  Otherwise, it's best to keep it as an implementation detail of a page.  So then the blocks folder will only have those blocks which are intended to appear on multiple different pages.  Individual pages could still make blocks as modules within their own page folder, and only move them to the blocks folder if they end up being used on other pages.

### Summary of dependencies of Tags, Blocks, and Pages
Again, Pages don't know about other Pages, Blocks don't know about other Blocks, Tags don't know about other tags.  Blocks can use Tags.  Pages can use Blocks and Tags.  A Tag is independently testable.  A Block is independently testable provide it (optionally) receives a reference to the main data object (or mockup).  A Part is independently testable, and a Page is independently testable, again assuming it has access to any Tag, Block, or Part it needs to interact with, and to the main. data object.

### Template pattern
Templates, meaning snippets of html that need to be merged with data are not official concepts in AppTemplate.  However, modern javascript provides us a convenient way to get very powerful templates.  This method can be used with blocks, tags, or pages as needed.  Templates can be defined inline, or in separate modules.





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




