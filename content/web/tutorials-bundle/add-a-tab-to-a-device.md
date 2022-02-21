---
title: Add a tab to a details views with context routes
layout: redirect
weight: 40
---

**Version:** 1009.0.18 | **Packages:** @c8y/cli, @c8y/apps and @c8y/ngx-components

It is a common use case that you want to show additional information to a user in a details view (for example for a device or a group).

This how-to recipe explains how to accomplish a new tab in the device details view:

![Device info with custom tab](/images/web-sdk/device-detail-custom-tab.png)

In Web SDK for Angular, this kind of views are called `ViewContext` as they provide a view for a certain context. There are a couple of context views, for example, `Device`, `Group`, `User`, `Application` and `Tenant`. The user can access them by navigating to a certain `Route` with the hash navigation. For example, if you go to the route `apps/cockpit/#/device/1234` the application tries to resolve the device with the ID `1234`.

The details view usually shows a couple of `Tabs`, like the **Info** tab in the screenshot above which is referenced by another route called `/info` but reuses the context of the device to show information about it.

In the following, we will guide you through the process of creating a new tab to this view that is accessible through the route `apps/cockpit/#/device/:id/hello`.

### 1. Initialize the example app

As a starting point, you need an application supporting context routes. For this purpose, create a new Cockpit application using the `c8ycli`:

```js
c8ycli new my-cockpit cockpit  -a @c8y/apps@1009.0.18
```

Next, you need to install all dependencies. Switch to the new folder and run `npm install`.

> **Tip:** The `c8ycli new` command has a `-a` flag which defines which package to use for scaffolding. This way you can also define which version of the app you want to scaffold, for example:
>
> - `c8ycli new my-cockpit cockpit -a @c8y/apps@1009.0.18` will scaffold an app with the version `1009.0.18`
> - `c8ycli new my-cockpit cockpit -a @c8y/apps@latest` will scaffold an app with the latest official release. Same as if used without the `-a` flag
> - `c8ycli new my-cockpit cockpit -a @c8y/apps@next` will scaffold an app with the latest beta release.


### 2. Add a new ROUTE&#95;HOOK_ONCE

The hook concept allows you to hook into the existing code. In this example we want to add a so-called ChildRoute (by Angular) on the existing route `device/:id`.

To achieve this, add the following code to the `app.module.ts`:

```js
import { NgModule } from '@angular/core';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { RouterModule as NgRouterModule } from '@angular/router';
import { UpgradeModule as NgUpgradeModule } from '@angular/upgrade/static';
// ---- 8< changed part ----
import { CoreModule, RouterModule, HOOK_ONCE_ROUTE, ViewContext } from '@c8y/ngx-components';
// ---- >8 ----
import { DashboardUpgradeModule, UpgradeModule, HybridAppModule, UPGRADE_ROUTES} from '@c8y/ngx-components/upgrade';
import { AssetsNavigatorModule } from '@c8y/ngx-components/assets-navigator';
import { CockpitDashboardModule, ReportDashboardModule } from '@c8y/ngx-components/context-dashboard';
import { ReportsModule } from '@c8y/ngx-components/reports';
import { SensorPhoneModule } from '@c8y/ngx-components/sensor-phone';
import { BinaryFileDownloadModule } from '@c8y/ngx-components/binary-file-download';

@NgModule({
  imports: [
    // Upgrade module must be the first
    UpgradeModule,
    BrowserAnimationsModule,
    RouterModule.forRoot(),
    NgRouterModule.forRoot([...UPGRADE_ROUTES], { enableTracing: false, useHash: true }),
    CoreModule.forRoot(),
    AssetsNavigatorModule,
    ReportsModule,
    NgUpgradeModule,
    DashboardUpgradeModule,
    CockpitDashboardModule,
    SensorPhoneModule,
    ReportDashboardModule,
    BinaryFileDownloadModule
  ],

  // ---- 8< added part ----
  providers: [{
    provide: HOOK_ONCE_ROUTE,          // 1.
    useValue: [{                       // 2.
      context: ViewContext.Device,     // 3.
      path: 'hello',                   // 4.
      component: HelloComponent,       // 5.
      label: 'hello',                  // 6.
      priority: 100,
      icon: 'rocket'
    }],
    multi: true
  }]
  // ---- >8 ----

})
export class AppModule extends HybridAppModule {
  constructor(protected upgrade: NgUpgradeModule) {
    super();
  }
}
```

