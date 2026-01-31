---
name: best-practices
description: Expert guidance for TypeScript and Angular web application development. Use this skill when building Angular applications, components, services, or any TypeScript code that follows Angular best practices. Triggers on Angular component creation, service development, state management with signals, reactive forms, template syntax, accessibility requirements, and scalable web application architecture. Covers Angular v20+ standalone components, signal-based state management, and modern Angular patterns.
---

# Angular & TypeScript Development Skill

Build functional, maintainable, performant, and accessible Angular applications following modern best practices.

## TypeScript Guidelines

### Type Safety

- Enable strict type checking in `tsconfig.json`
- Prefer type inference when obvious; avoid redundant annotations
- Never use `any`; use `unknown` when type is uncertain
- Use discriminated unions for complex state

```typescript
// Prefer inference
const count = 0; // inferred as number
const items = ['a', 'b']; // inferred as string[]

// Explicit when needed
function process(data: unknown): Result {
  if (isValidData(data)) {
    return transform(data);
  }
  throw new Error('Invalid data');
}
```

## Angular Component Patterns

### Standalone Components (Default in v20+)

Never set `standalone: true` in decorators—it's the default.

```typescript
import { Component, ChangeDetectionStrategy, input, output, computed, signal } from '@angular/core';

@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <article class="user-card" [class.active]="isActive()">
      <h2>{{ name() }}</h2>
      <p>{{ formattedRole() }}</p>
      <button (click)="onSelect.emit(userId())">Select</button>
    </article>
  `,
  styles: `
    .user-card { padding: 1rem; border: 1px solid #ccc; }
    .active { border-color: blue; }
  `
})
export class UserCardComponent {
  // Input signals (replacing @Input)
  name = input.required<string>();
  userId = input.required<string>();
  role = input<string>('user');
  isActive = input(false);

  // Output (replacing @Output)
  onSelect = output<string>();

  // Computed for derived state
  formattedRole = computed(() => this.role().toUpperCase());
}
```

### Component Rules

- Single responsibility per component
- Use `input()` and `output()` functions, not decorators
- Use `computed()` for derived state
- Always set `changeDetection: ChangeDetectionStrategy.OnPush`
- Prefer inline templates for small components (<20 lines)
- Use relative paths for external templates/styles

### Host Bindings

Use `host` object instead of `@HostBinding`/`@HostListener`:

```typescript
@Component({
  selector: 'app-tooltip',
  host: {
    '[class.visible]': 'isVisible()',
    '[attr.role]': '"tooltip"',
    '(mouseenter)': 'show()',
    '(mouseleave)': 'hide()'
  },
  template: `<ng-content />`
})
export class TooltipComponent {
  isVisible = signal(false);
  show() { this.isVisible.set(true); }
  hide() { this.isVisible.set(false); }
}
```

## State Management with Signals

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Doubled: {{ doubled() }}</p>
      <button (click)="increment()">+</button>
      <button (click)="decrement()">-</button>
    </div>
  `
})
export class CounterComponent {
  // Local state with signals
  count = signal(0);
  
  // Derived state with computed
  doubled = computed(() => this.count() * 2);
  
  // Use update() or set(), never mutate()
  increment() { this.count.update(n => n + 1); }
  decrement() { this.count.update(n => n - 1); }
}
```

## Template Syntax

### Control Flow (Use Native Syntax)

```typescript
@Component({
  template: `
    @if (isLoading()) {
      <app-spinner />
    } @else if (error()) {
      <app-error [message]="error()" />
    } @else {
      <ul>
        @for (item of items(); track item.id) {
          <li>{{ item.name }}</li>
        } @empty {
          <li>No items found</li>
        }
      </ul>
    }

    @switch (status()) {
      @case ('pending') { <span>Pending...</span> }
      @case ('success') { <span>Complete!</span> }
      @default { <span>Unknown</span> }
    }
  `
})
```

### Template Rules

- Use `@if`, `@for`, `@switch` (not `*ngIf`, `*ngFor`, `*ngSwitch`)
- Use `track` in `@for` loops for performance
- Use `class` bindings instead of `ngClass`
- Use `style` bindings instead of `ngStyle`
- Never use arrow functions in templates
- Never assume globals like `Date` or `Math` are available
- Use async pipe for observables

```typescript
// Class bindings
<div [class.active]="isActive()" [class.disabled]="isDisabled()">

