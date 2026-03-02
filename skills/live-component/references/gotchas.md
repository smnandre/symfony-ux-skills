# LiveComponent Gotchas & Debugging

Common pitfalls and solutions.

---

## Props Not Persisting

### Non-LiveProp Values Reset

```php
// ❌ Regular property - resets on every re-render
public string $query = '';

// ✅ LiveProp - persists between re-renders
#[LiveProp]
public string $query = '';
```

### Props Must Be Writable for data-model

```php
// ❌ Not writable - data-model won't work
#[LiveProp]
public string $search = '';

// ✅ Writable - data-model works
#[LiveProp(writable: true)]
public string $search = '';
```

### Object Props Need Hydration

```php
// ❌ Complex objects can't auto-hydrate
#[LiveProp]
public ComplexDTO $data;

// ✅ Entity with ID - auto-hydrates
#[LiveProp]
public Product $product;

// ✅ Custom hydration for DTOs
#[LiveProp(hydrateWith: 'hydrateData', dehydrateWith: 'dehydrateData')]
public ComplexDTO $data;
```

---

## Missing DefaultActionTrait

```php
// ❌ Won't work - missing trait
#[AsLiveComponent]
final class MyComponent
{
}

// ✅ Must include DefaultActionTrait
#[AsLiveComponent]
final class MyComponent
{
    use DefaultActionTrait;
}
```

---

## Missing attributes in Template

```twig
{# ❌ Component won't work - missing attributes #}
<div>
    Content
</div>

{# ✅ Root element must have {{ attributes }} #}
<div {{ attributes }}>
    Content
</div>
```

---

## data-model Binding Issues

### Wrong Element for Binding

```twig
{# ❌ data-model on non-input element #}
<div data-model="query">...</div>

{# ✅ data-model on input/select/textarea #}
<input data-model="query">
<select data-model="category">
<textarea data-model="content">
```

### Nested Property Access

```php
#[LiveProp(writable: ['email', 'name'])]
public User $user;
```

```twig
{# ✅ Access nested properties #}
<input data-model="user.email">
<input data-model="user.name">

{# ❌ Property not in writable list #}
<input data-model="user.password">
```

### Checkbox/Radio Binding

```twig
{# Checkbox needs value or boolean prop #}
<input type="checkbox" data-model="enabled">

{# Multiple checkboxes to array #}
<input type="checkbox" data-model="selectedIds" value="1">
<input type="checkbox" data-model="selectedIds" value="2">
```

---

## Action Errors

### Action Not Found

```php
// ❌ Missing LiveAction attribute
public function save(): void { }

// ✅ Must have LiveAction attribute
#[LiveAction]
public function save(): void { }
```

### Wrong Parameter Name

```twig
{# Template #}
<button 
    data-action="live#action"
    data-live-action-param="delete"
    data-live-id-param="{{ id }}"  {# param name: id #}
>
```

```php
// ❌ Param name mismatch
#[LiveAction]
public function delete(#[LiveArg] int $itemId): void { }

// ✅ Matching param name
#[LiveAction]
public function delete(#[LiveArg] int $id): void { }

// ✅ Or specify custom name
#[LiveAction]
public function delete(#[LiveArg('id')] int $itemId): void { }
```

---

## Form Issues

### Form Not Validating

```php
// ❌ Form not submitted
#[LiveAction]
public function save(): void
{
    $data = $this->getForm()->getData();  // Not validated!
}

// ✅ Submit form first
#[LiveAction]
public function save(): void
{
    $this->submitForm();  // Validates
    
    if (!$this->getForm()->isValid()) {
        return;  // Re-renders with errors
    }
    
    $data = $this->getForm()->getData();
}
```

### initialFormData Type Mismatch

```php
// ❌ Wrong type for initialFormData
#[LiveProp]
public ?array $initialFormData = null;

// ✅ Match form's data_class
#[LiveProp]
public ?User $initialFormData = null;
```

### Form Re-creation Issues

```php
protected function instantiateForm(): FormInterface
{
    // ❌ Creating form with data from prop that changes
    return $this->createForm(UserType::class, $this->user);
    
    // ✅ Use initialFormData for initial state
    return $this->createForm(UserType::class, $this->initialFormData);
}
```

---

## Hydration Errors

### Entity Not Found

