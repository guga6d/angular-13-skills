---
name: angular-http
description: Implement HTTP data fetching in Angular 13 using HttpClient, RxJS Observables, services, interceptors, and reactive state patterns. Use for API calls, request/response handling, loading states, error handling, authentication, and reusable data services. Triggers on API integration, backend communication, data loading, CRUD operations, or HTTP architecture implementation. Angular 13 does not support signal-based APIs like resource() or httpResource().
---

# Angular 13 HTTP & Data Fetching

Fetch data in Angular 13 using:

- `HttpClient`
- RxJS Observables
- Services
- Interceptors
- Reactive loading and error handling patterns

Angular 13 does NOT support:

- `resource()`
- `httpResource()`
- Signals API
- `toSignal()`
- Signal-based HTTP state management

The recommended Angular 13 approach is:

- `HttpClient`
- RxJS
- Service-based architecture
- Observable state management

---

# Importing HttpClientModule

HTTP requests require `HttpClientModule`.

```typescript
import { NgModule } from '@angular/core';

import {
  HttpClientModule,
} from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
  ],
})
export class AppModule {}
```

---

# Basic HttpClient Usage

## GET Request Example

```typescript
import { Component } from '@angular/core';

import {
  HttpClient,
} from '@angular/common/http';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-user-profile',

  template: `
    <div *ngIf="loading">
      Loading...
    </div>

    <div *ngIf="error">
      {{ error }}
    </div>

    <div *ngIf="user">
      <h1>{{ user.name }}</h1>

      <p>{{ user.email }}</p>
    </div>
  `,
})
export class UserProfileComponent {
  user: User | null = null;

  loading = false;

  error = '';

  constructor(
    private http: HttpClient
  ) {}

  ngOnInit(): void {
    this.loadUser();
  }

  loadUser(): void {
    this.loading = true;

    this.http
      .get<User>(
        '/api/users/123'
      )
      .subscribe({
        next: (user) => {
          this.user = user;

          this.loading = false;
        },

        error: () => {
          this.error =
            'Failed to load user';

          this.loading = false;
        },
      });
  }
}
```

---

# Service-Based HTTP Architecture

Angular 13 applications should centralize API communication inside services.

---

# User Service Example

```typescript
import { Injectable } from '@angular/core';

import {
  HttpClient,
} from '@angular/common/http';

import {
  Observable,
} from 'rxjs';

export interface User {
  id: number;
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root',
})
export class UserService {
  private readonly apiUrl =
    '/api/users';

  constructor(
    private http: HttpClient
  ) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(
      this.apiUrl
    );
  }

  getUser(
    id: string
  ): Observable<User> {
    return this.http.get<User>(
      `${this.apiUrl}/${id}`
    );
  }

  createUser(
    user: Partial<User>
  ): Observable<User> {
    return this.http.post<User>(
      this.apiUrl,
      user
    );
  }

  updateUser(
    id: string,
    user: Partial<User>
  ): Observable<User> {
    return this.http.put<User>(
      `${this.apiUrl}/${id}`,
      user
    );
  }

  patchUser(
    id: string,
    changes: Partial<User>
  ): Observable<User> {
    return this.http.patch<User>(
      `${this.apiUrl}/${id}`,
      changes
    );
  }

  deleteUser(
    id: string
  ): Observable<void> {
    return this.http.delete<void>(
      `${this.apiUrl}/${id}`
    );
  }
}
```

---

# HTTP Methods

## GET

```typescript
this.http.get<User[]>(
  '/api/users'
);
```

---

## POST

```typescript
this.http.post<User>(
  '/api/users',
  payload
);
```

---

## PUT

```typescript
this.http.put<User>(
  `/api/users/${id}`,
  payload
);
```

---

## PATCH

```typescript
this.http.patch<User>(
  `/api/users/${id}`,
  changes
);
```

---

## DELETE

```typescript
this.http.delete<void>(
  `/api/users/${id}`
);
```

---

# Request Options

Angular 13 `HttpClient` supports request configuration.

