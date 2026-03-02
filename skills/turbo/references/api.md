# Turbo API Reference

## Table of Contents

- [Turbo Drive](#turbo-drive)
- [Turbo Frames](#turbo-frames)
- [Turbo Streams](#turbo-streams)
- [Data Attributes](#data-attributes)
- [Meta Tags](#meta-tags)
- [Events](#events)
- [JavaScript API](#javascript-api)

---

## Turbo Drive

Intercepts link clicks and form submissions, replacing full page loads with AJAX requests.

### How It Works

1. User clicks link or submits form
2. Turbo makes fetch request
3. Response `<body>` replaces current `<body>`
4. `<head>` is merged (scripts/styles preserved)
5. Browser history updated

### Visit Types

| Type | Trigger | Cache | History |
|------|---------|-------|---------|
| Application | Link click, `Turbo.visit()` | Preview from cache | Push |
| Restoration | Back/Forward button | Restore from cache | Replace |

### Prefetching

Enabled by default (Turbo 8+). Links prefetch on hover.

```html
<!-- Disable prefetching globally -->
<meta name="turbo-prefetch" content="false">

<!-- Disable for specific link -->
<a href="/heavy-page" data-turbo-prefetch="false">Heavy Page</a>
```

### Preloading

```html
<!-- Preload into cache on page load -->
<a href="/dashboard" data-turbo-preload>Dashboard</a>
```

---

## Turbo Frames

### Element Attributes

| Attribute | Description |
|-----------|-------------|
| `id` | Required. Unique identifier |
| `src` | URL to load content from |
| `loading` | `eager` (default) or `lazy` |
| `disabled` | Disables frame navigation |
| `target` | Default frame target for links inside |
| `autoscroll` | Scroll frame into view after load |

### Basic Structure

```html
<turbo-frame id="[unique-id]" 
             src="[optional-url]"
             loading="[eager|lazy]"
             disabled
             target="[frame-id]"
             autoscroll>
    <!-- Content -->
</turbo-frame>
```

### Frame Loading

```html
<!-- Eager (default): loads immediately if src present -->
<turbo-frame id="sidebar" src="/sidebar"></turbo-frame>

<!-- Lazy: loads when frame enters viewport -->
<turbo-frame id="comments" src="/comments" loading="lazy">
    <p>Loading comments...</p>
</turbo-frame>
```

### Frame Targeting

```html
<!-- Link targets specific frame -->
<a href="/page" data-turbo-frame="content">Load in content frame</a>

<!-- Link breaks out to full page -->
<a href="/page" data-turbo-frame="_top">Full page navigation</a>

<!-- Frame sets default target for all links inside -->
<turbo-frame id="nav" target="main">
    <a href="/page1">Goes to main frame</a>
    <a href="/page2">Also goes to main frame</a>
</turbo-frame>
```

### Frame with URL Updates

```html
<!-- Advance: push to history -->
<turbo-frame id="products" data-turbo-action="advance">

<!-- Replace: replace current history entry -->
<turbo-frame id="filters" data-turbo-action="replace">
```

### Server Response Requirements

Response must contain a `<turbo-frame>` with matching `id`:

```html
<!-- Request to /items/1 from frame id="item_1" -->

<!-- Valid response -->
<turbo-frame id="item_1">
    <h2>Item 1</h2>
</turbo-frame>

<!-- Invalid: frame not found, shows error -->
<div>No matching frame!</div>
```

---

## Turbo Streams

### Actions

| Action | Description |
|--------|-------------|
| `append` | Add content at end of target |
| `prepend` | Add content at start of target |
| `replace` | Replace entire target element |
| `update` | Replace target's innerHTML |
| `remove` | Remove target element |
| `before` | Insert content before target |
| `after` | Insert content after target |
| `morph` | Morph target to new content (smooth) |
| `refresh` | Trigger page refresh |

### Syntax

```html
<!-- Single target (by ID) -->
<turbo-stream action="[action]" target="[element-id]">
    <template>
        <!-- New content here -->
    </template>
</turbo-stream>

<!-- Multiple targets (by CSS selector) -->
<turbo-stream action="[action]" targets="[css-selector]">
    <template>
        <!-- Content applied to all matches -->
    </template>
</turbo-stream>
```

### Action Examples

```html
<!-- Append new item to list -->
<turbo-stream action="append" target="items">
    <template>
        <li id="item_42">New Item</li>
    </template>
</turbo-stream>

<!-- Update counter text -->
<turbo-stream action="update" target="item-count">
    <template>42 items</template>
</turbo-stream>

<!-- Remove deleted item -->
<turbo-stream action="remove" target="item_5"></turbo-stream>

<!-- Replace with updated version -->
<turbo-stream action="replace" target="user_card">
    <template>
        <div id="user_card">
            <h3>Updated User</h3>
        </div>
    </template>
</turbo-stream>

<!-- Morph for smooth transitions -->
<turbo-stream action="morph" target="dashboard">
    <template>
        <div id="dashboard">Updated stats...</div>
    </template>
</turbo-stream>
```

### Delivery Methods

**1. HTTP Response (form submission)**

```php
// Controller returns stream response
$request->setRequestFormat(TurboBundle::STREAM_FORMAT);
return $this->render('item/create.stream.html.twig');
```

**2. WebSocket / SSE (real-time)**

```html
<turbo-stream-source src="wss://example.com/updates">
</turbo-stream-source>

<!-- Or with Mercure -->
<turbo-stream-source src="{{ mercure('topic')|escape('html_attr') }}">
</turbo-stream-source>
```

**3. Inline in Page**

```html
<!-- Streams in page are processed and removed -->
<turbo-stream action="remove" target="old-banner"></turbo-stream>
```

### Content-Type

Stream responses must have:
```
Content-Type: text/vnd.turbo-stream.html
```

Symfony sets this automatically with `TurboBundle::STREAM_FORMAT`.

---

## Data Attributes

### On Links and Forms

| Attribute | Values | Description |
|-----------|--------|-------------|
| `data-turbo` | `true`, `false` | Enable/disable Turbo |
| `data-turbo-frame` | frame id, `_top` | Target frame for navigation |
| `data-turbo-action` | `advance`, `replace` | History behavior |
| `data-turbo-method` | HTTP method | Change link method (use forms instead) |
| `data-turbo-stream` | (boolean) | Accept stream response |
| `data-turbo-confirm` | message | Show confirm dialog |
| `data-turbo-prefetch` | `true`, `false` | Control prefetching |
| `data-turbo-preload` | (boolean) | Preload into cache |

### On Any Element

| Attribute | Values | Description |
|-----------|--------|-------------|
| `data-turbo-permanent` | (boolean) | Persist element across navigations |
| `data-turbo-temporary` | (boolean) | Remove before caching |
| `data-turbo-cache` | `false` | Exclude from cache |

### Examples

```html
<!-- Disable Turbo -->
<a href="/external" data-turbo="false">External</a>

<!-- Custom confirm dialog -->
<a href="/delete" data-turbo-method="delete" 
   data-turbo-confirm="Are you sure?">Delete</a>

<!-- Persist element (e.g., audio player) -->
<div id="player" data-turbo-permanent>
    <audio src="..."></audio>
</div>

<!-- Don't cache this element -->
<div data-turbo-temporary>
    <p>Session-specific content</p>
</div>
```

---

## Meta Tags

Place in `<head>` to configure Turbo behavior.

```html
<!-- Force full page reload -->
<meta name="turbo-visit-control" content="reload">

<!-- Disable caching -->
<meta name="turbo-cache-control" content="no-cache">

<!-- No preview from cache -->
<meta name="turbo-cache-control" content="no-preview">

<!-- Scope Turbo to path -->
<meta name="turbo-root" content="/app">

<!-- Enable View Transitions API -->
<meta name="view-transition" content="same-origin">

<!-- Page refresh with morphing -->
<meta name="turbo-refresh-method" content="morph">

<!-- Preserve scroll on refresh -->
<meta name="turbo-refresh-scroll" content="preserve">
```

---

## Events

### Drive Events

| Event | When | Target |
|-------|------|--------|
| `turbo:click` | Link clicked | Link element |
| `turbo:before-visit` | Before visit starts | document |
| `turbo:visit` | Visit started | document |
| `turbo:before-cache` | Before page cached | document |
| `turbo:before-render` | Before rendering | document |
| `turbo:render` | After rendering | document |
| `turbo:load` | Page fully loaded | document |

### Frame Events

| Event | When | Target |
|-------|------|--------|
| `turbo:before-frame-render` | Before frame renders | turbo-frame |
| `turbo:frame-render` | After frame renders | turbo-frame |
| `turbo:frame-load` | Frame fully loaded | turbo-frame |
| `turbo:frame-missing` | Response missing frame | turbo-frame |

### Stream Events

| Event | When | Target |
|-------|------|--------|
| `turbo:before-stream-render` | Before stream action | turbo-stream |

### Form Events

| Event | When | Target |
|-------|------|--------|
| `turbo:submit-start` | Form submission started | form |
| `turbo:submit-end` | Form submission complete | form |

### Event Examples

```javascript
// Prevent navigation
document.addEventListener('turbo:before-visit', (event) => {
    if (!confirm('Leave page?')) {
        event.preventDefault();
    }
});

// Cleanup before caching
document.addEventListener('turbo:before-cache', () => {
    document.querySelectorAll('.modal').forEach(m => m.remove());
});

// Initialize after render
document.addEventListener('turbo:load', () => {
    initializeComponents();
});

// Handle missing frame
document.addEventListener('turbo:frame-missing', (event) => {
    event.preventDefault();
    event.detail.visit(event.detail.response);  // Do full page visit
});

// Modify stream before render
document.addEventListener('turbo:before-stream-render', (event) => {
    // Access the stream element
    const stream = event.target;
    // Modify or prevent
});
```

---

## JavaScript API

### Turbo.visit()

```javascript
// Basic visit
Turbo.visit('/dashboard');

// With options
Turbo.visit('/page', { 
    action: 'replace',    // 'advance' (default) or 'replace'
    frame: 'main-content' // Target frame
});
```

### Turbo.cache

```javascript
// Clear cache
Turbo.cache.clear();
```

### Turbo.session

```javascript
// Disable Drive
Turbo.session.drive = false;

// Check if Drive is enabled
if (Turbo.session.drive) { /* ... */ }
```

### Turbo.navigator

```javascript
// Get current location
Turbo.navigator.location;

// Programmatic form submission
Turbo.navigator.submitForm(formElement);
```

### Turbo.renderStreamMessage()

```javascript
// Manually process stream HTML
const streamHTML = `
    <turbo-stream action="append" target="messages">
        <template><div>New message</div></template>
    </turbo-stream>
`;
Turbo.renderStreamMessage(streamHTML);
```

### Frame Methods

```javascript
const frame = document.querySelector('turbo-frame#products');

// Reload frame
frame.reload();

// Check loading state
frame.loading; // 'eager' or 'lazy'

// Get/set source
frame.src = '/new-source';

// Disable/enable
frame.disabled = true;
```
