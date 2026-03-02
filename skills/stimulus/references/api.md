# Stimulus API Reference

## Table of Contents

- [Lifecycle Callbacks](#lifecycle-callbacks)
- [Targets](#targets)
- [Values](#values)
- [Actions](#actions)
- [CSS Classes](#css-classes)
- [Outlets](#outlets)
- [Controller Properties](#controller-properties)

---

## Lifecycle Callbacks

```javascript
export default class extends Controller {
    initialize() {
        // Called once when controller is first instantiated
        // Use for one-time setup that doesn't require DOM
    }

    connect() {
        // Called each time controller connects to DOM
        // Element is available, targets are accessible
        // May be called multiple times (Turbo navigation)
    }

    disconnect() {
        // Called each time controller disconnects from DOM
        // Clean up: remove listeners, cancel timers, abort fetches
    }
}
```

Connection happens when:
- Element with `data-controller` is in document
- Controller identifier matches

Disconnection happens when:
- Element removed from DOM
- `data-controller` attribute removed/changed
- Turbo replaces page body

---

## Targets

### Definition

```javascript
static targets = ['query', 'results', 'errorMessage']
```

### Generated Properties

| Target name | Property | Returns |
|-------------|----------|---------|
| `item` | `this.itemTarget` | First element (throws if none) |
| `item` | `this.itemTargets` | Array of all elements |
| `item` | `this.hasItemTarget` | Boolean |

### HTML

```html
<div data-controller="search">
    <input data-search-target="query">
    <div data-search-target="results"></div>
    <span data-search-target="errorMessage"></span>
</div>
```

### Target Callbacks

```javascript
static targets = ['item']

itemTargetConnected(element) {
    // Called when a target element is added to DOM
    // Fires before connect() on initial load
}

itemTargetDisconnected(element) {
    // Called when a target element is removed from DOM
    // Fires before disconnect()
}
```

### Shared Targets

Element can be target for multiple controllers:

```html
<input data-search-target="input" data-validation-target="field">
```

### Naming

Use camelCase in JS, kebab-case in HTML:

```javascript
static targets = ['errorMessage']
```
```html
<span data-controller-target="errorMessage">  <!-- camelCase preserved -->
```

---

## Values

### Definition

```javascript
static values = {
    url: String,                              // required, no default
    count: Number,
    enabled: Boolean,
    items: Array,
    config: Object,
    delay: { type: Number, default: 300 },    // with default
    query: { type: String, default: '' }
}
```

### Types and Defaults

| Type | Empty default | HTML encoding |
|------|---------------|---------------|
| `String` | `""` | As-is |
| `Number` | `0` | Numeric string |
| `Boolean` | `false` | `"true"` or `"false"` |
| `Array` | `[]` | JSON |
| `Object` | `{}` | JSON |

### Generated Properties

| Value name | Property | Description |
|------------|----------|-------------|
| `url` | `this.urlValue` | Get/set value |
| `url` | `this.hasUrlValue` | Boolean, true if attribute exists |

### HTML

```html
<div data-controller="loader"
     data-loader-url-value="/api/data"
     data-loader-count-value="5"
     data-loader-enabled-value="true"
     data-loader-items-value='["a","b","c"]'
     data-loader-config-value='{"timeout":30}'>
</div>
```

### Value Change Callbacks

```javascript
static values = { count: Number }

countValueChanged(value, previousValue) {
    // Called when data-*-count-value changes
    // Also called on connect() with previousValue = undefined
    console.log(`Changed from ${previousValue} to ${value}`);
}
```

### Setter Behavior

```javascript
this.countValue = 10;        // Updates DOM attribute
this.countValue = undefined; // Removes attribute, reverts to default
```

---

## Actions

### Syntax

```
event->controller#method
event->controller#method:option
```

### HTML Examples

```html
<!-- Explicit event -->
<button data-action="click->dialog#open">Open</button>

<!-- Default event (click for buttons, submit for forms, input for inputs) -->
<button data-action="dialog#open">Open</button>

<!-- Multiple actions -->
<button data-action="click->a#method click->b#method">

<!-- With options -->
<form data-action="submit->form#save:prevent">
<a data-action="click->nav#go:prevent:stop">
```

### Default Events

| Element | Default event |
|---------|---------------|
| `<button>` | `click` |
| `<input type="submit">` | `click` |
| `<input>` | `input` |
| `<textarea>` | `input` |
| `<select>` | `change` |
| `<form>` | `submit` |
| `<details>` | `toggle` |
| Other | `click` |

### Key Filters

```html
<input data-action="keydown.enter->form#submit">
<input data-action="keydown.esc->form#cancel">
<input data-action="keydown.ctrl+s->form#save">
<input data-action="keydown.shift+tab->nav#previous">
```

Available keys: `enter`, `tab`, `esc`, `space`, `up`, `down`, `left`, `right`, `home`, `end`, `page_up`, `page_down`

Modifiers: `ctrl`, `alt`, `shift`, `meta`

Combine with `+`: `ctrl+s`, `ctrl+shift+z`

### Global Events

```html
<!-- Window events -->
<div data-action="resize@window->gallery#layout">
<div data-action="scroll@window->nav#highlight">

<!-- Document events -->
<div data-action="click@document->dropdown#closeOutside">
<div data-action="turbo:load@document->app#init">
```

### Action Options

| Option | Effect |
|--------|--------|
| `:prevent` | `event.preventDefault()` |
| `:stop` | `event.stopPropagation()` |
| `:self` | Only if `event.target === element` |
| `:passive` | Passive listener (better scroll perf) |
| `:capture` | Capture phase |
| `:once` | Remove after first invocation |

### Action Parameters

Pass data via `data-*-param`:

```html
<button data-action="click->item#delete"
        data-item-id-param="123"
        data-item-confirm-param="true">
    Delete
</button>
```

```javascript
delete(event) {
    const id = event.params.id;           // "123" (string)
    const confirm = event.params.confirm; // "true" (string)
}
```

### Event Object

```javascript
handleClick(event) {
    event.target;         // Element that triggered
    event.currentTarget;  // Element with data-action
    event.params;         // Action parameters
    event.preventDefault();
    event.stopPropagation();
}
```

---

## CSS Classes

### Definition

```javascript
static classes = ['loading', 'active', 'hidden']
```

### Generated Properties

| Class name | Property | Returns |
|------------|----------|---------|
| `loading` | `this.loadingClass` | First class string |
| `loading` | `this.loadingClasses` | Array of all classes |
| `loading` | `this.hasLoadingClass` | Boolean |

### HTML

```html
<div data-controller="toggle"
     data-toggle-loading-class="opacity-50 cursor-wait"
     data-toggle-active-class="bg-blue-500 text-white">
</div>
```

### Usage

```javascript
static classes = ['loading']

async submit() {
    this.element.classList.add(this.loadingClass);
    // or for multiple classes:
    this.element.classList.add(...this.loadingClasses);
    
    await fetch(...);
    
    this.element.classList.remove(this.loadingClass);
}
```

---

## Outlets

### Definition

```javascript
static outlets = ['user-status', 'notification']
```

### Generated Properties

| Outlet name | Property | Returns |
|-------------|----------|---------|
| `user-status` | `this.userStatusOutlet` | First controller instance |
| `user-status` | `this.userStatusOutlets` | Array of all instances |
| `user-status` | `this.hasUserStatusOutlet` | Boolean |
| `user-status` | `this.userStatusOutletElement` | First element |
| `user-status` | `this.userStatusOutletElements` | Array of elements |

### HTML

```html
<div data-controller="chat"
     data-chat-user-status-outlet=".online-user"
     data-chat-notification-outlet="#notifications">
</div>

<div class="online-user" data-controller="user-status">...</div>
<div class="online-user" data-controller="user-status">...</div>
<div id="notifications" data-controller="notification">...</div>
```

### Outlet Callbacks

```javascript
static outlets = ['user-status']

userStatusOutletConnected(outlet, element) {
    // outlet = controller instance
    // element = DOM element
}

userStatusOutletDisconnected(outlet, element) {
    // Called when outlet element removed
}
```

### Cross-Controller Communication

```javascript
// chat_controller.js
static outlets = ['user-status']

broadcastMessage(message) {
    this.userStatusOutlets.forEach(userCtrl => {
        userCtrl.showNotification(message);
    });
}

// user_status_controller.js
showNotification(message) {
    // Called from chat controller
}
```

### Namespaced Outlets

For controllers like `admin--user-status`:

```javascript
static outlets = ['admin--user-status']

// Access (note: no double underscore)
this.adminUserStatusOutlet
this.adminUserStatusOutlets
```

---

## Controller Properties

### Built-in Properties

```javascript
this.element           // The controller's root element
this.application       // The Stimulus Application instance
this.identifier        // Controller name (e.g., "search")
this.scope             // Scope object with schema and element info
```

### Data API

```javascript
// Read/write data attributes on controller element
this.data.get('index')          // data-controller-index
this.data.set('index', '5')
this.data.has('index')
this.data.delete('index')
```

Note: Prefer Values API over Data API for typed access.

### Dispatch Custom Events

```javascript
// Dispatch from controller element
this.dispatch('success', { detail: { id: 123 } });

// Listen in HTML
<div data-action="search:success->results#refresh">
```

Options:
```javascript
this.dispatch('save', {
    target: otherElement,      // default: this.element
    detail: { data: '...' },   // event.detail
    prefix: 'custom',          // event name: custom:save
    bubbles: true,             // default: true
    cancelable: true           // default: true
});
```
