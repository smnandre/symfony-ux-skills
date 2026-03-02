# LiveComponent Patterns

Common patterns for building reactive components.

---

## Live Search

```php
#[AsLiveComponent]
final class ProductSearch
{
    use DefaultActionTrait;
    
    #[LiveProp(writable: true, url: true)]
    public string $query = '';
    
    #[LiveProp(writable: true, url: true)]
    public string $category = '';
    
    #[LiveProp(writable: true, url: true)]
    public int $page = 1;
    
    public function __construct(
        private readonly ProductRepository $products,
    ) {}
    
    #[LiveProp(writable: true, onUpdated: 'resetPage')]
    public string $query = '';
    
    public function resetPage(): void
    {
        $this->page = 1;
    }
    
    public function getResults(): Paginator
    {
        return $this->products->search(
            query: $this->query,
            category: $this->category,
            page: $this->page,
        );
    }
}
```

```twig
<div {{ attributes }}>
    <div class="search-controls">
        <input 
            type="search" 
            data-model="debounce(300)|query"
            placeholder="Search..."
        >
        
        <select data-model="category">
            <option value="">All Categories</option>
            {% for cat in categories %}
                <option value="{{ cat.id }}">{{ cat.name }}</option>
            {% endfor %}
        </select>
    </div>
    
    <div class="results" data-loading="addClass(opacity-50)">
        <span data-loading="show" class="spinner">Searching...</span>
        
        {% for product in this.results %}
            <twig:ProductCard :product="product" />
        {% else %}
            <p>No products found for "{{ query }}"</p>
        {% endfor %}
    </div>
    
    {% if this.results.hasNextPage %}
        <button 
            data-action="live#action"
            data-live-action-param="loadMore"
            data-loading="attr(disabled)"
        >
            Load More
        </button>
    {% endif %}
</div>
```

---

## Editable List (CRUD)

```php
#[AsLiveComponent]
final class TodoList
{
    use DefaultActionTrait;
    use ComponentToolsTrait;
    
    #[LiveProp]
    public array $todos = [];
    
    #[LiveProp(writable: true)]
    public string $newTodo = '';
    
    #[LiveProp]
    public ?int $editingId = null;
    
    #[LiveProp(writable: true)]
    public string $editingText = '';
    
    public function __construct(
        private readonly TodoRepository $repository,
    ) {}
    
    public function mount(): void
    {
        $this->todos = $this->repository->findAll();
    }
    
    #[LiveAction]
    public function add(): void
    {
        if (empty($this->newTodo)) return;
        
        $todo = new Todo($this->newTodo);
        $this->repository->save($todo);
        
        $this->todos[] = $todo;
        $this->newTodo = '';
    }
    
    #[LiveAction]
    public function startEdit(#[LiveArg] int $id): void
    {
        $this->editingId = $id;
        $this->editingText = $this->findTodo($id)->getText();
    }
    
    #[LiveAction]
    public function saveEdit(): void
    {
        $todo = $this->findTodo($this->editingId);
        $todo->setText($this->editingText);
        $this->repository->save($todo);
        
        $this->editingId = null;
    }
    
    #[LiveAction]
    public function cancelEdit(): void
    {
        $this->editingId = null;
    }
    
    #[LiveAction]
    public function delete(#[LiveArg] int $id): void
    {
        $this->repository->delete($id);
        $this->todos = array_filter(
            $this->todos, 
            fn($t) => $t->getId() !== $id
        );
    }
    
    #[LiveAction]
    public function toggle(#[LiveArg] int $id): void
    {
        $todo = $this->findTodo($id);
        $todo->toggle();
        $this->repository->save($todo);
    }
}
```

```twig
<div {{ attributes }}>
    <form data-action="live#action:prevent" data-live-action-param="add">
        <input 
            type="text" 
            data-model="norender|newTodo"
            placeholder="Add todo..."
        >
        <button type="submit">Add</button>
    </form>
    
    <ul>
        {% for todo in todos %}
            <li data-live-id="{{ todo.id }}">
                {% if editingId == todo.id %}
                    <input 
                        type="text" 
                        data-model="norender|editingText"
                        value="{{ editingText }}"
                    >
                    <button data-action="live#action" data-live-action-param="saveEdit">
                        Save
                    </button>
                    <button data-action="live#action" data-live-action-param="cancelEdit">
                        Cancel
                    </button>
                {% else %}
                    <input 
                        type="checkbox"
                        {{ todo.done ? 'checked' }}
                        data-action="change->live#action"
                        data-live-action-param="toggle"
                        data-live-id-param="{{ todo.id }}"
                    >
                    <span class="{{ todo.done ? 'line-through' }}">
                        {{ todo.text }}
                    </span>
                    <button 
                        data-action="live#action"
                        data-live-action-param="startEdit"
                        data-live-id-param="{{ todo.id }}"
                    >
                        Edit
                    </button>
                    <button 
                        data-action="live#action"
                        data-live-action-param="delete"
                        data-live-id-param="{{ todo.id }}"
                    >
                        Delete
                    </button>
                {% endif %}
            </li>
        {% endfor %}
    </ul>
</div>
```

