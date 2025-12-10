# 📘 Angular Workshop Notes

## 1. Control Flow
Angular's new built-in control flow offers a more performant and developer-friendly alternative to traditional structural directives like `*ngIf` and `*ngFor`.

**Documentation:** [Angular Control Flow Guide](https://angular.dev/guide/templates/control-flow)

### Syntax Comparison

**Conditional Rendering (`@if`)**
Previously, we used the `*ngIf` directive. The new syntax is cleaner and supports `else` blocks natively without needing `ng-template`.

```ts
// ❌ Old Version
<div *ngIf="a > b; else other">
  {{a}} is greater than {{b}}
</div>
<ng-template #other>...</ng-template>

// ✅ New Version
@if (a > b) {
  <p>{{a}} is greater than {{b}}</p>
} @else {
  <p>{{b}} is greater than or equal to {{a}}</p>
}
```

**Loops (`@for`)**
The new loop syntax forces the use of `track`, which significantly improves rendering performance by helping Angular identify unique items in a list.

```ts
@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>There are no items.</li>
}
```

> **Note:** **Nx** (monorepo tool) is highly recommended for modern Angular development to manage complexity and build processes efficiently.

---

## 2. Signals
Signals are reactive primitives that act as a wrapper around a value. They notify consumers when that value changes.

*   **Reactivity:** Signals are the foundation for fine-grained reactivity (Change Detection is more performant as it doesn't need to check the whole tree).
*   **Synchronous:** Reading a signal is always synchronous.
*   **Types:** Can be `WritableSignal` (settable) or `Signal` (read-only).

### Basic Usage
```ts
// Creating a signal
const count = signal(0);

// Reading a signal (always execute as a function)
console.log('The count is: ' + count());
```

**Best Practices:**
*   **Avoid Method Calls in Templates:** Do not call complex methods in templates (e.g., `{{ calculate() }}`) as they re-run on every change detection cycle. However, calling Signals (`{{ count() }}`) is efficient and recommended.
*   **Consistency:** Use signals consistently throughout the project where reactivity is needed.
*   **Typing:** Explicitly type your signals unless the type is easily inferred.

### Updating Values
```ts
const count = signal(0);

// Set a new value directly
count.set(3);

// Update based on the previous value
count.update(value => value + 1);
```

### Computed Signals
`computed` creates a read-only signal that derives its value from other signals.

```ts
const count: WritableSignal<number> = signal(0);

// doubleCount updates automatically when count changes
const doubleCount: Signal<number> = computed(() => count() * 2);
```
*   **Dynamic Dependencies:** Angular tracks dependencies automatically. However, dependencies are only tracked if they are executed. If a signal is inside an `if` block that doesn't run, it won't be tracked as a dependency at that moment.
*   **Role:** Signal = Producer; Computed = Consumer & Producer.

### Effects
`effect` runs an operation whenever one of its signal dependencies changes. It is primarily for side effects (e.g., logging, manual DOM manipulation).

```ts
effect(() => {
  // Runs whenever currentUser or counter changes
  console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
});
```
*   **Warning:** Use effects sparingly. They can lead to infinite loops if not careful. The goal is to let data flow through computed signals to the template.

---

## 3. Component Architecture: Smart vs. Dumb
Separating concerns is crucial for scalability. This pattern is often called **Container/Presentational** or **Smart/Dumb**.

### The Pattern
1.  **Container (Smart/Stateful):**
    *   Knows about the state and services.
    *   Fetches data.
    *   Passes data down to presentational components via inputs.
    *   Handles events bubbling up from presentational components.
2.  **Presentational (Dumb/Stateless):**
    *   Focuses purely on UI/Layout.
    *   Receives data via `@Input()`.
    *   Emits user actions via `@Output()`.
    *   Contains no business logic or service injections.

### Recommended Folder Structure
Create unique components for features, even if they aren't reused, to keep the code modular.

```text
src/
└── app/
    └── todo/
        ├── container/      # Smart components (State managers)
        ├── presentational/ # Dumb components (UI only)
        ├── models/         # Interfaces/Types
        └── services/       # Data fetching logic
```

### Data Flow & Events
*   **Bubbling:** Avoid passing events up through more than 3-5 layers of components ("prop drilling"). If the hierarchy is deeper, consider using a shared Service or Store.
*   **Forms:**
    *   `form.patchValue()`: Updates a subset of the form model.
    *   `form.setValue()`: Requires the exact structure of the entire form model.
*   **Observables vs. Signals:** The modern goal is **"subscribeless"** code.
    *   Avoid `.subscribe()` in the TS file.
    *   Use the `AsyncPipe` in templates or convert Observables to Signals (`toSignal`).
    *   Streams should ideally "end" in the HTML.

**Further Reading:**
*   [Remote Data & Signals (Blog)](https://offering.solutions/blog/articles/2025/06/14/angular-2025-remote-data-signals-and-the-questions-youre-asking/)

---

## 4. Feature Architecture
Organize your application by business domain (Features), not by technical type.

**Example Structure (Book Shop):**
```text
src/
├── products/          # Feature: Product List
│   ├── container
│   ├── presentational
│   └── services
├── product-detail/    # Feature: Single Product
│   ├── container
│   ├── presentational
│   └── services
├── checkout/          # Feature: Payment/Cart
│   ├── container
│   ├── presentational
│   └── services
└── shared/            # Reusable UI/Logic
    └── cart-products  # e.g., "Add to Cart" button used in multiple features
```

---

## 5. Security (OAuth2 & OpenID Connect)
Modern authentication usually involves delegating login to an Identity Provider (IDP).

*   **OAuth2:** Handles **Authorization** (What you can do). Issues the **Access Token**.
*   **OpenID Connect (OIDC):** Handles **Authentication** (Who you are). Issues the **ID Token**.
*   **Duende / Auth0 / Azure AD:** Examples of Identity Providers (IDP).

### Authorization Code Flow (PKCE)
This is the standard flow for SPAs (Single Page Applications):
1.  **User acts:** Clicks "Login" in the Angular app.
2.  **Redirect:** App redirects browser to the IDP login page.
3.  **Auth:** User enters credentials at the IDP.
4.  **Code:** IDP redirects back to the Angular app with a temporary code (e.g., `localhost:4200?code=xyz`).
5.  **Exchange:**
    *   Angular (via a library like `angular-auth-oidc-client`) reads the code.
    *   Sends the code to the IDP via an HTTP POST.
    *   IDP validates the code and responds with the Tokens (Access, ID, Refresh).

**Important Notes:**
*   **Refresh Tokens:** Used to get a new Access Token silently (e.g., 30 seconds before the current one expires) without forcing the user to log in again.
*   **BFF (Backend for Frontend):** A highly recommended security pattern where cookies are handled by a lightweight backend proxy, keeping tokens out of the browser storage entirely.

---

## 6. Testing
Focus on writing readable, maintainable tests.

*   **Hierarchy:**
    1.  **Unit / Shallow Testing:** Testing individual components/services in isolation.
    2.  **Integration Testing:** Testing how components work together.
    3.  **E2E (End-to-End):** Testing the full user flow.
*   **Best Practices:**
    *   Keep logic out of components to make them easier to test.
    *   Mock dependencies.
    *   `ng-mocks`: Excellent library for easily mocking child components and services.
    *   **Playwright:** The current industry standard recommendation over Cypress for E2E testing due to speed and reliability.

---

## 7. State Management: NgRx Signal Store
NgRx Signal Store is a modern, lightweight, functional state management solution designed specifically for Angular Signals.

*   **Key Traits:** Functional, extensible, modular, and RxJS-optional.

### Setup
The store is created using `signalStore()`. You compose it by adding "features" like `withState`, `withMethods`, and `withComputed`.

### 1. Defining State (`withState`)
Define the shape of your state and the initial values.

```ts
import { signalStore, withState } from '@ngrx/signals';
import { Book } from './book';

type BookSearchState = {
  books: Book[];
  isLoading: boolean;
  filter: { query: string; order: 'asc' | 'desc' };
};

const initialState: BookSearchState = {
  books: [],
  isLoading: false,
  filter: { query: '', order: 'asc' },
};

export const BookSearchStore = signalStore(
  withState(initialState)
);
```

### 2. Updating State (`withMethods`)
Define methods to modify the state. Use `patchState` for updates.

```ts
withMethods((store) => ({
  updateQuery(query: string): void {
    patchState(store, (state) => ({ filter: { ...state.filter, query } }));
  },
  // Async method with Promise
  async loadAll(): Promise<void> {
    patchState(store, { isLoading: true });
    const books = await inject(BooksService).getAll();
    patchState(store, { books, isLoading: false });
  },
}))
```

### 3. RxJS Interop (`rxMethod`)
For complex async flows (debouncing, switchMap), utilize `rxMethod`. This bridges the gap between reactive streams and Signals.

**Full Example:**

```ts
import { computed, inject } from '@angular/core';
import { debounceTime, distinctUntilChanged, pipe, switchMap, tap } from 'rxjs';
import { patchState, signalStore, withComputed, withMethods, withState } from '@ngrx/signals';
import { rxMethod } from '@ngrx/signals/rxjs-interop';
import { tapResponse } from '@ngrx/operators';
import { BooksService } from './books-service';

export const BookSearchStore = signalStore(
  withState(initialState),

  // Computed values (Selectors)
  withComputed(({ books, filter }) => ({
    booksCount: computed(() => books().length),
    sortedBooks: computed(() => {
      const direction = filter.order() === 'asc' ? 1 : -1;
      return books().toSorted((a, b) => direction * a.title.localeCompare(b.title));
    }),
  })),

  // Methods (Actions/Reducers/Effects)
  withMethods((store, booksService = inject(BooksService)) => ({

    // Simple State Update
    updateQuery(query: string): void {
      patchState(store, (state) => ({ filter: { ...state.filter, query } }));
    },

    // RxJS Stream for complex async logic
    loadByQuery: rxMethod<string>(
      pipe(
        debounceTime(300),
        distinctUntilChanged(),
        tap(() => patchState(store, { isLoading: true })),
        switchMap((query) => {
          return booksService.getByQuery(query).pipe(
            tapResponse({
              next: (books) => patchState(store, { books }),
              error: (err) => console.error(err),
              finalize: () => patchState(store, { isLoading: false }),
            })
          );
        })
      )
    ),
  }))
);
```

***
*Enhanced and formatted by Gemini 3 Pro.*