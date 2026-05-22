---
name: angular13-directive-patterns
description: Implement reusable Angular 13 directive patterns for DOM interaction, forms, observers, drag-and-drop, permissions, and dynamic behaviors. Use for creating reusable attribute and structural directives with Renderer2, HostBinding, HostListener, RxJS, and Angular lifecycle hooks. Triggers on implementing reusable UI behaviors, DOM observation, permissions, clipboard utilities, form enhancements, or drag-and-drop interactions in Angular 13 applications.
---

# Angular 13 Directive Patterns

## Table of Contents

- DOM Manipulation
- Form Directives
- Intersection Observer
- Resize Observer
- Drag and Drop
- Permission Directive
- Exporting Directive APIs

---

# Angular 13 Directive Notes

Angular 13 does NOT support:

- Signals
- `input()`
- `output()`
- `effect()`
- `afterNextRender()`
- `@defer`
- Native control flow (`@if`, `@for`, `@switch`)
- Host Directives
- Directive Composition API

Use:

- `@Input`
- `@Output`
- `EventEmitter`
- `@HostBinding`
- `@HostListener`
- `Renderer2`
- RxJS
- Angular lifecycle hooks

---

# DOM Manipulation

---

# Auto Focus Directive

```typescript
import {
  AfterViewInit,
  Directive,
  ElementRef,
  Input,
} from '@angular/core';

@Directive({
  selector: '[appAutoFocus]',
})
export class AutoFocusDirective
  implements AfterViewInit {

  @Input('appAutoFocus')
  enabled = true;

  @Input()
  delay = 0;

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  ngAfterViewInit(): void {

    if (!this.enabled) {
      return;
    }

    setTimeout(() => {
      this.el.nativeElement.focus();
    }, this.delay);
  }
}
```

Usage:

```html
<input appAutoFocus />

<input
  [appAutoFocus]="shouldFocus"
  [delay]="100" />
```

---

# Select All Directive

```typescript
import {
  Directive,
  ElementRef,
  HostListener,
} from '@angular/core';

@Directive({
  selector: '[appSelectAll]',
})
export class SelectAllDirective {

  constructor(
    private el:
      ElementRef<HTMLInputElement>,
  ) {}

  @HostListener('focus')
  onFocus(): void {

    setTimeout(() => {
      this.el.nativeElement.select();
    }, 0);
  }

  @HostListener(
    'click',
    ['$event'],
  )
  onClick(
    event: MouseEvent,
  ): void {

    if (
      document.activeElement
      !== this.el.nativeElement
    ) {
      this.el.nativeElement.select();
    }
  }
}
```

Usage:

```html
<input
  appSelectAll
  value="Select me on focus" />
```

---

# Copy To Clipboard Directive

```typescript
import {
  Directive,
  EventEmitter,
  HostBinding,
  HostListener,
  Input,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appCopyToClipboard]',
})
export class CopyToClipboardDirective {

  @Input('appCopyToClipboard')
  text!: string;

  @Output()
  copied =
    new EventEmitter<void>();

  @Output()
  error =
    new EventEmitter<Error>();

  @HostBinding('style.cursor')
  cursor = 'pointer';

  @HostListener('click')
  async copy(): Promise<void> {

    try {
      await navigator.clipboard
        .writeText(this.text);

      this.copied.emit();

    } catch (err) {

      this.error.emit(
        err as Error,
      );
    }
  }
}
```

Usage:

```html
<button
  [appCopyToClipboard]="textToCopy"
  (copied)="showToast('Copied!')">

  Copy
</button>
```

---

# Form Directives

---

# Trim Input Directive

```typescript
import {
  Directive,
  ElementRef,
  HostListener,
  Optional,
  Self,
} from '@angular/core';

import {
  NgControl,
} from '@angular/forms';

@Directive({
  selector:
    'input[appTrim], textarea[appTrim]',
})
export class TrimDirective {

  constructor(
    private el:
      ElementRef<
        HTMLInputElement
        | HTMLTextAreaElement
      >,

    @Optional()
    @Self()
    private ngControl?:
      NgControl,
  ) {}

  @HostListener('blur')
  onBlur(): void {

    const value =
      this.el.nativeElement.value;

    const trimmed =
      value.trim();

    if (value !== trimmed) {

      this.el.nativeElement.value =
        trimmed;

      this.ngControl
        ?.control
        ?.setValue(trimmed);
    }
  }
}
```

Usage:

```html
<input
  appTrim
  formControlName="name" />
```

---

