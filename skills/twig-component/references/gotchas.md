# TwigComponent Gotchas & Debugging

## Attributes

### Missing `{{ attributes }}` on Root Element

```twig
{# WRONG: no attributes on root -- extra HTML attributes are silently lost #}
<div class="alert alert-{{ type }}">
    {{ message }}
</div>

{# CORRECT: attributes rendered on root element #}
<div class="alert alert-{{ type }}" {{ attributes }}>
    {{ message }}
</div>
```

Without `{{ attributes }}`, any HTML attributes passed when using the component (e.g., `<twig:Alert class="mb-4" id="main-alert" />`) are silently dropped.

### Attributes Merge with Existing

```twig
{# Component template #}
<div class="alert" {{ attributes }}>

{# Usage #}
<twig:Alert class="mb-4" />

{# Renders as: <div class="alert mb-4"> -- classes are MERGED #}
```

`{{ attributes }}` merges `class` values. Other attributes (like `id`, `style`, `data-*`) are added or overwritten. If you don't want merging, use `{{ attributes.without('class') }}`.

### Rendering Attributes on the Wrong Element

```twig
{# WRONG: attributes on inner element, root has no attributes #}
<div>
    <button {{ attributes }}>Click me</button>
</div>

{# CORRECT: attributes on root element #}
<div {{ attributes }}>
    <button>Click me</button>
</div>
```

Attributes should always go on the **root** element of the component template.

---

## Naming and Resolution

### Component Name Doesn't Match Class

```php
// This component is available as "Alert" (class name, not file name)
#[AsTwigComponent]
final class Alert { }

// Template must be: templates/components/Alert.html.twig

// If you customize the name:
#[AsTwigComponent(name: 'ui:alert')]
final class Alert { }
// Template: templates/components/ui/alert.html.twig
// Usage: <twig:ui:alert />
```

### Template Not Found

```
# Expected path for component "Alert":
templates/components/Alert.html.twig    # default
templates/components/alert.html.twig    # WRONG (case-sensitive!)

# Expected path for component "ui:alert":
templates/components/ui/alert.html.twig
```

The template path is derived from the component name. Colons (`:`) become directory separators.

### Anonymous Component Not Discovered

Anonymous components (no PHP class) must be in the `anonymous_template_directory`:

```yaml
# config/packages/twig_component.yaml
twig_component:
    anonymous_template_directory: 'components/'
    defaults:
        App\Twig\Components\: 'components/'
```

```
# Template location:
templates/components/Badge.html.twig    # <twig:Badge />
templates/components/ui/Tag.html.twig   # <twig:ui:Tag />
```

If the template exists but the component isn't found, check the directory config.

---

## Props

### Undeclared Props Become Attributes

```php
#[AsTwigComponent]
final class Button
{
    public string $label;
}
```

```twig
{# "variant" is not a prop -- it becomes an HTML attribute #}
<twig:Button label="Save" variant="primary" />

{# Renders: <button variant="primary">Save</button> -- probably not what you want #}
```

If a value passed to a component doesn't match a declared property, it silently goes into `{{ attributes }}`. This can be confusing. Declare all expected props.

### Required Props Not Provided

```twig
{# This will throw an error if "label" has no default #}
<twig:Button />

{# Fix: either provide the prop or add a default in PHP #}
<twig:Button label="Save" />
```

### Prop Type Mismatch

```twig
{# WRONG: passing string "true" instead of boolean true #}
<twig:Toggle enabled="true" />

{# CORRECT: use : prefix for expressions #}
<twig:Toggle :enabled="true" />
<twig:Toggle :count="42" />
<twig:Toggle :items="['a', 'b']" />
```

The `:` prefix evaluates the value as a Twig expression. Without it, the value is always a string.

---

## Blocks and Slots

### Block Name Collision with Parent Layout

```twig
{# Component template #}
<div {{ attributes }}>
    {% block content %}{% endblock %}
</div>
```

If your component's block name conflicts with a block in the parent layout, unexpected things happen. Use specific names:

```twig
{# BETTER: use prefixed or specific block names #}
<div {{ attributes }}>
    {% block card_content %}{% endblock %}
</div>
```

### Nested Component Block Scope

```twig
{# Blocks only apply to the IMMEDIATE component, not nested ones #}
<twig:Card>
    <twig:block name="header">Card Header</twig:block>

    <twig:Button>
        {# This block is for Button, NOT for Card #}
        <twig:block name="content">Button Content</twig:block>
    </twig:Button>
</twig:Card>
```

### outerScope for Accessing Parent Variables

Inside a component template, the parent's Twig variables are NOT available. Use `outerScope`:

```twig
{# Parent template #}
{% set color = 'blue' %}
<twig:Badge />

{# Badge.html.twig -- this.color is NOT "blue" #}
{# Use outerScope to access parent variables: #}
<span class="badge badge-{{ outerScope.color }}">{{ label }}</span>
```

---

## PreMount and Mount

### PreMount Validation Runs Before Mount

```php
#[AsTwigComponent]
final class Pagination
{
    public int $page;
    public int $total;

    #[PreMount]
    public function preMount(array $data): array
    {
        // This runs BEFORE mount() and receives the raw data
        // Use it for validation or transformation
        if (isset($data['page']) && $data['page'] < 1) {
            $data['page'] = 1;
        }
        return $data;
    }
}
```

### Mount Is Called After Property Assignment

```php
#[AsTwigComponent]
final class DataTable
{
    public array $items = [];
    public int $perPage = 10;

    private array $paginatedItems;

    public function mount(): void
    {
        // Properties are already set when mount() is called
        $this->paginatedItems = array_slice($this->items, 0, $this->perPage);
    }
}
```

---

## Computed Properties

### Computed Methods Are Cached Per Render

```php
#[AsTwigComponent]
final class ExpensiveComponent
{
    #[ExposeInTemplate]
    public function expensiveCalculation(): array
    {
        // This is called ONCE per render, even if used multiple times in template
        return $this->repository->findAll();
    }
}
```

```twig
{# Both access the same cached result #}
{{ this.expensiveCalculation|length }} items
{% for item in this.expensiveCalculation %}
    ...
{% endfor %}
```

---

## HTML vs Twig Syntax

### Self-Closing vs Block Syntax

```twig
{# Self-closing: no blocks needed #}
<twig:Alert type="success" message="Done!" />

{# Block syntax: when you need to pass blocks #}
<twig:Card>
    <twig:block name="header">Title</twig:block>
    Content here
</twig:Card>

{# WRONG: self-closing with blocks (blocks are ignored) #}
<twig:Card header="Title" />
{# This only works if "header" is a prop, not a block #}
```

### Expression Syntax

```twig
{# Static string value #}
<twig:Alert type="success" />

{# Dynamic Twig expression (note the : prefix) #}
<twig:Alert :type="alertType" />
<twig:Alert :type="isError ? 'danger' : 'info'" />
<twig:UserCard :user="app.user" />
```

---

## Debugging

### Check Component Registration

```bash
php bin/console debug:twig-component
```

Lists all registered components, their class, template path, and type (live or not).

### Component Not Rendering

1. Check the component name matches exactly (case-sensitive)
2. Check the template exists at the expected path
3. Check the class has `#[AsTwigComponent]`
4. Check the `defaults` config maps the namespace correctly
5. Clear the cache: `php bin/console cache:clear`
