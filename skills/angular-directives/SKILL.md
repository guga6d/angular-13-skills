---
name: angular-directives
description: Create custom directives in Angular 13 for DOM manipulation and reusable behavior extension. Use for attribute directives that modify element behavior/appearance, structural directives for dynamic rendering, overlays and portals, and reusable DOM behaviors. Triggers on creating reusable DOM behaviors, extending element functionality, handling keyboard/mouse interactions, or composing reusable behaviors across Angular 13 components.
---

# Angular 13 Directives

Create reusable directives in Angular 13 using:
- `@Directive`
- `@Input`
- `@Output`
- `@HostBinding`
- `@HostListener`
- `Renderer2`
- `ElementRef`

Angular 13 does NOT support:
- Signals
- `input()`
- `output()`
- `effect()`
- `hostDirectives`
- Directive Composition API
- Native control flow (`@if`, `@for`, `@switch`)

Use classic Angular directive patterns.

---

# Attribute Directives

Attribute directives modify:
- appearance
- behavior
- interaction
- accessibility

of existing DOM elements.

---

# Basic Highlight Directive

```typescript
import {
  Directive,
  ElementRef,
  Input,
  OnChanges,
  Renderer2,
  SimpleChanges,
} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective
  implements OnChanges {

  @Input('appHighlight')
  color = 'yellow';

  constructor(
    private el:
      ElementRef<HTMLElement>,

    private renderer:
      Renderer2,
  ) {}

  ngOnChanges(
    changes: SimpleChanges,
  ): void {

    this.renderer.setStyle(
      this.el.nativeElement,
      'background-color',
      this.color,
    );
  }
}
```

Usage:

```html
<p appHighlight="lightblue">
  Highlighted text
</p>

<p appHighlight>
  Default yellow highlight
</p>
```

---

# Host Bindings and Host Listeners

Angular 13 commonly uses:
- `@HostBinding`
- `@HostListener`

for directive host interaction.

---

# Tooltip Directive

```typescript
import {
  Directive,
  ElementRef,
  HostListener,
  Input,
  OnDestroy,
} from '@angular/core';

@Directive({
  selector: '[appTooltip]',
})
export class TooltipDirective
  implements OnDestroy {

  @Input('appTooltip')
  text!: string;

  @Input()
  position:
    'top'
    | 'bottom'
    | 'left'
    | 'right'
    = 'top';

  private tooltipEl?:
    HTMLElement;

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  @HostListener('mouseenter')
  show(): void {

    this.tooltipEl =
      document.createElement(
        'div',
      );

    this.tooltipEl.textContent =
      this.text;

    this.tooltipEl.className =
      `tooltip tooltip-${this.position}`;

    this.tooltipEl.setAttribute(
      'role',
      'tooltip',
    );

    document.body.appendChild(
      this.tooltipEl,
    );

    this.positionTooltip();
  }

  @HostListener('mouseleave')
  hide(): void {

    this.tooltipEl?.remove();

    this.tooltipEl = undefined;
  }

  ngOnDestroy(): void {
    this.hide();
  }

  private positionTooltip():
    void {

    // Tooltip positioning logic
  }
}
```

Usage:

```html
<button
  appTooltip="Click to save"
  position="bottom">

  Save
</button>
```

---

# Class and Style Manipulation

```typescript
import {
  Directive,
  HostBinding,
  Input,
} from '@angular/core';

@Directive({
  selector: '[appButton]',
})
export class ButtonDirective {
  @Input()
  variant:
    'primary'
    | 'secondary'
    = 'primary';

  @Input()
  size:
    'small'
    | 'medium'
    | 'large'
    = 'medium';

  @Input()
  disabled = false;

  @HostBinding('class')
  get hostClasses(): string {

    return [
      'btn',
      `btn-${this.variant}`,
      `btn-${this.size}`,
      this.disabled
        ? 'disabled'
        : '',
    ].join(' ');
  }

  @HostBinding('attr.disabled')
  get disabledAttr():
    '' | null {

    return this.disabled
      ? ''
      : null;
  }
}
```

Usage:

```html
<button
  appButton
  variant="primary"
  size="large">

  Click
</button>
```

---

# Event Handling Directive

