---
title: "Build Angular Universal Application Series 2: SEO and TransferState"
date: 2020-01-12 18:23:17 +0800
header:
  image: "/assets/images/headers/build-angular-universal-application-series-2-seo-and-transferstate-header.jpg"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com/photos/_st8-JkLstI)"
tags:
  - angular universal
toc: true
---

This is the first series for build angular universal application: SEO and transferstate.

## SEO for Angular Universal

Angular has provide the `Meta` and `Title` service to populate the meta tags for seo. You can also update the twitter card and facebook opengraph through these services.

```typescript
import { Meta, Title } from "@angular/platform-browser";

...

constructor(private meta: Meta, private title: Title) {
  title.setTitle('sample title');
  meta.addTags([
    { name: 'keywords', content: 'angular, universal' }
  ]);
}
```

## TransferState

By default, the angular universal application will request the API data at server-side to render the HTML, when client-side take over, the application will request the API data again.

In order to solve this problem of duplicate data fetching, we need find a way to store the *state*. The State Transfer API provides us with a storage container for easilly transfering data between the server and the client application, avoiding the need for the client application to have to contact the server to get the data.

### Fetch API by TransferState

Import the `BrowserTransferStateModule` to `AppBrowserModule`.

```typescript
import { BrowserTransferStateModule } from '@angular/platform-browser';

@NgModule({
  imports: [
    BrowserTransferStateModule,
    ...
  ]
})
export class AppBrowserModule { }
```

Import the `ServerTransferStateModule` to `AppServerModule`.

```typescript
import { ServerTransferStateModule } from '@angular/platform-server';

@NgModule({
  imports: [
    ServerTransferStateModule,
    ...
  ]
})
export class AppServerModule { }
```

Import the `TransferHttpCacheModule` to `AppModule`.

```typescript
import { TransferHttpCacheModule } from '@nguniversal/common';

@NgModule({
  imports: [
    TransferHttpCacheModule,
    ...
  ]
})
export class AppModule { }
```

Using the `TransferState` to fetch the API data.

```typescript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { TransferState, makeStateKey } from '@angular/platform-browser';

const RESULT_KEY = makeStateKey<string>('json-result');

@Component({
  selector: 'json-view',
  templateUrl: './json.component.html'
})
export class JsonComponent implements OnInit {
  public subs: any;

  constructor(private http: HttpClient, private state: TransferState) { }

  ngOnInit() {
    const baseUrl = 'http://localhost:3000';
    if (this.state.hasKey(RESULT_KEY)) {
      this.subs = this.state.get(RESULT_KEY, null as any);
    } else {
      this.http.get(baseUrl + `/api/json`).subscribe(data => {
        this.subs = data;
        this.state.set(RESULT_KEY, this.subs);
      });
    }
  }
}
```

### where to store the data

In the server rendered HTML, it contains a script tag at the bottom where the transfered data get's stored.

![angular universal transferstate in html](/assets/images/posts/angular-universal-transferstate-in-html.png)

## Reference

the complete code repository is: [angular-universal-demo](https://github.com/4garfield/angular-universal-demo), you can also check below reference links:

* [Angular Universal: a Complete Practical Guide](https://blog.angular-university.io/angular-universal/)
* [angular/universal](https://github.com/angular/universal)
* [TransferHttpCacheModule](https://github.com/angular/universal/blob/master/docs/transfer-http.md)
* [Server Side Rendering (SSR) in Angular 5+](https://itnext.io/server-side-rendering-ssr-in-angular-5-the-simplest-and-quickest-ssr-approach-34cf53224f32)
