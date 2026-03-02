# Turbo Patterns

Common patterns for Turbo Frames and Streams in Symfony applications.

---

## Inline Editing

Edit items in place without page navigation.

### Template (list view)

```twig
{# templates/task/index.html.twig #}
<ul id="tasks">
    {% for task in tasks %}
        <turbo-frame id="task_{{ task.id }}">
            {{ include('task/_task.html.twig') }}
        </turbo-frame>
    {% endfor %}
</ul>
```

```twig
{# templates/task/_task.html.twig #}
<li>
    <span>{{ task.title }}</span>
    <a href="{{ path('task_edit', {id: task.id}) }}">Edit</a>
</li>
```

### Edit Form

```twig
{# templates/task/edit.html.twig #}
<turbo-frame id="task_{{ task.id }}">
    <li>
        <form action="{{ path('task_update', {id: task.id}) }}" method="post">
            <input name="title" value="{{ task.title }}">
            <button type="submit">Save</button>
            <a href="{{ path('task_show', {id: task.id}) }}">Cancel</a>
        </form>
    </li>
</turbo-frame>
```

### Controller

```php
#[Route('/tasks/{id}/edit', name: 'task_edit')]
public function edit(Task $task): Response
{
    return $this->render('task/edit.html.twig', ['task' => $task]);
}

#[Route('/tasks/{id}', name: 'task_update', methods: ['POST'])]
public function update(Request $request, Task $task): Response
{
    $task->setTitle($request->request->get('title'));
    $this->em->flush();
    
    return $this->render('task/_task_frame.html.twig', ['task' => $task]);
}
```

---

## Search with Live Results

Search form with results updating as user types.

```twig
{# Single frame for form + results #}
<turbo-frame id="search">
    <form action="{{ path('search') }}" method="get" data-turbo-frame="search">
        <input type="search" name="q" value="{{ q }}"
               data-controller="autosubmit"
               data-action="input->autosubmit#submit">
    </form>
    
    <ul>
        {% for item in results %}
            <li>{{ item.name }}</li>
        {% endfor %}
    </ul>
</turbo-frame>
```

```javascript
// assets/controllers/autosubmit_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    submit() {
        clearTimeout(this.timeout);
        this.timeout = setTimeout(() => {
            this.element.form.requestSubmit();
        }, 300);
    }
}
```

---

## Tabs

Tab navigation with frame-based content loading.

```twig
<nav>
    <a href="{{ path('tab_one') }}" data-turbo-frame="tab-content">Tab 1</a>
    <a href="{{ path('tab_two') }}" data-turbo-frame="tab-content">Tab 2</a>
    <a href="{{ path('tab_three') }}" data-turbo-frame="tab-content">Tab 3</a>
</nav>

<turbo-frame id="tab-content" data-turbo-action="advance">
    {# Active tab content renders here #}
    {{ include('tabs/_tab_one.html.twig') }}
</turbo-frame>
```

Each tab route returns:

```twig
<turbo-frame id="tab-content">
    {# Tab specific content #}
</turbo-frame>
```

---

## Modal Dialog

Load modal content via frame.

```twig
{# Empty modal frame #}
<turbo-frame id="modal"></turbo-frame>

{# Link to open modal #}
<a href="{{ path('item_delete_confirm', {id: item.id}) }}" 
   data-turbo-frame="modal">
    Delete
</a>
```

```twig
{# templates/item/delete_confirm.html.twig #}
<turbo-frame id="modal">
    <dialog open data-controller="modal">
        <h2>Confirm Delete</h2>
        <p>Delete "{{ item.name }}"?</p>
        
        <form action="{{ path('item_delete', {id: item.id}) }}" method="post">
            <input type="hidden" name="_method" value="DELETE">
            <button type="submit">Delete</button>
            <button type="button" data-action="modal#close">Cancel</button>
        </form>
    </dialog>
</turbo-frame>
```

---

## Pagination

