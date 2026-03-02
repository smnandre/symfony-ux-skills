# LiveComponent API Reference

## AsLiveComponent Attribute

```php
use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;

#[AsLiveComponent(
    name: 'ProductSearch',                      // Optional, defaults to class name
    template: 'components/Search.html.twig',   // Optional
    method: 'get',                             // HTTP method: 'get' or 'post'
    route: 'ux_live_component',                // Custom route
)]
final class ProductSearch
{
    use DefaultActionTrait;  // Required!
}
```

---

## LiveProp

### Options

```php
use Symfony\UX\LiveComponent\Attribute\LiveProp;

#[LiveProp(
    writable: true,                    // Allow frontend modification
    hydrateWith: 'hydrateMethod',      // Custom hydration method
    dehydrateWith: 'dehydrateMethod',  // Custom dehydration method
    url: true,                         // Sync to URL query parameter
    modifier: 'modifyMethod',          // Runtime modification
    onUpdated: 'onPropUpdated',        // Callback when value changes
)]
public string $search = '';
```

### Writable Props

```php
// Simple writable
#[LiveProp(writable: true)]
public string $query = '';

// Writable with allowed paths (for objects/arrays)
#[LiveProp(writable: ['name', 'email', 'address.city'])]
public User $user;

// All paths writable
#[LiveProp(writable: true)]
public array $filters = [];
```

### URL Mapping

```php
use Symfony\UX\LiveComponent\Metadata\UrlMapping;

// Basic URL sync
#[LiveProp(writable: true, url: true)]
public string $query = '';
// ?query=value

// Custom parameter name
#[LiveProp(writable: true, url: new UrlMapping(as: 'q'))]
public string $query = '';
// ?q=value

// With alias for history state
#[LiveProp(writable: true, url: new UrlMapping(as: 'q', history: 'push'))]
public string $query = '';
```

### onUpdated Callback

```php
#[LiveProp(writable: true, onUpdated: 'onQueryChanged')]
public string $query = '';

public function onQueryChanged(string $previousValue): void
{
    // Called after $query changes
    $this->page = 1;  // Reset pagination
}
```

### Hydration

```php
// Entity: auto-hydrates from ID
#[LiveProp]
public ?Product $product = null;

// Custom hydration
#[LiveProp(
    hydrateWith: 'hydrateDate',
    dehydrateWith: 'dehydrateDate'
)]
public \DateTimeInterface $date;

public function hydrateDate(string $value): \DateTimeInterface
{
    return new \DateTime($value);
}

public function dehydrateDate(\DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

---

## LiveAction

```php
use Symfony\UX\LiveComponent\Attribute\LiveAction;

#[LiveAction]
public function save(): void
{
    // Simple action
}

#[LiveAction]
public function delete(): Response
{
    // Can return Response for redirect
    return $this->redirectToRoute('app_list');
}
```

### LiveArg

```php
use Symfony\UX\LiveComponent\Attribute\LiveArg;

#[LiveAction]
public function updateItem(
    #[LiveArg] int $id,
    #[LiveArg('qty')] int $quantity,  // Custom param name
): void {
    // ...
}
```

Template:
```twig
<button 
    data-action="live#action"
    data-live-action-param="updateItem"
    data-live-id-param="{{ item.id }}"
    data-live-qty-param="5"
>
```

---

## Traits

### DefaultActionTrait (Required)

```php
use Symfony\UX\LiveComponent\DefaultActionTrait;

#[AsLiveComponent]
final class MyComponent
{
    use DefaultActionTrait;
}
```

Provides the `__invoke()` method for re-rendering.

### ComponentToolsTrait

```php
use Symfony\UX\LiveComponent\ComponentToolsTrait;

#[AsLiveComponent]
final class MyComponent
{
    use DefaultActionTrait;
    use ComponentToolsTrait;
    
    #[LiveAction]
    public function doSomething(): void
    {
        // Emit event to other components
        $this->emit('eventName', ['data' => 'value']);
        
        // Emit only to parent
        $this->emitUp('eventName');
        
        // Emit only to named component
        $this->emitTo('OtherComponent', 'eventName');
        
        // Dispatch browser event
        $this->dispatchBrowserEvent('custom:event', ['detail' => 'data']);
    }
}
```

### ComponentWithFormTrait

```php
use Symfony\UX\LiveComponent\ComponentWithFormTrait;
use Symfony\Component\Form\FormInterface;

#[AsLiveComponent]
final class MyForm extends AbstractController
{
    use DefaultActionTrait;
    use ComponentWithFormTrait;
    
    #[LiveProp]
    public ?Entity $initialFormData = null;
    
    protected function instantiateForm(): FormInterface
    {
        return $this->createForm(EntityType::class, $this->initialFormData);
    }
    
    #[LiveAction]
    public function save(): void
    {
        $this->submitForm();  // Validates and submits
        
        if (!$this->getForm()->isValid()) {
            return;  // Re-renders with errors
        }
        
        $entity = $this->getForm()->getData();
        // ...
    }
}
```

### ValidatableComponentTrait

```php
use Symfony\UX\LiveComponent\ValidatableComponentTrait;
use Symfony\Component\Validator\Constraints as Assert;

#[AsLiveComponent]
final class ContactForm
{
    use DefaultActionTrait;
    use ValidatableComponentTrait;
    
