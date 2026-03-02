# Stimulus Patterns

Common controller patterns for real-world use cases.

---

## Debounced Search

```javascript
// search_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['input', 'results'];
    static values = { 
        url: String,
        delay: { type: Number, default: 300 }
    };

    connect() {
        this.timeout = null;
    }

    disconnect() {
        clearTimeout(this.timeout);
    }

    search() {
        clearTimeout(this.timeout);
        this.timeout = setTimeout(() => this.performSearch(), this.delayValue);
    }

    async performSearch() {
        const query = this.inputTarget.value;
        if (!query) {
            this.resultsTarget.innerHTML = '';
            return;
        }

        const url = `${this.urlValue}?q=${encodeURIComponent(query)}`;
        const response = await fetch(url);
        this.resultsTarget.innerHTML = await response.text();
    }
}
```

```html
<div data-controller="search" data-search-url-value="/api/search">
    <input type="text" 
           data-search-target="input" 
           data-action="input->search#search">
    <div data-search-target="results"></div>
</div>
```

---

## Loading State

```javascript
// loading_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['button', 'spinner'];
    static classes = ['loading'];

    start() {
        this.element.classList.add(this.loadingClass);
        this.buttonTarget.disabled = true;
        this.spinnerTarget.hidden = false;
    }

    stop() {
        this.element.classList.remove(this.loadingClass);
        this.buttonTarget.disabled = false;
        this.spinnerTarget.hidden = true;
    }
}
```

```html
<form data-controller="loading" data-loading-loading-class="opacity-50">
    <button data-loading-target="button">
        Submit
        <span data-loading-target="spinner" hidden>...</span>
    </button>
</form>
```

---

## Toggle / Disclosure

```javascript
// toggle_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['content'];
    static classes = ['open'];
    static values = { open: { type: Boolean, default: false } };

    toggle() {
        this.openValue = !this.openValue;
    }

    show() {
        this.openValue = true;
    }

    hide() {
        this.openValue = false;
    }

    openValueChanged() {
        this.contentTarget.hidden = !this.openValue;
        this.element.classList.toggle(this.openClass, this.openValue);
    }
}
```

```html
<div data-controller="toggle" data-toggle-open-class="is-open">
    <button data-action="click->toggle#toggle">Toggle</button>
    <div data-toggle-target="content" hidden>
        Content here
    </div>
</div>
```

---

## Modal / Dialog

```javascript
// modal_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['dialog'];

    open() {
        this.dialogTarget.showModal();
    }

    close() {
        this.dialogTarget.close();
    }

    closeOnBackdrop(event) {
        if (event.target === this.dialogTarget) {
            this.close();
        }
    }

    closeOnEscape(event) {
        if (event.key === 'Escape') {
            this.close();
        }
    }
}
```

```html
<div data-controller="modal">
    <button data-action="click->modal#open">Open Modal</button>
    
    <dialog data-modal-target="dialog"
            data-action="click->modal#closeOnBackdrop keydown.esc@window->modal#closeOnEscape">
        <h2>Modal Title</h2>
        <p>Modal content</p>
        <button data-action="click->modal#close">Close</button>
    </dialog>
</div>
```

---

## Dropdown

```javascript
// dropdown_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['menu'];
    static values = { open: { type: Boolean, default: false } };

    toggle() {
        this.openValue = !this.openValue;
    }

    close() {
        this.openValue = false;
    }

    closeOnClickOutside(event) {
        if (!this.element.contains(event.target)) {
            this.close();
        }
    }

    openValueChanged() {
        this.menuTarget.hidden = !this.openValue;
    }
}
```

```html
<div data-controller="dropdown" 
     data-action="click@document->dropdown#closeOnClickOutside">
    <button data-action="click->dropdown#toggle">Menu</button>
    <ul data-dropdown-target="menu" hidden>
        <li><a href="#">Option 1</a></li>
        <li><a href="#">Option 2</a></li>
    </ul>
</div>
```

---

## Tabs

```javascript
// tabs_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['tab', 'panel'];
    static classes = ['active'];
    static values = { index: { type: Number, default: 0 } };

    select(event) {
        this.indexValue = this.tabTargets.indexOf(event.currentTarget);
    }

    indexValueChanged() {
        this.tabTargets.forEach((tab, i) => {
            tab.classList.toggle(this.activeClass, i === this.indexValue);
            tab.setAttribute('aria-selected', i === this.indexValue);
        });
        this.panelTargets.forEach((panel, i) => {
            panel.hidden = i !== this.indexValue;
        });
    }
}
```

```html
<div data-controller="tabs" data-tabs-active-class="border-b-2 border-blue-500">
    <div role="tablist">
        <button data-tabs-target="tab" data-action="click->tabs#select" role="tab">Tab 1</button>
        <button data-tabs-target="tab" data-action="click->tabs#select" role="tab">Tab 2</button>
    </div>
    <div data-tabs-target="panel" role="tabpanel">Content 1</div>
    <div data-tabs-target="panel" role="tabpanel" hidden>Content 2</div>
</div>
```

---

## Form Validation