Explanation of the above numbers:

 1. Provides the multi-provider hook `HOOK_ROUTE_ONCE`. This tells the application to extend the current route configuration.
 2. Specifies that we want to use a value to define the route hook. You can also use a class here, for example, if you want to resolve the routes async.
 3. Defines the context of the route. You should use the `ViewContext` enum to define it. In this case we want to extend the context of a device.
 4. The path where it is going to be shown. Is added to the context path. In this case the complete path is: `device/:id/hello`.
 5. Defines which component should be shown if the path is hit by a user.
 6. The properties `label` and `icon` define how the tab should look like. The `priority` defines on which position it should be shown.

> **Info:** The HOOK_ONCE_ROUTE inherits the Angular Route type, so all properties of it can be reused here.

After this alignments the route is registered but the application will fail to compile, as the `HelloComponent` does not exist yet. We will create it in the next section.

### 3. Add a component to display context data

The `HelloComponent` might want to display details about the device. Hence it needs the information in which context it has been opened. The context route resolves the device upfront, so there is no need to handle this. You can directly access it via the parent route.

So let's create a new file called `hello.component.ts`:

```js
import { Component } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-hello',
  template: `
  <c8y-title>world</c8y-title>
  <pre>
    <code>
      {{route.snapshot.parent.data.contextData | json}}
    </code>
  </pre>
  `
})
export class HelloComponent {
  constructor(public route: ActivatedRoute) {}
}
```

There is nothing special to mention about this component other than that it injects the `ActivatedRoute` and accesses the parent data of it. This is the key point: as the parent context route already has resolved the data of the device, this component will always show the detailed data of the current device.

Adding this to the `entryComponents` in `app.module.ts` will allow to compile the application:

```js

import { NgModule } from '@angular/core';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { RouterModule as NgRouterModule } from '@angular/router';
import { UpgradeModule as NgUpgradeModule } from '@angular/upgrade/static';
import { CoreModule, RouterModule, HOOK_ONCE_ROUTE, ViewContext } from '@c8y/ngx-components';
import { DashboardUpgradeModule, UpgradeModule, HybridAppModule, UPGRADE_ROUTES} from '@c8y/ngx-components/upgrade';
import { AssetsNavigatorModule } from '@c8y/ngx-components/assets-navigator';
import { CockpitDashboardModule, ReportDashboardModule } from '@c8y/ngx-components/context-dashboard';
import { ReportsModule } from '@c8y/ngx-components/reports';
import { SensorPhoneModule } from '@c8y/ngx-components/sensor-phone';
import { BinaryFileDownloadModule } from '@c8y/ngx-components/binary-file-download';
// ---- 8< added part ----
import { HelloComponent } from './hello.component';
// ---- >8 ----

@NgModule({
  imports: [
    // Upgrade module must be the first
    UpgradeModule,
    BrowserAnimationsModule,
    RouterModule.forRoot(),
    NgRouterModule.forRoot([...UPGRADE_ROUTES], { enableTracing: false, useHash: true }),
    CoreModule.forRoot(),
    AssetsNavigatorModule,
    ReportsModule,
    NgUpgradeModule,
    DashboardUpgradeModule,
    CockpitDashboardModule,
    SensorPhoneModule,
    ReportDashboardModule,
    BinaryFileDownloadModule
  ],

  // ---- 8< added part ----
  declarations: [HelloComponent],
  entryComponents: [HelloComponent],
  // ---- >8 ----

  providers: [{
    provide: HOOK_ONCE_ROUTE,
    useValue: [{
      context: ViewContext.Device,
      path: 'hello',
      component: HelloComponent,
      label: 'hello',
      priority: 100,
      icon: 'rocket'
    }],
    multi: true
  }]
})
export class AppModule extends HybridAppModule {
  constructor(protected upgrade: NgUpgradeModule) {
    super();
  }
}

```

When you now start your application with `npm start` and navigate to a details view of a device it should look like this:

![Device info with custom tab](/images/web-sdk/device-detail-custom-tab.png)

Congratulations, you added a tab to a device. You can do the same for tenants, users or applications details views.

