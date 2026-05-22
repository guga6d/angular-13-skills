---
name: angular-routing-patterns
description: Implement advanced routing patterns in Angular 13 applications using RouterModule, lazy loaded modules, class-based guards, resolvers, breadcrumbs, tab navigation, modal routes, preloading strategies, route animations, and scroll restoration. Use for enterprise routing architecture, authentication flows, advanced navigation patterns, and scalable Angular 13 routing setups.
---

# Angular 13 Routing Patterns

## Table of Contents
- [Route Configuration Options](#route-configuration-options)
- [Authentication Flow](#authentication-flow)
- [Breadcrumbs](#breadcrumbs)
- [Tab Navigation](#tab-navigation)
- [Modal Routes](#modal-routes)
- [Preloading Strategies](#preloading-strategies)
- [Route Animations](#route-animations)
- [Scroll Position Restoration](#scroll-position-restoration)
- [Angular 13 Routing Best Practices](#angular-13-routing-best-practices)

---

# Route Configuration Options

## Full Route Options

Angular 13 routing is configured using the `Routes` array and `RouterModule.forRoot()`.

```typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { UserComponent } from './pages/user/user.component';
import { AuthGuard } from './guards/auth.guard';
import { UnsavedChangesGuard } from './guards/unsaved-changes.guard';
import { FeatureFlagGuard } from './guards/feature-flag.guard';
import { UserResolver } from './resolvers/user.resolver';

const routes: Routes = [
  {
    path: 'users/:id',

    // Component
    component: UserComponent,

    // Lazy loading module
    loadChildren: () =>
      import('./features/user/user.module').then(
        (m) => m.UserModule
      ),

    // Guards
    canActivate: [AuthGuard],
    canActivateChild: [AuthGuard],
    canDeactivate: [UnsavedChangesGuard],
    canLoad: [FeatureFlagGuard],

    // Resolver
    resolve: {
      user: UserResolver,
    },

    // Route data
    data: {
      title: 'User Profile',
      animation: 'userPage',
    },

    // Child routes
    children: [],

    // Named outlet
    outlet: 'sidebar',

    // Path matching
    pathMatch: 'full',
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

## Dynamic Page Titles

Angular 13 does not support route title resolvers directly.

Use a resolver + `Title` service.

```typescript
import { Injectable } from '@angular/core';
import {
  Resolve,
  ActivatedRouteSnapshot,
} from '@angular/router';

import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

import { UserService } from '../services/user.service';

@Injectable({
  providedIn: 'root',
})
export class UserTitleResolver implements Resolve<string> {
  constructor(private userService: UserService) {}

  resolve(
    route: ActivatedRouteSnapshot
  ): Observable<string> {
    const id = route.paramMap.get('id') as string;

    return this.userService.getById(id).pipe(
      map((user) => `${user.name} - Profile`)
    );
  }
}
```

### Route Configuration

```typescript
{
  path: 'users/:id',
  component: UserComponent,
  resolve: {
    pageTitle: UserTitleResolver,
  },
}
```

### Set Document Title

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Title } from '@angular/platform-browser';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
})
export class UserComponent implements OnInit {
  constructor(
    private route: ActivatedRoute,
    private title: Title
  ) {}

  ngOnInit(): void {
    const pageTitle =
      this.route.snapshot.data['pageTitle'];

    this.title.setTitle(pageTitle);
  }
}
```

---

# Authentication Flow

## Auth Service Pattern

```typescript
import { Injectable } from '@angular/core';
import { Router } from '@angular/router';
import { HttpClient } from '@angular/common/http';

import { BehaviorSubject, Observable, of } from 'rxjs';
import { tap, map } from 'rxjs/operators';

export interface User {
  id: string;
  name: string;
  email: string;
}

export interface Credentials {
  email: string;
  password: string;
}

export interface AuthResponse {
  token: string;
  user: User;
}

@Injectable({
  providedIn: 'root',
})
export class AuthService {
  private userSubject =
    new BehaviorSubject<User | null>(null);

  private tokenSubject =
    new BehaviorSubject<string | null>(null);

  user$ = this.userSubject.asObservable();

  isAuthenticated$ = this.user$.pipe(
    map((user) => !!user)
  );

  constructor(
    private http: HttpClient,
    private router: Router
  ) {}

  login(
    credentials: Credentials
  ): Observable<AuthResponse> {
    return this.http
      .post<AuthResponse>(
        '/api/login',
        credentials
      )
      .pipe(
        tap((response) => {
          localStorage.setItem(
            'token',
            response.token
          );

          this.tokenSubject.next(response.token);
          this.userSubject.next(response.user);
        })
      );
  }

  logout(): void {
    localStorage.removeItem('token');

    this.userSubject.next(null);
    this.tokenSubject.next(null);

    this.router.navigate(['/login']);
  }

  checkAuth(): Observable<boolean> {
    const token = localStorage.getItem('token');

    if (!token) {
      return of(false);
    }

    return this.http
      .get<User>('/api/me')
      .pipe(
        tap((user) => {
          this.tokenSubject.next(token);
          this.userSubject.next(user);
        }),
        map(() => true)
      );
  }

  get currentUser(): User | null {
    return this.userSubject.value;
  }

  get token(): string | null {
    return this.tokenSubject.value;
  }
}
```

---

## Auth Guard

Angular 13 uses class-based guards.

```typescript
import { Injectable } from '@angular/core';

import {
  CanActivate,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router,
  UrlTree,
} from '@angular/router';

import { Observable, of } from 'rxjs';
import { switchMap, catchError } from 'rxjs/operators';

import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ):
    | Observable<boolean | UrlTree>
    | boolean
    | UrlTree {

    if (this.authService.currentUser) {
      return true;
    }

    return this.authService.checkAuth().pipe(
      switchMap((isValid) => {
        if (isValid) {
          return of(true);
        }

        return of(
          this.router.createUrlTree(
            ['/login'],
            {
              queryParams: {
                returnUrl: state.url,
              },
            }
          )
        );
      }),
      catchError(() => {
        return of(
          this.router.createUrlTree(
            ['/login']
          )
        );
      })
    );
  }
}
```

---

## Login Component Pattern

```typescript
import { Component } from '@angular/core';

import {
  ActivatedRoute,
  Router,
} from '@angular/router';

import {
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';

import { AuthService } from '../services/auth.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
})
export class LoginComponent {
  form: FormGroup;

  isSubmitting = false;
  errorMessage = '';

  constructor(
    private fb: FormBuilder,
    private authService: AuthService,
    private router: Router,
    private route: ActivatedRoute
  ) {
    this.form = this.fb.group({
      email: [
        '',
        [Validators.required, Validators.email],
      ],
      password: [
        '',
        [Validators.required],
      ],
    });
  }

  login(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.isSubmitting = true;

    this.authService
      .login(this.form.value)
      .subscribe({
        next: () => {
          const returnUrl =
            this.route.snapshot.queryParams[
              'returnUrl'
            ] || '/';

          this.router.navigateByUrl(returnUrl);
        },
        error: () => {
          this.errorMessage =
            'Invalid email or password';

          this.isSubmitting = false;
        },
      });
  }
}
```

---

# Breadcrumbs

## Breadcrumb Service

```typescript
import { Injectable } from '@angular/core';

import {
  ActivatedRoute,
  NavigationEnd,
  Router,
} from '@angular/router';

import { BehaviorSubject } from 'rxjs';

import { filter } from 'rxjs/operators';

export interface Breadcrumb {
  label: string;
  url: string;
}

@Injectable({
  providedIn: 'root',
})
export class BreadcrumbService {
  private breadcrumbsSubject =
    new BehaviorSubject<Breadcrumb[]>([]);

  breadcrumbs$ =
    this.breadcrumbsSubject.asObservable();

  constructor(
    private router: Router,
    private route: ActivatedRoute
  ) {
    this.router.events
      .pipe(
        filter(
          (event) =>
            event instanceof NavigationEnd
        )
      )
      .subscribe(() => {
        const breadcrumbs =
          this.buildBreadcrumbs(
            this.route.root
          );

        this.breadcrumbsSubject.next(
          breadcrumbs
        );
      });
  }

  private buildBreadcrumbs(
    route: ActivatedRoute,
    url: string = '',
    breadcrumbs: Breadcrumb[] = []
  ): Breadcrumb[] {

    const children = route.children;

    if (children.length === 0) {
      return breadcrumbs;
    }

    for (const child of children) {
      const routeUrl =
        child.snapshot.url
          .map((segment) => segment.path)
          .join('/');

      if (routeUrl) {
        url += `/${routeUrl}`;
      }

      const label =
        child.snapshot.data['breadcrumb'];

      if (label) {
        breadcrumbs.push({
          label,
          url,
        });
      }

      return this.buildBreadcrumbs(
        child,
        url,
        breadcrumbs
      );
    }

    return breadcrumbs;
  }
}
```

---

## Route Configuration

```typescript
const routes: Routes = [
  {
    path: 'products',
    data: {
      breadcrumb: 'Products',
    },
    children: [
      {
        path: '',
        component: ProductListComponent,
      },
      {
        path: ':id',
        component: ProductDetailComponent,
        data: {
          breadcrumb: 'Product Details',
        },
      },
    ],
  },
];
```

---

## Breadcrumb Component

```typescript
import { Component } from '@angular/core';

import { Observable } from 'rxjs';

import {
  Breadcrumb,
  BreadcrumbService,
} from './breadcrumb.service';

@Component({
  selector: 'app-breadcrumb',
  template: `
    <nav aria-label="Breadcrumb">
      <ol>
        <li>
          <a routerLink="/">Home</a>
        </li>

        <li
          *ngFor="
            let crumb of breadcrumbs$ | async
          "
        >
          <a [routerLink]="crumb.url">
            {{ crumb.label }}
          </a>
        </li>
      </ol>
    </nav>
  `,
})
export class BreadcrumbComponent {
  breadcrumbs$: Observable<Breadcrumb[]> =
    this.breadcrumbService.breadcrumbs$;

  constructor(
    private breadcrumbService: BreadcrumbService
  ) {}
}
```

---

# Tab Navigation

## Tabs Layout Pattern

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-tabs-layout',
  template: `
    <div class="tabs">
      <a
        routerLink="./"
        routerLinkActive="active"
        [routerLinkActiveOptions]="{
          exact: true
        }"
      >
        Overview
      </a>

      <a
        routerLink="details"
        routerLinkActive="active"
      >
        Details
      </a>

      <a
        routerLink="settings"
        routerLinkActive="active"
      >
        Settings
      </a>
    </div>

    <div class="tab-content">
      <router-outlet></router-outlet>
    </div>
  `,
})
export class TabsLayoutComponent {}
```

---

## Routes

```typescript
{
  path: 'account',
  component: TabsLayoutComponent,
  children: [
    {
      path: '',
      component: AccountOverviewComponent,
    },
    {
      path: 'details',
      component: AccountDetailsComponent,
    },
    {
      path: 'settings',
      component: AccountSettingsComponent,
    },
  ],
}
```

---

# Modal Routes

Angular 13 supports auxiliary outlets.

## Route Configuration

```typescript
const routes: Routes = [
  {
    path: 'products',
    component: ProductListComponent,
  },
  {
    path: 'product-modal/:id',
    component: ProductModalComponent,
    outlet: 'modal',
  },
];
```

---

## App Component

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <router-outlet></router-outlet>

    <router-outlet
      name="modal"
    ></router-outlet>
  `,
})
export class AppComponent {}
```

---

## Open Modal Route

```typescript
this.router.navigate([
  {
    outlets: {
      modal: [
        'product-modal',
        productId,
      ],
    },
  },
]);
```

---

## Close Modal Route

```typescript
this.router.navigate([
  {
    outlets: {
      modal: null,
    },
  },
]);
```

---

## RouterLink Example

```html
<a
  [routerLink]="[
    {
      outlets: {
        modal: [
          'product-modal',
          product.id
        ]
      }
    }
  ]"
>
  View Details
</a>
```

---

# Preloading Strategies

## Preload All Lazy Modules

```typescript
import { NgModule } from '@angular/core';

import {
  PreloadAllModules,
  RouterModule,
  Routes,
} from '@angular/router';

const routes: Routes = [];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy:
        PreloadAllModules,
    }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

## Disable Preloading

```typescript
import { NoPreloading } from '@angular/router';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: NoPreloading,
    }),
  ],
})
export class AppRoutingModule {}
```

---

## Custom Preloading Strategy

```typescript
import { Injectable } from '@angular/core';

import {
  Route,
  PreloadingStrategy,
} from '@angular/router';

import {
  Observable,
  of,
} from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class SelectivePreloadStrategy
  implements PreloadingStrategy {

  preload(
    route: Route,
    load: () => Observable<any>
  ): Observable<any> {

    if (route.data?.['preload']) {
      return load();
    }

    return of(null);
  }
}
```

---

## Route Configuration

```typescript
{
  path: 'dashboard',
  loadChildren: () =>
    import('./dashboard/dashboard.module')
      .then((m) => m.DashboardModule),

  data: {
    preload: true,
  },
}
```

---

## Register Custom Strategy

```typescript
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy:
        SelectivePreloadStrategy,
    }),
  ],
})
export class AppRoutingModule {}
```

---

# Route Animations

## Route Configuration

```typescript
const routes: Routes = [
  {
    path: 'home',
    component: HomeComponent,
    data: {
      animation: 'HomePage',
    },
  },
  {
    path: 'about',
    component: AboutComponent,
    data: {
      animation: 'AboutPage',
    },
  },
];
```

---

## App Component Animations

```typescript
import { Component } from '@angular/core';

import {
  animate,
  animateChild,
  group,
  query,
  style,
  transition,
  trigger,
} from '@angular/animations';

import {
  RouterOutlet,
} from '@angular/router';

@Component({
  selector: 'app-root',
  template: `
    <div
      [@routeAnimations]="
        prepareRoute(outlet)
      "
    >
      <router-outlet
        #outlet="outlet"
      ></router-outlet>
    </div>
  `,
  animations: [
    trigger('routeAnimations', [
      transition(
        'HomePage <=> AboutPage',
        [
          style({
            position: 'relative',
          }),

          query(
            ':enter, :leave',
            [
              style({
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
              }),
            ],
            {
              optional: true,
            }
          ),

          query(
            ':enter',
            [
              style({
                left: '-100%',
              }),
            ],
            {
              optional: true,
            }
          ),

          query(
            ':leave',
            animateChild(),
            {
              optional: true,
            }
          ),

          group([
            query(
              ':leave',
              [
                animate(
                  '300ms ease-out',
                  style({
                    left: '100%',
                  })
                ),
              ],
              {
                optional: true,
              }
            ),

            query(
              ':enter',
              [
                animate(
                  '300ms ease-out',
                  style({
                    left: '0%',
                  })
                ),
              ],
              {
                optional: true,
              }
            ),
          ]),
        ]
      ),
    ]),
  ],
})
export class AppComponent {
  prepareRoute(
    outlet: RouterOutlet
  ): string {
    return (
      outlet &&
      outlet.activatedRouteData &&
      outlet.activatedRouteData[
        'animation'
      ]
    );
  }
}
```

---

# Scroll Position Restoration

Angular 13 supports scroll restoration via router options.

```typescript
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      scrollPositionRestoration:
        'enabled',

      anchorScrolling:
        'enabled',

      onSameUrlNavigation:
        'reload',
    }),
  ],
})
export class AppRoutingModule {}
```

## Available Options

```typescript
scrollPositionRestoration:
  | 'disabled'
  | 'enabled'
  | 'top';

anchorScrolling:
  | 'disabled'
  | 'enabled';

onSameUrlNavigation:
  | 'ignore'
  | 'reload';
```

---

# Angular 13 Routing Best Practices

## Use Feature Routing Modules

```text
features/
├── users/
│   ├── users-routing.module.ts
│   ├── users.module.ts
│   └── pages/
├── admin/
│   ├── admin-routing.module.ts
│   ├── admin.module.ts
│   └── pages/
```

---

## Prefer Lazy Loading

Use lazy loading for:

- Admin areas
- Large feature modules
- Dashboard modules
- Settings modules
- Report modules

---

## Keep Guards Lightweight

Guards should:

- Validate permissions
- Redirect users
- Avoid heavy business logic
- Avoid long-running requests

---

## Use Resolvers for Required Data

Resolvers are ideal when:

- The page cannot render without data
- SEO matters
- You want to avoid loading flickers
- Route activation depends on backend data

---

## Centralize Authentication Logic

Keep authentication logic inside:

```text
core/
├── services/
│   └── auth.service.ts
├── guards/
│   └── auth.guard.ts
├── interceptors/
│   └── auth.interceptor.ts
```

---

## Use Named Outlets Sparingly

Named outlets are useful for:

- Modal routing
- Sidebars
- Chat panels
- Secondary layouts

Avoid overusing them in standard navigation flows.

---

## Organize Routes by Feature

```text
app/
├── core/
├── shared/
├── features/
│   ├── auth/
│   ├── users/
│   ├── dashboard/
│   └── products/
```

---

## Use Route Data for Metadata

Prefer route data for:

- Page titles
- Breadcrumb labels
- Route animations
- Role requirements
- Feature flags