// Style bindings
<div [style.width.px]="width()" [style.color]="textColor()">

// Async pipe for observables
<div>{{ data$ | async }}</div>
```

## Services

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  
  getUsers() {
    return this.http.get<User[]>('/api/users');
  }
}
```

### Service Rules

- Single responsibility per service
- Use `providedIn: 'root'` for singletons
- Use `inject()` function, not constructor injection

## Reactive Forms

Prefer reactive forms over template-driven:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  imports: [ReactiveFormsModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <label for="name">Name</label>
      <input id="name" formControlName="name" />
      @if (form.controls.name.errors?.['required']) {
        <span role="alert">Name is required</span>
      }
      
      <label for="email">Email</label>
      <input id="email" type="email" formControlName="email" />
      
      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  private fb = inject(FormBuilder);
  
  form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]]
  });
  
  onSubmit() {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

## Routing with Lazy Loading

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { 
    path: 'home', 
    loadComponent: () => import('./home/home.component').then(m => m.HomeComponent)
  },
  {
    path: 'users',
    loadChildren: () => import('./users/users.routes').then(m => m.USERS_ROUTES)
  }
];

// users/users.routes.ts
export const USERS_ROUTES: Routes = [
  { path: '', loadComponent: () => import('./user-list.component').then(m => m.UserListComponent) },
  { path: ':id', loadComponent: () => import('./user-detail.component').then(m => m.UserDetailComponent) }
];
```

## Images

Use `NgOptimizedImage` for static images (not for base64):

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `
    <img ngSrc="/assets/hero.jpg" width="800" height="600" priority alt="Hero image" />
    <img ngSrc="/assets/thumbnail.jpg" width="200" height="150" alt="Thumbnail" />
  `
})
```

## Accessibility Requirements

All components must pass AXE checks and meet WCAG AA:

### Focus Management

```typescript
@Component({
  template: `
    <dialog #dialog [attr.aria-labelledby]="'dialog-title'">
      <h2 id="dialog-title">Confirm Action</h2>
      <p>Are you sure?</p>
      <button (click)="confirm()" #confirmBtn>Confirm</button>
      <button (click)="cancel()">Cancel</button>
    </dialog>
  `
})
export class ConfirmDialogComponent {
  @ViewChild('confirmBtn') confirmBtn!: ElementRef<HTMLButtonElement>;
  
  open() {
    // Focus first interactive element when dialog opens
    setTimeout(() => this.confirmBtn.nativeElement.focus());
  }
}
```

### ARIA and Semantics

```typescript
@Component({
  template: `
    <nav aria-label="Main navigation">
      <ul role="menubar">
        @for (item of menuItems(); track item.id) {
          <li role="none">
            <a 
              role="menuitem"
              [href]="item.url"
              [attr.aria-current]="isActive(item) ? 'page' : null">
              {{ item.label }}
            </a>
          </li>
        }
      </ul>
    </nav>

    <main>
      <h1>{{ pageTitle() }}</h1>
      <section aria-labelledby="section-heading">
        <h2 id="section-heading">Section Title</h2>
        <!-- content -->
      </section>
    </main>
  `
})
```

### Accessibility Checklist

- All interactive elements are keyboard accessible
- Color contrast ratio ≥ 4.5:1 for normal text, ≥ 3:1 for large text
- All images have descriptive `alt` text
- Form inputs have associated labels
- Error messages are announced to screen readers
- Focus indicators are visible
- Skip links for main content
- Heading hierarchy is logical (h1 → h2 → h3)

## Quick Reference

| Pattern | Use | Avoid |
|---------|-----|-------|
| Components | Standalone (default) | NgModules |
| Inputs | `input()` function | `@Input()` decorator |
| Outputs | `output()` function | `@Output()` decorator |
| State | Signals + `computed()` | BehaviorSubject for local state |
| Signal updates | `set()`, `update()` | `mutate()` |
| Host bindings | `host` object | `@HostBinding`/`@HostListener` |
| DI | `inject()` function | Constructor injection |
| Control flow | `@if`, `@for`, `@switch` | `*ngIf`, `*ngFor`, `*ngSwitch` |
| Classes | `[class.name]="expr"` | `[ngClass]` |
| Styles | `[style.prop]="expr"` | `[ngStyle]` |
| Forms | Reactive Forms | Template-driven |
| Routes | Lazy loading | Eager loading |
| Images | `NgOptimizedImage` | Plain `<img>` |