```typescript
this.http.get<User[]>(
  '/api/users',
  {
    headers: {
      Authorization:
        'Bearer token',

      'Content-Type':
        'application/json',
    },

    params: {
      page: '1',
      limit: '10',
      sort: 'name',
    },

    observe: 'response',

    responseType: 'json',
  }
);
```

---

# Observable-Based State Management

Angular 13 uses RxJS observables instead of signals.

---

# Using Async Pipe (Recommended)

Prefer `async` pipe over manual subscriptions when possible.

```typescript
import { Component } from '@angular/core';

import {
  Observable,
} from 'rxjs';

@Component({
  selector: 'app-users',

  template: `
    <div
      *ngIf="
        users$ | async as users
      "
    >
      <div
        *ngFor="
          let user of users
        "
      >
        {{ user.name }}
      </div>
    </div>
  `,
})
export class UsersComponent {
  users$: Observable<User[]> = this.userService.getUsers();;

  constructor(
    private userService:
      UserService
  ) {}
}
```

---

# Loading State Pattern

Use component state for loading indicators.

```typescript
loading = false;

loadUsers(): void {
  this.loading = true;

  this.userService
    .getUsers()
    .subscribe({
      next: (users) => {
        this.users = users;

        this.loading = false;
      },

      error: () => {
        this.loading = false;
      },
    });
}
```

---

# Error Handling

Use RxJS operators for reusable error handling.

---

# catchError Example

```typescript
import {
  catchError,
} from 'rxjs/operators';

import {
  throwError,
} from 'rxjs';

getUser(id: string) {
  return this.http
    .get<User>(
      `/api/users/${id}`
    )
    .pipe(
      catchError((error) => {
        console.error(
          'Error fetching user',
          error
        );

        return throwError(
          () =>
            new Error(
              'Failed to load user'
            )
        );
      })
    );
}
```

---

# Retry Requests

```typescript
import {
  retry,
} from 'rxjs/operators';

this.http
  .get<User[]>(
    '/api/users'
  )
  .pipe(retry(2));
```

---

# finalize Operator

Use `finalize()` for cleanup logic like loading states.

```typescript
import {
  finalize,
} from 'rxjs/operators';

this.loading = true;

this.http
  .get('/api/users')
  .pipe(
    finalize(() => {
      this.loading = false;
    })
  )
  .subscribe();
```

---

# switchMap for Dependent Requests

Use `switchMap` for chained HTTP calls.

```typescript
import {
  switchMap,
} from 'rxjs/operators';

this.route.params
  .pipe(
    switchMap((params) =>
      this.userService.getUser(
        params['id']
      )
    )
  )
  .subscribe((user) => {
    this.user = user;
  });
```

---

# HTTP Interceptors

Angular 13 uses class-based interceptors.

Angular 13 does NOT support:

- `HttpInterceptorFn`
- Functional interceptors
- `provideHttpClient`
- `withInterceptors`

---

# Auth Interceptor Example

```typescript
import {
  Injectable,
} from '@angular/core';

import {
  HttpEvent,
  HttpHandler,
  HttpInterceptor,
  HttpRequest,
} from '@angular/common/http';

import {
  Observable,
} from 'rxjs';

@Injectable()
export class AuthInterceptor
  implements HttpInterceptor
{
  constructor(
    private authService:
      AuthService
  ) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const token =
      this.authService.getToken();

    if (!token) {
      return next.handle(req);
    }

    const cloned =
      req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`,
        },
      });

    return next.handle(cloned);
  }
}
```

---

# Error Interceptor Example

```typescript
import {
  Injectable,
} from '@angular/core';

import {
  HttpErrorResponse,
  HttpEvent,
  HttpHandler,
  HttpInterceptor,
  HttpRequest,
} from '@angular/common/http';

import {
  Observable,
  throwError,
} from 'rxjs';

import {
  catchError,
} from 'rxjs/operators';

@Injectable()
export class ErrorInterceptor
  implements HttpInterceptor
{
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError(
        (
          error:
            HttpErrorResponse
        ) => {
          if (
            error.status ===
            401
          ) {
            console.error(
              'Unauthorized'
            );
          }

          return throwError(
            () => error
          );
        }
      )
    );
  }
}
```

---

# Logging Interceptor Example

```typescript
import {
  Injectable,
} from '@angular/core';

