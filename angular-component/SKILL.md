---
name: angular-component
description: Create Angular 13 components following Angular 13 best practices. Use for building reusable UI components with @Input/@Output decorators, OnPush change detection, lifecycle hooks, content projection, reactive patterns with RxJS, and accessibility support. Triggers on component creation, refactoring legacy AngularJS patterns, adding host bindings/listeners, or implementing accessible interactive components in Angular 13 applications.
---

# Angular 13 Component

Create components compatible with Angular 13 standards and APIs.

Angular 13 does NOT support:
- Signals (`input()`, `output()`, `computed()`)
- Native control flow (`@if`, `@for`, `@switch`)
- Standalone components
- `outputFromObservable`
- `afterRender` / `afterNextRender`

Use classic Angular patterns with:
- `@Input()`
- `@Output()`
- `EventEmitter`
- `*ngIf`
- `*ngFor`
- `*ngSwitch`
- NgModules

## Component Structure

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  Input,
  Output,
  EventEmitter,
} from '@angular/core';

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent {
  @Input()
  name!: string;

  @Input()
  email = '';

  @Input()
  showEmail = false;

  @Input()
  isActive = false;

  @Output()
  selected = new EventEmitter<string>();

  get avatarUrl(): string {
    return `https://api.example.com/avatar/${this.name}`;
  }

  handleClick(): void {
    this.selected.emit(this.name);
  }
}
```

## Inputs

Use `@Input()` decorators for component inputs.

```typescript
// Required input
@Input()
name!: string;

// Optional input with default value
@Input()
count = 0;

// Optional nullable input
@Input()
label?: string;

// Input alias
@Input('buttonSize')
size: 'small' | 'medium' | 'large' = 'medium';
```

## Outputs

Use `@Output()` with `EventEmitter`.

```typescript
import { Output, EventEmitter } from '@angular/core';

// Basic output
@Output()
clicked = new EventEmitter<void>();

// Typed output
@Output()
selected = new EventEmitter<Item>();

// Output alias
@Output('change')
valueChange = new EventEmitter<number>();

// Emit values
this.clicked.emit();
this.selected.emit(item);
```

## Host Bindings and Host Listeners

In Angular 13 it is acceptable to use:
- `host` object
- `@HostBinding`
- `@HostListener`

Prefer the `host` object when possible.

```typescript
@Component({
  selector: 'app-button',
  template: `
    <ng-content></ng-content>
  `,
  host: {
    'role': 'button',
    '[class.primary]': 'variant === "primary"',
    '[class.disabled]': 'disabled',
    '[style.--btn-color]': 'color',
    '[attr.aria-disabled]': 'disabled',
    '[attr.tabindex]': 'disabled ? -1 : 0',
    '(click)': 'onClick($event)',
    '(keydown.enter)': 'onClick($event)',
    '(keydown.space)': 'onClick($event)',
  },
})
export class ButtonComponent {
  @Input()
  variant: 'primary' | 'secondary' = 'primary';

  @Input()
  disabled = false;

  @Input()
  color = '#007bff';

  @Output()
  clicked = new EventEmitter<void>();

  onClick(event: Event): void {
    if (!this.disabled) {
      this.clicked.emit();
    }
  }
}
```

## Content Projection

Use `<ng-content>` for content projection.

```typescript
@Component({
  selector: 'app-card',
  template: `
    <header>
      <ng-content select="[card-header]"></ng-content>
    </header>

    <main>
      <ng-content></ng-content>
    </main>

    <footer>
      <ng-content select="[card-footer]"></ng-content>
    </footer>
  `,
})
export class CardComponent {}
```

Usage:

```html
<app-card>
  <h2 card-header>Title</h2>

  <p>Main content</p>

  <button card-footer>
    Action
  </button>
</app-card>
```

## Lifecycle Hooks

Angular 13 lifecycle hooks:

```typescript
import {
  OnInit,
  OnDestroy,
  OnChanges,
  AfterViewInit,
  SimpleChanges,
} from '@angular/core';

export class MyComponent
  implements
    OnInit,
    OnChanges,
    AfterViewInit,
    OnDestroy {

  ngOnInit(): void {
    // Component initialized
  }

  ngOnChanges(changes: SimpleChanges): void {
    // Inputs changed
  }

  ngAfterViewInit(): void {
    // View initialized
  }

  ngOnDestroy(): void {
    // Cleanup subscriptions
  }
}
```

## RxJS Cleanup Pattern

Angular 13 applications commonly use RxJS cleanup with `Subject`.

```typescript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

