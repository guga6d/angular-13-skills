---
name: angular-routing
description: Implement routing in Angular 13 applications using RouterModule, lazy loaded modules, class-based guards, resolvers, child routes, and route parameters with ActivatedRoute. Use for navigation setup, protected routes, feature module routing, nested routes, and route data handling. Triggers on route configuration, authentication guards, lazy loading modules, navigation handling, or reading route/query parameters in Angular 13.
---

# Angular Routing (Angular 13)

Implement routing in Angular 13 applications using `RouterModule`, lazy loaded feature modules, guards, resolvers, and `ActivatedRoute`.

Angular 13 uses NgModules and class-based routing patterns. Functional guards, standalone components, `provideRouter`, and signal-based route APIs are not available.

---

## Basic Setup

Angular 13 routing is configured using `RouterModule.forRoot()`.

## App Routing Module

```typescript
// app-routing.module.ts

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { HomeComponent } from './pages/home/home.component';
import { AboutComponent } from './pages/about/about.component';
import { NotFoundComponent } from './pages/not-found/not-found.component';

const routes: Routes = [
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full',
  },
  {
    path: 'home',
    component: HomeComponent,
  },
  {
    path: 'about',
    component: AboutComponent,
  },
  {
    path: '**',
    component: NotFoundComponent,
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

# Router Outlet & Navigation

## App Component

```typescript
// app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent {}
```

```html
<!-- app.component.html -->

<nav>
  <a routerLink="/home" routerLinkActive="active">
    Home
  </a>

  <a routerLink="/about" routerLinkActive="active">
    About
  </a>
</nav>

<router-outlet></router-outlet>
```

---

# Lazy Loading Feature Modules

Angular 13 lazy loading is implemented with `loadChildren()` and feature modules.

## App Routes

```typescript
const routes: Routes = [
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full',
  },

  {
    path: 'home',
    component: HomeComponent,
  },

  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.module').then(
        (m) => m.AdminModule
      ),
  },
];
```

---

# Feature Routing Modules

Each feature module should have its own routing module.

## Admin Routing Module

```typescript
// admin-routing.module.ts

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { AdminDashboardComponent } from './pages/admin-dashboard/admin-dashboard.component';
import { AdminUsersComponent } from './pages/admin-users/admin-users.component';
import { AdminSettingsComponent } from './pages/admin-settings/admin-settings.component';

const routes: Routes = [
  {
    path: '',
    component: AdminDashboardComponent,
  },
  {
    path: 'users',
    component: AdminUsersComponent,
  },
  {
    path: 'settings',
    component: AdminSettingsComponent,
  },
];

@NgModule({
  imports: [
    RouterModule.forChild(routes),
  ],
  exports: [RouterModule],
})
export class AdminRoutingModule {}
```

## Admin Module

```typescript
// admin.module.ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { AdminRoutingModule } from './admin-routing.module';

@NgModule({
  declarations: [
    AdminDashboardComponent,
    AdminUsersComponent,
    AdminSettingsComponent,
  ],
  imports: [
    CommonModule,
    AdminRoutingModule,
  ],
})
export class AdminModule {}
```

---

# Route Parameters

## Route Configuration

```typescript
const routes: Routes = [
  {
    path: 'users/:id',
    component: UserDetailComponent,
  },
];
```

## Reading Route Params

```typescript
// user-detail.component.ts