    #[LiveProp(writable: true)]
    #[Assert\NotBlank]
    #[Assert\Email]
    public string $email = '';
    
    #[LiveAction]
    public function submit(): void
    {
        $this->validate();  // Throws if invalid
        // Validation passed...
    }
}
```

### LiveCollectionTrait

```php
use Symfony\UX\LiveComponent\LiveCollectionTrait;

#[AsLiveComponent]
final class OrderForm extends AbstractController
{
    use DefaultActionTrait;
    use ComponentWithFormTrait;
    use LiveCollectionTrait;  // Enables add/remove for collections
}
```

---

## Data Attributes

### data-model

```twig
{# Basic binding #}
<input data-model="fieldName">

{# Modifiers #}
<input data-model="on(change)|fieldName">     {# Update on change event #}
<input data-model="on(blur)|fieldName">       {# Update on blur #}
<input data-model="norender|fieldName">       {# Update without re-render #}
<input data-model="debounce(500)|fieldName">  {# Debounce in ms #}

{# Validation modifiers #}
<input data-model="validatedFields" name="email">  {# Validates on change #}

{# Nested properties #}
<input data-model="user.email">
<input data-model="filters.category">
```

### data-action

```twig
{# Call LiveAction #}
<button data-action="live#action" data-live-action-param="save">

{# Prevent default #}
<form data-action="live#action:prevent" data-live-action-param="submit">

{# With event #}
<input data-action="keydown.enter->live#action" data-live-action-param="search">

{# Re-render only #}
<button data-action="live#$render">Refresh</button>
```

### data-loading

```twig
{# Classes #}
<div data-loading="addClass(loading)">
<div data-loading="removeClass(visible)">

{# Visibility #}
<span data-loading="show">Loading...</span>
<span data-loading="hide">Content</span>

{# Attributes #}
<button data-loading="attr(disabled)">
<input data-loading="attr(readonly)">

{# Scoped to action #}
<span data-loading="action(save)|show">Saving...</span>

{# Scoped to model #}
<span data-loading="model(search)|show">Searching...</span>

{# Delay #}
<span data-loading="delay(200)|show">

{# Combine #}
<button data-loading="action(save)|addClass(opacity-50) attr(disabled)">
```

### data-poll

```twig
{# Poll every 2s (default) #}
<div {{ attributes }} data-poll>

{# Custom delay #}
<div {{ attributes }} data-poll="delay(5000)">

{# Call action #}
<div {{ attributes }} data-poll="action(refresh)">
```

### data-live-ignore

```twig
{# Don't morph this element #}
<div data-live-ignore>
    <third-party-widget></third-party-widget>
</div>

{# Ignore children only #}
<div data-live-ignore="children">
```

### data-live-id

```twig
{# Stable identity for morphing #}
{% for item in items %}
    <div data-live-id="{{ item.id }}">{{ item.name }}</div>
{% endfor %}
```

---

## Events

### Component Events

```php
// Emit to all listening components
$this->emit('product:saved', ['id' => $id]);

// Emit only to parent
$this->emitUp('item:updated');

// Emit to specific component
$this->emitTo('Cart', 'item:added', ['productId' => $id]);

// Self emit (triggers own listener)
$this->emitSelf('refresh');
```

### Listening

```php
use Symfony\UX\LiveComponent\Attribute\LiveListener;

#[LiveListener('product:saved')]
public function onProductSaved(#[LiveArg] int $id): void
{
    // Re-renders component
}

#[LiveListener('cart:updated')]
public function refreshCart(): void
{
    // Called when event dispatched
}
```

### Browser Events

```php
// Dispatch to browser
$this->dispatchBrowserEvent('modal:close');
$this->dispatchBrowserEvent('toast:show', ['message' => 'Saved!']);
```

```twig
{# Listen in Stimulus #}
<div data-action="modal:close@window->modal#close">
```

---

## Hooks

### PreReRender

```php
use Symfony\UX\LiveComponent\Attribute\PreReRender;

#[PreReRender]
public function beforeRender(): void
{
    // Called before each re-render (not initial)
}
```

### PostHydrate

```php
use Symfony\UX\LiveComponent\Attribute\PostHydrate;

#[PostHydrate]
public function afterHydrate(): void
{
    // Called after props hydrated from request
}
```

---

## Testing

```php
use Symfony\UX\LiveComponent\Test\InteractsWithLiveComponents;

class MyComponentTest extends KernelTestCase
{
    use InteractsWithLiveComponents;
    
    public function testSearch(): void
    {
        $component = $this->createLiveComponent(
            name: 'ProductSearch',
            data: ['query' => '']
        );
        
        // Set prop
        $component->set('query', 'laptop');
        
        // Call action
        $component->call('search');
        
        // Assert render
        $this->assertStringContainsString('laptop', $component->render());
        
        // Assert prop
        $this->assertSame('laptop', $component->get('query'));
    }
}
```

---

## Lazy & Defer Loading

```twig
{# Load immediately after page (defer) #}
<twig:HeavyComponent defer />

{# Load when visible (lazy) #}
<twig:HeavyComponent lazy />

{# With placeholder #}
<twig:HeavyComponent lazy>
    <div class="skeleton">Loading...</div>
</twig:HeavyComponent>

{# Custom loading attribute in component #}
```

```php
#[AsLiveComponent]
final class HeavyComponent
{
    // loading="lazy" or loading="defer" handled automatically
}
```
