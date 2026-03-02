# Stimulus Gotchas & Debugging

Common mistakes, pitfalls, and debugging strategies.

---

## Naming Mistakes

### Controller Name Mismatch

```javascript
// File: assets/controllers/userProfile_controller.js  ❌ Wrong
// File: assets/controllers/user_profile_controller.js ✅ Correct
```

```html
<!-- data-controller must match file name (underscores become dashes) -->
<div data-controller="userProfile">  ❌ Wrong
<div data-controller="user-profile"> ✅ Correct
```

### Target Name Case

```javascript
static targets = ['errorMessage']  // camelCase in JS
```

```html
<!-- camelCase preserved in HTML -->
<span data-search-target="errorMessage">   ✅ Correct
<span data-search-target="error-message">  ❌ Wrong
```

### Value Name Case

```javascript
static values = { apiUrl: String }  // camelCase in JS
```

```html
<!-- kebab-case in HTML -->
<div data-search-api-url-value="/api">  ✅ Correct
<div data-search-apiUrl-value="/api">   ❌ Wrong
```

---

## Scope Issues

### Target Outside Controller Scope

```html
<div data-controller="tabs">
    <button data-tabs-target="tab">Tab 1</button>  ✅ Inside scope
</div>
<div data-tabs-target="panel">Content</div>  ❌ Outside scope - won't work
```

### Nested Controller Confusion

```html
<div data-controller="parent">
    <div data-controller="child">
        <span data-parent-target="item">  ❌ Child blocks parent's access
        <span data-child-target="item">   ✅ Correct scope
    </div>
</div>
```

Parent cannot see targets inside nested child controller scope.

---

## Lifecycle Timing

### Accessing Targets Too Early

```javascript
export default class extends Controller {
    static targets = ['output'];

    initialize() {
        this.outputTarget.textContent = 'Hello';  ❌ Targets not ready
    }

    connect() {
        this.outputTarget.textContent = 'Hello';  ✅ Targets available
    }
}
```

### Not Cleaning Up

```javascript
export default class extends Controller {
    connect() {
        this.interval = setInterval(() => this.tick(), 1000);
        window.addEventListener('resize', this.handleResize);  ❌ Leaks
    }

    // Missing disconnect() - causes memory leaks with Turbo!

    disconnect() {
        clearInterval(this.interval);
        window.removeEventListener('resize', this.handleResize);  ✅
    }
}
```

### Binding `this` for Event Listeners

```javascript
export default class extends Controller {
    connect() {
        // ❌ Wrong - `this` will be wrong when called
        window.addEventListener('resize', this.handleResize);
        
        // ✅ Correct - bind or use arrow function
        this.boundResize = this.handleResize.bind(this);
        window.addEventListener('resize', this.boundResize);
    }

    disconnect() {
        window.removeEventListener('resize', this.boundResize);
    }
}
```

---

## Value Gotchas

### Values Are Strings in HTML

```html
<div data-counter-count-value="5">  <!-- "5" is a string -->
```

Stimulus converts based on type declaration:

```javascript
static values = { 
    count: Number,  // Converted to number
    enabled: Boolean  // "true"/"false" converted to boolean
}
```

### Boolean Values

```html
<div data-toggle-enabled-value="true">   ✅ Works
<div data-toggle-enabled-value="false">  ✅ Works
<div data-toggle-enabled-value="">       ❌ Empty string, not false
<div>                                    <!-- Missing = uses default -->
```

### Array/Object Values Need Valid JSON

```html
<div data-chart-data-value='[1,2,3]'>           ✅ Valid JSON
<div data-chart-data-value="[1,2,3]">           ✅ Also works
<div data-chart-data-value='{"a": 1}'>          ✅ Valid JSON
<div data-chart-data-value='{a: 1}'>            ❌ Invalid JSON
<div data-chart-data-value="{{ data }}">        ❌ May produce invalid JSON
<div data-chart-data-value="{{ data|json_encode|e('html_attr') }}">  ✅ Twig
```

### Value Changed Fires on Connect

```javascript
countValueChanged(value, previousValue) {
    // On connect: previousValue is undefined
    if (previousValue === undefined) return;  // Skip initial call if needed
}
```

---

## Action Gotchas

### Forgetting Event Parameter

```javascript
// ❌ Can't access event
submit() {
    // event is undefined
}

// ✅ Correct
submit(event) {
    event.preventDefault();
}
```

