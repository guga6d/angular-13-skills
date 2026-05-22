# Angular 13 Dependency Injection Patterns

## Table of Contents
- [Service Patterns](#service-patterns)
- [Abstract Classes as Tokens](#abstract-classes-as-tokens)
- [Hierarchical Injection](#hierarchical-injection)
- [Dynamic Providers](#dynamic-providers)
- [Testing with DI](#testing-with-di)
- [Cleanup and Subscription Management](#cleanup-and-subscription-management)

---

# Service Patterns

## Facade Service Pattern

Use facade services to simplify communication between components and multiple services.

Angular 13 uses:
- Constructor injection
- RxJS Observables
- BehaviorSubject

instead of Signals.

```typescript
@Injectable({
  providedIn: 'root',
})
export class ShopFacadeService {
  readonly products$ =
    this.productService.products$;

  readonly cart$ =
    this.cartService.items$;

  readonly cartTotal$ =
    this.cartService.total$;

  constructor(
    private productService:
      ProductService,

    private cartService:
      CartService,

    private orderService:
      OrderService,
  ) {}

  addToCart(
    productId: string,
    quantity: number,
  ): void {

    const product =
      this.productService.getById(
        productId,
      );

    if (product) {
      this.cartService.add(
        product,
        quantity,
      );
    }
  }

  async checkout():
    Promise<Order> {

    const items =
      await firstValueFrom(
        this.cartService.items$,
      );

    const order =
      await this.orderService.create(
        items,
      );

    this.cartService.clear();

    return order;
  }
}
```

---

# State Service Pattern

Angular 13 applications commonly use:
- `BehaviorSubject`
- `Observable`
- immutable state updates

instead of Signals and `computed()`.

```typescript
import {
  Injectable,
} from '@angular/core';

import {
  BehaviorSubject,
  Observable,
} from 'rxjs';

import {
  map,
} from 'rxjs/operators';

export interface UserStateModel {
  user: User | null;

  loading: boolean;

  error: string | null;
}

@Injectable({
  providedIn: 'root',
})
export class UserStateService {
  private stateSubject =
    new BehaviorSubject<UserStateModel>({
      user: null,
      loading: false,
      error: null,
    });

  readonly state$ =
    this.stateSubject.asObservable();

  readonly user$ =
    this.state$.pipe(
      map(state => state.user),
    );

  readonly loading$ =
    this.state$.pipe(
      map(state => state.loading),
    );

  readonly error$ =
    this.state$.pipe(
      map(state => state.error),
    );

  readonly isAuthenticated$ =
    this.user$.pipe(
      map(user => !!user),
    );

  setUser(user: User): void {
    this.updateState({
      user,
      loading: false,
      error: null,
    });
  }

  setLoading(): void {
    this.updateState({
      loading: true,
      error: null,
    });
  }

  setError(error: string): void {
    this.updateState({
      loading: false,
      error,
    });
  }

  clear(): void {
    this.stateSubject.next({
      user: null,
      loading: false,
      error: null,
    });
  }

  private updateState(
    partial:
      Partial<UserStateModel>,
  ): void {

    this.stateSubject.next({
      ...this.stateSubject.value,
      ...partial,
    });
  }
}
```

---

# Repository Pattern

Use abstract classes or interfaces for repositories.

```typescript
export abstract class Repository<
  T extends { id: string }
> {
  abstract getAll():
    Promise<T[]>;

  abstract getById(
    id: string,
  ): Promise<T | null>;

  abstract create(
    item: Omit<T, 'id'>,
  ): Promise<T>;

  abstract update(
    id: string,
    item: Partial<T>,
  ): Promise<T>;

  abstract delete(
    id: string,
  ): Promise<void>;
}
```

---

## HTTP Repository Implementation

```typescript
@Injectable()
export class HttpUserRepository
  extends Repository<User> {

  constructor(
    private http: HttpClient,

    @Inject(API_URL)
    private apiUrl: string,
  ) {
    super();
  }

  async getAll():
    Promise<User[]> {

    return firstValueFrom(
      this.http.get<User[]>(
        `${this.apiUrl}/users`,
      ),
    );
  }

  async getById(
    id: string,
  ): Promise<User | null> {

    return firstValueFrom(
      this.http
        .get<User>(
          `${this.apiUrl}/users/${id}`,
        )
        .pipe(
          catchError(() =>
            of(null),
          ),
        ),
    );
  }

  async create(
    user: Omit<User, 'id'>,
  ): Promise<User> {

    return firstValueFrom(
      this.http.post<User>(
        `${this.apiUrl}/users`,
        user,
      ),
    );
  }

  async update(
    id: string,
    user: Partial<User>,
  ): Promise<User> {

    return firstValueFrom(
      this.http.patch<User>(
        `${this.apiUrl}/users/${id}`,
        user,
      ),
    );
  }

  async delete(
    id: string,
  ): Promise<void> {

    await firstValueFrom(
      this.http.delete(
        `${this.apiUrl}/users/${id}`,
      ),
    );
  }
}
```

---

## Providing Repository Implementations

```typescript
providers: [
  {
    provide: Repository,
    useClass:
      HttpUserRepository,
  },
]
```

---

# Abstract Classes as Tokens

Abstract classes work well as Angular DI tokens.

```typescript
export abstract class LoggerService {
  abstract log(
    message: string,
  ): void;

  abstract error(
    message: string,
    error?: Error,
  ): void;

  abstract warn(
    message: string,
  ): void;
}
```

---

## Console Logger

```typescript
@Injectable()
export class ConsoleLoggerService
  extends LoggerService {

  log(message: string): void {
    console.log(
      `[LOG] ${message}`,
    );
  }

  error(
    message: string,
    error?: Error,
  ): void {

    console.error(
      `[ERROR] ${message}`,
      error,
    );
  }

  warn(message: string): void {
    console.warn(
      `[WARN] ${message}`,
    );
  }
}
```

---

## Remote Logger

```typescript
@Injectable()
export class RemoteLoggerService
  extends LoggerService {

  constructor(
    private http: HttpClient,
  ) {
    super();
  }

  log(message: string): void {
    this.send(
      'log',
      message,
    );
  }

  error(
    message: string,
    error?: Error,
  ): void {

    this.send(
      'error',
      message,
      error,
    );
  }

  warn(message: string): void {
    this.send(
      'warn',
      message,
    );
  }

  private send(
    level: string,
    message: string,
    error?: Error,
  ): void {

    this.http.post(
      '/api/logs',
      {
        level,
        message,
        error:
          error?.message,
      },
    ).subscribe();
  }
}
```

---

## Environment-Based Provider

```typescript
providers: [
  {
    provide: LoggerService,

    useClass:
      environment.production
        ? RemoteLoggerService
        : ConsoleLoggerService,
  },
]
```

---

## Injecting Abstract Services

```typescript
@Injectable({
  providedIn: 'root',
})
export class UserService {
  constructor(
    private logger:
      LoggerService,
  ) {}

  createUser(
    user: UserData,
  ): void {

    this.logger.log(
      `Creating user:
       ${user.email}`,
    );

    // ...
  }
}
```

---

# Hierarchical Injection

## Component Tree Injection

Services provided in a parent component are shared by all descendants.

```typescript
@Component({
  selector:
    'app-form-container',

  providers: [
    FormStateService,
  ],

  template: `
    <app-form-header>
    </app-form-header>

    <app-form-body>
    </app-form-body>

    <app-form-footer>
    </app-form-footer>
  `,
})
export class FormContainerComponent {
  constructor(
    private formState:
      FormStateService,
  ) {}
}
```

---

## Child Component

```typescript
@Component({
  selector: 'app-form-body',
  template: `...`,
})
export class FormBodyComponent {
  constructor(
    private formState:
      FormStateService,
  ) {}
}
```

---

## viewProviders vs providers

```typescript
@Component({
  selector: 'app-tabs',

  // Shared with view
  // children and
  // projected content
  providers: [
    TabsService,
  ],

  // Shared only with
  // internal view children
  viewProviders: [
    InternalTabsService,
  ],

  template: `
    <div class="tabs">
      <ng-content>
      </ng-content>
    </div>
  `,
})
export class TabsComponent {}
```

### providers
Available to:
- Component itself
- View children
- Content children (`ng-content`)

### viewProviders
Available to:
- Component itself
- View children only

NOT available to:
- Projected content

---

# Dynamic Providers

## Feature Flags

Use injection tokens with factory providers.

```typescript
export interface FeatureFlags {
  newDashboard: boolean;

  betaFeatures: boolean;

  experimentalApi: boolean;
}

export const FEATURE_FLAGS =
  new InjectionToken<FeatureFlags>(
    'FEATURE_FLAGS',
  );
```

---

## Factory Provider

```typescript
providers: [
  {
    provide: FEATURE_FLAGS,

    useFactory: (
      http: HttpClient,
    ) => {
      return {
        newDashboard: true,
        betaFeatures: false,
        experimentalApi: false,
      };
    },

    deps: [
      HttpClient,
    ],
  },
]
```

---

## Using Feature Flags

```typescript
@Component({
  selector: 'app-dashboard',
  template: `...`,
})
export class DashboardComponent {
  showNewDashboard = false;

  constructor(
    @Inject(FEATURE_FLAGS)
    private features:
      FeatureFlags,
  ) {
    this.showNewDashboard =
      this.features.newDashboard;
  }
}
```

---

# Platform-Specific Services

```typescript
export abstract class StorageService {
  abstract get(
    key: string,
  ): string | null;

  abstract set(
    key: string,
    value: string,
  ): void;

  abstract remove(
    key: string,
  ): void;
}
```

---

## Browser Implementation

```typescript
@Injectable()
export class BrowserStorageService
  extends StorageService {

  get(
    key: string,
  ): string | null {

    return localStorage.getItem(
      key,
    );
  }

  set(
    key: string,
    value: string,
  ): void {

    localStorage.setItem(
      key,
      value,
    );
  }

  remove(
    key: string,
  ): void {

    localStorage.removeItem(
      key,
    );
  }
}
```

---

## Server Implementation

```typescript
@Injectable()
export class ServerStorageService
  extends StorageService {

  private store =
    new Map<string, string>();

  get(
    key: string,
  ): string | null {

    return (
      this.store.get(key)
      ?? null
    );
  }

  set(
    key: string,
    value: string,
  ): void {

    this.store.set(
      key,
      value,
    );
  }

  remove(
    key: string,
  ): void {

    this.store.delete(key);
  }
}
```

---

## Platform Factory Provider

```typescript
import {
  PLATFORM_ID,
} from '@angular/core';

import {
  isPlatformBrowser,
} from '@angular/common';

providers: [
  {
    provide: StorageService,

    useFactory: (
      platformId: object,
    ) => {

      return isPlatformBrowser(
        platformId,
      )
        ? new BrowserStorageService()
        : new ServerStorageService();
    },

    deps: [
      PLATFORM_ID,
    ],
  },
]
```

---

# Testing with DI

Angular 13 uses:
- `TestBed`
- Jasmine spies
- `overrideProvider`

---

# Mocking Services

```typescript
describe(
  'UserComponent',
  () => {

    let userServiceSpy:
      jasmine.SpyObj<UserService>;

    beforeEach(
      async () => {

        userServiceSpy =
          jasmine.createSpyObj(
            'UserService',
            [
              'getUser',
              'updateUser',
            ],
          );

        userServiceSpy
          .getUser
          .and
          .returnValue(
            of({
              id: '1',
              name: 'Test',
            }),
          );

        await TestBed
          .configureTestingModule({
            declarations: [
              UserComponent,
            ],

            providers: [
              {
                provide:
                  UserService,

                useValue:
                  userServiceSpy,
              },
            ],
          })
          .compileComponents();
      },
    );

    it(
      'should load user',
      () => {

        const fixture =
          TestBed.createComponent(
            UserComponent,
          );

        fixture.detectChanges();

        expect(
          userServiceSpy
            .getUser,
        ).toHaveBeenCalled();
      },
    );
  },
);
```

---

# Overriding Providers

```typescript
beforeEach(async () => {
  await TestBed
    .configureTestingModule({
      declarations: [
        AppComponent,
      ],
    })

    .overrideProvider(
      APP_CONFIG,
      {
        useValue: {
          apiUrl:
            'http://test-api.com',
        },
      },
    )

    .compileComponents();
});
```

---

# Testing Injection Tokens

```typescript
describe(
  'API_URL token',
  () => {

    it(
      'should provide URL',
      () => {

        TestBed
          .configureTestingModule({
            providers: [
              {
                provide:
                  API_URL,

                useValue:
                  'https://api.test.com',
              },
            ],
          });

        const apiUrl =
          TestBed.inject(
            API_URL,
          );

        expect(apiUrl)
          .toBe(
            'https://api.test.com',
          );
      },
    );
  },
);
```

---

# Cleanup and Subscription Management

Angular 13 does NOT support:
- `DestroyRef`
- `takeUntilDestroyed`
- `assertInInjectionContext`
- signal-based inject utilities

Use classic RxJS cleanup patterns.

---

# takeUntil Cleanup Pattern

```typescript
import {
  Subject,
} from 'rxjs';

import {
  takeUntil,
} from 'rxjs/operators';

@Component({
  selector: 'app-data',
  template: `...`,
})
export class DataComponent
  implements
    OnInit,
    OnDestroy {

  private destroy$ =
    new Subject<void>();

  constructor(
    private dataService:
      DataService,
  ) {}

  ngOnInit(): void {
    this.dataService.data$
      .pipe(
        takeUntil(
          this.destroy$,
        ),
      )
      .subscribe(data => {
        console.log(data);
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

# Manual Subscription Cleanup

```typescript
import {
  Subscription,
} from 'rxjs';

@Component({
  selector: 'app-example',
  template: `...`,
})
export class ExampleComponent
  implements OnDestroy {

  private subscription?:
    Subscription;

  ngOnInit(): void {
    this.subscription =
      this.http
        .get('/api/data')
        .subscribe();
  }

  ngOnDestroy(): void {
    this.subscription
      ?.unsubscribe();
  }
}
```

---

# Service Cleanup

```typescript
@Injectable()
export class WebSocketService
  implements OnDestroy {

  private socket?:
    WebSocket;

  connect(url: string): void {
    this.socket =
      new WebSocket(url);
  }

  ngOnDestroy(): void {
    this.socket?.close();
  }
}
```

---

# Angular 13 DI Best Practices

Prefer:
- Constructor injection
- Abstract classes as tokens
- `providedIn: 'root'`
- RxJS-based state management
- Facade services
- Feature module isolation
- Smart/container components
- `takeUntil` cleanup
- Async pipe over manual subscriptions

Avoid:
- Nested subscriptions
- Global mutable state
- Memory leaks
- Large god services
- Experimental injection APIs
- Excessive shared state
- Direct DOM access in services
- Manual injector lookups
- Subscription cleanup omissions