# Angular 13 HTTP Patterns

## Table of Contents

- [Service Layer Pattern](#service-layer-pattern)
- [Caching Strategies](#caching-strategies)
- [Pagination](#pagination)
- [Infinite Scroll](#infinite-scroll)
- [File Upload](#file-upload)
- [Request Cancellation](#request-cancellation)
- [Debounced Search](#debounced-search)
- [Testing HTTP](#testing-http)
- [Angular 13 Limitations Compared to Angular v20+](#angular-13-limitations-compared-to-angular-v20)

---

# Service Layer Pattern

In Angular 13, all HTTP communication should be encapsulated inside services.

Angular 13 does NOT support:

- `httpResource()`
- `resource()`
- `signal()`
- `computed()`
- `inject()`
- signal-based HTTP state management

Recommended Angular 13 architecture:

- `HttpClient`
- RxJS Observables
- Services
- Component state
- Async pipe

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
  id: string;
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root',
})
export class UserService {
  private readonly baseUrl =
    '/api/users';

  constructor(
    private http: HttpClient
  ) {}

  // GET all users
  getAll(): Observable<User[]> {
    return this.http.get<User[]>(
      this.baseUrl
    );
  }

  // GET by id
  getById(
    id: string
  ): Observable<User> {
    return this.http.get<User>(
      `${this.baseUrl}/${id}`
    );
  }

  // CREATE
  create(
    user: Omit<User, 'id'>
  ): Observable<User> {
    return this.http.post<User>(
      this.baseUrl,
      user
    );
  }

  // UPDATE
  update(
    id: string,
    user: Partial<User>
  ): Observable<User> {
    return this.http.patch<User>(
      `${this.baseUrl}/${id}`,
      user
    );
  }

  // DELETE
  delete(
    id: string
  ): Observable<void> {
    return this.http.delete<void>(
      `${this.baseUrl}/${id}`
    );
  }
}
```

---

# Component Usage Pattern

```typescript
import {
  Component,
  OnInit,
} from '@angular/core';

@Component({
  selector: 'app-users',

  template: `
    <div *ngIf="loading">
      Loading...
    </div>

    <div *ngIf="error">
      {{ error }}
    </div>

    <ul *ngIf="users.length">
      <li
        *ngFor="
          let user of users
        "
      >
        {{ user.name }}
      </li>
    </ul>
  `,
})
export class UsersComponent
  implements OnInit
{
  users: User[] = [];

  loading = false;

  error = '';

  constructor(
    private userService:
      UserService
  ) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.loading = true;

    this.userService
      .getAll()
      .subscribe({
        next: (users) => {
          this.users = users;

          this.loading = false;
        },

        error: () => {
          this.error =
            'Failed to load users';

          this.loading = false;
        },
      });
  }
}
```

---

# Caching Strategies

Angular 13 uses RxJS and in-memory caching instead of signal-based caches.

---

# Simple In-Memory Cache

```typescript
import { Injectable } from '@angular/core';

import {
  HttpClient,
} from '@angular/common/http';

import {
  Observable,
  of,
} from 'rxjs';

import {
  tap,
} from 'rxjs/operators';

@Injectable({
  providedIn: 'root',
})
export class CachedUserService {
  private cache =
    new Map<
      string,
      {
        data: User;
        timestamp: number;
      }
    >();

  private cacheDuration =
    5 * 60 * 1000;

  constructor(
    private http: HttpClient
  ) {}

  getUser(
    id: string
  ): Observable<User> {
    const cached =
      this.cache.get(id);

    if (
      cached &&
      Date.now() -
        cached.timestamp <
        this.cacheDuration
    ) {
      return of(cached.data);
    }

    return this.http
      .get<User>(
        `/api/users/${id}`
      )
      .pipe(
        tap((user) => {
          this.cache.set(id, {
            data: user,
            timestamp:
              Date.now(),
          });
        })
      );
  }

  invalidateCache(
    id?: string
  ): void {
    if (id) {
      this.cache.delete(id);

      return;
    }

    this.cache.clear();
  }
}
```

---

# shareReplay Cache Pattern

A common Angular 13 pattern is caching observables with `shareReplay`.

```typescript
import {
  shareReplay,
} from 'rxjs/operators';

users$ =
  this.http
    .get<User[]>(
      '/api/users'
    )
    .pipe(shareReplay(1));
```

Benefits:

- Prevents duplicate HTTP requests
- Shares data across subscribers
- Improves performance

---

# Pagination

Angular 13 pagination is typically implemented using:

- component state
- query params
- RxJS
- reusable services

---

# Paginated Response Interface

```typescript
export interface PaginatedResponse<T> {
  data: T[];

  total: number;

  page: number;

  pageSize: number;

  totalPages: number;
}
```

---

# Pagination Service Method

```typescript
getUsers(
  page: number,
  pageSize: number
): Observable<
  PaginatedResponse<User>
> {
  return this.http.get<
    PaginatedResponse<User>
  >('/api/users', {
    params: {
      page: page.toString(),

      pageSize:
        pageSize.toString(),
    },
  });
}
```

---

# Pagination Component Example

```typescript
@Component({
  selector: 'app-users-list',

  template: `
    <div *ngIf="loading">
      Loading...
    </div>

    <ul>
      <li
        *ngFor="
          let user of users
        "
      >
        {{ user.name }}
      </li>
    </ul>

    <div class="pagination">
      <button
        (click)="prevPage()"
        [disabled]="page === 1"
      >
        Previous
      </button>

      <span>
        Page {{ page }}
        of
        {{ totalPages }}
      </span>

      <button
        (click)="nextPage()"
        [disabled]="
          page >= totalPages
        "
      >
        Next
      </button>
    </div>
  `,
})
export class UsersListComponent {
  users: User[] = [];

  loading = false;

  page = 1;

  pageSize = 10;

  totalPages = 1;

  constructor(
    private userService:
      UserService
  ) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.loading = true;

    this.userService
      .getUsers(
        this.page,
        this.pageSize
      )
      .subscribe({
        next: (response) => {
          this.users =
            response.data;

          this.totalPages =
            response.totalPages;

          this.loading =
            false;
        },

        error: () => {
          this.loading =
            false;
        },
      });
  }

  nextPage(): void {
    this.page++;

    this.loadUsers();
  }

  prevPage(): void {
    if (this.page === 1) {
      return;
    }

    this.page--;

    this.loadUsers();
  }
}
```

---

# Infinite Scroll

Angular 13 infinite scroll commonly uses:

- page tracking
- array concatenation
- loading guards

---

# Infinite Scroll Example

```typescript
@Component({
  selector:
    'app-infinite-users',

  template: `
    <ul>
      <li
        *ngFor="
          let user of users
        "
      >
        {{ user.name }}
      </li>
    </ul>

    <div *ngIf="loading">
      Loading...
    </div>

    <button
      *ngIf="hasMore"
      (click)="loadMore()"
    >
      Load More
    </button>
  `,
})
export class InfiniteUsersComponent {
  users: User[] = [];

  loading = false;

  page = 1;

  totalPages = 1;

  hasMore = true;

  constructor(
    private userService:
      UserService
  ) {}

  ngOnInit(): void {
    this.loadPage(1);
  }

  loadMore(): void {
    this.loadPage(
      this.page + 1
    );
  }

  private loadPage(
    page: number
  ): void {
    this.loading = true;

    this.userService
      .getUsers(page, 20)
      .subscribe({
        next: (response) => {
          this.users = [
            ...this.users,
            ...response.data,
          ];

          this.page = page;

          this.totalPages =
            response.totalPages;

          this.hasMore =
            this.page <
            this.totalPages;

          this.loading =
            false;
        },

        error: () => {
          this.loading =
            false;
        },
      });
  }
}
```

---

# File Upload

Angular 13 file uploads use:

- `FormData`
- `HttpClient`
- `HttpEventType`

---

# Single File Upload

```typescript
import {
  HttpClient,
  HttpEventType,
} from '@angular/common/http';

@Component({
  selector:
    'app-file-upload',

  template: `
    <input
      type="file"
      (change)="
        onFileSelected($event)
      "
    />

    <progress
      *ngIf="
        uploadProgress !==
        null
      "
      [value]="uploadProgress"
      max="100"
    >
    </progress>
  `,
})
export class FileUploadComponent {
  uploadProgress:
    | number
    | null = null;

  constructor(
    private http: HttpClient
  ) {}

  onFileSelected(
    event: Event
  ): void {
    const input =
      event.target as HTMLInputElement;

    const file =
      input.files?.[0];

    if (!file) {
      return;
    }

    const formData =
      new FormData();

    formData.append(
      'file',
      file
    );

    this.http
      .post(
        '/api/upload',
        formData,
        {
          reportProgress:
            true,

          observe: 'events',
        }
      )
      .subscribe((event) => {
        if (
          event.type ===
            HttpEventType.UploadProgress &&
          event.total
        ) {
          this.uploadProgress =
            Math.round(
              (100 *
                event.loaded) /
                event.total
            );
        }

        if (
          event.type ===
          HttpEventType.Response
        ) {
          this.uploadProgress =
            null;

          console.log(
            'Upload complete'
          );
        }
      });
  }
}
```

---

# Multiple File Upload

```typescript
uploadFiles(
  files: FileList
) {
  const formData =
    new FormData();

  for (
    let i = 0;
    i < files.length;
    i++
  ) {
    formData.append(
      'files',
      files[i]
    );
  }

  return this.http.post<{
    urls: string[];
  }>(
    '/api/upload-multiple',
    formData
  );
}
```

---

# Request Cancellation

Angular 13 uses RxJS unsubscription for request cancellation.
Angular 13 can use RxJS switchmap on streams to cancel previous requests.

Angular 13 does NOT support:

- `abortSignal`
- signal-based request cancellation
- automatic resource cancellation

---

# Manual Cancellation Example

```typescript
import {
  Subscription,
} from 'rxjs';

@Component({
  selector: 'app-search',

  template: `
    <input
      (input)="search($event)"
    />
  `,
})
export class SearchComponent {
  private searchSubscription?:
    Subscription;

  results: Result[] = [];

  constructor(
    private http: HttpClient
  ) {}

  search(
    event: Event
  ): void {
    const input =
      event.target as HTMLInputElement;

    const query =
      input.value;

    // Cancel previous request
    this.searchSubscription?.unsubscribe();

    this.searchSubscription =
      this.http
        .get<Result[]>(
          `/api/search?q=${query}`
        )
        .subscribe(
          (results) => {
            this.results =
              results;
          }
        );
  }
}
```

---

# Automatic Cancellation Example with RxJS SwitchMap

```typescript
import {
  Component,
  OnInit,
} from '@angular/core';

import {
  HttpClient,
} from '@angular/common/http';

import {
  Subject,
} from 'rxjs';

import {
  switchMap,
} from 'rxjs/operators';

@Component({
  selector: 'app-search-switchmap',

  template: `
    <input
      (input)="onSearch($event)"
    />
  `,
})
export class SearchSwitchMapComponent
  implements OnInit
{
  private searchTerms =
    new Subject<string>();

  results: Result[] = [];

  constructor(
    private http: HttpClient
  ) {}

  ngOnInit(): void {
    this.searchTerms
      .pipe(
        switchMap((term) =>
          this.http.get<Result[]>(
            `/api/search?q=${term}`
          )
        )
      )
      .subscribe((results) => {
        this.results =
          results;
      });
  }

  onSearch(
    event: Event
  ): void {
    const input =
      event.target as HTMLInputElement;

    this.searchTerms.next(
      input.value
    );
  }
}
```

# Debounced Search

Angular 13 debounced searches should use RxJS operators.

---

# Debounced Search Example

```typescript
import {
  Subject,
  of,
} from 'rxjs';

import {
  debounceTime,
  distinctUntilChanged,
  switchMap,
  catchError,
  filter,
} from 'rxjs/operators';

@Component({
  selector:
    'app-search-debounced',

  template: `
    <input
      (input)="
        onSearch($event)
      "
    />

    <ul>
      <li
        *ngFor="
          let result of results
        "
      >
        {{ result.name }}
      </li>
    </ul>
  `,
})
export class SearchDebouncedComponent {
  private search$ =
    new Subject<string>();

  results: Result[] = [];

  constructor(
    private http: HttpClient
  ) {}

  ngOnInit(): void {
    this.search$
      .pipe(
        debounceTime(300),

        distinctUntilChanged(),

        filter(
          (query) =>
            query.length >= 2
        ),

        switchMap((query) =>
          this.http
            .get<Result[]>(
              `/api/search?q=${query}`
            )
            .pipe(
              catchError(() =>
                of([])
              )
            )
        )
      )
      .subscribe((results) => {
        this.results =
          results;
      });
  }

  onSearch(
    event: Event
  ): void {
    const input =
      event.target as HTMLInputElement;

    this.search$.next(
      input.value
    );
  }
}
```

---

# Testing HTTP

Angular 13 HTTP testing uses:

- `HttpClientTestingModule`
- `HttpTestingController`

Angular 13 does NOT support:

- `provideHttpClientTesting()`
- standalone testing providers

---

# Service Testing Example

```typescript
import {
  TestBed,
} from '@angular/core/testing';

import {
  HttpClientTestingModule,
  HttpTestingController,
} from '@angular/common/http/testing';

describe(
  'UserService',
  () => {
    let service:
      UserService;

    let httpMock:
      HttpTestingController;

    beforeEach(() => {
      TestBed.configureTestingModule(
        {
          imports: [
            HttpClientTestingModule,
          ],

          providers: [
            UserService,
          ],
        }
      );

      service =
        TestBed.inject(
          UserService
        );

      httpMock =
        TestBed.inject(
          HttpTestingController
        );
    });

    it(
      'should create user',
      () => {
        const newUser = {
          name: 'Test',
          email:
            'test@example.com',
        };

        service
          .create(newUser)
          .subscribe(
            (user) => {
              expect(
                user.id
              ).toBeDefined();

              expect(
                user.name
              ).toBe(
                'Test'
              );
            }
          );

        const req =
          httpMock.expectOne(
            '/api/users'
          );

        expect(
          req.request.method
        ).toBe('POST');

        expect(
          req.request.body
        ).toEqual(newUser);

        req.flush({
          id: '1',
          ...newUser,
        });
      }
    );

    afterEach(() => {
      httpMock.verify();
    });
  }
);
```

---

# Component HTTP Testing Example

```typescript
describe(
  'UsersComponent',
  () => {
    let component:
      UsersComponent;

    let fixture: any;

    let httpMock:
      HttpTestingController;

    beforeEach(() => {
      TestBed.configureTestingModule(
        {
          declarations: [
            UsersComponent,
          ],

          imports: [
            HttpClientTestingModule,
          ],
        }
      );

      fixture =
        TestBed.createComponent(
          UsersComponent
        );

      component =
        fixture.componentInstance;

      httpMock =
        TestBed.inject(
          HttpTestingController
        );
    });

    it(
      'should load users',
      () => {
        fixture.detectChanges();

        const req =
          httpMock.expectOne(
            '/api/users'
          );

        req.flush([
          {
            id: '1',
            name: 'John',
            email:
              'john@example.com',
          },
        ]);

        expect(
          component.users.length
        ).toBe(1);
      }
    );
  }
);
```

---

# Angular 13 Limitations Compared to Angular v20+

The following APIs do NOT exist in Angular 13:

- `httpResource()`
- `resource()`
- `signal()`
- `computed()`
- `toSignal()`
- `inject()`
- `takeUntilDestroyed()`
- `provideHttpClientTesting()`
- `@if`
- `@for`

Angular 13 alternatives:

| Angular v20+ | Angular 13 Equivalent |
|---|---|
| `httpResource()` | `HttpClient + RxJS` |
| `resource()` | `Observable services` |
| Signals | RxJS Subjects |
| `toSignal()` | Async pipe |
| `inject()` | Constructor injection |
| `takeUntilDestroyed()` | `takeUntil()` |
| `@if` | `*ngIf` |
| `@for` | `*ngFor` |

---

# Recommended Angular 13 HTTP Architecture

For enterprise Angular 13 applications:

- Service-based API layer
- RxJS state management
- Async pipe
- Shared caching services
- Interceptors
- Pagination abstraction
- Centralized error handling
- Observable patterns
- Reusable HTTP services
- Proper subscription cleanup

This architecture is production-stable and widely adopted in Angular 13 applications.