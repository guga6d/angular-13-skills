# Angular 13 Form Patterns

## Table of Contents

- [Reactive Forms (Production-Stable)](#reactive-forms-production-stable)
- [Typed Reactive Forms](#typed-reactive-forms)
- [FormBuilder Patterns](#formbuilder-patterns)
- [Dynamic Forms with FormArray](#dynamic-forms-with-formarray)
- [Custom Validators](#custom-validators)
- [Form State Management](#form-state-management)
- [Error Display Pattern](#error-display-pattern)
- [Form Submission Pattern](#form-submission-pattern)
- [Angular 13 Best Practices](#angular-13-best-practices)

---

# Reactive Forms (Production-Stable)

For Angular 13 production applications requiring long-term stability and scalability, use Reactive Forms.

Reactive Forms provide:

- Explicit form state management
- Better scalability for enterprise applications
- Strong validation support
- Observable-based reactive patterns
- Easier unit testing

```typescript
import { Component } from '@angular/core';
import {
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-login',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input
        type="email"
        formControlName="email"
        placeholder="Email"
      />

      <span
        class="error"
        *ngIf="
          form.get('email')?.errors?.required &&
          form.get('email')?.touched
        "
      >
        Email is required
      </span>

      <span
        class="error"
        *ngIf="
          form.get('email')?.errors?.email &&
          form.get('email')?.touched
        "
      >
        Invalid email format
      </span>

      <input
        type="password"
        formControlName="password"
        placeholder="Password"
      />

      <button
        type="submit"
        [disabled]="form.invalid"
      >
        Login
      </button>
    </form>
  `,
})
export class LoginComponent {
  form: FormGroup;

  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      email: [
        '',
        [Validators.required, Validators.email],
      ],
      password: [
        '',
        [
          Validators.required,
          Validators.minLength(8),
        ],
      ],
    });
  }

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    console.log(this.form.value);
  }
}
```

---

# Typed Reactive Forms

Angular 13 does not yet support fully typed forms introduced in later Angular versions.

To improve type safety:

- Use interfaces for data contracts
- Keep form models aligned with interfaces
- Create typed helper methods when needed

---

## Typed FormControl Pattern

```typescript
import {
  FormControl,
  Validators,
} from '@angular/forms';

// String control
const name = new FormControl('');

// Nullable number control
const age = new FormControl<number | null>(null);

// Control with validators
const username = new FormControl('', [
  Validators.required,
  Validators.minLength(3),
]);
```

---

## Typed FormGroup Pattern

```typescript
import {
  FormControl,
  FormGroup,
} from '@angular/forms';

interface UserFormValue {
  name: string;
  email: string;
  age: number | null;
}

const form = new FormGroup({
  name: new FormControl(''),
  email: new FormControl(''),
  age: new FormControl<number | null>(null),
});

// Cast form value when necessary
const value =
  form.value as UserFormValue;
```

---

## Typed Getter Pattern

Prefer getters to avoid repeated `get()` calls inside templates.

```typescript
get emailControl(): FormControl {
  return this.form.get('email') as FormControl;
}
```

---

# FormBuilder Patterns

Use `FormBuilder` to reduce boilerplate and improve readability.

---

## Basic FormBuilder Usage

```typescript
constructor(private fb: FormBuilder) {}

form = this.fb.group({
  name: ['', Validators.required],
  email: [
    '',
    [Validators.required, Validators.email],
  ],
});
```

---

## Nested FormGroups

Use nested groups to organize complex forms.

```typescript
import { Component } from '@angular/core';
import {
  FormBuilder,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-profile',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input
        formControlName="name"
        placeholder="Name"
      />

      <div formGroupName="address">
        <input
          formControlName="street"
          placeholder="Street"
        />

        <input
          formControlName="city"
          placeholder="City"
        />

        <input
          formControlName="zip"
          placeholder="ZIP"
        />
      </div>

      <button type="submit">
        Submit
      </button>
    </form>
  `,
})
export class ProfileComponent {
  form = this.fb.group({
    name: ['', Validators.required],

    address: this.fb.group({
      street: [''],

      city: ['', Validators.required],

      zip: [
        '',
        [
          Validators.required,
          Validators.pattern(/^\d{5}$/),
        ],
      ],
    }),
  });

  constructor(private fb: FormBuilder) {}

  onSubmit(): void {
    console.log(this.form.value);
  }
}
```

---

# Dynamic Forms with FormArray

Use `FormArray` for dynamic collections such as:

- Order items
- Dynamic addresses
- Phone numbers
- Multi-step form sections

---

## FormArray Example

```typescript
import { Component } from '@angular/core';

import {
  FormArray,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-order',
  template: `
    <form [formGroup]="form">
      <div formArrayName="items">
        <div
          *ngFor="
            let item of items.controls;
            let i = index
          "
          [formGroupName]="i"
        >
          <input
            formControlName="product"
            placeholder="Product"
          />

          <input
            type="number"
            formControlName="quantity"
          />

          <button
            type="button"
            (click)="removeItem(i)"
          >
            Remove
          </button>
        </div>
      </div>

      <button
        type="button"
        (click)="addItem()"
      >
        Add Item
      </button>
    </form>
  `,
})
export class OrderComponent {
  form = this.fb.group({
    items: this.fb.array([
      this.createItem(),
    ]),
  });

  constructor(private fb: FormBuilder) {}

  get items(): FormArray {
    return this.form.get(
      'items'
    ) as FormArray;
  }

  createItem(): FormGroup {
    return this.fb.group({
      product: [
        '',
        Validators.required,
      ],

      quantity: [
        1,
        [
          Validators.required,
          Validators.min(1),
        ],
      ],
    });
  }

  addItem(): void {
    this.items.push(this.createItem());
  }

  removeItem(index: number): void {
    this.items.removeAt(index);
  }
}
```

---

# Custom Validators

Custom validators centralize validation logic and improve reusability.

---

## Sync Validator

```typescript
import {
  AbstractControl,
  ValidationErrors,
  ValidatorFn,
} from '@angular/forms';

export function forbiddenValue(
  forbidden: string
): ValidatorFn {
  return (
    control: AbstractControl
  ): ValidationErrors | null => {
    return control.value === forbidden
      ? {
          forbiddenValue: {
            value: control.value,
          },
        }
      : null;
  };
}
```

---

## Sync Validator Usage

```typescript
this.form = this.fb.group({
  name: [
    '',
    [
      Validators.required,
      forbiddenValue('admin'),
    ],
  ],
});
```

---

## Cross-Field Validator

Use form-level validators when validation depends on multiple controls.

```typescript
import {
  AbstractControl,
  ValidationErrors,
  ValidatorFn,
} from '@angular/forms';

export function passwordMatch(): ValidatorFn {
  return (
    group: AbstractControl
  ): ValidationErrors | null => {
    const password =
      group.get('password')?.value;

    const confirm =
      group.get('confirmPassword')?.value;

    return password === confirm
      ? null
      : { passwordMismatch: true };
  };
}
```

---

## Cross-Field Validator Usage

```typescript
this.form = this.fb.group(
  {
    password: [
      '',
      [
        Validators.required,
        Validators.minLength(8),
      ],
    ],

    confirmPassword: [
      '',
      Validators.required,
    ],
  },
  {
    validators: passwordMatch(),
  }
);
```

---

## Async Validator

Use asynchronous validators for API validation.

```typescript
import {
  AbstractControl,
  AsyncValidatorFn,
} from '@angular/forms';

import {
  catchError,
  map,
} from 'rxjs/operators';

import { of } from 'rxjs';

export function uniqueEmail(
  userService: UserService
): AsyncValidatorFn {
  return (
    control: AbstractControl
  ) => {
    return userService
      .checkEmail(control.value)
      .pipe(
        map((exists) =>
          exists
            ? { emailTaken: true }
            : null
        ),

        catchError(() => of(null))
      );
  };
}
```

---

## Async Validator Usage

```typescript
this.form = this.fb.group({
  email: [
    '',
    [Validators.required, Validators.email],
    [uniqueEmail(this.userService)],
  ],
});
```

---

# Form State Management

Reactive Forms expose useful state information for validation and UI updates.

---

## State Properties

```typescript
// Validation state
form.valid;
form.invalid;
form.pending;

// Interaction state
form.dirty;
form.pristine;
form.touched;
form.untouched;

// Update values
form.setValue({
  name: 'John',
  email: 'john@example.com',
});

form.patchValue({
  name: 'John',
});

// Reset form
form.reset();

form.reset({
  name: 'Default',
});

// Disable / enable
form.disable();
form.enable();

form.get('email')?.disable();

// Mark states
form.markAllAsTouched();

form.markAsPristine();

form.markAsDirty();
```

---

# ValueChanges Observable

Reactive Forms are built around RxJS observables.

---

## Subscribe to Form Changes

```typescript
this.form.valueChanges.subscribe(
  (value) => {
    console.log('Form value:', value);
  }
);
```

---

## Debounced Single Control

```typescript
import {
  debounceTime,
  distinctUntilChanged,
} from 'rxjs/operators';

this.form
  .get('email')
  ?.valueChanges.pipe(
    debounceTime(300),
    distinctUntilChanged()
  )
  .subscribe((email) => {
    this.validateEmail(email);
  });
```

---

## Status Changes

```typescript
this.form.statusChanges.subscribe(
  (status) => {
    console.log('Form status:', status);
  }
);
```

Possible statuses:

- VALID
- INVALID
- PENDING
- DISABLED

---

# Angular v18+ Features Not Available in Angular 13

The following APIs do NOT exist in Angular 13:

- `form.events`
- `ValueChangeEvent`
- `StatusChangeEvent`
- `TouchedChangeEvent`
- `FormSubmittedEvent`
- Signal Forms
- Typed Forms API
- `@if`
- `@for`

For Angular 13 projects:

- Use `valueChanges`
- Use `statusChanges`
- Use `*ngIf`
- Use `*ngFor`

---

# Form Submission Pattern

Use loading states and proper validation handling during submit.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-form',
  template: `
    <form
      [formGroup]="form"
      (ngSubmit)="onSubmit()"
    >
      <!-- fields -->

      <button
        type="submit"
        [disabled]="
          form.invalid ||
          isSubmitting
        "
      >
        {{
          isSubmitting
            ? 'Submitting...'
            : 'Submit'
        }}
      </button>
    </form>
  `,
})
export class FormComponent {
  isSubmitting = false;

  async onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.isSubmitting = true;

    this.api.submit(
      this.form.getRawValue()
    ).subscribe({
      next: () => {
        this.form.reset();
      },

      error: (error) => {
        console.error(error);
      },

      complete: () => {
        this.isSubmitting = false;
      },
    });
  }
}
```

---

# Angular 13 Best Practices

## Prefer Reactive Forms for Complex Forms

Use Reactive Forms when forms include:

- Dynamic controls
- Complex validation
- API validation
- Conditional fields
- Multi-step flows
- Enterprise-level complexity

---

## Prefer FormBuilder

Avoid unnecessary manual control creation.

Good:

```typescript
this.fb.group({
  name: [''],
});
```

Avoid:

```typescript
new FormGroup({
  name: new FormControl(''),
});
```

---

## Avoid Heavy Template Logic

Move validation logic to helper methods with pure pipes and getters.

---

## Use markAllAsTouched on Submit

Ensures all validation errors appear consistently.

```typescript
if (this.form.invalid) {
  this.form.markAllAsTouched();
  return;
}
```

---

## Unsubscribe from Observables

Avoid memory leaks.

```typescript
import { Subject } from 'rxjs';

import { takeUntil } from 'rxjs/operators';

private destroy$ =
  new Subject<void>();

ngOnInit(): void {
  this.form.valueChanges
    .pipe(
      takeUntil(this.destroy$)
    )
    .subscribe();
}

ngOnDestroy(): void {
  this.destroy$.next();

  this.destroy$.complete();
}
```

---

## Use getRawValue for Submission

`getRawValue()` includes disabled controls.

```typescript
const payload =
  this.form.getRawValue();
```

---

## Use updateOn for Performance

Reduce unnecessary validation executions.

```typescript
this.form = this.fb.group({
  email: [
    '',
    {
      validators: [
        Validators.required,
        Validators.email,
      ],
      updateOn: 'blur',
    },
  ],
});
```