```javascript
// validation_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['submit'];

    validate(event) {
        const field = event.target;
        const error = field.nextElementSibling;
        
        if (field.validity.valid) {
            error.hidden = true;
            field.classList.remove('is-invalid');
        } else {
            error.textContent = field.validationMessage;
            error.hidden = false;
            field.classList.add('is-invalid');
        }
        
        this.updateSubmit();
    }

    updateSubmit() {
        const form = this.element;
        this.submitTarget.disabled = !form.checkValidity();
    }

    submit(event) {
        if (!this.element.checkValidity()) {
            event.preventDefault();
            this.element.reportValidity();
        }
    }
}
```

```html
<form data-controller="validation" data-action="submit->validation#submit:prevent">
    <input type="email" required data-action="blur->validation#validate">
    <span class="error" hidden></span>
    
    <input type="text" minlength="3" required data-action="blur->validation#validate">
    <span class="error" hidden></span>
    
    <button data-validation-target="submit" disabled>Submit</button>
</form>
```

---

## Clipboard Copy

```javascript
// clipboard_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['source', 'button'];
    static values = { successText: { type: String, default: 'Copied!' } };

    async copy() {
        const text = this.sourceTarget.value || this.sourceTarget.textContent;
        await navigator.clipboard.writeText(text);
        
        const originalText = this.buttonTarget.textContent;
        this.buttonTarget.textContent = this.successTextValue;
        
        setTimeout(() => {
            this.buttonTarget.textContent = originalText;
        }, 2000);
    }
}
```

```html
<div data-controller="clipboard">
    <input data-clipboard-target="source" value="Copy this text" readonly>
    <button data-clipboard-target="button" data-action="click->clipboard#copy">
        Copy
    </button>
</div>
```

---

## Auto-Submit Form

```javascript
// auto_submit_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static values = { delay: { type: Number, default: 500 } };

    connect() {
        this.timeout = null;
    }

    disconnect() {
        clearTimeout(this.timeout);
    }

    submit() {
        clearTimeout(this.timeout);
        this.timeout = setTimeout(() => {
            this.element.requestSubmit();
        }, this.delayValue);
    }
}
```

```html
<form data-controller="auto-submit" data-turbo-frame="results">
    <input type="search" name="q" data-action="input->auto-submit#submit">
    <select name="category" data-action="change->auto-submit#submit">
        <option>All</option>
        <option>Books</option>
    </select>
</form>
```

---

## Character Counter

```javascript
// counter_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['input', 'count'];
    static values = { max: Number };
    static classes = ['over'];

    connect() {
        this.update();
    }

    update() {
        const count = this.inputTarget.value.length;
        const remaining = this.maxValue - count;
        
        this.countTarget.textContent = `${count}/${this.maxValue}`;
        this.countTarget.classList.toggle(this.overClass, remaining < 0);
    }
}
```

```html
<div data-controller="counter" 
     data-counter-max-value="280"
     data-counter-over-class="text-red-500">
    <textarea data-counter-target="input" 
              data-action="input->counter#update"></textarea>
    <span data-counter-target="count"></span>
</div>
```

---

## Infinite Scroll

```javascript
// infinite_scroll_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['sentinel', 'container'];
    static values = { url: String, page: { type: Number, default: 1 } };

    connect() {
        this.observer = new IntersectionObserver(entries => {
            entries.forEach(entry => {
                if (entry.isIntersecting) this.loadMore();
            });
        });
        this.observer.observe(this.sentinelTarget);
    }

    disconnect() {
        this.observer.disconnect();
    }

    async loadMore() {
        this.pageValue++;
        const url = `${this.urlValue}?page=${this.pageValue}`;
        const response = await fetch(url);
        const html = await response.text();
        
        this.sentinelTarget.insertAdjacentHTML('beforebegin', html);
    }
}
```

```html
<div data-controller="infinite-scroll" 
     data-infinite-scroll-url-value="/api/items">
    <div data-infinite-scroll-target="container">
        <!-- Items here -->
    </div>
    <div data-infinite-scroll-target="sentinel"></div>
</div>
```

---

## Sortable List

```javascript
// sortable_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static targets = ['item'];

    dragstart(event) {
        event.dataTransfer.effectAllowed = 'move';
        event.dataTransfer.setData('text/plain', event.target.dataset.id);
        event.target.classList.add('dragging');
    }

    dragend(event) {
        event.target.classList.remove('dragging');
    }

    dragover(event) {
        event.preventDefault();
        const dragging = this.element.querySelector('.dragging');
        const target = event.target.closest('[data-sortable-target="item"]');
        
        if (target && target !== dragging) {
            const rect = target.getBoundingClientRect();
            const after = event.clientY > rect.top + rect.height / 2;
            target.parentNode.insertBefore(dragging, after ? target.nextSibling : target);
        }
    }
}
```

```html
<ul data-controller="sortable" data-action="dragover->sortable#dragover">
    <li data-sortable-target="item" 
        data-id="1" 
        draggable="true"
        data-action="dragstart->sortable#dragstart dragend->sortable#dragend">
        Item 1
    </li>
    <!-- more items -->
</ul>
```
