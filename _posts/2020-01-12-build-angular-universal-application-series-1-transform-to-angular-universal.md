---
title: "Build Angular Universal Application Series 1: Transform to Angular Universal"
date: 2020-01-12 17:44:09 +0800
header:
  image: "/assets/images/headers/build-angular-universal-application-series-1-transform-to-angular-universal-header.jpg"
  teaser: "/assets/images/headers/build-angular-universal-application-series-1-transform-to-angular-universal-teaser.jpg"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com/photos/XnvLe0u9iM8)"
tags:
  - ssr
  - angular universal
toc: true
---

This is the first series for build angular universal application: transform to Angular Universal.

## What and Why

When we use Angular Universal, we will render the initial HTML(`index.html`) and CSS shown to the user ahead of time. We can do it for example at build time, or on-the-fly on the server when the user requests the page.

This server-side rendered HTML will be served initially to the user, so the user can see something on the screen quickly. Together with the server-side rendered HTML, we will also ship to the browser a normal client-side Angular Application. This client-side Angular application will then take over the page, and from there on everything is working like a normal single page application, meaning that all the runtime rendering will occur directly on the client as usual.

There are three main reasons to create a Universal version of your app.

* Facilitate web crawlers through search engine optimization (SEO)
* Improve performance on mobile and low-powered devices
* Show the first page quickly with a first-contentful paint (FCP)

## Add Angular Universal to normal application

For the existing Angular application which already uses the Angular CLI, we will add the Angular Universal to the application.

### Angular Universal Express Server

Create `server.ts` file contains the backend express server for universal.

```typescript
// These are important and needed before anything else
import 'zone.js/dist/zone-node';
import 'reflect-metadata';

import { enableProdMode } from '@angular/core';

import * as express from 'express';
import { join } from 'path';

// Faster server renders w/ Prod mode (dev mode never needed)
enableProdMode();

// Express server
const app = express();

const PORT = process.env.PORT || 3000;
const DIST_FOLDER = join(process.cwd(), 'dist');

// * NOTE :: leave this as require() since this file is built Dynamically from webpack
const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = require('./dist/server/main.bundle');

// Express Engine
import { ngExpressEngine } from '@nguniversal/express-engine';
// Import module map for lazy loading
import { provideModuleMap } from '@nguniversal/module-map-ngfactory-loader';

app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));

app.set('view engine', 'html');
app.set('views', join(DIST_FOLDER, 'browser'));

function cacheControl(req, res, next) {
  res.header('Cache-Control', 'max-age=60');
  next();
}
// Server static files
app.get('*.*', cacheControl, express.static(join(DIST_FOLDER, 'browser'), { index: false }));

app.get('*', cacheControl, (req, res) => {
  console.time(`GET: ${req.originalUrl}`);
  res.render('index', {
    req: req,
    res: res,
    time: true,    // use this to determine what part of your app is slow, only in development
    providers: []
  });
  console.timeEnd(`GET: ${req.originalUrl}`);
});

// catch 404
app.use(function (req, res, next) {
  res.status(404);
  res.json({ msg: 'Request Resource Not Found' });
});

// error handler
app.use(function (err, req, res, next) {
  res.status(err.status || 500);
  res.json({ msg: err.message });
});

// Start up the Node server
app.listen(PORT, () => {
  console.log(`Node server listening on http://localhost:${PORT}`);
});
```

let's see this line of code:

```typescript
const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = require('./dist/server/main.bundle');
```

This is for importing the universal bundle `main.bundle.js` from the build output folder.

### Server build config

Add the `webpack.server.config.js`, for angular universal express server build config.

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    server: path.join(__dirname, 'server.ts')
  },
  resolve: {
    extensions: ['.js', '.ts']
  },
  target: 'node',
  externals: [],
  output: {
    path: path.join(__dirname, './dist'),
    filename: '[name].js'
  },
  module: {
    rules: [{
      test: /\.ts$/,
      loader: 'ts-loader',
      options: {
        onlyCompileBundledFiles: true
      }
    }]
  },
  plugins: [
    // Temporary Fix for issue: https://github.com/angular/angular/issues/11580
    // for 'WARNING Critical dependency: the request of a dependency is an expression'
    new webpack.ContextReplacementPlugin(
      /(.+)?angular(\\|\/)core(.+)?/,
      path.join(__dirname, 'src'), // location of your src
      {} // a map of your routes
    ),
    new webpack.ContextReplacementPlugin(
      /(.+)?express(\\|\/)(.+)?/,
      path.join(__dirname, 'src'), {}
    )
  ]
}
```

### Separate Client & Server Application Entry Point

Due to service-side and client-side has different runtime, we need separate the application entry point.

#### .angular-cli.json

Update the `.angular-cli.json` to naming the application in client-side and server-side.