---

## Form with Validation

```php
#[AsLiveComponent]
final class ContactForm extends AbstractController
{
    use DefaultActionTrait;
    use ComponentWithFormTrait;
    
    #[LiveProp]
    public bool $submitted = false;
    
    protected function instantiateForm(): FormInterface
    {
        return $this->createForm(ContactType::class);
    }
    
    #[LiveAction]
    public function submit(MailerInterface $mailer): void
    {
        $this->submitForm();
        
        if (!$this->getForm()->isValid()) {
            return;
        }
        
        $data = $this->getForm()->getData();
        
        // Send email...
        $mailer->send(...);
        
        $this->submitted = true;
    }
}
```

```twig
<div {{ attributes }}>
    {% if submitted %}
        <div class="alert alert-success">
            Thank you! We'll be in touch.
        </div>
    {% else %}
        {{ form_start(form, {
            attr: {
                'data-action': 'live#action:prevent',
                'data-live-action-param': 'submit'
            }
        }) }}
        
        <div class="form-group">
            {{ form_label(form.email) }}
            {{ form_widget(form.email, {
                attr: {'data-model': 'on(blur)|validatedFields'}
            }) }}
            {{ form_errors(form.email) }}
        </div>
        
        <div class="form-group">
            {{ form_label(form.message) }}
            {{ form_widget(form.message) }}
            {{ form_errors(form.message) }}
        </div>
        
        <button type="submit" data-loading="attr(disabled)">
            <span data-loading="hide">Send Message</span>
            <span data-loading="show">Sending...</span>
        </button>
        
        {{ form_end(form) }}
    {% endif %}
</div>
```

---

## Dependent Selects

```php
#[AsLiveComponent]
final class LocationSelector
{
    use DefaultActionTrait;
    
    #[LiveProp(writable: true, onUpdated: 'onCountryChanged')]
    public ?string $country = null;
    
    #[LiveProp(writable: true, onUpdated: 'onStateChanged')]
    public ?string $state = null;
    
    #[LiveProp(writable: true)]
    public ?string $city = null;
    
    public function __construct(
        private readonly LocationRepository $locations,
    ) {}
    
    public function onCountryChanged(): void
    {
        $this->state = null;
        $this->city = null;
    }
    
    public function onStateChanged(): void
    {
        $this->city = null;
    }
    
    public function getCountries(): array
    {
        return $this->locations->getCountries();
    }
    
    public function getStates(): array
    {
        if (!$this->country) return [];
        return $this->locations->getStates($this->country);
    }
    
    public function getCities(): array
    {
        if (!$this->state) return [];
        return $this->locations->getCities($this->state);
    }
}
```

```twig
<div {{ attributes }}>
    <select data-model="country">
        <option value="">Select Country</option>
        {% for country in this.countries %}
            <option value="{{ country.code }}">{{ country.name }}</option>
        {% endfor %}
    </select>
    
    <select data-model="state" {{ not country ? 'disabled' }}>
        <option value="">Select State</option>
        {% for state in this.states %}
            <option value="{{ state.code }}">{{ state.name }}</option>
        {% endfor %}
    </select>
    
    <select data-model="city" {{ not state ? 'disabled' }}>
        <option value="">Select City</option>
        {% for city in this.cities %}
            <option value="{{ city.id }}">{{ city.name }}</option>
        {% endfor %}
    </select>
</div>
```

---

## Modal with Form

```php
#[AsLiveComponent]
final class CreateItemModal extends AbstractController
{
    use DefaultActionTrait;
    use ComponentWithFormTrait;
    use ComponentToolsTrait;
    
    #[LiveProp]
    public bool $isOpen = false;
    
    protected function instantiateForm(): FormInterface
    {
        return $this->createForm(ItemType::class);
    }
    
    #[LiveAction]
    public function open(): void
    {
        $this->isOpen = true;
        $this->resetForm();
    }
    
    #[LiveAction]
    public function close(): void
    {
        $this->isOpen = false;
    }
    
    #[LiveAction]
    public function save(EntityManagerInterface $em): void
    {
        $this->submitForm();
        
        if (!$this->getForm()->isValid()) {
            return;
        }
        
        $item = $this->getForm()->getData();
        $em->persist($item);
        $em->flush();
        
        $this->emit('item:created', ['id' => $item->getId()]);
        $this->isOpen = false;
    }
}
```

