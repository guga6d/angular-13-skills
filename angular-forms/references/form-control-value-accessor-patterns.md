# Angular 13 Custom Form Controls (ControlValueAccessor)

## Table of Contents

- [Custom Form Controls](#custom-form-controls)
- [Why ControlValueAccessor](#why-controlvalueaccessor)
- [Rating Component Example](#rating-component-example)
- [Using the Custom Control](#using-the-custom-control)
- [Validation Support](#validation-support)
- [Disabled State Support](#disabled-state-support)
- [Best Practices](#best-practices)
- [Angular 13 Limitations Compared to Signal Forms](#angular-13-limitations-compared-to-signal-forms)

---

# Custom Form Controls

Angular 13 does NOT support:

- Signal Forms
- `FormValueControl`
- `form()`
- `FormField`
- Signals API
- `model()`
- `input()`
- `InputSignal`
- `@for`
- `@if`

To create reusable custom form controls in Angular 13, use:

- `ControlValueAccessor`
- `NG_VALUE_ACCESSOR`
- Reactive Forms

`ControlValueAccessor` is the Angular 13 equivalent pattern for integrating custom components with Reactive Forms.

---

# Why ControlValueAccessor

Use `ControlValueAccessor` when creating:

- Star rating components
- Custom selects
- Date pickers
- Toggles
- Sliders
- Rich text editors
- Reusable Material components

It allows custom components to behave like native Angular form controls.

---

# Rating Component Example

## Rating Interface

```typescript
export interface RatingFormValue {
  rating: number;
}
```

---

## Rating Component

```typescript
import {
  Component,
  forwardRef,
} from '@angular/core';

import {
  ControlValueAccessor,
  NG_VALUE_ACCESSOR,
} from '@angular/forms';

@Component({
  selector: 'app-rating',

  template: `
    <div class="star-rating-container">
      <span
        *ngFor="
          let star of starArray;
          let i = index
        "
        class="star-icon"
        [class.filled]="star <= value"
        [class.disabled]="disabled"
        (click)="rate(star)"
      >
        ★
      </span>

      <div
        class="error"
        *ngIf="errorMessage"
      >
        {{ errorMessage }}
      </div>
    </div>
  `,

  styles: [
    `
      .star-rating-container {
        display: flex;
        align-items: center;
        gap: 4px;
      }

      .star-icon {
        cursor: pointer;
        font-size: 24px;
      }

      .filled {
        color: gold;
      }

      .disabled {
        opacity: 0.5;
        pointer-events: none;
      }

      .error {
        color: red;
        margin-left: 8px;
      }
    `,
  ],

  providers: [
    {
      provide: NG_VALUE_ACCESSOR,

      useExisting: forwardRef(
        () => RatingComponent
      ),

      multi: true,
    },
  ],
})
export class RatingComponent
  implements ControlValueAccessor
{
  value = 0;

  disabled = false;

  errorMessage = '';

  starArray = [1, 2, 3, 4, 5];

  // Required callbacks
  private onChange = (
    value: number
  ) => {};

  private onTouched = () => {};

  // Called by Angular when value changes externally
  writeValue(value: number): void {
    this.value = value || 0;
  }

  // Register change callback
  registerOnChange(
    fn: (value: number) => void
  ): void {
    this.onChange = fn;
  }

  // Register touched callback
  registerOnTouched(
    fn: () => void
  ): void {
    this.onTouched = fn;
  }

  // Disabled state support
  setDisabledState(
    isDisabled: boolean
  ): void {
    this.disabled = isDisabled;
  }

  // Internal click handler
  rate(star: number): void {
    if (this.disabled) {
      return;
    }

    this.value = star;

    this.onChange(star);

    this.onTouched();
  }
}
```

---

# Using the Custom Control

## Reactive Form Example

```typescript
import { Component } from '@angular/core';

import {
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-rating-form',

  template: `
    <form
      [formGroup]="ratingForm"
      (ngSubmit)="submit()"
    >
      <div class="form-field">
        <app-rating
          formControlName="rating"
        >
        </app-rating>

        <!-- Current value -->
        <p>
          Rating:
          {{
            ratingForm.get('rating')
              ?.value
          }}
        </p>

        <!-- Validation -->
        <div
          class="error"
          *ngIf="
            ratingForm
              .get('rating')
              ?.touched &&
            ratingForm
              .get('rating')
              ?.invalid
          "
        >
          Rating is required
        </div>
      </div>

      <button type="submit">
        Submit
      </button>
    </form>
  `,
})
export class RatingFormComponent {
  ratingForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.ratingForm = this.fb.group({
      rating: [
        0,
        Validators.required,
      ],
    });
  }

  submit(): void {
    if (this.ratingForm.invalid) {
      this.ratingForm.markAllAsTouched();

      return;
    }

    console.log(
      this.ratingForm.value
    );
  }
}
```

---

# Importing ReactiveFormsModule

Reactive Forms require `ReactiveFormsModule`.

```typescript
import { NgModule } from '@angular/core';

import {
  ReactiveFormsModule,
} from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule,
  ],
})
export class AppModule {}
```

---

# Validation Support

Angular automatically integrates validation with custom controls using `ControlValueAccessor`.

---

## Required Validator Example

```typescript
this.form = this.fb.group({
  rating: [
    null,
    Validators.required,
  ],
});
```

---

## Custom Validator Example

```typescript
import {
  AbstractControl,
  ValidationErrors,
} from '@angular/forms';

export function minimumRating(
  min: number
) {
  return (
    control: AbstractControl
  ): ValidationErrors | null => {
    if (
      control.value < min
    ) {
      return {
        minimumRating: true,
      };
    }

    return null;
  };
}
```

---

## Custom Validator Usage

```typescript
this.form = this.fb.group({
  rating: [
    0,
    [
      Validators.required,
      minimumRating(3),
    ],
  ],
});
```

---

# Disabled State Support

Angular Reactive Forms automatically call `setDisabledState()`.

---

## Disable Entire Form

```typescript
this.form.disable();
```

---

## Disable Single Control

```typescript
this.form
  .get('rating')
  ?.disable();
```

---

# Accessing Form State

Reactive Forms expose validation and interaction states.

```typescript
const control =
  this.form.get('rating');

control?.valid;

control?.invalid;

control?.touched;

control?.dirty;

control?.disabled;

control?.errors;
```

---

# Styling Based on Form State

```html
<app-rating
  formControlName="rating"
  [class.invalid]="
    form.get('rating')
      ?.touched &&
    form.get('rating')
      ?.invalid
  "
>
</app-rating>
```

---

# Form Submission Pattern

```typescript
submit(): void {
  if (this.form.invalid) {
    this.form.markAllAsTouched();

    return;
  }

  const payload =
    this.form.getRawValue();

  console.log(payload);
}
```

---

# Angular 13 Best Practices

## Always Implement Full ControlValueAccessor

Implement all required methods:

- `writeValue`
- `registerOnChange`
- `registerOnTouched`
- `setDisabledState`

---

## Always Call onTouched

Ensure touched state updates correctly.

```typescript
this.onTouched();
```

---

## Prefer Reactive Forms

Custom controls integrate better with Reactive Forms than template-driven forms.

---

## Avoid Direct FormControl Injection

Prefer `formControlName` integration instead of manually injecting controls into components.

---

## Keep Custom Controls Stateless When Possible

The form should remain the source of truth.

---

## Support Disabled State

Always implement:

```typescript
setDisabledState(
  isDisabled: boolean
): void {}
```

---

## Use markAllAsTouched on Submit

```typescript
if (this.form.invalid) {
  this.form.markAllAsTouched();

  return;
}
```

---

# Angular 13 Limitations Compared to Signal Forms

The following APIs do NOT exist in Angular 13:

- `FormValueControl`
- `FormField`
- `form()`
- Signal Forms
- `signal()`
- `model()`
- `InputSignal`
- `readonly()`
- `errors()`
- `invalid()`
- `@if`
- `@for`

Angular 13 alternatives:

| Angular v21+ Signal Forms | Angular 13 Equivalent |
|---|---|
| `FormValueControl` | `ControlValueAccessor` |
| `form()` | `FormGroup` |
| `FormField` | `formControlName` |
| `signal()` | RxJS + FormControl |
| `@if` | `*ngIf` |
| `@for` | `*ngFor` |

---

# Recommended Angular 13 Architecture

For reusable forms in Angular 13:

- Reactive Forms
- ControlValueAccessor
- FormBuilder
- FormGroup
- FormArray
- RxJS Observables
- Angular Material integration
- Reusable validator functions

This architecture is production-stable and widely adopted in enterprise Angular 13 applications.