```json
{
  ...
  "apps": [{
    "name": "browser",
    "platform": "browser",
    "root": "src",
    "outDir": "dist/browser",
    "assets": [
      "favicon.ico",
      "assets"
    ],
    "index": "index.html",
    "main": "main.browser.ts",
    "polyfills": "polyfills.ts",
    "tsconfig": "tsconfig.app.json",
    "prefix": "app",
    "styles": [],
    "scripts": [],
    "environmentSource": "environments/environment.ts",
    "environments": {
      "dev": "environments/environment.ts",
      "prod": "environments/environment.prod.ts"
    }
  }, {
    "name": "server",
    "platform": "server",
    "root": "src",
    "outDir": "dist/server",
    "main": "main.server.ts",
    "tsconfig": "tsconfig.server.json",
    "environmentSource": "environments/environment.ts",
    "environments": {
      "dev": "environments/environment.ts",
      "prod": "environments/environment.prod.ts"
    }
  }],
  ...
}
```

#### package.json

Update `package.json` build scripts for angular universal.

```json
{
  ...
  "scripts": {
    "build:ssr": "npm run build:ssr:browser && npm run build:ssr:server && npm run webpack:server",
    "build:ssr:browser": "ng build --prod --aot --app browser",
    "build:ssr:server": "ng build --prod --aot --app server --output-hashing none",
    "serve:ssr": "node dist/server.js",
    "webpack:server": "webpack --config webpack.server.config.js --progress --colors"
  },
  ...
}
```

#### main.browser.ts

Add client-side application entry point: `main.browser.ts`.

```typescript
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppBrowserModule } from './app/app.browser.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppBrowserModule)
  .catch(err => console.log(err));
```

#### main.server.ts

Add server-side application entry point: `main.server.ts`.

```main.server.ts
export { AppServerModule } from './app/app.server.module';
```

#### tsconfig.server.json

Add the server-side typescript build configuration: tsconfig.server.json.

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "baseUrl": "./",
    "module": "commonjs",
    "types": []
  },
  "exclude": [
    "**/*.spec.ts"
  ],
  "angularCompilerOptions": {
    "entryModule": "app/app.server.module#AppServerModule"
  }
}
```

#### app.browser.module.ts

Add `app.browser.module.ts` for client-side angular app root module.

```typescript
import { NgModule } from '@angular/core'
import { AppComponent } from './app.component';
import { AppModule } from './app.module';

@NgModule({
  bootstrap: [AppComponent],
  imports: [
    AppModule
  ]
})
export class AppBrowserModule { }
```

#### app.server.module.ts

Add `app.server.module.ts` for server-side angular app root module.

```typescript
import { NgModule } from '@angular/core';
import { ServerModule } from '@angular/platform-server';
import { ModuleMapLoaderModule } from '@nguniversal/module-map-ngfactory-loader';

import { AppComponent } from './app.component';
import { AppModule } from './app.module';

@NgModule({
  bootstrap: [AppComponent],
  imports: [
    AppModule,
    ServerModule,
    ModuleMapLoaderModule
  ],
  providers: [
    // Add universal-only providers here
  ],
})
export class AppServerModule { }
```

#### app.module.ts

Update the `app.module.ts` to add the identifier for angular universal application.

```typescript
import { BrowserModule } from '@angular/platform-browser';

@NgModule({
  imports: [
    BrowserModule.withServerTransition({
      appId: 'angular-universal-demo-app'
    }),
    ...
  ],
  ...
})
export class AppModule { }
```

### Import Universal Compatible Module

Due to the client-side and server-side have different js runtime. We must take care of every modules used in application, makesure non of them throwing error in server-side.

Like the Angular Animation, you need separately import for client-side and server-side.

`app.browser.module.ts`.

```typescript
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

@NgModule({
  ...
  imports: [
    BrowserAnimationsModule,
    ...
  ]
})
export class AppBrowserModule { }
```

`app.server.module.ts`.

```typescript
import { NoopAnimationsModule } from '@angular/platform-browser/animations';

@NgModule({
  ...
  imports: [
    NoopAnimationsModule,
    ...
  ]
})
export class AppServerModule { }
```

## Universal "Gotchas"

Below things need keep in mind for build the angular universal application.

* `window`, `document` and others browser only type does not exist on the server. You need wrap them in the browser only environment.

```typescript
import { PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

constructor(@Inject(PLATFORM_ID) private platformId: Object) { ... }

ngOnInit() {
  if (isPlatformBrowser(this.platformId)) {
    // Client only code.
    ...
  }
  if (isPlatformServer(this.platformId)) {
    // Server only code.
    ...
  }
}
```

* **Don't manipulate the nativeElement directly.** Instead, you can use the Renderer2 to manipulate.

```typescript
constructor(element: ElementRef, renderer: Renderer2) {
  renderer.setStyle(element.nativeElement, 'font-size', 'x-large');
}
```