```php
#[LiveProp]
public Product $product;  // ❌ If product deleted, hydration fails
```

Solution: Make nullable or handle in mount:

```php
#[LiveProp]
public ?Product $product = null;

public function mount(?Product $product): void
{
    $this->product = $product;
}
```

### Collection Hydration

```php
// ❌ Array of entities won't auto-hydrate
#[LiveProp]
public array $products = [];

// ✅ Store IDs and fetch in getter
#[LiveProp]
public array $productIds = [];

public function getProducts(): array
{
    return $this->productRepository->findBy(['id' => $this->productIds]);
}
```

---

## Performance Issues

### Too Many Re-renders

```twig
{# ❌ Re-renders on every keystroke #}
<input data-model="search">

{# ✅ Debounce to reduce requests #}
<input data-model="debounce(300)|search">

{# ✅ Only update on blur #}
<input data-model="on(blur)|search">
```

### Heavy Computed Properties

```php
// ❌ Expensive query on every render
public function getProducts(): array
{
    return $this->repository->findWithComplexQuery();
}

// ✅ Use computed caching
```

```twig
{# Computed is cached within single render #}
{% for product in computed.products %}
```

### Large DOM Updates

```twig
{# ❌ Large list without stable IDs #}
{% for item in items %}
    <div>{{ item.name }}</div>
{% endfor %}

{# ✅ Add data-live-id for efficient morphing #}
{% for item in items %}
    <div data-live-id="{{ item.id }}">{{ item.name }}</div>
{% endfor %}
```

---

## DOM Preservation

### Third-Party Widgets Destroyed

```twig
{# ❌ Widget re-initialized on every render #}
<select class="select2">

{# ✅ Prevent morphing #}
<div data-live-ignore>
    <select class="select2">
</div>
```

### User Input Lost

```twig
{# ❌ Input value reset during morph #}
<input type="text" name="temp">

{# ✅ Preserve specific attribute #}
<input type="text" name="temp" data-live-preserve="value">
```

---

## Event Issues

### Event Not Received

```php
// Child emits
$this->emit('item:saved', ['id' => 1]);

// ❌ Parent listener method name mismatch
#[LiveListener('itemSaved')]  // Wrong event name
public function onItemSaved(): void { }

// ✅ Exact event name match
#[LiveListener('item:saved')]
public function onItemSaved(#[LiveArg] int $id): void { }
```

### emitUp vs emit

```php
// emit - broadcasts to all listeners
$this->emit('event');

// emitUp - only to parent components
$this->emitUp('event');

// emitTo - to specific component
$this->emitTo('ComponentName', 'event');
```

---

## URL Sync Issues

### URL Not Updating

```php
// ❌ Missing writable
#[LiveProp(url: true)]
public string $query = '';

// ✅ Must be writable for URL sync
#[LiveProp(writable: true, url: true)]
public string $query = '';
```

### History State

```php
// Default: replaces URL without history entry
#[LiveProp(writable: true, url: true)]

// Push to history (back button works)
use Symfony\UX\LiveComponent\Metadata\UrlMapping;

#[LiveProp(writable: true, url: new UrlMapping(history: 'push'))]
```

---

## Debugging

### Enable Debug Mode

Check browser console for LiveComponent logs.

### Inspect Component State

```twig
{# Debug: show current state #}
<pre>{{ _context|json_encode(constant('JSON_PRETTY_PRINT')) }}</pre>
```

### Network Tab

1. Open DevTools → Network
2. Filter by Fetch/XHR
3. Look for requests to `/_components/`
4. Check request payload for props
5. Check response for HTML

### Common Errors

| Error | Cause |
|-------|-------|
| "Component not found" | Missing `#[AsLiveComponent]` or not in configured namespace |
| "Checksum mismatch" | LiveProp changed without being writable |
| "Cannot hydrate" | Entity deleted or custom type without hydrator |
| "Action not found" | Missing `#[LiveAction]` attribute |

### Test in Isolation

```php
use Symfony\UX\LiveComponent\Test\InteractsWithLiveComponents;

public function testComponent(): void
{
    $component = $this->createLiveComponent('MyComponent', [
        'initialProp' => 'value'
    ]);
    
    // Debug
    dump($component->render());
    dump($component->get('propName'));
}
```
