---
name: angular-forms
description: Build reactive, type-safe forms in Angular 13 using Reactive Forms. Use for form creation with FormGroup, FormControl, FormArray, synchronous and asynchronous validation, dynamic forms, conditional validation, and multi-step forms. Triggers on form implementation, adding validation, creating reusable form components, or managing complex form state. Recommended for scalable Angular 13 applications. Don't use for template-driven forms or signal-based APIs unavailable in Angular 13.
---

# Angular 13 Reactive Forms

Build scalable and maintainable forms using Angular 13 Reactive Forms. Reactive Forms provide explicit form state management, strong validation support, observables for reactive updates, and better scalability for enterprise applications.

Reactive Forms are the recommended approach for complex forms in Angular 13 applications.

---

# Basic Setup

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
    <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
      <label>
        Email
        <input type="email" formControlName="email" />
      </label>

      <p
        class="error"
        *ngIf="
          loginForm.get('email')?.touched &&
          loginForm.get('email')?.invalid
        "
      >
        Valid email is required
      </p>

      <label>
        Password
        <input type="password" formControlName="password" />
      </label>

      <p
        class="error"
        *ngIf="
          loginForm.get('password')?.touched &&
          loginForm.get('password')?.invalid
        "
      >
        Password is required
      </p>

      <button type="submit" [disabled]="loginForm.invalid">
        Login
      </button>
    </form>
  `,
})
export class LoginComponent {
  loginForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', Validators.required],
    });
  }

  onSubmit(): void {
    if (this.loginForm.invalid) {
      this.loginForm.markAllAsTouched();
      return;
    }

    console.log('Submitting:', this.loginForm.value);
  }
}
```

---

# Importing ReactiveFormsModule

Reactive Forms require `ReactiveFormsModule`.

```typescript
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [ReactiveFormsModule],
})
export class AppModule {}
```

---

# Creating Forms

## Using FormBuilder

Angular 13 projects should prefer `FormBuilder` to reduce boilerplate.

```typescript
this.profileForm = this.fb.group({
  name: [''],
  email: [''],
  age: [null],
});
```

---

# Typed Form Model Structure

Angular 13 does not yet support fully typed forms natively. Define interfaces to maintain consistency between the form model and business data.

```typescript
interface UserProfile {
  name: string;
  email: string;
  age: number | null;
  preferences: {
    newsletter: boolean;
    theme: 'light' | 'dark';
  };
}
```

```typescript
this.profileForm = this.fb.group({
  name: [''],
  email: [''],
  age: [null],
  preferences: this.fb.group({
    newsletter: [false],
    theme: ['light'],
  }),
});
```

---

# Accessing Form Values

## Reading Entire Form

```typescript
const formValue = this.profileForm.value;
```

## Reading Single Control

```typescript
const email = this.profileForm.get('email')?.value;
```

## Reading Nested Controls

```typescript
const theme = this.profileForm.get(
  'preferences.theme'
)?.value;
```

---

# Updating Values

## setValue

Requires all fields.

```typescript
this.profileForm.setValue({
  name: 'Alice',
  email: 'alice@example.com',
  age: 30,
  preferences: {
    newsletter: true,
    theme: 'dark',
  },
});
```

## patchValue

Updates only specific fields.

```typescript
this.profileForm.patchValue({
  name: 'Bob',
});
```

## Updating Single Control

```typescript
this.profileForm.get('name')?.setValue('Charlie');
```

---

# Form State

Reactive Forms expose useful control state information.

```typescript
const emailControl = this.profileForm.get('email');
```

## Validation State

```typescript
emailControl?.valid;
emailControl?.invalid;
emailControl?.errors;
emailControl?.pending;
```

## Interaction State

```typescript
emailControl?.touched;
emailControl?.dirty;
emailControl?.pristine;
```

## Availability State

```typescript
emailControl?.disabled;
emailControl?.enabled;
```

---

# Form-Level State