```twig
<div {{ attributes }}>
    <button data-action="live#action" data-live-action-param="open">
        Create Item
    </button>
    
    {% if isOpen %}
        <dialog open data-action="keydown.esc@window->live#action" data-live-action-param="close">
            <header>
                <h2>Create New Item</h2>
                <button data-action="live#action" data-live-action-param="close">&times;</button>
            </header>
            
            {{ form_start(form, {
                attr: {
                    'data-action': 'live#action:prevent',
                    'data-live-action-param': 'save'
                }
            }) }}
            
            {{ form_row(form.name) }}
            {{ form_row(form.description) }}
            
            <footer>
                <button type="button" data-action="live#action" data-live-action-param="close">
                    Cancel
                </button>
                <button type="submit" data-loading="attr(disabled)">
                    Create
                </button>
            </footer>
            
            {{ form_end(form) }}
        </dialog>
    {% endif %}
</div>
```

---

## Auto-Save

```php
#[AsLiveComponent]
final class AutoSaveEditor
{
    use DefaultActionTrait;
    use ComponentToolsTrait;
    
    #[LiveProp]
    public Document $document;
    
    #[LiveProp(writable: true)]
    public string $content = '';
    
    #[LiveProp]
    public ?string $savedAt = null;
    
    public function mount(Document $document): void
    {
        $this->document = $document;
        $this->content = $document->getContent();
    }
    
    #[LiveAction]
    public function save(EntityManagerInterface $em): void
    {
        $this->document->setContent($this->content);
        $em->flush();
        
        $this->savedAt = (new \DateTime())->format('H:i:s');
    }
}
```

```twig
<div {{ attributes }}>
    <textarea 
        data-model="debounce(1000)|content"
        data-action="input->live#action"
        data-live-action-param="save"
    >{{ content }}</textarea>
    
    <div class="status">
        <span data-loading="action(save)|show">Saving...</span>
        {% if savedAt %}
            <span data-loading="action(save)|hide">Saved at {{ savedAt }}</span>
        {% endif %}
    </div>
</div>
```

---

## Voting Component

```php
#[AsLiveComponent]
final class VoteButton
{
    use DefaultActionTrait;
    
    #[LiveProp]
    public Post $post;
    
    #[LiveProp]
    public ?User $currentUser = null;
    
    public function getUserVote(): ?int
    {
        if (!$this->currentUser) return null;
        return $this->post->getVoteByUser($this->currentUser)?->getValue();
    }
    
    #[LiveAction]
    public function vote(#[LiveArg] int $value, VoteService $voteService): void
    {
        if (!$this->currentUser) return;
        
        $voteService->vote($this->post, $this->currentUser, $value);
    }
}
```

```twig
<div {{ attributes }} class="vote-buttons">
    <button 
        data-action="live#action"
        data-live-action-param="vote"
        data-live-value-param="1"
        class="{{ this.userVote == 1 ? 'active' }}"
        {{ not currentUser ? 'disabled' }}
    >
        ▲
    </button>
    
    <span>{{ post.score }}</span>
    
    <button 
        data-action="live#action"
        data-live-action-param="vote"
        data-live-value-param="-1"
        class="{{ this.userVote == -1 ? 'active' }}"
        {{ not currentUser ? 'disabled' }}
    >
        ▼
    </button>
</div>
```

---

## Parent-Child Communication

### Parent Component

```php
#[AsLiveComponent]
final class ShoppingCart
{
    use DefaultActionTrait;
    
    #[LiveProp]
    public array $items = [];
    
    #[LiveListener('cart:item-removed')]
    public function onItemRemoved(#[LiveArg] int $itemId): void
    {
        $this->items = array_filter(
            $this->items,
            fn($item) => $item->getId() !== $itemId
        );
    }
    
    #[LiveListener('cart:item-updated')]
    public function onItemUpdated(): void
    {
        // Re-renders to show updated total
    }
    
    public function getTotal(): float
    {
        return array_sum(array_map(
            fn($item) => $item->getPrice() * $item->getQuantity(),
            $this->items
        ));
    }
}
```

### Child Component

```php
#[AsLiveComponent]
final class CartItem
{
    use DefaultActionTrait;
    use ComponentToolsTrait;
    
    #[LiveProp]
    public CartItem $item;
    
    #[LiveProp(writable: true)]
    public int $quantity;
    
    public function mount(CartItem $item): void
    {
        $this->item = $item;
        $this->quantity = $item->getQuantity();
    }
    
    #[LiveAction]
    public function updateQuantity(CartService $cart): void
    {
        $cart->updateQuantity($this->item, $this->quantity);
        $this->emitUp('cart:item-updated');
    }
    
    #[LiveAction]
    public function remove(CartService $cart): void
    {
        $cart->remove($this->item);
        $this->emitUp('cart:item-removed', ['itemId' => $this->item->getId()]);
    }
}
```