```typescript
import {
  Directive,
  ElementRef,
  EventEmitter,
  HostListener,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appClickOutside]',
})
export class ClickOutsideDirective {
  @Output()
  clickOutside =
    new EventEmitter<void>();

  constructor(
    private el:
      ElementRef<HTMLElement>,
  ) {}

  @HostListener(
    'document:click',
    ['$event'],
  )
  onDocumentClick(
    event: MouseEvent,
  ): void {

    const clickedInside =
      this.el.nativeElement.contains(
        event.target as Node,
      );

    if (!clickedInside) {
      this.clickOutside.emit();
    }
  }
}
```

Usage:

```html
<div
  appClickOutside
  (clickOutside)="closeMenu()">

  ...
</div>
```

---

# Keyboard Shortcut Directive

```typescript
import {
  Directive,
  EventEmitter,
  HostListener,
  Input,
  Output,
} from '@angular/core';

@Directive({
  selector: '[appShortcut]',
})
export class ShortcutDirective {
  @Input('appShortcut')
  key!: string;

  @Input()
  ctrl = false;

  @Input()
  shift = false;

  @Input()
  alt = false;

  @Output()
  triggered =
    new EventEmitter<
      KeyboardEvent
    >();

  @HostListener(
    'document:keydown',
    ['$event'],
  )
  onKeydown(
    event: KeyboardEvent,
  ): void {

    const keyMatch =
      event.key.toLowerCase()
      === this.key.toLowerCase();

    const ctrlMatch =
      this.ctrl
        ? event.ctrlKey
          || event.metaKey
        : true;

    const shiftMatch =
      this.shift
        ? event.shiftKey
        : true;

    const altMatch =
      this.alt
        ? event.altKey
        : true;

    if (
      keyMatch
      && ctrlMatch
      && shiftMatch
      && altMatch
    ) {
      event.preventDefault();

      this.triggered.emit(
        event,
      );
    }
  }
}
```

Usage:

```html
<button
  appShortcut="s"
  [ctrl]="true"
  (triggered)="save()">

  Save
</button>
```

---

# Structural Directives

Angular 13 structural directives use:
- `TemplateRef`
- `ViewContainerRef`

Angular 13 still relies on:
- `*ngIf`
- `*ngFor`
- `*ngSwitch`

for standard control flow.

Custom structural directives should be used only for advanced rendering behavior.

---

# Portal Directive

```typescript
import {
  Directive,
  EmbeddedViewRef,
  Input,
  OnDestroy,
  OnInit,
  TemplateRef,
  ViewContainerRef,
} from '@angular/core';

@Directive({
  selector: '[appPortal]',
})
export class PortalDirective
  implements
    OnInit,
    OnDestroy {

  @Input('appPortal')
  target:
    string
    | HTMLElement
    = 'body';

  private viewRef?:
    EmbeddedViewRef<any>;

  constructor(
    private templateRef:
      TemplateRef<any>,

    private viewContainer:
      ViewContainerRef,
  ) {}

  ngOnInit(): void {
    const container =
      this.getContainer();

    if (container) {
      this.viewRef =
        this.viewContainer
          .createEmbeddedView(
            this.templateRef,
          );

      this.viewRef.rootNodes
        .forEach(node => {
          container.appendChild(
            node,
          );
        });
    }
  }

  ngOnDestroy(): void {
    this.viewRef?.destroy();
  }

  private getContainer():
    HTMLElement | null {

    if (
      typeof this.target
      === 'string'
    ) {
      return document
        .querySelector(
          this.target,
        );
    }

    return this.target;
  }
}
```

Usage:

```html
<div *appPortal="'body'">
  <div class="modal">
    Modal content
  </div>
</div>
```

---

# Lazy Render Directive

Angular 13 does NOT support:
- `@defer`
- signal-based rendering

Use classic structural directives.

```typescript
import {
  Directive,
  Input,
  OnChanges,
  SimpleChanges,
  TemplateRef,
  ViewContainerRef,
} from '@angular/core';

@Directive({
  selector: '[appLazyRender]',
})
export class LazyRenderDirective
  implements OnChanges {

  @Input('appLazyRender')
  condition = false;

  private rendered = false;

  constructor(
    private templateRef:
      TemplateRef<any>,

    private viewContainer:
      ViewContainerRef,
  ) {}

  ngOnChanges(
    changes: SimpleChanges,
  ): void {

    if (
      this.condition
      && !this.rendered
    ) {
      this.viewContainer
        .createEmbeddedView(
          this.templateRef,
        );

      this.rendered = true;
    }
  }
}
```

