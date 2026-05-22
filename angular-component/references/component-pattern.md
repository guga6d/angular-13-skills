# Angular 13 Component Patterns

## Table of Contents
- [Two-Way Binding](#two-way-binding)
- [View Queries](#view-queries)
- [Content Queries](#content-queries)
- [Dependency Injection in Components](#dependency-injection-in-components)
- [Component Communication Patterns](#component-communication-patterns)
- [Dynamic Components](#dynamic-components)
- [Attribute Directives on Components](#attribute-directives-on-components)
- [Error Handling Components](#error-handling-components)

---

# Two-Way Binding

Angular 13 uses:
- `@Input()`
- `@Output()`
- `EventEmitter`
- `[(ngModel)]`

instead of `model()` signals.

## Manual Two-Way Binding

```typescript
import {
  Component,
  Input,
  Output,
  EventEmitter,
} from '@angular/core';

@Component({
  selector: 'app-slider',
  template: `
    <input
      type="range"
      [value]="value"
      [min]="min"
      [max]="max"
      (input)="onInput($event)"
    />

    <span>{{ value }}</span>
  `,
})
export class SliderComponent {
  @Input()
  value = 0;

  @Input()
  min = 0;

  @Input()
  max = 100;

  @Output()
  valueChange = new EventEmitter<number>();

  onInput(event: Event): void {
    const target = event.target as HTMLInputElement;

    this.valueChange.emit(Number(target.value));
  }
}
```

Usage:

```html
<app-slider [(value)]="sliderValue"></app-slider>
```

---

# Using ngModel

Import `FormsModule` to use `[(ngModel)]`.

```typescript
import { FormsModule } from '@angular/forms';

@NgModule({
  imports: [
    FormsModule,
  ],
})
export class SharedModule {}
```

```html
<input [(ngModel)]="username" />
```

---

# View Queries

Angular 13 uses:
- `@ViewChild`
- `@ViewChildren`

instead of `viewChild()` and `viewChildren()`.

```typescript
import {
  Component,
  ViewChild,
  ViewChildren,
  QueryList,
  ElementRef,
  AfterViewInit,
} from '@angular/core';

@Component({
  selector: 'app-gallery',
  template: `
    <div #container class="gallery">
      <app-image-card
        *ngFor="let image of images"
        [image]="image">
      </app-image-card>
    </div>
  `,
})
export class GalleryComponent implements AfterViewInit {
  @Input()
  images: Image[] = [];

  // Query DOM element
  @ViewChild('container')
  container!: ElementRef<HTMLDivElement>;

  // Query single component
  @ViewChild(ImageCardComponent)
  firstCard?: ImageCardComponent;

  // Query all matching components
  @ViewChildren(ImageCardComponent)
  allCards!: QueryList<ImageCardComponent>;

  ngAfterViewInit(): void {
    console.log(this.container);
    console.log(this.allCards.toArray());
  }
}
```

---

# Content Queries

Angular 13 uses:
- `@ContentChild`
- `@ContentChildren`

instead of `contentChild()` and `contentChildren()`.

```typescript
import {
  Component,
  ContentChildren,
  QueryList,
  AfterContentInit,
} from '@angular/core';

@Component({
  selector: 'app-tabs',
  template: `
    <div class="tab-headers">
      <button
        *ngFor="let tab of tabs.toArray()"
        [class.active]="tab === activeTab"
        (click)="selectTab(tab)"
      >
        {{ tab.label }}
      </button>
    </div>

    <div class="tab-content">
      <ng-content></ng-content>
    </div>
  `,
})
export class TabsComponent implements AfterContentInit {
  @ContentChildren(TabComponent)
  tabs!: QueryList<TabComponent>;

  activeTab?: TabComponent;

  ngAfterContentInit(): void {
    const firstTab = this.tabs.first;

    if (firstTab) {
      this.selectTab(firstTab);
    }
  }

  selectTab(tab: TabComponent): void {
    this.activeTab = tab;

    this.tabs.forEach(item => {
      item.isActive = item === tab;
    });
  }
}
```

```typescript
@Component({
  selector: 'app-tab',
  template: `
    <div [style.display]="isActive ? 'block' : 'none'">
      <ng-content></ng-content>
    </div>
  `,
})
export class TabComponent {
  @Input()
  label!: string;

  isActive = false;
}
```

---

# Dependency Injection in Components

Angular 13 primarily uses constructor injection.

`inject()` exists only experimentally and should generally be avoided in Angular 13 projects.

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-dashboard',
  template: `...`,
})
export class DashboardComponent {
  constructor(
    private router: Router,
    private userService: UserService,
    @Optional() private analytics?: AnalyticsService,
  ) {}

  navigateToProfile(): void {
    this.router.navigate(['/profile']);
  }
}
```

## Injection Tokens

```typescript
import {
  Inject,
  InjectionToken,
} from '@angular/core';

export const APP_CONFIG =
  new InjectionToken<AppConfig>('app.config');

constructor(
  @Inject(APP_CONFIG)
  private config: AppConfig,
) {}
```

---

# Component Communication Patterns

## Parent to Child (Inputs)

```typescript
// Parent
@Component({
  template: `
    <app-child
      [data]="parentData"
      [config]="config">
    </app-child>
  `,
})
export class ParentComponent {
  parentData = {
    name: 'Test',
  };

  config = {
    theme: 'dark',
  };
}
```

```typescript
// Child
@Component({
  selector: 'app-child',
  template: `...`,
})
export class ChildComponent {
  @Input()
  data!: Data;

  @Input()
  config?: Config;
}
```

---

## Child to Parent (Outputs)

```typescript
// Child
@Component({
  selector: 'app-child',
  template: `
    <button (click)="save()">
      Save
    </button>
  `,
})
export class ChildComponent {
  @Output()
  saved = new EventEmitter<Data>();

  save(): void {
    this.saved.emit({
      id: 1,
      name: 'Item',
    });
  }
}
```

```typescript
// Parent
@Component({
  template: `
    <app-child
      (saved)="onSaved($event)">
    </app-child>
  `,
})
export class ParentComponent {
  onSaved(data: Data): void {
    console.log('Saved:', data);
  }
}
```

---

# Shared Service Pattern

Angular 13 applications commonly use:
- `BehaviorSubject`
- RxJS streams
- Observables

instead of Signals.

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root',
})
export class CartService {
  private itemsSubject =
    new BehaviorSubject<CartItem[]>([]);

  items$ =
    this.itemsSubject.asObservable();

  total$ = this.items$.pipe(
    map(items =>
      items.reduce(
        (sum, item) => sum + item.price,
        0,
      ),
    ),
  );

  addItem(item: CartItem): void {
    const current =
      this.itemsSubject.value;

    this.itemsSubject.next([
      ...current,
      item,
    ]);
  }

  removeItem(id: string): void {
    const filtered =
      this.itemsSubject.value.filter(
        item => item.id !== id,
      );

    this.itemsSubject.next(filtered);
  }
}
```

## Component Using Shared Service

```typescript
@Component({
  template: `
    <button (click)="add()">
      Add
    </button>
  `,
})
export class ProductComponent {
  @Input()
  product!: Product;

  constructor(
    private cart: CartService,
  ) {}

  add(): void {
    this.cart.addItem({
      ...this.product,
      quantity: 1,
    });
  }
}
```

```typescript
@Component({
  template: `
    <span>
      Total:
      {{ total$ | async }}
    </span>
  `,
})
export class CartSummaryComponent {
  total$ = this.cart.total$;

  constructor(
    public cart: CartService,
  ) {}
}
```

---

# Dynamic Components

Angular 13 does NOT support `@defer`.

Use:
- Lazy-loaded modules
- Dynamic component loading
- Route-based lazy loading

## Route Lazy Loading

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () =>
      import('./dashboard/dashboard.module')
        .then(m => m.DashboardModule),
  },
];
```

---

# Dynamic Component Rendering

```typescript
import {
  Component,
  ViewChild,
  ViewContainerRef,
  ComponentFactoryResolver,
} from '@angular/core';

@Component({
  selector: 'app-host',
  template: `
    <ng-template #container></ng-template>
  `,
})
export class HostComponent {
  @ViewChild(
    'container',
    {
      read: ViewContainerRef,
      static: true,
    },
  )
  container!: ViewContainerRef;

  constructor(
    private resolver:
      ComponentFactoryResolver,
  ) {}

  loadComponent(): void {
    this.container.clear();

    const factory =
      this.resolver.resolveComponentFactory(
        HeavyChartComponent,
      );

    this.container.createComponent(factory);
  }
}
```

---

# Attribute Directives on Components

Angular 13 directives use:
- `@Input()`
- `@HostBinding`

```typescript
import {
  Directive,
  Input,
  HostBinding,
} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  @Input('appHighlight')
  color = 'yellow';

  @HostBinding('style.backgroundColor')
  get backgroundColor(): string {
    return this.color;
  }
}
```

Usage:

```html
<app-card appHighlight="lightblue">
</app-card>
```

---

# Error Handling Components

Angular 13 does NOT support signal-based error boundaries.

Use:
- `ErrorHandler`
- local state variables
- RxJS error handling

```typescript
import {
  Component,
  ErrorHandler,
} from '@angular/core';

@Component({
  selector: 'app-error-boundary',
  template: `
    <div *ngIf="hasError; else content">
      <div class="error">
        <h3>
          Something went wrong
        </h3>

        <button (click)="retry()">
          Retry
        </button>
      </div>
    </div>

    <ng-template #content>
      <ng-content></ng-content>
    </ng-template>
  `,
})
export class ErrorBoundaryComponent {
  hasError = false;

  constructor(
    private errorHandler: ErrorHandler,
  ) {}

  handleError(error: unknown): void {
    this.hasError = true;

    this.errorHandler.handleError(error);
  }

  retry(): void {
    this.hasError = false;
  }
}
```

---

# Angular 13 Recommended Patterns

Prefer:
- RxJS state management
- Smart/container components
- OnPush change detection
- Feature modules
- Strong typing
- Reactive forms
- Lazy loading
- Services for shared state
- Async pipe over manual subscriptions

Avoid:
- Deeply nested subscriptions
- Business logic in templates
- Excessive `any`
- Manual DOM manipulation
- Large shared modules
- Mutable shared state
- Memory leaks from subscriptions
- Overusing template methods
- Using experimental APIs in Angular 13