# Code Organization Guidelines

The source code in the repo can be divided into the following four categories:

1. Apps. These are your products, something you ship to the user.
2. App-specific libs. These are apps' sections that can be developed and tested independently.
3. Reusable libs. These are your components, services, utilities used in many different apps.
4. Third-party libs and tools.

## Apps

Apps should contain only a handful of files setting up the environment and configuring global services/capabilities. All `.forRoot` calls should be in `apps`. Apps should not contain any business logic, any services, and should ideally contain a single component.

An example of an app module:

```
@NgModule({
  imports: [
    BrowserModule,
    NxModule.forRoot(),
    TypeaheadModule.forRoot(),
    StoreModule.forRoot({user}),
    EffectsModule.forRoot([AuthEffects]),
    RouterModule.forRoot([
      {
        path: 'history',
        loadChildren: '@techops-ui/safe/history-search#HistorySearchModule'
      }
    ]),
    !environment.production ? StoreDevtoolsModule.instrument() : []
  ]
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## App-specific libs

App-specific libs is where the application functionality is implemented. They should contain components, services, state management, business logic. They should not have any `.forRoot` calls.

An example:

```
@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    TypeaheadModule,
    StoreModule.forFeature('historySearch', reducers),
    EffectsModule.forFeature([
      appEffects.FleetEffects,
      appEffects.DeferralCodeEffects
    ]),
    RouterModule.forChild([{ path: '', component: HistorySearchComponent }])
  ],
  declarations: [
    SearchResultsComponent,
    TotalItemsFoundComponent,
    HistorySearchComponent
  ],
  providers: [
    SearchDiscrepancyCriteriaService,
    SearchDiscrepancyResultsService
  ]
})
export class HistorySearchModule {}
```

App-specific libs should be as independent as possible to allow us to develop and test them independently.

### Recommendations

* Places app-specific libs under the directory matching the app name (e.g., `libs/safe/history-search`). 
* If an app-specific lib defines the router configuration, make sure it does not export any components.
* If an app-specific lib is loaded lazily, **add it to the lazyLoad property in tslint.json**.

### Extracting an app-specific lib from an app

The process of extracting of an app-specific lib looks like this:

* Run `ng g lib lib-name --directory=app-name` (add `--routing`, `--lazy`, and `--parentModule` if needed)
* Copy the content of the app (everything except index.html, main.ts, polyfills.ts, app.module.ts, app.component.ts and other "global" files) into the lib.
* Update `app.module.ts` to remove all the imports and declarations that are no longer needed. Leave `forRoot` calls there, but you may have to update them (e.g., `StoreModule.forRoot({})`).
* Replace all `forRoot` calls with `forFeature` calls in the app-specific lib. You will have to introduce a prefix for the store, so you will have to update the tests and the production code to reflect that.
* Replace `BrowserModule` with `CommonModule` in the app-specific lib. BrowserModule can only be imported once, and it should be done at the app level.
* Create an integration test that exercises the new module without requiring the containing app.
* If your new app-specific lib is loaded lazily, **add it to the lazyLoad property in tslint.json**.


## Reusable libs

Reusable libs are the collections of components and services.

An example:

```
@NgModule({
  imports: [CommonModule],
  declarations: [AppsModalComponent, NotificationModalComponent, UserModalComponent, HeaderComponent],
  exports: [HeaderComponent, AppsModalComponent, NotificationModalComponent, UserModalComponent]
})
export class TechopsHeaderModule {}
```