# Input Mask Directive

```typescript
import {
  Directive,
  ElementRef,
  HostListener,
  Input,
} from '@angular/core';

@Directive({
  selector: '[appMask]',
})
export class MaskDirective {

  @Input('appMask')
  mask!: string;

  constructor(
    private el:
      ElementRef<HTMLInputElement>,
  ) {}

  @HostListener(
    'input',
    ['$event'],
  )
  onInput(
    event: InputEvent,
  ): void {

    const input =
      this.el.nativeElement;

    const value =
      input.value;

    const masked =
      this.applyMask(value);

    if (value !== masked) {
      input.value = masked;
    }
  }

  @HostListener(
    'keydown',
    ['$event'],
  )
  onKeydown(
    event: KeyboardEvent,
  ): void {

    const allowedKeys = [
      'Backspace',
      'Delete',
      'ArrowLeft',
      'ArrowRight',
      'Tab',
    ];

    if (
      allowedKeys.includes(
        event.key,
      )
    ) {
      return;
    }

    const input =
      this.el.nativeElement;

    const position =
      input.selectionStart ?? 0;

    const maskChar =
      this.mask[position];

    if (!maskChar) {
      event.preventDefault();
      return;
    }

    if (
      !this.isValidChar(
        event.key,
        maskChar,
      )
    ) {
      event.preventDefault();
    }
  }

  private applyMask(
    value: string,
  ): string {

    let result = '';
    let valueIndex = 0;

    for (
      let i = 0;
      i < this.mask.length
        && valueIndex < value.length;
      i++
    ) {

      const maskChar =
        this.mask[i];

      const inputChar =
        value[valueIndex];

      if (
        maskChar === '9'
        || maskChar === 'A'
        || maskChar === '*'
      ) {

        if (
          this.isValidChar(
            inputChar,
            maskChar,
          )
        ) {

          result += inputChar;
          valueIndex++;

        } else {

          valueIndex++;
          i--;
        }

      } else {

        result += maskChar;

        if (
          inputChar === maskChar
        ) {
          valueIndex++;
        }
      }
    }

    return result;
  }

  private isValidChar(
    char: string,
    maskChar: string,
  ): boolean {

    switch (maskChar) {

      case '9':
        return /\d/.test(char);

      case 'A':
        return /[a-zA-Z]/.test(char);

      case '*':
        return /[a-zA-Z0-9]/.test(char);

      default:
        return char === maskChar;
    }
  }
}
```

Usage:

```html
<input
  appMask="(999) 999-9999"
  placeholder="(555) 123-4567" />
```

---

# Character Counter Directive

Angular 13 does not support signals.

Use directive state with exported references.

```typescript
import {
  Directive,
  ElementRef,
  HostListener,
  Input,
} from '@angular/core';

@Directive({
  selector: '[appCharCount]',
  exportAs: 'appCharCount',
})
export class CharCountDirective {

  @Input('appCharCount')
  maxLength!: number;

  currentLength = 0;

  constructor(
    private el:
      ElementRef<
        HTMLInputElement
        | HTMLTextAreaElement
      >,
  ) {
    this.updateCount();
  }

  @HostListener('input')
  onInput(): void {
    this.updateCount();
  }

  get remaining(): number {
    return (
      this.maxLength
      - this.currentLength
    );
  }

  get isOverLimit(): boolean {
    return this.remaining < 0;
  }

  private updateCount(): void {
    this.currentLength =
      this.el.nativeElement
        .value.length;
  }
}
```

Usage:

```html
<textarea
  appCharCount="500"
  #counter="appCharCount">
</textarea>

<span>
  {{ counter.remaining }}
  characters remaining
</span>
```

---

# Intersection Observer

---

# Lazy Load Directive

