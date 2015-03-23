[previousePost]: http://peter.grman.at/typescript-development-in-practice/

Last month I wrote about [using TypeScript efficiently with NodeJS and Gulp][previousePost]. But after an AirPair session with [Jonathon Kresner](https://twitter.com/hackerpreneur) I realized, how little I actually have said. [Last month's post][previousePost] is still useful as a general introduction of things you can consider and you should [check it out][previousePost]. In the following I'll go more in the details of how I'm specifically using [TypeScript](http://www.typescriptlang.org/) with [Gulp](http://gulpjs.com/), [Sublime Text 3](http://www.sublimetext.com/3) and [TSD (TypeScript Definition manager for DefinitelyTyped)](http://definitelytyped.org/tsd/), to give you a clear picture how you can start development easily. You can find the session with Jonathon [on YouTube](https://www.youtube.com/watch?v=-4VYhrea904) and the code which was produced [on GitHub](https://github.com/airpair/experts.airpair.com/pull/21).

## 0. Introduction

For those of you, not knowing TypeScript, it's a statically typed superset of JavaScript. This means, it brings all the advantages of a statically typed language, but valid JavaScript is still valid in TypeScript. To help with the transition from JavaScript to TypeScript, developers of JavaScript libraries can create definition files (`*.d.ts`) which could be compared to C-header files. They contain all the functions and variables accessible in a library and annotate them with types. More to this in **3. Using TypeScript's Definition Files**.

## 1. All the Apps you Need

- [Sublime Text 3](http://www.sublimetext.com/3): Of course you're free to use any other editor or IDE. Anything from Visual Studio and WebStorm to Atom or Vim, but for this post, let's look specifically at Sublime Text 3.
- [T3S](https://github.com/Railk/T3S): Now there are plenty of plug-ins for Sublime Text, which you can install with [Package Control](https://packagecontrol.io/) but one you can't, T3S. You should clone it from GitHub into your Sublime Text's package directory. You can find it via `Preferences > Browse Packages…` menu. Everything is nicely explained in their [README](https://github.com/Railk/T3S/blob/master/README.md).
    - The catch: There are multiple branches. There is `ST2` for Sublime Text 2, and the `master` branch should be for Sublime Text **3**, BUT - in my experience it didn't work with the latest version of Sublime Text 3. Instead, I went for the `dev` branch witch is couple commits behind as well as couple commits ahead. You can get it by running `git fetch && git checkout dev` in the directory where you've checked out T3S.
- NodeJS, npm
- [TypeScript](http://www.typescriptlang.org/): `npm install -g typescript` However, you run the TypeScript compiler with the `tsc` command.
- [Gulp](http://gulpjs.com/): `npm install -g gulp` For build and task automation.
- [TSD](http://definitelytyped.org/tsd/): `npm install -g tsd` To easier manage TypeScript's dependencies.

If you plan on writing a Single Page Application with AngularJS, you can go directly with a great Yeoman generator for that: [gulp-angular](https://github.com/Swiip/generator-gulp-angular). Install it with `npm install -g yeoman generator-gulp-angular`.

*(optionally run the `npm install ...` commands with `sudo` if that is necessary for you)*

Of course, there are other tools like [Bower](http://bower.io/) for instance, you might need or want, but I won't go into details there.

## 2. Set up your folder and environment

So this is where things can get very easy or very tricky. If you want to do a Single Page Application with AngularJS, you just need to run the [Yeoman generator](https://github.com/Swiip/generator-gulp-angular), I've mentioned above. Start Yeoman (`yo`) and follow the instruction on the screen. Afterwards jump to the next sub-point (**Configuring T3S in Sublime Text 3**).

If you're still reading this, you either might be a very curious person, or run TypeScript on the server with NodeJS. I want to make a generator for these upcoming steps eventually, but so far you'll have to do it manually.

- Create a directory, initialize it with `npm init` (and optionally `git init`)
- Install `express` and all the other packages you want to use with `npm install express --save` (I'll only talk about express later)
- Install `gulp`, `gulp-typescript` and `gulp-nodemon` for development with `npm install gulp gulp-typescript gulp-nodemon --save-dev` (you might want or need more, again I'll only mention these)
- Initialize tsd with `tsd init` and install the required type definitions (`*.d.ts`) with `tsd query express node --action install --save`
- Create the folder `server` where all your code will be. Add an empty `index.ts` inside.

### Configuring T3S in Sublime Text 3

Now you should have some TypeScript files and you can open them in Sublime Text. The T3S package will notice that you didn't configure it yet, so it will prompt to do so. Choose the option to create a `.sublimets` which only allows for one root file. This is a big disadvantage, yes, but the other option did never work for me and, eventually, I gave up trying. The root TypeScript file will be your `index.ts`, `app.ts`, `main.ts` or anything similar, which is the main entry point for your application and will have all the references. Also write `yes` in the question to add more options. This will allow you to change the default `ES3` target to `ES5` (as far as I know, these are the only 2 valid options so far).

## 3. Using TypeScript's Definition Files (`*.d.ts`)

In order for TypeScript to understand JavaScript better, you can give it information via definition files. The easiest way to handle and maintain them is by using `tsd`.

- `tsd query angular*` lets you search for files related to AngularJS and its modules.
- `tsd query angular --action install --save` lets you install the specific AngularJS definition file, while keeping the information additionally in `tsd.json`, to be able to update all your definition files at once later.

You can see the `typings` folder with `tsd.d.ts` inside. This is the only file you need to reference, in order to get access to all the underlying definition files, which you have installed. If you are creating a server for instance, you can reference the file by adding the following line in the beginning of your `index.ts`:

    /// <reference path="../typings/tsd.d.ts" />

## 4. Developing in TypeScript

If you used the Yeoman-Generator for the Single Page Application, you have some code you can look at and play with. In this case please proceed to **5. Using Gulp for Compilation**. 

If you didn't use the generator, you have an almost empty `index.ts`, let me help you with that. Let's write the MEAN-equivalent of a *Hello World* application. I made it on purpose slightly larger than necessary to be able to see, how TypeScript handles type annotations. We start with:

    import express = require('express');

    var app = express();

    app.get('/', (req, res) => {
        res.send('Hello TypeScript')
    });

Here we create a new `express` object and add a single method to `GET /`. Notice we write `import express` instead of `var express`. This way we tell TypeScript to find the necessary definitions. Otherwise, it would be treating the `express` variable as an `any` object. This is an object, TypeScript has no type information about. In case you didn't just copy & paste it, you should have also noticed the rich and useful auto completion provided by T3S and Sublime Text. In case it was missing, try to reload the project by typing `ctrl+shift+P` (`cmd+shift+P` on Mac) and searching for `Typescript: Reload Project`. Let's add some more lines:

    function repeatQuery(req: express.Request, res: express.Response) {
        res.json(req.query);
    }

    app.get('/repeat', repeatQuery);

Here you see that in `repeatQuery` we had to annotate the parameters explicitly, because TypeScript wouldn't know how the function is going to be used, and so couldn't know which types those parameters will be. And now we only need to let express listen on a port and we're done:

    var port: number = +process.env.PORT || 3000;

    var server = app.listen(port, function() {
      console.log('Express server listening on port ' + port);
    });

You see, we also had to annotate `port` as being a number, because `process.env` is of type `any`. And, therefore, TypeScript wouldn't know what properties or objects it contains in the first place, and even less what types those are. The final file should look like this:

    /// <reference path="../typings/tsd.d.ts" />

    import express = require('express');

    var app = express();

    app.get('/', (req, res) => {
        res.send('Hello TypeScript')
    });

    function repeatQuery(req: express.Request, res: express.Response) {
        res.json(req.query);
    }
    app.get('/repeat', repeatQuery);

    var port: number = +process.env.PORT || 3000;

    var server = app.listen(port, function() {
      console.log('Express server listening on port ' + port);
    });

## 5. Using Gulp for Compilation

If you used the Yeoman Generator, just run `gulp serve` and enjoy. **You are done!** 

Everyone else, bear with me. Here comes the tricky part and the reason, why the explicit `server` folder was necessary in the first place. You can't run TypeScript, you first need to compile it to JavaScript. A simple `gulpfile.js` for that could look like this:

    var gulp = require('gulp'),
      ts = require('gulp-typescript');

    gulp.task('typescript', function() {
      console.log('Compiling typescript');
      return gulp.src(['server/**/*.ts'])
        .pipe(ts({module: 'commonjs'})).js.pipe(gulp.dest('./deploy/server'))
    });

When you run `gulp typescript`, TypeScript will compile everything inside the `server` folder and send the JavaScript output to `deploy/server`, from where you run it simply with `npm index.js`. But as that is too boring, we'll automate that as well, in order to have a more fluent development experience. In the final result, we will watch for TypeScript changes, recompile those and serve them with `nodemon` which will update automatically. The update `gulpfile.js` could look like this:

    var gulp = require('gulp'),
      nodemon = require('gulp-nodemon'),
      ts = require('gulp-typescript');

    gulp.task('typescript', function() {
      console.log('Compiling typescript');
      return gulp.src(['server/**/*.ts'])
        .pipe(ts({module: 'commonjs'})).js.pipe(gulp.dest('./deploy/server'))
    });

    gulp.task('watch', function() {
      gulp.watch('./server/**/*.ts', ['typescript']);
    });

    gulp.task('serve', ['typescript'], function () {
      livereload.listen();
      nodemon({
        script: 'deploy/server/index.js',
        ext: 'js',
      }).on('restart', function () {
        setTimeout(function () {
          livereload.changed();
        }, 500);
      });
    });

You see, gulp watches for changes in `*.ts` files in order to recompile them. Nodemon will afterwards automatically restart the application. This is great for development, but how can we deploy this application to our server or Heroku? For that, one more file is missing, `package.json`. Yes, it is in the root folder of the application, but we don't want to deploy the root folder, instead, only the `deploy` folder. Add this to your `gulpfile.js`:

    gulp.task('deploy', ['build'], function() {
      return gulp.src(['package.json'])
        .pipe(gulp.dest('./deploy'));
    });

    gulp.task('default', ['deploy']);

This will copy the `package.json` file into your `deploy` directory. In case you want to run it on Heroku, you can also add the `Procfile` or anything else you'll need. Also don't forget to **exclude** the `deploy` folder from your git repository.

## 6. The End?

Well, it's the end of THIS post. It is by far not the end of what to consider when you use TypeScript, and, hopefully, only the beginning of your journey with TypeScript. You can check out my tiny NodeJS project [VimFika](https://github.com/pgrm/vimfika) which is written in TypeScript. There you can see examples of how to interact with MongoDB (using Mongoose) and other libraries. You can also watch out for the new [AirPair front-end](https://github.com/airpair/experts.airpair.com/), which is going to be open source and maybe in TypeScript, as well as my new project, [logTank](http://www.logtank.com/). It will be also open source and written entirely in TypeScript. Those projects will be, of course, much bigger than VimFika so you can get more inspiration and examples on how to solve some problems.

### Optional Extras

I'm now using `tslint` together with `gulp-tslint` which helps keeping a cleaner code and also prevents you from some JavaScript errors, which still can occur in TypeScript. For instance, unlike CoffeeScript, TypeScript's `==` will stay `==` and if you don't have exact type information, you should use `===`.

### What about the tests?

I'm currently working on an API and decided not to write tests in TypeScript. For me it would be too much hassle to need to compile the tests in some temporary directory before you can run them. There is great support to write tests in CoffeeScript, or you can use plain old JavaScript.

### How do I combine TypeScript on Client and Server in one project?

Well, if you really can't split it, you can give T3S' other option a try, which supports multiple TypeScript root files. If that doesn't work (and you don't want to fix it, as it is an open source project after all) you could try splitting the TypeScript files into different and mostly independent folders, `server` and `scripts` for instance. You could afterwards try to run different Sublime Text instances on these folders, with their separate  `.sublimets` files.

## The End!

If you have any questions or comments, please write them below this post, or contact me on twitter.