Next you will learn how to show this tab only if a certain criteria is met.

### (Bonus) 4. Show the tab only on certain criteria

In some cases, additional information is available only if a certain criteria is met. For example, it only makes sense to show a location if the device has a location fragment associated. To add such a criteria, the context routes inherit the [guard concept of Angular](https://angular.io/guide/router#milestone-5-route-guards).

To add a guard, you simply need to add the `canActivate` property to the route definition:

```js
import { NgModule } from '@angular/core';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { RouterModule as NgRouterModule } from '@angular/router';
import { UpgradeModule as NgUpgradeModule } from '@angular/upgrade/static';
import { CoreModule, RouterModule, HOOK_ONCE_ROUTE, ViewContext } from '@c8y/ngx-components';
import { DashboardUpgradeModule, UpgradeModule, HybridAppModule, UPGRADE_ROUTES} from '@c8y/ngx-components/upgrade';
import { AssetsNavigatorModule } from '@c8y/ngx-components/assets-navigator';
import { CockpitDashboardModule, ReportDashboardModule } from '@c8y/ngx-components/context-dashboard';
import { ReportsModule } from '@c8y/ngx-components/reports';
import { SensorPhoneModule } from '@c8y/ngx-components/sensor-phone';
import { BinaryFileDownloadModule } from '@c8y/ngx-components/binary-file-download';
import { HelloComponent } from './hello.component';

// ---- 8< added part ----
import { HelloGuard } from './hello.guard';
// ---- >8 ----

@NgModule({
  imports: [
    // Upgrade module must be the first
    UpgradeModule,
    BrowserAnimationsModule,
    RouterModule.forRoot(),
    NgRouterModule.forRoot([...UPGRADE_ROUTES], { enableTracing: false, useHash: true }),
    CoreModule.forRoot(),
    AssetsNavigatorModule,
    ReportsModule,
    NgUpgradeModule,
    DashboardUpgradeModule,
    CockpitDashboardModule,
    SensorPhoneModule,
    ReportDashboardModule,
    BinaryFileDownloadModule
  ],
  declarations: [HelloComponent],
  entryComponents: [HelloComponent],
  providers: [

    // ---- 8< added part ----
    HelloGuard,
    // ---- >8 ----

    {
    provide: HOOK_ONCE_ROUTE,
    useValue: [{
      context: ViewContext.Device,
      path: 'hello',
      component: HelloComponent,
      label: 'hello',
      priority: 100,
      icon: 'rocket',

      // ---- 8< added part ----
      canActivate: [HelloGuard]
      // ---- >8 ----

    }],
    multi: true
  }]
})
export class AppModule extends HybridAppModule {
  constructor(protected upgrade: NgUpgradeModule) {
    super();
  }
}
```

Now you can write a guard which checks certain criteria. If it resolves to true, the tab will be shown, otherwise not.

A guard to check for a certain fragment on a device can look like this `hello.guard.ts`:

```js
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

@Injectable()
export class HelloGuard implements CanActivate {

  canActivate(route: ActivatedRouteSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    const contextData = route.data.contextData || route.parent.data.contextData;          // 1.
    const { 'acme_HelloWorld': helloWorldFragment } = contextData;                        // 2.
    return !!helloWorldFragment;
  }
}
```

Explanation of the above numbers:

 1. This is the only tricky part which is not aligned with the Angular router as in a context route the `CanActivate` will be called twice, once when the parent route is activated and once when the child route is activated. The first call checks if the tab should be shown at all, while the second call checks if the user is allowed to navigate to it. Hence the `ActivatedRouteSnapshot` is different in both calls and we need to resolve the `contextData` in the second case from the parent.
 2. Checks if the `acme_HelloWorld` fragment is set on the context.

If you now post a device with the fragment `"acme_HelloWorld": {}` to the API, the **Hello** tab will just be shown for that device and not for others.

### Conclusion

Context routes help you to extend existing routes with further information.

At the same time, the concept allows the application to be consistent since the context is just resolved once and context not found can be handled by the parent.

However, there is currently no default way of abstracting the context route concept and implementing your own context route. But since the concept is heavily based on [Angular routing](https://angular.io/guide/router) you can implement the concept quite easily.
