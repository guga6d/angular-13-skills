# Angular Skills for Angular 13

This repository contains adaptations of the original Angular skills created by the AnalogJS project, rewritten to work with older Angular versions, mainly Angular 13.

## Based On

The skills in this repository were inspired by the official AnalogJS Angular Skills project:

- https://github.com/analogjs/angular-skills

## Documentation Used

All adaptations were created following the official Angular 13 documentation:

- https://v13.angular.io/docs

## Purpose

The goal of this project is to provide best practices, examples, and patterns compatible with Angular 13, since many modern Angular skills rely on APIs and features introduced only in newer Angular versions, such as:

- Signals
- Standalone Components
- Functional Guards
- `provideRouter`
- `inject()`
- `httpResource()`
- `resource()`
- Signal Forms
- Modern Angular APIs introduced after Angular 14+

This repository adapts those concepts to Angular 13-compatible patterns using:

- NgModules
- Reactive Forms
- RxJS
- Class-based Guards
- Traditional HttpClient
- ActivatedRoute
- Services
- Classic Interceptors
- Module-based Lazy Loading

## Project Status

> ⚠️ This repository is **not actively maintained**.

Reasons:

- Older Angular versions no longer receive official updates.
- Many modern APIs do not exist in Angular 13.
- The Angular ecosystem is moving toward Signals and Standalone APIs.
- The Angular team has created an official skills repository:

  - https://github.com/angular/skills

## Compatibility

These adaptations were primarily designed for:

- Angular 13
- TypeScript versions compatible with Angular 13

## Structure

The skills were adapted to preserve as much of the original intent as possible while respecting:

- Angular 13 syntax
- APIs available at the time
- Official Angular 13 best practices

## Important Notes

Some features available in modern Angular/AnalogJS skills do not have direct equivalents in Angular 13. In those cases, idiomatic Angular 13 alternatives were used.

Examples:

| Modern Angular | Angular 13 |
|---|---|
| Signals | RxJS + BehaviorSubject |
| Signal Forms | Reactive Forms |
| Standalone Components | NgModules |
| Functional Guards | Class-based Guards |
| `inject()` | Constructor Injection |
| `httpResource()` | HttpClient + RxJS |
| `resource()` | Observables/Services |
| `input()` signals | `@Input()` |

## Credits

All credit for the original skill ideas goes to:

- AnalogJS Team  
  https://github.com/analogjs/angular-skills