```typescript
import {
  AfterViewInit,
  Directive,
  ElementRef,
  EventEmitter,
  Input,
  OnDestroy,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appLazyLoad]',
})
export class LazyLoadDirective
  implements
    AfterViewInit,
    OnDestroy {

  @Input('appLazyLoad')
  src!: string;

  @Input()
  placeholder =
    '/assets/placeholder.png';

  @Output()
  loaded =
    new EventEmitter<void>();

  private observer?:
    IntersectionObserver;

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  ngAfterViewInit(): void {
    this.setupObserver();
  }

  private setupObserver():
    void {

    this.observer =
      new IntersectionObserver(
        entries => {

          entries.forEach(
            entry => {

              if (
                entry.isIntersecting
              ) {

                this.loadImage();

                this.observer
                  ?.disconnect();
              }
            },
          );
        },
        {
          rootMargin: '50px',
        },
      );

    this.observer.observe(
      this.el.nativeElement,
    );

    if (
      this.el.nativeElement
      instanceof HTMLImageElement
    ) {

      this.el.nativeElement.src =
        this.placeholder;
    }
  }

  private loadImage():
    void {

    const element =
      this.el.nativeElement;

    if (
      element
      instanceof HTMLImageElement
    ) {

      element.src = this.src;

      element.onload = () => {
        this.loaded.emit();
      };

    } else {

      element.style.backgroundImage =
        `url(${this.src})`;

      this.loaded.emit();
    }
  }

  ngOnDestroy(): void {
    this.observer?.disconnect();
  }
}
```

Usage:

```html
<img
  [appLazyLoad]="imageUrl"
  alt="Lazy loaded image" />
```

---

# Infinite Scroll Directive

```typescript
import {
  AfterViewInit,
  Directive,
  ElementRef,
  EventEmitter,
  Input,
  OnDestroy,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appInfiniteScroll]',
})
export class InfiniteScrollDirective
  implements
    AfterViewInit,
    OnDestroy {

  @Input()
  threshold = 0.1;

  @Input()
  disabled = false;

  @Output()
  scrolled =
    new EventEmitter<void>();

  private observer?:
    IntersectionObserver;

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  ngAfterViewInit(): void {
    this.setupObserver();
  }

  ngOnChanges(): void {

    if (this.disabled) {
      this.observer?.disconnect();
      return;
    }

    this.setupObserver();
  }

  private setupObserver():
    void {

    this.observer?.disconnect();

    this.observer =
      new IntersectionObserver(
        entries => {

          if (
            entries[0]
              .isIntersecting
            && !this.disabled
          ) {

            this.scrolled.emit();
          }
        },
        {
          threshold:
            this.threshold,
        },
      );

    this.observer.observe(
      this.el.nativeElement,
    );
  }

  ngOnDestroy(): void {
    this.observer?.disconnect();
  }
}
```

Usage:

```html
<div class="list">

  <div
    *ngFor="
      let item of items
    ">
    {{ item.name }}
  </div>

  <div
    appInfiniteScroll
    (scrolled)="loadMore()"
    [disabled]="isLoading">

    Loading...
  </div>

</div>
```

---

# Resize Observer Directive

```typescript
import {
  AfterViewInit,
  Directive,
  ElementRef,
  EventEmitter,
  OnDestroy,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appResize]',
  exportAs: 'appResize',
})
export class ResizeDirective
  implements
    AfterViewInit,
    OnDestroy {

  width = 0;

  height = 0;

  @Output()
  resized =
    new EventEmitter<{
      width: number;
      height: number;
    }>();

  private observer?:
    ResizeObserver;

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  ngAfterViewInit(): void {

    this.observer =
      new ResizeObserver(
        entries => {

          const entry =
            entries[0];

          const {
            width,
            height,
          } =
            entry.contentRect;

          this.width = width;
          this.height = height;

          this.resized.emit({
            width,
            height,
          });
        },
      );

    this.observer.observe(
      this.el.nativeElement,
    );
  }

  ngOnDestroy(): void {
    this.observer?.disconnect();
  }
}
```

Usage:

```html
<div
  appResize
  #resize="appResize">

  Size:
  {{ resize.width }}
  x
  {{ resize.height }}
</div>
```

---

# Drag And Drop

---

# Draggable Directive

```typescript
import {
  Directive,
  EventEmitter,
  HostBinding,
  HostListener,
  Input,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appDraggable]',
})
export class DraggableDirective {

  @Input('appDraggable')
  data: any = null;

  @Input()
  effectAllowed:
    DataTransfer['effectAllowed']
    = 'move';

  @Output()
  dragStart =
    new EventEmitter<DragEvent>();

  @Output()
  dragEnd =
    new EventEmitter<DragEvent>();

  @HostBinding('draggable')
  draggable = true;

  @HostBinding('class.dragging')
  isDragging = false;

  @HostListener(
    'dragstart',
    ['$event'],
  )
  onDragStart(
    event: DragEvent,
  ): void {

    this.isDragging = true;

    if (event.dataTransfer) {

      event.dataTransfer
        .effectAllowed =
          this.effectAllowed;

      event.dataTransfer
        .setData(
          'application/json',
          JSON.stringify(
            this.data,
          ),
        );
    }

    this.dragStart.emit(
      event,
    );
  }

  @HostListener(
    'dragend',
    ['$event'],
  )
  onDragEnd(
    event: DragEvent,
  ): void {

    this.isDragging = false;

    this.dragEnd.emit(
      event,
    );
  }
}
```

