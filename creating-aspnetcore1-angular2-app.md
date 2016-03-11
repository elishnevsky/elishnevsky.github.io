# Creating an ASP.NET Core 1 with Angular 2 Application

This is a step-by-step guide how to create an ASP.NET Core 1 with Angular 2
application from scretch.

## Prerequisites

1. Download and install the latest stable version of [NodeJS](https://nodejs.org)
2. In Visual Studio go to Options and navigate to Projects and Solutions > External Web Tools
3. Add `c:\Program Files\nodejs` to the list of locations and move it to the top

## Step 1: Creating a project

Create a new project and select ASP.NET Web Application. In the next window select any template from the list of ASP.NET 5 Templates. I suggest to start with an _Empty_ template and add what you need later.

Once the project is created right-click the project and select *Manage NuGet Packages...* Browse and install Microsoft.AspNet.StaticFiles dependency. Make sure *Include prelease* checkbox is checked.

Once that is complete open `Startup.cs` file and replace the call to `app.Run(...)` method with the following in this exact order:

```cs
app.UseDefaultFiles();
app.UseStaticFiles();
```

## Step 2: Adding index.html

Add a new HTML Page file to `wwwroot` folder and call it `index.html`. Add some content and run the application. If you have done everything correctly it should fire up the browser and display your page.

## Step 3: Adding tsconfig.json

In the root of your project create a new TypeScript JSON Configuration file. Keep the name `tsconfig.json'. Then copy/paste the following:

```js
{
  "compilerOptions": {
    "target": "es5",
    "module": "system",
    "moduleResolution": "node",
    "sourceMap": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "removeComments": false,
    "noImplicitAny": false,
    "rootDir": "./scripts",
    "outDir": "./wwwroot/app"
  },
  "exclude": [
    "node_modules",
    "typings/main",
    "typings/main.d.ts",
    "wwwroot"
  ]
}

```

`rootDir` is where you put your Angular 2 TypeScript source files. `outDir` is where the transpiled JavaScript files will be created. It is important that they will go in `wwwroot` folder, because it is where
all the static files are served from in ASP.NET Core 1 applications. The folder structure will be preserved.

## Step 4: Adding typings.json

Add a new JSON File to the root of the project, call it `typings.json` then copy/paste the following:

```js
{
  "ambientDependencies": {
    "es6-shim": "github:DefinitelyTyped/DefinitelyTyped/es6-shim/es6-shim.d.ts#4de74cb527395c13ba20b438c3a7a419ad931f1c",
    "jasmine": "github:DefinitelyTyped/DefinitelyTyped/jasmine/jasmine.d.ts#d594ef506d1efe2fea15f8f39099d19b39436b71"
  }
}

```

## Step 5: Adding gulpfile.js

Add a new Gulp Configuration File to the root of the project. Keep the name `gulpfile.js`. Replace the default content with the following:

```js
var gulp = require("gulp");

var paths = {
    npmSource: "./node_modules/",
    libTarget: "./wwwroot/lib/"
};

gulp.task("lib", function () {
    var files = [
        "es6-shim/es6-shim.min.js",
        "systemjs/dist/system-polyfills.js",
        "angular2/es6/dev/src/testing/shims_for_IE.js",
        "angular2/bundles/angular2-polyfills.js",
        "systemjs/dist/system.src.js",
        "rxjs/bundles/Rx.js",
        "angular2/bundles/angular2.dev.js",
        "angular2/bundles/router.dev.js",
        "angular2/bundles/http.dev.js"
    ].map(function (file) {
        return paths.npmSource + file;
    });

    return gulp.src(files, { base: paths.npmSource }).pipe(gulp.dest(paths.libTarget));
});

```

## Step 6: Adding package.json

Add a new NPM Configuration File to the root of the project. Keep the name `package.json`. Replace the default content with the following:

```js
{
  "version": "1.0.0",
  "name": "ASP.NET",
  "scripts": {
    "typings": "typings",
    "postinstall": "typings install && gulp lib"
  },
  "license": "ISC",
  "dependencies": {
    "angular2": "2.0.0-beta.9",
    "systemjs": "0.19.24",
    "es6-promise": "^3.0.2",
    "es6-shim": "^0.33.3",
    "reflect-metadata": "0.1.2",
    "rxjs": "5.0.0-beta.2",
    "zone.js": "0.5.15"
  },
  "devDependencies": {
    "gulp": "^3.9.1",
    "typings": "^0.7.5"
  }
}

```

As soon as you save this file Visual Studio starts restoring dependincies.

## Step 7: Writing some Angular 2 code

Create a new **scripts** folder. Add a TypeScript file called **app.component.ts**:

```ts
import {Component} from 'angular2/core';

@Component({
    selector: 'my-app',
    template: `<h1>My First ASP.NET Core 1 with Angular 2 App</h1>`
})
export class AppComponent { }
```

Add a new TypeScript file and call it **main.ts**:

```ts
import {bootstrap} from 'angular2/platform/browser';
import {AppComponent} from './app.component';

bootstrap(AppComponent);
```

## Step 8: Adding references, bootstrapping and running the application

Replace the content of your `index.html` file with the following:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>ASP.NET Core 1 with Angular 2</title>

    <!-- 1. Load libraries -->
    <!-- IE required polyfills, in this exact order -->
    <script src="lib/es6-shim/es6-shim.min.js"></script>
    <script src="lib/systemjs/dist/system-polyfills.js"></script>
    <script src="lib/angular2/es6/dev/src/testing/shims_for_IE.js"></script>

    <script src="lib/angular2/bundles/angular2-polyfills.js"></script>
    <script src="lib/systemjs/dist/system.src.js"></script>
    <script src="lib/rxjs/bundles/Rx.js"></script>
    <script src="lib/angular2/bundles/angular2.dev.js"></script>

    <!-- 2. Configure SystemJS -->
    <script>
        System.config({
            packages: {
                app: {
                    format: 'register',
                    defaultExtension: 'js'
                }
            }
        });
        System.import('app/main')
            .then(null, console.error.bind(console));
    </script>
</head>
<body>
    <my-app>Loading...</my-app>
</body>
</html>
```

Done!