import { Component, OnDestroy, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

import { Subscription } from 'rxjs';

@Component({
  selector: 'app-user-detail',
  templateUrl: './user-detail.component.html',
})
export class UserDetailComponent implements OnInit, OnDestroy {

  userId!: string;

  private subscription!: Subscription;

  constructor(
    private route: ActivatedRoute,
  ) {}

  ngOnInit(): void {
    this.subscription = this.route.paramMap.subscribe((params) => {
      this.userId = params.get('id') || '';
    });
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
}
```

## Template

```html
<h1>User {{ userId }}</h1>
```

---

# Query Parameters

## Route Example

```text
/search?q=angular&page=1
```

## Reading Query Params

```typescript
import { Component, OnDestroy, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

import { Subscription } from 'rxjs';

@Component({
  selector: 'app-search',
  templateUrl: './search.component.html',
})
export class SearchComponent implements OnInit, OnDestroy {

  query = '';
  page = 1;

  private subscription!: Subscription;

  constructor(
    private route: ActivatedRoute,
  ) {}

  ngOnInit(): void {
    this.subscription = this.route.queryParamMap.subscribe((params) => {
      this.query = params.get('q') || '';
      this.page = Number(params.get('page')) || 1;
    });
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
}
```

---

# Route Guards

Angular 13 uses class-based guards.

## Auth Guard

```typescript
// auth.guard.ts

import { Injectable } from '@angular/core';

import {
  ActivatedRouteSnapshot,
  CanActivate,
  Router,
  RouterStateSnapshot,
  UrlTree,
} from '@angular/router';

import { Observable } from 'rxjs';

import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {

  constructor(
    private authService: AuthService,
    private router: Router,
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot,
  ):
    | boolean
    | UrlTree
    | Observable<boolean | UrlTree>
    | Promise<boolean | UrlTree> {

    if (this.authService.isAuthenticated()) {
      return true;
    }

    return this.router.createUrlTree(
      ['/login'],
      {
        queryParams: {
          returnUrl: state.url,
        },
      },
    );
  }
}
```

## Route Usage

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard],
  },
];
```

---

# Role Guard

## Role Guard Service

```typescript
// role.guard.ts

import { Injectable } from '@angular/core';

import {
  ActivatedRouteSnapshot,
  CanActivate,
  Router,
  UrlTree,
} from '@angular/router';

import { Observable } from 'rxjs';

import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root',
})
export class RoleGuard implements CanActivate {

  constructor(
    private authService: AuthService,
    private router: Router,
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
  ):
    | boolean
    | UrlTree
    | Observable<boolean | UrlTree>
    | Promise<boolean | UrlTree> {

    const allowedRoles = route.data['roles'] as string[];

    const userRole = this.authService.currentUser?.role;

    if (
      userRole &&
      allowedRoles.includes(userRole)
    ) {
      return true;
    }

    return this.router.createUrlTree(['/unauthorized']);
  }
}
```

## Usage

```typescript
{
  path: 'admin',
  component: AdminComponent,
  canActivate: [AuthGuard, RoleGuard],
  data: {
    roles: ['admin', 'superadmin'],
  },
}
```

---

# CanDeactivate Guard

Use `CanDeactivate` to prevent navigation with unsaved changes.

## Interface

```typescript
export interface CanComponentDeactivate {
  canDeactivate: () => boolean;
}
```

## Guard

```typescript
// unsaved-changes.guard.ts

import { Injectable } from '@angular/core';

import {
  CanDeactivate,
} from '@angular/router';

import { CanComponentDeactivate } from './can-component-deactivate';

@Injectable({
  providedIn: 'root',
})
export class UnsavedChangesGuard
  implements CanDeactivate<CanComponentDeactivate> {

  canDeactivate(
    component: CanComponentDeactivate,
  ): boolean {

    if (component.canDeactivate()) {
      return true;
    }

    return confirm(
      'You have unsaved changes. Leave anyway?'
    );
  }
}
```

## Component

```typescript
@Component({
  selector: 'app-edit-user',
  templateUrl: './edit-user.component.html',
})
export class EditUserComponent
  implements CanComponentDeactivate {

  form = this.fb.group({
    name: [''],
  });

  constructor(
    private fb: FormBuilder,
  ) {}

  canDeactivate(): boolean {
    return !this.form.dirty;
  }
}
```

## Route

```typescript
{
  path: 'users/:id/edit',
  component: EditUserComponent,
  canDeactivate: [UnsavedChangesGuard],
}
```

---

# Resolvers

Resolvers load data before route activation.

## Resolver

```typescript
// user.resolver.ts

import { Injectable } from '@angular/core';

import {
  ActivatedRouteSnapshot,
  Resolve,
} from '@angular/router';

import { Observable } from 'rxjs';

import { UserService } from '../services/user.service';
import { User } from '../models/user';

@Injectable({
  providedIn: 'root',
})
export class UserResolver implements Resolve<User> {

  constructor(
    private userService: UserService,
  ) {}

  resolve(
    route: ActivatedRouteSnapshot,
  ): Observable<User> {

    const id = route.paramMap.get('id') || '';

    return this.userService.getById(id);
  }
}
```

## Route Configuration

```typescript
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: {
    user: UserResolver,
  },
}
```

## Accessing Resolved Data

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

import { User } from '../models/user';

@Component({
  selector: 'app-user-detail',
  templateUrl: './user-detail.component.html',
})
export class UserDetailComponent implements OnInit {

  user!: User;

  constructor(
    private route: ActivatedRoute,
  ) {}

  ngOnInit(): void {
    this.user = this.route.snapshot.data['user'];
  }
}
```

---

# Nested Routes

## Parent Route

```typescript
const routes: Routes = [
  {
    path: 'products',
    component: ProductsLayoutComponent,
    children: [
      {
        path: '',
        component: ProductListComponent,
      },
      {
        path: ':id',
        component: ProductDetailComponent,
      },
      {
        path: ':id/edit',
        component: ProductEditComponent,
      },
    ],
  },
];
```

## Parent Layout Component

```html
<h1>Products</h1>

<router-outlet></router-outlet>
```

---

# Programmatic Navigation

## Navigation Examples

```typescript
import { Component } from '@angular/core';
import {
  ActivatedRoute,
  Router,
} from '@angular/router';

@Component({
  selector: 'app-product',
  templateUrl: './product.component.html',
})
export class ProductComponent {

  constructor(
    private router: Router,
    private route: ActivatedRoute,
  ) {}

  goToProducts(): void {
    this.router.navigate(['/products']);
  }

  goToProduct(id: string): void {
    this.router.navigate(['/products', id]);
  }

  search(query: string): void {
    this.router.navigate(
      ['/search'],
      {
        queryParams: {
          q: query,
          page: 1,
        },
      },
    );
  }

  goToEdit(): void {
    this.router.navigate(
      ['edit'],
      {
        relativeTo: this.route,
      },
    );
  }

  replaceUrl(): void {
    this.router.navigate(
      ['/new-page'],
      {
        replaceUrl: true,
      },
    );
  }
}
```

---

# Route Data

## Static Route Data

```typescript
{
  path: 'admin',
  component: AdminComponent,
  data: {
    title: 'Admin Dashboard',
    roles: ['admin'],
  },
}
```

## Accessing Route Data

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-admin',
  templateUrl: './admin.component.html',
})
export class AdminComponent implements OnInit {

  title = '';
  roles: string[] = [];

  constructor(
    private route: ActivatedRoute,
  ) {}

  ngOnInit(): void {
    this.title = this.route.snapshot.data['title'];
    this.roles = this.route.snapshot.data['roles'];
  }
}
```

---

# Router Events

Listen to navigation events for loaders, analytics, or page transitions.

```typescript
import { Component, OnDestroy, OnInit } from '@angular/core';

import {
  NavigationEnd,
  NavigationStart,
  Router,
} from '@angular/router';

import { filter, Subscription } from 'rxjs';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent
  implements OnInit, OnDestroy {

  isNavigating = false;

  private subscription!: Subscription;

  constructor(
    private router: Router,
  ) {}

  ngOnInit(): void {
    this.subscription = this.router.events.pipe(
      filter(
        (event) =>
          event instanceof NavigationStart ||
          event instanceof NavigationEnd,
      ),
    )
    .subscribe((event) => {
      this.isNavigating =
        event instanceof NavigationStart;
    });
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
}
```

---

# Preloading Strategies

Angular 13 supports route preloading for lazy modules.

## Enable Preloading

```typescript
import {
  NgModule,
} from '@angular/core';

import {
  PreloadAllModules,
  RouterModule,
  Routes,
} from '@angular/router';

const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.module')
        .then((m) => m.AdminModule),
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(
      routes,
      {
        preloadingStrategy: PreloadAllModules,
      },
    ),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

# 404 Not Found Route

Always place wildcard routes last.

```typescript
const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },

  {
    path: '**',
    component: NotFoundComponent,
  },
];
```

---

# Best Practices

## Use Feature Routing Modules

Organize large applications by feature:

```text
users/
├── users.module.ts
├── users-routing.module.ts
├── pages/
├── services/
└── components/
```

---

## Lazy Load Large Features

Lazy load admin areas, dashboards, and large modules.

```typescript
{
  path: 'admin',
  loadChildren: () =>
    import('./admin/admin.module')
      .then((m) => m.AdminModule),
}
```

---

## Use Route Resolvers for Required Data

Avoid loading screens inside components when the route depends on required data.

```typescript
resolve: {
  user: UserResolver,
}
```

---

## Unsubscribe From Route Observables

Prevent memory leaks.

```typescript
private subscription!: Subscription;

ngOnDestroy(): void {
  this.subscription.unsubscribe();
}
```

---

## Prefer Snapshot for Static Params

Use `snapshot` when route params do not change during component lifecycle.

```typescript
const id = this.route.snapshot.paramMap.get('id');
```

Use subscriptions only when route params may change without recreating the component.

---