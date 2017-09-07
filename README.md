Initializing npm
Whenever we start a new project we need to run the following command from the top level of our project directory to begin using npm:

$ npm init

Installing npm Packages
Now, let's install our first package: gulp. Gulp is a JavaScript package that runs development tasks for us.

$ npm install gulp --save-dev

More Packages: Browserify
Let's download another package with npm! This one is called browserify. Again, in the command line we want to make sure we are in the top level of our project directory, and then enter the following command:

$ npm install browserify --save-dev

Here is an example .gitignore file. This should go in the top level of your project folder.
.gitignore
node_modules/
.DS_Store

Installing Gulp
We already installed gulp with npm inside of our project folder using this command:

$ npm install gulp --save-dev

However, we will also need to install gulp globally to use it in the terminal. We do this by using a new flag: -g for "global".

$ npm install gulp -g

First, we'll need to get a new package called vinyl-source-stream using npm

npm install vinyl-source-stream --save-dev

We'll also need a new package called gulp-concat. We'll install it with npm now:

$ npm install gulp-concat --save-dev

Let's run the new version of our browserify task:

$ gulp jsBrowserify

Minifying with gulp-uglify
We can add minification to the end of our JavaScript chain of tasks so that after we have concatenated our files 
and browserified the result, we can then minify it. We'll need another package from npm:

$ npm install gulp-uglify --save-dev

Managing Environmental Variables with gulp-util
First, as usual, we need a new package from npm. It's called gulp-util. Let's install it as a development dependency:

$ npm install gulp-util --save-dev

Clean Tasks
The next thing that we're going to need is a task to clean up our environment before we make a build. 
We want to make sure that we are using up-to-date versions of our files every time that we build. 
To this end, we need a way to delete files using gulp. Surprise! There's a package for that, called del, which stands for delete.
Let's install it:

$ npm install del --save-dev

Linting with JSHint
JSHint is available through npm, so let's start off by installing it as a development dependency. 
We'll need the jshint package itself, and then we'll need the gulp-jshint package to allow us to write a gulp task to automatically check
our code using the linter.

We are pulling in all the JavaScript files in our js folder, running jshint on them, then using the default jshint reporter to show 
us our errors. To try it out, just run the command:

$ gulp jshint

Starting a new project and adding dependencies
Whenever we start a new project, we need to run the following commands to create our manifest files package.json and bower.json:

$ npm init
$ bower init

Cloning a project
When we clone a project to continue working on it, we run these commands:

$ npm install
$ bower install
$ npm install jshint --save-dev
$ npm install gulp-jshint --save-dev

Installing Bower
Let's get started. Bower is itself a Node module so we can install it easily with npm. It is recommended that we install it globally.

$ npm install bower -g

Installing Front-End Dependencies via Bower
jQuery
Next, let's install jQuery and use that for our pingpong app instead of relying on the Google CDN in our script tag. 
We can install jQuery with this simple command. It has the same structure as an npm install command, but we are using the --save 
flag instead of our usual --save-dev flag because we do want to use jQuery in the final production build.

$ bower install jquery --save

The syntax similarities don't stop there. When we clone a project that uses npm to manage its dependencies, 
we run npm install to put the packages on our local machine. We do the same thing to get our Bower dependencies:

$ bower install

Bootstrap
Next, let's add Bootstrap the same way:

bower install bootstrap --save

Moment.js
Let's add another JavaScript dependency. We're going to add Moment.js so that we can easily work with dates and times in various formats 
in our apps. First, we'll install it with Bower:

$ bower install moment --save

Let's use gulp to streamline this rather than adding a ton of <script> and <link> tags to our HTML file and being responsible for 
adding a new one every time we add another Bower dependency. We're going to use npm to install another gulp package called bower-files:

$ npm install bower-files --save-dev

BrowserSync
We're going to use a package called BrowserSync to implement our development server with live reloading. First, let's download it with NPM as always:

$ npm install browser-sync --save-dev

Jasmine
We’ll take a different approach. First, we’ll use npm to install Jasmine. The Jasmine Node module comes with code that allows us to run 
our tests in the terminal. Once we’ve familiarized ourselves with Jasmine, we’ll learn to use a test-runner called Karma to run our tests.
As always, we should create a package.json file by running npm init. Now we can install the Node module for Jasmine:

$ npm install jasmine --save-dev

Next, we'll initialize Jasmine:

$ ./node_modules/.bin/jasmine init

Last, we’ll make a small update in our package.json file. Open it with atom and make this change:
package.json
...
"scripts": {
  "test": "jasmine"
}

$ npm test

gulpfile.js

var gulp = require('gulp');
var concat = require('gulp-concat');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var uglify = require('gulp-uglify');
var utilities = require('gulp-util');
var del = require('del');
var jshint = require('gulp-jshint');
var lib = require('bower-files')({
  "overrides":{
    "bootstrap" : {
      "main": [
        "less/bootstrap.less",
        "dist/css/bootstrap.css",
        "dist/js/bootstrap.js"
      ]
    }
  }
});
var browserSync = require('browser-sync').create();