Paginate within a frame, optionally updating URL.

```twig
<turbo-frame id="items" data-turbo-action="advance">
    <ul>
        {% for item in items %}
            <li>{{ item.name }}</li>
        {% endfor %}
    </ul>
    
    <nav>
        {% if prev_page %}
            <a href="{{ path('items', {page: prev_page}) }}">Previous</a>
        {% endif %}
        {% if next_page %}
            <a href="{{ path('items', {page: next_page}) }}">Next</a>
        {% endif %}
    </nav>
</turbo-frame>
```

---

## Form with Multiple Updates (Streams)

Form submission that updates multiple page sections.

### Controller

```php
#[Route('/comments', name: 'comment_create', methods: ['POST'])]
public function create(Request $request): Response
{
    $comment = new Comment();
    $form = $this->createForm(CommentType::class, $comment);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        $this->em->persist($comment);
        $this->em->flush();
        
        $request->setRequestFormat(TurboBundle::STREAM_FORMAT);
        
        return $this->render('comment/create.stream.html.twig', [
            'comment' => $comment,
            'count' => $this->commentRepo->count([]),
        ]);
    }
    
    // Handle validation errors
    return $this->render('comment/_form.html.twig', [
        'form' => $form,
    ]);
}
```

### Stream Template

```twig
{# templates/comment/create.stream.html.twig #}

{# Add comment to list #}
<turbo-stream action="append" target="comments">
    <template>
        {{ include('comment/_comment.html.twig') }}
    </template>
</turbo-stream>

{# Update counter #}
<turbo-stream action="update" target="comment-count">
    <template>{{ count }}</template>
</turbo-stream>

{# Reset form #}
<turbo-stream action="replace" target="comment-form">
    <template>
        {{ include('comment/_form.html.twig', {comment: null}) }}
    </template>
</turbo-stream>

{# Show flash message #}
<turbo-stream action="prepend" target="flash-messages">
    <template>
        <div class="alert alert-success">Comment added!</div>
    </template>
</turbo-stream>
```

---

## Delete with Confirmation

Remove item with stream response.

```twig
<div id="item_{{ item.id }}">
    <span>{{ item.name }}</span>
    <form action="{{ path('item_delete', {id: item.id}) }}" 
          method="post"
          data-turbo-confirm="Delete this item?">
        <input type="hidden" name="_method" value="DELETE">
        <button type="submit">Delete</button>
    </form>
</div>
```

### Controller

```php
#[Route('/items/{id}', name: 'item_delete', methods: ['DELETE'])]
public function delete(Request $request, Item $item): Response
{
    $itemId = $item->getId();
    
    $this->em->remove($item);
    $this->em->flush();
    
    $request->setRequestFormat(TurboBundle::STREAM_FORMAT);
    
    return $this->render('item/delete.stream.html.twig', [
        'itemId' => $itemId,
    ]);
}
```

```twig
{# templates/item/delete.stream.html.twig #}
<turbo-stream action="remove" target="item_{{ itemId }}"></turbo-stream>
```

---

## Real-time Updates with Mercure

Broadcast changes to all connected clients.

### Entity Setup

```php
use Symfony\UX\Turbo\Attribute\Broadcast;

#[ORM\Entity]
#[Broadcast]
class Message
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;
    
    #[ORM\Column]
    private string $content;
    
    // ...
}
```

### Template with Stream Source

```twig
{# Subscribe to Mercure updates #}
<turbo-stream-source src="{{ mercure('Message')|escape('html_attr') }}">
</turbo-stream-source>

<div id="messages">
    {% for message in messages %}
        {{ include('message/_message.html.twig') }}
    {% endfor %}
</div>

{# New messages auto-appear for all users #}
```

### Broadcast Templates

