# Angular Workshop Notes
## Control flow
*ngIf wird jetzt neu **control flow**
https://angular.dev/guide/templates/control-flow

```ts
// old version
<div *ngIf="a > b">...</div>

// new version
@if (a > b) {
  <p>{{a}} is greater than {{b}}</p>
}

@for (item of items; track item.name) {
  <li> {{ item.name }}</li>
}

@empty {
  <li> There are no items. </li>
}
```

---

Nx wird immer empfohlen!!

---

## Signals
sind nur property und function hey es hat sich was geändert
sind für ui (changedetection -> performanter)
sind immer synchron
können read only oder writeable sein

```ts
const count = signal(0);

console.log('The count is: ' + count())
```

nicht im component template keine method calls, wegen performance!!
signals kann man machen

signals kann man auch typisieren
auch immer empfohlen außer wenn es ist ableitbar
im projekt consistent sein!

mit .set(...) werte setzen
es gibt auch update

```ts
const count = signal(0);

count.set(3);

// Increment the count by 1.
count.update(value => value + 1);
```

update eigentlich das gleiche wie set nur mit callback funktion

computed um etwas abzuleiten von einem signal

```ts
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```

wenn sich count ändert weiß angular das im doubleCount computed eine referenz hat und updated sich automatisch
erkennt nur abhänigkeiten wenn der code auch wirklich im compute durchlaufen wird
abhängigkeiten im computed immer initial definieren um das problem zu umgehen
signals machen wenn geht, hat fast nur vorteile und reduced auch rsjx

signal ist produce
computed ist consumer and producer
effect ist nur consumer

```ts
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
});
```

effects nur bedacht verwenden

## Statefull and Stateless Components
statefull und stateless components sind zu empfehlen
es gibt immer eine container component der für das holen der daten zuständig
und dann in den presentation components werden nur die daten angezeigt die der container mitgibt
man kann im code zb auch die components in folders aufteilen, z.b. smart and dumb or statefull and stateless or container and presentation

jemehr dumb components desto leichter wird es das projekt zu erweitern

projektstruktur:
```ts
src/
└── app/
    └── todo/
        ├── container //smart components with states
        ├── models
        ├── presentational //dumb components only displaying states
        └── services
```

es ist auch wichtig components zu erstellen für uniqe components nicht nur für reusable components

sidefact: todo-form.component.ts with new angular version when creating looks now like this "todo-form.ts"

bubbling the events is a problem with this kind of project structure, it is advised to go not more than 3-5 layers done for bubbling events up to the statefull component

there could also be a shell component with many statefull components in there

form.patchValue can also pass a subset value and form.setValue needs the whole state

for observables:
.subscribe is waiting for the whole response and then returns it

the goal is to not have any subscribes in the code so that the whole codebaes is reactive
the stream should always end in the html

Blog post remote data signals questions:
https://offering.solutions/blog/articles/2025/06/14/angular-2025-remote-data-signals-and-the-questions-youre-asking/


## Architecture
Build it from all the features
for example book shop
products page
product detail page
checkout page

```ts
src/
├── products/
│   ├── container
│   ├── presentational
│   └── services
├── product-detail/
│   ├── container
│   ├── presentational
│   └── services
├── checkout/
│   ├── container
│   ├── presentational
│   └── services
└── shared/
    └── cart-products
```

products, product-detail and checkout feature all use the "add to Card" so the cart-products needs to be shared accross all of them

## Security
OAuth2 & OpenID Connect

OAuth2 is for the AccessToken
OpenID is for your Information (IDToken)

OAuth2 = WHAT you can do
OpenID Connect = WHO you are

RefreshToken

Dudende = Identity and access management solution which returns you the token

1: Login Button in our App
2: Redirect to the IDP (Duende, Auth0, Azure AD)
3: User/Pass on the IDP
4: Redirected back to app WITH A CODE!!! (localhost:4200?code=123123132123123123)
5: Angular Application Loads again
5.1: Lib reads the code
5.2: Lib send this code via HTTP Post reqiest back to the IDP
5.3: The IDP response with the token

https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow

https://damienbod.com/