var buildProduction = utilities.env.production;

gulp.task('concatInterface', function(){
  return gulp.src(['./js/*-interface.js'])
    .pipe(concat('allConcat.js'))
    .pipe(gulp.dest('./tmp'));
});

gulp.task('jsBrowserify', ['concatInterface'], function() {
  return browserify({ entries: ['./tmp/allConcat.js'] })
    .bundle()
    .pipe(source('app.js'))
    .pipe(gulp.dest('./build/js'));
});

gulp.task("minifyScripts", ["jsBrowserify"], function(){
  return gulp.src("./build/js/app.js")
    .pipe(uglify())
    .pipe(gulp.dest("./build/js"));
});

gulp.task("build", ['clean'], function(){
  if (buildProduction) {
    gulp.start('minifyScripts');
  } else {
    gulp.start('jsBrowserify');
  }
  gulp.start('bower');
});

gulp.task("clean", function(){
  return del(['build', 'tmp']);
});

gulp.task('jshint', function(){
  return gulp.src(['js/*.js'])
  .pipe(jshint())
  .pipe(jshint.reporter('default'));
});

gulp.task('bowerJS', function() {
  return gulp.src(lib.ext('js').files)
    .pipe(concat('vendor.min.js'))
    .pipe(uglify())
    .pipe(gulp.dest('./build/js'));
});

gulp.task('bowerCSS', function(){
  return gulp.src(lib.ext('css').files)
    .pipe(concat('vendor.css'))
    .pipe(gulp.dest('./build/css'));
});

gulp.task('bower', ['bowerJS', 'bowerCSS']);

gulp.task('serve', function(){
  browserSync.init({
    server: {
      baseDir: "./",
      index: "index.html"
    }
  });

  gulp.watch(['js/*.js'], ['jsBuild']);
  gulp.watch(['bower.json'], ['bowerBuild']);
});

gulp.task('jsBuild', ['jsBrowserify', 'jshint'], function(){
  browserSync.reload();
});

gulp.task('bowerBuild', ['bower'], function(){
  browserSync.reload();
});


Installing Karma

With that in mind, let’s set up Karma. Take the time to input each of these commands the first time you go through this lesson. 
You can reuse your completed package.json file for future projects. (Don’t forget to run $ npm install.)
First, let’s install Karma itself.

$ npm install karma --save-dev

Next, we’ll need to add some packages so that Karma and Jasmine can work together.

$ npm install karma-jasmine jasmine-core --save-dev
We also need to specify which browser (or browsers) we want Karma to launch. We’ll install the Chrome launcher. 

$ npm install karma-chrome-launcher --save-dev

$ npm install karma-cli --save-dev

In addition, we need Karma to browserify files, since the browser can't understand require statements and our code is separated into different modules:

$ npm install karma-browserify --save-dev

Karma doesn’t understand jQuery on its own, so we need a plugin for that, too:

$ npm install karma-jquery --save-dev

Last but not least, we’ll want to make our testing report easy on the eye:

$ npm install karma-jasmine-html-reporter --save-dev

It’s time to initialize Karma by running 
$ karma init 
in the root directory of your project.

There’s one last thing to do. $ npm test is still pointing at Jasmine. 
We need to update package.json one more time:

package.json
...
  "scripts": {
    "test": "karma start karma.conf.js"
  },
…

Now, when we run $ npm test, Karma will launch a Chrome browser. 

TranspilationTranspilation is the process of compiling code from one language to another (or in this case, one version to another).
To do this, wWe’ll use Babel, a popular JavaScript transpiler. There are many different ways to set up Babel; 
we’ll use an npm package called babelify.
We’ll also need to install the es2015 presets so Babel knows how to transpile the code. We’ll install both with one command:

$ npm installbabelify babel-preset-es2015 --save-dev

Next we need to update our Gulpfile so our code is transpiled when we build. 
We’ll add babelify as a dependency and then update our jsBrowserify task.

gulpfile.js
...
var babelify = require("babelify");
...

gulp.task('jsBrowserify', ['concatInterface'], function() {
  return browserify({ entries: ['./tmp/allConcat.js']})
    .transform(babelify.configure({
      presets: ["es2015"]
    }))
    .bundle()
    .pipe(source('app.js'))
    .pipe(gulp.dest('./build/js'))
});

Karma will browserify and test our code separately from Gulp so we also need to update our karma.conf.js file as well. 
Specifically, we’ll add an option for browserify. Here, we’ll babelify our code.

Karma.conf.js

module.exports = function(config) {
  config.set({
    ...
    preprocessors: {
      ...
    },
    plugins: [
      ...
    ],
    browserify: {
      debug: true,
      transform: [ [ 'babelify', {presets: ["es2015"]} ] ]
    },
    ...
  })
}

let’s also configure jshint for ES6 as well. If we don’t, we’ll get warnings and errors when we use ES6 syntax.
Create a .jshintrc file in the root directory.

.jshintrc

{ "esversion":6 }

Now when we run $ gulp jshint, our linter won’t complain about ES6 syntax.