```twig
{# templates/broadcast/Message.stream.html.twig #}
{# Rendered when entity is created/updated #}

{% block create %}
<turbo-stream action="append" target="messages">
    <template>{{ include('message/_message.html.twig') }}</template>
</turbo-stream>
{% endblock %}

{% block update %}
<turbo-stream action="replace" target="message_{{ message.id }}">
    <template>{{ include('message/_message.html.twig') }}</template>
</turbo-stream>
{% endblock %}

{% block remove %}
<turbo-stream action="remove" target="message_{{ message.id }}">
</turbo-stream>
{% endblock %}
```

---

## Lazy Loading Sections

Load heavy content after initial page load.

```twig
{# Main page loads fast #}
<h1>Dashboard</h1>

{# Stats load asynchronously #}
<turbo-frame id="stats" src="{{ path('dashboard_stats') }}" loading="lazy">
    <div class="skeleton">Loading stats...</div>
</turbo-frame>

{# Recent activity loads when scrolled into view #}
<turbo-frame id="activity" src="{{ path('dashboard_activity') }}" loading="lazy">
    <div class="skeleton">Loading activity...</div>
</turbo-frame>
```

---

## Infinite Scroll

Load more content as user scrolls.

```twig
<div id="items">
    {% for item in items %}
        {{ include('item/_item.html.twig') }}
    {% endfor %}
</div>

{% if has_more %}
    <turbo-frame id="load-more" 
                 src="{{ path('items', {page: next_page}) }}" 
                 loading="lazy">
        <p>Loading more...</p>
    </turbo-frame>
{% endif %}
```

Response appends items and includes next frame:

```twig
{# Rendered for page 2, 3, etc. #}
<turbo-frame id="load-more">
    {% for item in items %}
        {{ include('item/_item.html.twig') }}
    {% endfor %}
    
    {% if has_more %}
        <turbo-frame id="load-more" 
                     src="{{ path('items', {page: next_page}) }}" 
                     loading="lazy">
            <p>Loading more...</p>
        </turbo-frame>
    {% endif %}
</turbo-frame>
```

Combine with Stimulus to move items out of frame:

```javascript
// Move loaded items to main container
document.addEventListener('turbo:frame-load', (event) => {
    if (event.target.id === 'load-more') {
        const items = event.target.querySelectorAll('.item');
        document.getElementById('items').append(...items);
    }
});
```

---

## Form Validation Errors

Show validation errors without full page reload.

### Controller

```php
#[Route('/register', name: 'register', methods: ['GET', 'POST'])]
public function register(Request $request): Response
{
    $form = $this->createForm(RegistrationType::class);
    $form->handleRequest($request);
    
    if ($form->isSubmitted() && $form->isValid()) {
        // Success - redirect
        return $this->redirectToRoute('dashboard');
    }
    
    // GET or validation failed - render form
    // Frame request gets just the form
    return $this->render('auth/register.html.twig', [
        'form' => $form,
    ]);
}
```

### Template

```twig
{# Full page wraps form in frame #}
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Register</h1>
    
    <turbo-frame id="register-form">
        {{ include('auth/_register_form.html.twig') }}
    </turbo-frame>
{% endblock %}
```

```twig
{# Partial form template #}
<turbo-frame id="register-form">
    {{ form_start(form) }}
        {{ form_row(form.email) }}
        {{ form_row(form.password) }}
        <button type="submit">Register</button>
    {{ form_end(form) }}
</turbo-frame>
```

Validation errors re-render only the form frame.

---

## Toast Notifications

Show temporary notifications with streams.

### Base Layout

```twig
<div id="toasts" class="toast-container"></div>
```

### Stream Response (after any action)

```twig
<turbo-stream action="append" target="toasts">
    <template>
        <div class="toast" data-controller="toast" data-toast-duration-value="5000">
            {{ message }}
        </div>
    </template>
</turbo-stream>
```

### Auto-dismiss Controller

```javascript
// assets/controllers/toast_controller.js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
    static values = { duration: { type: Number, default: 3000 } };
    
    connect() {
        setTimeout(() => this.element.remove(), this.durationValue);
    }
}
```
