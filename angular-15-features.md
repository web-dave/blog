# Angular 15 is here!

Angular 15 is here and many new features have been added again.

As you can already see from the version, there are also breaking changes.
This time for a long time again one which requires manual intervention.

But one after anonther.

## NPM update

Angular now supports Node.js in the following versions 14.20.x, 16.13.x and 18.10.x.
All earlier versions are no longer supported. One of the reasons for this is EOL.

## ESBuild

The build process was optimized in the CLI. The process has been scaled down from Webpack and ESBuild to ESBuild. This reduced the build time by 57%.

## TypeScript 4.8

Angular 15 relies on Typescript 4.8. which brings some changes and new features. With that comes the breaking change that requires manual intervention (check npm update).
All changes can be found [here](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-8.html)

## injectFunction

What is interesting is a change that came with version 3.7.
This is about the way class attributes are defined.
Typesscript assumed an unofficial solution when initially implemented. Why do we care?
Because this implementation is leveraged by the dependency framework within Angular. There is a flag to keep the old implementation, which is also set by default in Angular 15, but this moves you away from the ES2022 standard.
You can find more details in the official [releasenote](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#the-usedefineforclassfields-flag-and-the-declare-property-modifier).

But what does the correct implementation of AI in Angular 15 look like now? The whole thing is solved by a function already known from Angular 14. `inject`

Dependency injection so far looked like this:

```ts
@Component({
  selector: "my-app",
  template: `{{ service.data$ | async }}`,
})
export class AppComponent {
  constructor(public service: myService) {}
}
```

From Angular 15 this way is a possible (and in my opinion also nicer)

```ts
@Component({
  selector: "my-app",
  template: `{{ data$ | async }}`,
})
export class AppComponent {
  data$ = inject(MyService).data$;
}
```

## Standalone Components

A less exciting feature (because it's been known for a while) are standalone components.
These are now stable in Angular 15.
This makes `@NgModule` optional. It is not yet possible to generate a project completely without `@NgModule`, the corresponding generator will probably be delivered with 15.x.

Below is a short example.

We see that the NgModule logic has been moved to the Component.
Standalone components are therefore not declared but imported, as you can see from the example.

```ts
@Component({
  selector: "app-root",
  standalone: true,
  imports: [CommonModule, MyOtherComponent, TopNavComponent],
  providers: [],
  template: `
    <top-nav></top-nav>
    <other-component></other-component>
  `,
})
export class AppComponent {}
```

The Boostrap process is also slightly adjusted.

```ts
import { bootstrapApplication } from "@angular/platform-browser";
bootstrapApplication(AppComponent).catch((err) => console.error(err));
```

But components aren't the only thing that's possible standalone.

## Standalone Router

That brings us straight to the router. This can now also be used without using NgModule.
For this purpose, the router is passed as the provider during bootstrapping.
Also the already familiar options which are otherwise given to `forRoot` are still available.

```ts
export const lazyRoutes: Routes = [{ path: "", component: PrivateComponent }];
```

```ts
const aboutRoutes: Routes = [
  {
    path: "about",
    component: AboutComponent,
  },
  {
    path: "private",
    loadChildren: () =>
      import("./feature/private").then((routes) => routes.lazyRoutes),
  },
];

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(
      aboutRoutes,
      withDebugTracing(),
      withPreloading(PreloadAllModules)
    ),
  ],
}).catch((err) => console.error(err));
```

### Functional routeGuards

The guards are now available as functions too, class based guards are expected to be deprecated in v16. This change came as a request from the community. This should make it easier to get started with Angular

So far, a guard looked like this:

```ts
@Injectable({ providedIn: "root" })
export class MyGuard implements CanActivate {
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    return true;
  }
}

const aboutRoutes: Routes = [
  {
    path: "about",
    component: AboutComponent,
    canActivate: [MyGuard],
  },
];
```

So we can now pack Guard functionality into functions. We also have the option of using the inject function here.

```ts
export const myGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot
): boolean => {
  return inject(MyService).status;
};

const aboutRoutes: Routes = [
  {
    path: "about",
    component: AboutComponent,
    canActivate: [myGuard],
  },
];
```

## Standalone HttpClient

The HttpClient has always been an injectable. We can also do without `@NgModule` here, any interceptors are passed via a factory mathode.

```ts
bootstrapApplication(AppComponent, {
  providers: [provideHttpClient(withInterceptors([]))],
}).catch((err) => console.error(err));
```

I was very happy about this development.
And if you've been around long enough, you might also get a [dejavu](https://github.com/web-dave/ng2lala/blob/17b4b55fb2b5fdb9b2977e5f47e0bc6f0dc0cd45/src/main.ts).

## Directive Composition Api

Sometimes there is a desire to define a component that inherits from multiple classes.
There is no such thing in Typescript.
With the Directive Composition API we can combine several directives into one directive and retain access to the public API of the individual directives.
Here's an example:

```ts
@Component({
  selector: "mat-menu",
  hostDirectives: [
    CdkTooltip,
    {
      directive: CdkMenu,
      inputs: ["cdkMenuDisabled: disabled"],
      outputs: ["cdkMenuClosed: closed"],
    },
  ],
})
class MatMenu {}
```

I looked at the feature in my live stream and tried it out.
It all feels very good. The video is available on my Youtube Channel. https://www.youtube.com/@webdave_de

If you like, you are welcome to join my stream, I deal with web technologies and stream on Twitch every Tuesday at 8:00 p.m. https://webdave.tv