```typescript
this.profileForm.valid;
this.profileForm.invalid;
this.profileForm.touched;
this.profileForm.dirty;
```

---

# Validation

## Built-in Validators

```typescript
this.userForm = this.fb.group({
  name: ['', Validators.required],

  email: [
    '',
    [Validators.required, Validators.email],
  ],

  age: [
    null,
    [Validators.min(18), Validators.max(120)],
  ],

  password: [
    '',
    [Validators.required, Validators.minLength(8)],
  ],

  bio: ['', Validators.maxLength(500)],

  phone: [
    '',
    Validators.pattern(/^\d{3}-\d{3}-\d{4}$/),
  ],
});
```

---

# Custom Validators

## Single Control Validator

```typescript
import {
  AbstractControl,
  ValidationErrors,
} from '@angular/forms';

export function noSpacesValidator(
  control: AbstractControl
): ValidationErrors | null {
  const value = control.value;

  if (value?.includes(' ')) {
    return {
      noSpaces: true,
    };
  }

  return null;
}
```

```typescript
this.signupForm = this.fb.group({
  username: ['', noSpacesValidator],
});
```

---

# Cross-Field Validation

Use group validators when validation depends on multiple fields.

```typescript
import {
  AbstractControl,
  ValidationErrors,
} from '@angular/forms';

export function passwordMatchValidator(
  control: AbstractControl
): ValidationErrors | null {
  const password = control.get('password')?.value;
  const confirmPassword =
    control.get('confirmPassword')?.value;

  if (password !== confirmPassword) {
    return {
      passwordMismatch: true,
    };
  }

  return null;
}
```

```typescript
this.passwordForm = this.fb.group(
  {
    password: ['', Validators.required],
    confirmPassword: ['', Validators.required],
  },
  {
    validators: passwordMatchValidator,
  }
);
```

---

# Async Validation

Use asynchronous validators for API validation.

```typescript
import {
  AbstractControl,
  AsyncValidatorFn,
  ValidationErrors,
} from '@angular/forms';

import {
  debounceTime,
  distinctUntilChanged,
  map,
  switchMap,
  take,
} from 'rxjs/operators';

import { Observable, of } from 'rxjs';

export function usernameTakenValidator(
  apiService: ApiService
): AsyncValidatorFn {
  return (
    control: AbstractControl
  ): Observable<ValidationErrors | null> => {
    return of(control.value).pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap((username) =>
        apiService.checkUsername(username)
      ),
      map((isTaken) =>
        isTaken ? { usernameTaken: true } : null
      ),
      take(1)
    );
  };
}
```

```typescript
this.signupForm = this.fb.group({
  username: [
    '',
    {
      validators: [Validators.required],
      asyncValidators: [
        usernameTakenValidator(this.apiService),
      ],
      updateOn: 'blur',
    },
  ],
});
```

---

# Conditional Validation

Reactive Forms allow dynamic validators.

```typescript
this.orderForm = this.fb.group({
  applyDiscount: [false],
  promoCode: [''],
});
```

```typescript
this.orderForm
  .get('applyDiscount')
  ?.valueChanges.subscribe((enabled) => {
    const promoCode =
      this.orderForm.get('promoCode');

    if (enabled) {
      promoCode?.setValidators([
        Validators.required,
      ]);
    } else {
      promoCode?.clearValidators();
    }

    promoCode?.updateValueAndValidity();
  });
```

---

# Conditional Fields

## Disable Controls Dynamically

```typescript
this.orderForm
  .get('total')
  ?.valueChanges.subscribe((total) => {
    const coupon =
      this.orderForm.get('couponCode');

    if (total < 50) {
      coupon?.disable();
    } else {
      coupon?.enable();
    }
  });
```

---

# Dynamic Forms with FormArray

Use `FormArray` for dynamic collections.

```typescript
import {
  FormArray,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
```

