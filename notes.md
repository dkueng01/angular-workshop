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