import {
  HttpEvent,
  HttpHandler,
  HttpInterceptor,
  HttpRequest,
} from '@angular/common/http';

import {
  Observable,
} from 'rxjs';

import {
  tap,
} from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor
  implements HttpInterceptor
{
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const started =
      Date.now();

    return next.handle(req).pipe(
      tap({
        next: () => {
          console.log(
            `${req.method} ${req.url} - ${
              Date.now() -
              started
            }ms`
          );
        },

        error: (err) => {
          console.error(
            `${req.method} ${req.url} failed`,
            err
          );
        },
      })
    );
  }
}
```

---

# Registering Interceptors

Angular 13 registers interceptors using `HTTP_INTERCEPTORS`.

```typescript
import {
  HTTP_INTERCEPTORS,
} from '@angular/common/http';

@NgModule({
  providers: [
    {
      provide:
        HTTP_INTERCEPTORS,

      useClass:
        AuthInterceptor,

      multi: true,
    },

    {
      provide:
        HTTP_INTERCEPTORS,

      useClass:
        ErrorInterceptor,

      multi: true,
    },

    {
      provide:
        HTTP_INTERCEPTORS,

      useClass:
        LoggingInterceptor,

      multi: true,
    },
  ],
})
export class AppModule {}
```

---

# HTTP Status Handling

```typescript
import {
  HttpErrorResponse,
} from '@angular/common/http';

catchError(
  (
    error:
      HttpErrorResponse
  ) => {
    switch (
      error.status
    ) {
      case 400:
        console.error(
          'Bad request'
        );
        break;

      case 401:
        console.error(
          'Unauthorized'
        );
        break;

      case 404:
        console.error(
          'Not found'
        );
        break;

      case 500:
        console.error(
          'Server error'
        );
        break;
    }

    return throwError(
      () => error
    );
  }
);
```

---

# Angular 13 Loading UI Pattern

Angular 13 uses:

- `*ngIf`
- RxJS
- Component state

instead of:

- `@if`
- `resource.status()`
- signal-based state

---

# Loading Template Example

```html
<div *ngIf="loading">
  Loading...
</div>

<div *ngIf="error">
  {{ error }}
</div>

<div *ngIf="!loading && data">
  <!-- content -->
</div>
```

---

# Unsubscribing Best Practice

Avoid memory leaks when manually subscribing.

```typescript
import {
  Subject,
} from 'rxjs';

import {
  takeUntil,
} from 'rxjs/operators';

private destroy$ =
  new Subject<void>();

ngOnInit(): void {
  this.userService
    .getUsers()
    .pipe(
      takeUntil(
        this.destroy$
      )
    )
    .subscribe();
}

ngOnDestroy(): void {
  this.destroy$.next();

  this.destroy$.complete();
}
```

---

# Angular 13 Limitations Compared to Angular v20+

The following APIs do NOT exist in Angular 13:

- `resource()`
- `httpResource()`
- Signals API
- `signal()`
- `computed()`
- `toSignal()`
- `HttpInterceptorFn`
- `provideHttpClient`
- `withInterceptors`
- `@if`
- `@for`
- `@switch`

Angular 13 alternatives:

| Angular v20+ | Angular 13 Equivalent |
|---|---|
| `httpResource()` | `HttpClient + RxJS` |
| `resource()` | `Observable + Service` |
| Signals | RxJS Subjects/Observables |
| Functional interceptors | Class interceptors |
| `@if` | `*ngIf` |
| `@for` | `*ngFor` |
| `@switch` | `ngSwitch` |

---

# Recommended Angular 13 HTTP Architecture

For scalable Angular 13 applications:

- `HttpClient`
- Service layer abstraction
- RxJS operators
- Async pipe
- Interceptors
- Centralized error handling
- Observable state management
- Environment-based API URLs
- Reusable data services

This architecture is production-stable and widely adopted in enterprise Angular 13 applications.