```typescript
@Component({
  selector: 'app-order',
  template: `
    <form [formGroup]="orderForm">
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

      <button type="button" (click)="addItem()">
        Add Item
      </button>
    </form>
  `,
})
export class OrderComponent {
  orderForm = this.fb.group({
    items: this.fb.array([
      this.createItem(),
    ]),
  });

  constructor(private fb: FormBuilder) {}

  get items(): FormArray {
    return this.orderForm.get(
      'items'
    ) as FormArray;
  }

  createItem(): FormGroup {
    return this.fb.group({
      product: ['', Validators.required],
      quantity: [
        1,
        [Validators.required, Validators.min(1)],
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

# Displaying Errors

```html
<input type="email" formControlName="email" />

<ul
  class="errors"
  *ngIf="
    loginForm.get('email')?.touched &&
    loginForm.get('email')?.errors
  "
>
  <li *ngIf="loginForm.get('email')?.errors?.required">
    Email is required
  </li>

  <li *ngIf="loginForm.get('email')?.errors?.email">
    Invalid email format
  </li>
</ul>
```

---

# Styling Based on State

```html
<input
  type="email"
  formControlName="email"
  [class.is-invalid]="
    loginForm.get('email')?.touched &&
    loginForm.get('email')?.invalid
  "
  [class.is-valid]="
    loginForm.get('email')?.touched &&
    loginForm.get('email')?.valid
  "
/>
```

---

# Observing Form Changes

Reactive Forms use RxJS observables.

## valueChanges

```typescript
this.profileForm.valueChanges.subscribe(
  (value) => {
    console.log('Form changed:', value);
  }
);
```

## statusChanges

```typescript
this.profileForm.statusChanges.subscribe(
  (status) => {
    console.log('Form status:', status);
  }
);
```

---

# Form Submission

```typescript
onSubmit(): void {
  if (this.loginForm.invalid) {
    this.loginForm.markAllAsTouched();
    return;
  }

  const credentials =
    this.loginForm.getRawValue();

  this.authService.login(credentials);
}
```

---

# Resetting Forms

## Reset State and Values

```typescript
this.loginForm.reset();
```

## Reset with Default Values

```typescript
this.loginForm.reset({
  email: '',
  password: '',
});
```

---

# updateOn Strategy

Angular 13 supports validation timing configuration.

## Validate on Blur

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

## Validate on Submit

```typescript
this.form = this.fb.group(
  {
    email: ['', Validators.required],
  },
  {
    updateOn: 'submit',
  }
);
```

---

# Best Practices

## Prefer Reactive Forms for Complex Forms

Use Reactive Forms when forms include:

- Complex validation
- Dynamic fields
- Multi-step flows
- Conditional controls
- API-driven validation
- Large enterprise forms

---

## Use FormBuilder

Prefer `FormBuilder` over manually instantiating controls.

```typescript
this.fb.group({
  name: [''],
});
```

---

## Avoid Heavy Template Logic

Move validation and form logic to the component class.

Bad:

```html
<div *ngIf="form.get('email')?.errors?.required">
```

Better:

```typescript
get emailControl() {
  return this.form.get('email');
}
```

---

## Use markAllAsTouched on Submit

Ensures validation errors appear consistently.

```typescript
if (this.form.invalid) {
  this.form.markAllAsTouched();
  return;
}
```

---

## Unsubscribe from valueChanges

Avoid memory leaks in Angular 13.

```typescript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

private destroy$ = new Subject<void>();

ngOnInit(): void {
  this.form.valueChanges
    .pipe(takeUntil(this.destroy$))
    .subscribe();
}

ngOnDestroy(): void {
  this.destroy$.next();
  this.destroy$.complete();
}
```

---

# Template-Driven Forms

Reactive Forms are preferred for scalable Angular 13 applications.

Use template-driven forms only for:

- Simple forms
- Small prototypes
- Minimal validation requirements

For enterprise applications, prefer Reactive Forms.