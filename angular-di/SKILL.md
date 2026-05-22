---
name: angular-di
description: Implement dependency injection in Angular 13 using constructor injection, injection tokens, NgModule providers, and provider configuration. Use for service architecture, configuring providers, creating injectable tokens, and managing singleton vs scoped services in Angular 13 applications. Triggers on service creation, configuring providers, using injection tokens, or understanding Angular 13 DI hierarchy.
---

# Angular 13 Dependency Injection

Configure and use dependency injection in Angular 13 using:
- Constructor injection
- `@Injectable`
- Injection tokens
- NgModule providers
- Component-level providers

Angular 13 does NOT support:
- `provideAppInitializer`
- `ApplicationConfig`
- `EnvironmentInjector`
- `createEnvironmentInjector`
- `runInInjectionContext`
- Functional interceptors
- Modern `inject()`-first patterns

Prefer classic Angular dependency injection patterns.

---

# Basic Injection

## Constructor Injection

Angular 13 primarily uses constructor injection.

```typescript
import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';

import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `...`,
})
export class UserListComponent {
  users$ =
    this.userService.getUsers();

  constructor(
    private http: HttpClient,
    private userService: UserService,
  ) {}
}
```

---

# Injectable Services

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

import {
  BehaviorSubject,
  firstValueFrom,
} from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class UserService {
  private usersSubject =
    new BehaviorSubject<User[]>([]);

  users$ =
    this.usersSubject.asObservable();

  constructor(
    private http: HttpClient,
  ) {}

  async loadUsers(): Promise<void> {
    const users =
      await firstValueFrom(
        this.http.get<User[]>(
          '/api/users',
        ),
      );

    this.usersSubject.next(users);
  }
}
```

---

# Provider Scopes

## Root Level (Singleton)

Prefer `providedIn: 'root'`.

```typescript
@Injectable({
  providedIn: 'root',
})
export class AuthService {}
```

---

## NgModule Providers

```typescript
@NgModule({
  providers: [
    AuthService,
  ],
})
export class CoreModule {}
```

---

## Component Level Providers

Each component instance receives its own service instance.

```typescript
@Component({
  selector: 'app-editor',
  template: `...`,
  providers: [
    EditorStateService,
  ],
})
export class EditorComponent {
  constructor(
    private editorState:
      EditorStateService,
  ) {}
}
```

---

## Lazy Loaded Module Scope

Services provided inside lazy modules are scoped to that module.

```typescript
@NgModule({
  providers: [
    AdminService,
  ],
})
export class AdminModule {}
```

---

# Injection Tokens

## Creating Tokens

```typescript
import {
  InjectionToken,
} from '@angular/core';

export const API_URL =
  new InjectionToken<string>(
    'API_URL',
  );

export interface AppConfig {
  apiUrl: string;

  features: {
    darkMode: boolean;
    analytics: boolean;
  };
}

export const APP_CONFIG =
  new InjectionToken<AppConfig>(
    'APP_CONFIG',
  );
```

---

## Providing Token Values

```typescript
@NgModule({
  providers: [
    {
      provide: API_URL,
      useValue:
        'https://api.example.com',
    },

    {
      provide: APP_CONFIG,
      useValue: {
        apiUrl:
          'https://api.example.com',

        features: {
          darkMode: true,
          analytics: true,
        },
      },
    },
  ],
})
export class CoreModule {}
```

---

## Injecting Tokens

Use `@Inject()`.

```typescript
import {
  Injectable,
  Inject,
} from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  constructor(
    @Inject(API_URL)
    private apiUrl: string,

    @Inject(APP_CONFIG)
    private config: AppConfig,
  ) {}

  getBaseUrl(): string {
    return this.apiUrl;
  }
}
```

---

# Provider Types

## useClass

```typescript
providers: [
  {
    provide: LoggerService,
    useClass: ConsoleLoggerService,
  },
]
```

## Conditional Implementation

```typescript
providers: [
  {
    provide: LoggerService,
    useClass:
      environment.production
        ? ProductionLoggerService
        : ConsoleLoggerService,
  },
]
```

---

# useValue

```typescript
providers: [
  {
    provide: API_URL,
    useValue:
      'https://api.example.com',
  },

  {
    provide: APP_CONFIG,
    useValue: {
      theme: 'dark',
      language: 'en',
    },
  },
]
```

---

# useFactory

Factory providers can inject dependencies using `deps`.

```typescript
providers: [
  {
    provide: UserService,

    useFactory: (
      http: HttpClient,
      config: AppConfig,
    ) => {
      return new UserService(
        http,
        config.apiUrl,
      );
    },

    deps: [
      HttpClient,
      APP_CONFIG,
    ],
  },
]
```

---

# useExisting

Alias one provider to another.

```typescript
providers: [
  ConsoleLoggerService,

  {
    provide: LoggerService,
    useExisting:
      ConsoleLoggerService,
  },

  {
    provide: ErrorLoggerService,
    useExisting:
      ConsoleLoggerService,
  },
]
```

---

# Optional Injection

Use `@Optional()`.

```typescript
import {
  Optional,
} from '@angular/core';