### Action Parameters Are Always Strings

```html
<button data-item-id-param="123" data-item-active-param="true">
```

```javascript
delete(event) {
    event.params.id;      // "123" (string, not number)
    event.params.active;  // "true" (string, not boolean)
    
    // Convert manually if needed
    const id = parseInt(event.params.id, 10);
    const active = event.params.active === 'true';
}
```

### Default Events Don't Work Everywhere

```html
<!-- ❌ div has no default event -->
<div data-action="myController#click">

<!-- ✅ Specify event explicitly -->
<div data-action="click->myController#click">
```

---

## Turbo Compatibility

### Controller Connects Multiple Times

With Turbo Drive, controllers may connect/disconnect as you navigate:

```javascript
export default class extends Controller {
    connect() {
        // This may run multiple times during navigation
        // Don't assume it only runs once
        console.log('Connected');
    }
}
```

### Lazy Controllers + Turbo Issue

Known issue: lazy controllers may not load when navigating with Turbo if the controller wasn't on the first page visited.

Workarounds:
1. Use eager loading for critical controllers
2. Add `data-turbo-track="reload"` to force page reload
3. Ensure controller is referenced on initial page

### Morphing and Targets

When using Turbo morphing, targets may be replaced. Use `[target]TargetConnected` callbacks:

```javascript
static targets = ['item']

itemTargetConnected(element) {
    // Reinitialize any third-party libraries on this element
    this.initializeElement(element);
}
```

---

## Debugging Tips

### Enable Debug Mode

```javascript
// assets/bootstrap.js
import { Application } from '@hotwired/stimulus';

const app = Application.start();
app.debug = true;  // Logs lifecycle events to console
```

### Check Controller Registration

```javascript
// In browser console
Stimulus.debug = true;

// List registered controllers
Array.from(Stimulus.router.modules).map(m => m.identifier);
```

### Verify Controller is Connected

```javascript
connect() {
    console.log(`${this.identifier} connected to`, this.element);
}
```

### Check Targets Exist

```javascript
someMethod() {
    console.log('Has target?', this.hasOutputTarget);
    console.log('Targets:', this.outputTargets);
}
```

### Inspect Element Data

```javascript
connect() {
    console.log('Values:', {
        url: this.urlValue,
        count: this.countValue,
    });
    console.log('Element dataset:', this.element.dataset);
}
```

### Common Console Errors

| Error | Cause |
|-------|-------|
| `Cannot read property 'xyz' of undefined` | Target doesn't exist or is out of scope |
| `this.xyzTarget is not a function` | Typo in target name or not declared in `static targets` |
| `Controller not found: xyz` | File naming mismatch or controller not registered |

---

## Performance Tips

### Use Lazy Loading for Heavy Controllers

```javascript
/* stimulusFetch: 'lazy' */
import { Controller } from '@hotwired/stimulus';
import HeavyLibrary from 'heavy-library';  // Only loaded when needed
```

### Debounce Expensive Operations

```javascript
connect() {
    this.debouncedSearch = this.debounce(this.search.bind(this), 300);
}

debounce(fn, wait) {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn(...args), wait);
    };
}
```

### Use `disconnect()` to Clean Up

Always remove event listeners, clear intervals, abort fetches:

```javascript
connect() {
    this.abortController = new AbortController();
    fetch(url, { signal: this.abortController.signal });
}

disconnect() {
    this.abortController.abort();
}
```

### Avoid Querying DOM in Loops

```javascript
// ❌ Slow
items.forEach(item => {
    document.querySelector(`[data-id="${item.id}"]`).classList.add('active');
});

// ✅ Better - use targets
this.itemTargets.forEach(target => {
    target.classList.add('active');
});
```

---

## Third-Party Libraries

### Initializing Libraries on Connect

```javascript
import Choices from 'choices.js';

export default class extends Controller {
    connect() {
        this.choices = new Choices(this.element);
    }

    disconnect() {
        this.choices.destroy();  // Clean up!
    }
}
```

### Re-initializing After Turbo Navigation

```javascript
static targets = ['select']

selectTargetConnected(element) {
    // Called when target added to DOM (including after Turbo morphing)
    element._choices = new Choices(element);
}

selectTargetDisconnected(element) {
    element._choices?.destroy();
}
```