export class ExampleComponent implements OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit(): void {
    this.userService.users$
      .pipe(takeUntil(this.destroy$))
      .subscribe(users => {
        console.log(users);
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## Accessibility Requirements

Components MUST:
- Pass AXE accessibility checks
- Meet WCAG AA standards
- Include proper ARIA attributes
- Support keyboard navigation
- Maintain visible focus indicators

```typescript
@Component({
  selector: 'app-toggle',
  template: `
    <span class="toggle-track">
      <span class="toggle-thumb"></span>
    </span>
  `,
  host: {
    'role': 'switch',
    '[attr.aria-checked]': 'checked',
    '[attr.aria-label]': 'label',
    'tabindex': '0',
    '(click)': 'toggle()',
    '(keydown.enter)': 'toggle()',
    '(keydown.space)': 'toggle(); $event.preventDefault()',
  },
})
export class ToggleComponent {
  @Input()
  label!: string;

  @Input()
  checked = false;

  @Output()
  checkedChange = new EventEmitter<boolean>();

  toggle(): void {
    this.checkedChange.emit(!this.checked);
  }
}
```

## Template Syntax

Angular 13 uses structural directives.

```html
<!-- Conditionals -->
<app-spinner *ngIf="isLoading"></app-spinner>

<app-error
  *ngIf="error"
  [message]="error">
</app-error>

<app-content
  *ngIf="!isLoading && !error"
  [data]="data">
</app-content>

<!-- Loops -->
<app-item
  *ngFor="let item of items; trackBy: trackById"
  [item]="item">
</app-item>

<p *ngIf="items.length === 0">
  No items found
</p>

<!-- Switch -->
<div [ngSwitch]="status">
  <span *ngSwitchCase="'pending'">
    Pending
  </span>

  <span *ngSwitchCase="'active'">
    Active
  </span>

  <span *ngSwitchDefault>
    Unknown
  </span>
</div>
```

## TrackBy Functions

Always use `trackBy` with `*ngFor` for performance.

```typescript
trackById(index: number, item: Item): number {
  return item.id;
}
```

## Class and Style Bindings

Prefer direct bindings over `ngClass` and `ngStyle` when possible.

```html
<!-- Class bindings -->
<div [class.active]="isActive">
  Single class
</div>

<div [class]="classString">
  Class string
</div>

<!-- Style bindings -->
<div [style.color]="textColor">
  Styled text
</div>

<div [style.width.px]="width">
  With unit
</div>
```

## Images

Angular 13 does NOT include `NgOptimizedImage`.

Use standard image loading patterns:

```html
<img
  [src]="imageUrl"
  [alt]="imageAlt"
  loading="lazy"
/>
```

## Module Declaration

Angular 13 components MUST be declared in an NgModule.

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { UserCardComponent } from './user-card.component';

@NgModule({
  declarations: [
    UserCardComponent,
  ],
  imports: [
    CommonModule,
  ],
  exports: [
    UserCardComponent,
  ],
})
export class UserCardModule {}
```

## Angular 13 Best Practices

- Use `ChangeDetectionStrategy.OnPush`
- Prefer reactive programming with RxJS
- Avoid business logic in templates
- Keep components small and reusable
- Use smart/container and dumb/presentational separation
- Unsubscribe from Observables
- Prefer async pipe over subscribe
- Use strongly typed models/interfaces
- Use reactive forms over template-driven forms in complex scenarios
- Use lazy-loaded modules for large applications
- Use `trackBy` in loops
- Use pure pipes when transforming template data

## Reactive Form Example

```typescript
import {
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';

export class LoginComponent {
  form: FormGroup;

  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      email: [
        '',
        [
          Validators.required,
          Validators.email,
        ],
      ],
      password: [
        '',
        Validators.required,
      ],
    });
  }

  submit(): void {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

## Anti-Patterns to Avoid

Do NOT:
- Use `any` unnecessarily
- Subscribe without cleanup
- Put heavy logic in templates
- Mutate `@Input()` values directly
- Use default change detection everywhere
- Create giant shared modules
- Use `setTimeout` for change detection fixes
- Manipulate DOM directly without `Renderer2`
- Use template functions
```