---

# Drop Zone Directive

```typescript
import {
  Directive,
  EventEmitter,
  HostBinding,
  HostListener,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appDropZone]',
})
export class DropZoneDirective {

  @Output()
  dropped =
    new EventEmitter<any>();

  @HostBinding('class.drag-over')
  isDragOver = false;

  @HostListener(
    'dragover',
    ['$event'],
  )
  onDragOver(
    event: DragEvent,
  ): void {

    event.preventDefault();

    this.isDragOver = true;
  }

  @HostListener(
    'dragleave',
    ['$event'],
  )
  onDragLeave(
    event: DragEvent,
  ): void {

    this.isDragOver = false;
  }

  @HostListener(
    'drop',
    ['$event'],
  )
  onDrop(
    event: DragEvent,
  ): void {

    event.preventDefault();

    this.isDragOver = false;

    const data =
      event.dataTransfer
        ?.getData(
          'application/json',
        );

    if (data) {

      this.dropped.emit(
        JSON.parse(data),
      );
    }
  }
}
```

Usage:

```html
<div [appDraggable]="item">
  Drag me
</div>

<div
  appDropZone
  (dropped)="onItemDropped($event)">

  Drop here
</div>
```

---

# Permission Directive

```typescript
import {
  Directive,
  Input,
  OnChanges,
  TemplateRef,
  ViewContainerRef,
} from '@angular/core';

import { AuthService }
  from './auth.service';

@Directive({
  selector: '[appHasPermission]',
})
export class HasPermissionDirective
  implements OnChanges {

  @Input('appHasPermission')
  permission!:
    string | string[];

  @Input()
  mode:
    'any'
    | 'all'
    = 'any';

  private hasView = false;

  constructor(
    private templateRef:
      TemplateRef<any>,

    private viewContainer:
      ViewContainerRef,

    private authService:
      AuthService,
  ) {}

  ngOnChanges(): void {

    const hasPermission =
      this.checkPermission();

    if (
      hasPermission
      && !this.hasView
    ) {

      this.viewContainer
        .createEmbeddedView(
          this.templateRef,
        );

      this.hasView = true;

    } else if (
      !hasPermission
      && this.hasView
    ) {

      this.viewContainer.clear();

      this.hasView = false;
    }
  }

  private checkPermission():
    boolean {

    const permissions =
      Array.isArray(
        this.permission,
      )
        ? this.permission
        : [this.permission];

    const userPermissions =
      this.authService
        .permissions;

    if (
      this.mode === 'all'
    ) {

      return permissions.every(
        permission =>
          userPermissions
            .includes(
              permission,
            ),
      );
    }

    return permissions.some(
      permission =>
        userPermissions
          .includes(
            permission,
          ),
    );
  }
}
```

Usage:

```html
<button
  *appHasPermission="'admin'">

  Admin Only
</button>

<div
  *appHasPermission="
    ['edit', 'delete'];
    mode: 'all'
  ">

  Edit & Delete
</div>
```

---

# Export Directive Reference

Angular 13 uses:
- `exportAs`
- directive public properties
- directive public methods

instead of signals.

```typescript
import {
  Directive,
} from '@angular/core';

@Directive({
  selector: '[appToggle]',
  exportAs: 'appToggle',
})
export class ToggleDirective {

  isOpen = false;

  toggle(): void {
    this.isOpen =
      !this.isOpen;
  }

  open(): void {
    this.isOpen = true;
  }

  close(): void {
    this.isOpen = false;
  }
}
```

Usage:

```html
<div
  appToggle
  #toggle="appToggle">

  <button
    (click)="toggle.toggle()">

    Toggle
  </button>

  <div *ngIf="toggle.isOpen">
    Content
  </div>

</div>
```

---

# Angular 13 Directive Best Practices

Prefer:

- `Renderer2`
- `@HostBinding`
- `@HostListener`
- Small focused directives
- Cleanup in `ngOnDestroy`
- `exportAs` for reusable APIs
- Strong typing
- Accessibility support
- RxJS interoperability

Avoid:

- Signals
- Experimental Angular APIs
- Large directives with business logic
- Direct DOM mutation when Renderer2 is possible
- Memory leaks
- Unsafe HTML manipulation
- Overusing structural directives
```