@Component({
  selector: 'app-example',
  template: `...`,
})
export class ExampleComponent {
  constructor(
    @Optional()
    private analytics?:
      AnalyticsService,
  ) {}

  trackEvent(name: string): void {
    this.analytics?.track(name);
  }
}
```

---

# Self, SkipSelf and Host

Angular 13 uses decorators instead of injection options.

---

## @Self()

Only search current injector.

```typescript
import {
  Self,
} from '@angular/core';

constructor(
  @Self()
  private localService:
    LocalService,
) {}
```

---

## @SkipSelf()

Skip current injector and search parent.

```typescript
import {
  SkipSelf,
} from '@angular/core';

constructor(
  @SkipSelf()
  private parentService:
    ParentService,
) {}
```

---

## @Host()

Limit resolution to host element injector tree.

```typescript
import {
  Host,
} from '@angular/core';

constructor(
  @Host()
  private hostService:
    HostService,
) {}
```

---

# Multi Providers

Collect multiple values under the same token.

```typescript
export const VALIDATORS =
  new InjectionToken<Validator[]>(
    'VALIDATORS',
  );
```

## Providing Multiple Values

```typescript
providers: [
  {
    provide: VALIDATORS,
    useClass:
      RequiredValidator,
    multi: true,
  },

  {
    provide: VALIDATORS,
    useClass:
      EmailValidator,
    multi: true,
  },

  {
    provide: VALIDATORS,
    useClass:
      MinLengthValidator,
    multi: true,
  },
]
```

---

## Injecting Multi Providers

```typescript
@Injectable()
export class ValidationService {
  constructor(
    @Inject(VALIDATORS)
    private validators:
      Validator[],
  ) {}

  validate(
    value: string,
  ): ValidationError[] {
    return this.validators
      .map(v =>
        v.validate(value),
      )
      .filter(Boolean);
  }
}
```

---

# HTTP Interceptors

Angular 13 uses class-based interceptors.

```typescript
import {
  Injectable,
} from '@angular/core';

import {
  HttpInterceptor,
  HttpRequest,
  HttpHandler,
  HttpEvent,
} from '@angular/common/http';

import { Observable } from 'rxjs';

@Injectable()
export class AuthInterceptor
  implements HttpInterceptor {

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler,
  ): Observable<HttpEvent<any>> {

    const cloned =
      req.clone({
        setHeaders: {
          Authorization:
            'Bearer token',
        },
      });

    return next.handle(cloned);
  }
}
```

## Registering Interceptors

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
  ],
})
export class CoreModule {}
```

---

# APP_INITIALIZER

Angular 13 uses `APP_INITIALIZER`.

```typescript
import {
  APP_INITIALIZER,
} from '@angular/core';

export function initializeApp(
  config: ConfigService,
): () => Promise<void> {

  return () =>
    config.loadConfig();
}

@NgModule({
  providers: [
    ConfigService,

    {
      provide:
        APP_INITIALIZER,

      useFactory:
        initializeApp,

      deps: [
        ConfigService,
      ],

      multi: true,
    },
  ],
})
export class AppModule {}
```

---

# Multiple Initializers

```typescript
providers: [
  {
    provide:
      APP_INITIALIZER,

    useFactory:
      initializeConfig,

    deps: [
      ConfigService,
    ],

    multi: true,
  },

  {
    provide:
      APP_INITIALIZER,

    useFactory:
      initializeAuth,

    deps: [
      AuthService,
    ],

    multi: true,
  },
]
```

---

# Angular 13 DI Best Practices

Prefer:
- Constructor injection
- `providedIn: 'root'`
- Injection tokens for configuration
- Feature module isolation
- RxJS-based shared state
- Class-based interceptors
- APP_INITIALIZER for startup logic

Avoid:
- Service locator patterns
- Circular dependencies
- Large shared mutable services
- Manual injector usage
- Providing singleton services in components
- Excessive global state
- Using experimental DI APIs in Angular 13
- Direct DOM access inside services