Usage:

```html
<div
  *appLazyRender="
    activeTab === 'reports'
  ">

  <app-heavy-reports>
  </app-heavy-reports>
</div>
```

---

# Template Outlet Directive

```typescript
import {
  Directive,
  EmbeddedViewRef,
  Input,
  OnChanges,
  SimpleChanges,
  TemplateRef,
  ViewContainerRef,
} from '@angular/core';

interface TemplateContext<T> {
  $implicit: T;

  item: T;

  index: number;
}

@Directive({
  selector: '[appTemplateOutlet]',
})
export class TemplateOutletDirective<T>
  implements OnChanges {

  @Input('appTemplateOutlet')
  template!:
    TemplateRef<
      TemplateContext<T>
    >;

  @Input(
    'appTemplateOutletContext'
  )
  context!: T;

  @Input(
    'appTemplateOutletIndex'
  )
  index = 0;

  private currentView?:
    EmbeddedViewRef<
      TemplateContext<T>
    >;

  constructor(
    private viewContainer:
      ViewContainerRef,
  ) {}

  ngOnChanges(
    changes: SimpleChanges,
  ): void {

    if (!this.currentView) {

      this.currentView =
        this.viewContainer
          .createEmbeddedView(
            this.template,
            {
              $implicit:
                this.context,

              item:
                this.context,

              index:
                this.index,
            },
          );

      return;
    }

    this.currentView.context
      .$implicit =
        this.context;

    this.currentView.context
      .item =
        this.context;

    this.currentView.context
      .index =
        this.index;

    this.currentView
      .markForCheck();
  }
}
```

Usage:

```html
<ng-template
  #itemTemplate
  let-item
  let-i="index">

  <div>
    {{ i }}:
    {{ item.name }}
  </div>
</ng-template>

<ng-container
  *appTemplateOutlet="
    itemTemplate;
    context: item;
    index: i
  ">
</ng-container>
```

---

# Reusable Behavior Composition

Angular 13 does NOT support:
- `hostDirectives`
- Directive Composition API

Reuse behavior through:
- inheritance
- shared services
- utility classes
- directive stacking

---

# Directive Stacking

Multiple directives can be applied to the same element.

```html
<button
  appButton
  appTooltip="Save item"
  appShortcut="s">

  Save
</button>
```

---

# Base Directive Pattern

```typescript
export abstract class FocusableDirective {
  protected focused = false;

  onFocus(): void {
    this.focused = true;
  }

  onBlur(): void {
    this.focused = false;
  }
}
```

---

# Extended Directive

```typescript
@Directive({
  selector: '[appFocusable]',
})
export class AppFocusableDirective
  extends FocusableDirective {

  @HostBinding('class.focused')
  get isFocused():
    boolean {

    return this.focused;
  }

  @HostListener('focus')
  handleFocus(): void {
    this.onFocus();
  }

  @HostListener('blur')
  handleBlur(): void {
    this.onBlur();
  }
}
```

---

# Accessibility Requirements

Directives MUST:
- Preserve keyboard navigation
- Support screen readers
- Maintain focus visibility
- Use ARIA attributes when needed
- Avoid breaking semantic HTML

---

# Renderer2 Best Practices

Prefer `Renderer2` over direct DOM manipulation when possible.

```typescript
constructor(
  private renderer:
    Renderer2,

  private el:
    ElementRef,
) {}

this.renderer.addClass(
  this.el.nativeElement,
  'active',
);
```

Avoid:

```typescript
element.style.color = 'red';
element.innerHTML = 'unsafe';
```

---

# Angular 13 Directive Best Practices

Prefer:
- `Renderer2`
- `@HostBinding`
- `@HostListener`
- Reusable attribute directives
- Strong typing
- Cleanup in `ngOnDestroy`
- Directive composition through stacking
- Small focused directives

Avoid:
- Direct DOM manipulation
- Large complex directives
- Business logic inside directives
- Memory leaks
- Manipulating unsafe HTML
- Experimental Angular APIs
- Overusing structural directives