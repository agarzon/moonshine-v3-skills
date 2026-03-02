# Relationship Fields Reference

All relationship fields require a registered `ModelResource`. The resource must be registered in `MoonShineServiceProvider` via `$core->resources()`.

## Common Constructor

```php
RelationField::make(
    Closure|string $label,
    ?string $relationName = null,      // auto-detected from label (camelCase)
    Closure|string|null $formatted = null, // display column or closure
    ModelResource|string|null $resource = null, // auto-detected from relation name
)
```

If `$resource` is omitted, MoonShine looks for a resource matching `{RelationName}Resource` (kebab-case URI).

---

## BelongsTo

`MoonShine\Laravel\Fields\Relationships\BelongsTo`

Renders as a `<select>` dropdown. Links to edit page in preview.

### Basic Usage

```php
use MoonShine\Laravel\Fields\Relationships\BelongsTo;

BelongsTo::make('User', 'user', resource: UserResource::class)

// Shorthand (resource auto-detected):
BelongsTo::make('User')

// Custom display column:
BelongsTo::make('Country', 'country', formatted: 'name')

// Custom display callback:
BelongsTo::make('Country', 'country', fn($item) => "$item->id. $item->title")
```

### All Methods

| Method | Description |
|--------|-------------|
| `default(Model $model)` | Default value (pass model instance) |
| `nullable()` | Allow NULL |
| `placeholder(string $value)` | Placeholder text |
| `searchable()` | Enable search in dropdown |
| `native()` | Disable Choices.js |
| `creatable(?ActionButton $button)` | Create related object via modal |
| `valuesQuery(Closure $callback)` | Filter dropdown values |
| `asyncSearch(...)` | Async search |
| `associatedWith(string $column)` | Dependent dropdown |
| `withImage(string $column, string $disk, string $dir)` | Show images in dropdown |
| `link(...)` | Custom preview link |

### Async Search

```php
BelongsTo::make('Category', 'category', resource: CategoryResource::class)
    ->asyncSearch(
        'title',                    // column to search
        searchQuery: function(Builder $query, Request $request, string $term, Field $field) {
            return $query->where('id', '!=', 2);
        },
        formatted: fn($item, Field $field) => $item->id . ' | ' . $item->title,
        limit: 10,
    )
```

### Associated (Dependent) Fields

```php
Select::make('Country', 'country_id'),

BelongsTo::make('City', 'city', resource: CityResource::class)
    ->associatedWith('country_id')
    ->asyncOnInit(whenOpen: false)
```

### Values with Images

```php
BelongsTo::make('Country', resource: CountryResource::class)
    ->withImage('thumb', 'public', 'countries')
```

### Filter Dropdown Values

```php
BelongsTo::make('Category', 'category', resource: CategoryResource::class)
    ->valuesQuery(fn(Builder $query, Field $field) => $query->where('active', true))
```

### Creatable

```php
BelongsTo::make('Author', resource: AuthorResource::class)
    ->creatable()
```

---

## BelongsToMany

`MoonShine\Laravel\Fields\Relationships\BelongsToMany`

Renders as checkboxes by default. Can switch to select dropdown.

### Basic Usage

```php
use MoonShine\Laravel\Fields\Relationships\BelongsToMany;

BelongsToMany::make('Categories', 'categories', resource: CategoryResource::class)
```

### Display Modes

```php
// Checkbox list (default)
BelongsToMany::make('Categories', resource: CategoryResource::class)

// Dropdown select
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->selectMode()

// Tree with checkboxes (for hierarchical data)
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->tree('parent_id')

// Horizontal layout
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->horizontalMode(true, minColWidth: '100px', maxColWidth: '33%')
```

### Pivot Fields

```php
BelongsToMany::make('Contacts', resource: ContactResource::class)
    ->fields([
        Text::make('Contact', 'text'),
    ])
```

> Must specify pivot fields in the Laravel relationship: `->withPivot('text')`

### Preview Modes

```php
// Table (default)
BelongsToMany::make('Categories', resource: CategoryResource::class)

// Count only
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->onlyCount()

// Inline with badges
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->inLine(
        separator: ' ',
        badge: fn($model, $value) => Badge::make((string) $value, 'primary'),
    )

// Link with count
BelongsToMany::make('Categories', resource: CategoryResource::class)
    ->relatedLink('category')
```

### All Methods

| Method | Description |
|--------|-------------|
| `selectMode()` | Render as dropdown |
| `tree(string $parentColumn)` | Tree with checkboxes |
| `horizontalMode(bool, ...)` | Horizontal checkbox layout |
| `fields(iterable $fields)` | Pivot fields |
| `columnLabel(string $label)` | Override column header |
| `creatable(?ActionButton $button)` | Create related object via modal |
| `onlyCount()` | Show count in preview |
| `inLine(separator, badge, link)` | Inline preview |
| `relatedLink(...)` | Show as link with count |
| `valuesQuery(Closure)` | Filter values |
| `asyncSearch(...)` | Async search |
| `associatedWith(string $column)` | Dependent values |
| `withImage(string, string, string)` | Images in options |
| `withCheckAll()` | Add check/uncheck all button |
| `buttons(array $buttons)` | Custom buttons |

---

## HasOne

`MoonShine\Laravel\Fields\Relationships\HasOne`

Displays as embedded form/table. Displayed outside the main form by default.

### Basic Usage

```php
use MoonShine\Laravel\Fields\Relationships\HasOne;

HasOne::make('Profile', 'profile', resource: ProfileResource::class)
```

> The `$formatted` parameter is NOT used in HasOne.

### Specifying Fields

```php
HasOne::make('Profile', resource: ProfileResource::class)
    ->fields([
        Phone::make('Phone'),
        Text::make('Address'),
    ])
```

### Display Modes

```php
// In tabs
HasOne::make('Comment', resource: CommentResource::class)
    ->tabMode()

// In modal
HasOne::make('Comment', resource: CommentResource::class)
    ->modalMode()

// Inside the main form
HasOne::make('Comment', resource: CommentResource::class)
    ->disableOutside()  // only works with modalMode
```

### Modification

```php
HasOne::make('Comment', resource: CommentResource::class)
    ->modifyTable(fn(TableBuilder $table) => $table)
    ->modifyForm(fn(FormBuilder $form) => $form->submit('Save Profile'))
    ->redirectAfter(fn(int $parentId) => route('home'))
```

---

## HasMany

`MoonShine\Laravel\Fields\Relationships\HasMany`

Displays as a table with CRUD. Rendered outside the main form by default.

### Basic Usage

```php
use MoonShine\Laravel\Fields\Relationships\HasMany;

HasMany::make('Comments', 'comments', resource: CommentResource::class)
```

> The related resource must have a `BelongsTo` field pointing to the parent.

### Key Methods

| Method | Description |
|--------|-------------|
| `fields(iterable $fields)` | Override displayed fields |
| `creatable(?ActionButton $button)` | Enable creating related records |
| `limit(int $limit)` | Limit records in preview (default: 15) |
| `relatedLink()` | Show as link with count |
| `searchable(bool)` | Enable/disable search (default: true) |
| `withoutModals()` | Disable modal editing |
| `disableOutside()` | Render inside main form |
| `tabMode()` | Display in tabs |
| `modalMode(...)` | Display in modal |
| `activeActions(Action ...)` | Restrict available actions |
| `withoutActions(Action ...)` | Exclude specific actions |
| `indexButtons(array)` | Add buttons to table |
| `formButtons(array)` | Add buttons to edit form |
| `modifyTable(Closure)` | Modify TableBuilder |
| `modifyBuilder(Closure)` | Modify QueryBuilder |
| `modifyCreateButton(Closure)` | Modify create button |
| `modifyEditButton(Closure)` | Modify edit button |
| `modifyItemButtons(Closure)` | Modify all item buttons |
| `changeEditButton(ActionButton)` | Replace edit button |
| `redirectAfter(Closure)` | Custom redirect after save |

### Creatable

```php
HasMany::make('Comments', resource: CommentResource::class)
    ->creatable()
```

### Active Actions

```php
HasMany::make('Comments')
    ->activeActions(Action::VIEW, Action::UPDATE)

HasMany::make('Comments')
    ->withoutActions(Action::DELETE)
```

### Display Modes

```php
// In tabs
HasMany::make('Comments', resource: CommentResource::class)->tabMode()

// In modal
HasMany::make('Comments', resource: CommentResource::class)->modalMode()

// Inside main form
HasMany::make('Comments', resource: CommentResource::class)->disableOutside()
```

### Modification

```php
HasMany::make('Comments', resource: CommentResource::class)
    ->modifyTable(fn(TableBuilder $table, bool $preview) =>
        $table->customAttributes(['style' => 'background: #f0f0f0'])
    )
    ->modifyBuilder(fn(Relation $query, HasMany $ctx) => $query->orderBy('created_at', 'desc'))
    ->modifyItemButtons(fn($detail, $edit, $delete, $massDelete, HasMany $ctx) => [$detail, $edit])
```

### Related Link

```php
HasMany::make('Comments', resource: CommentResource::class)
    ->relatedLink()
    ->modifyRelatedLink(fn(ActionButton $button, bool $preview) =>
        $button->when($preview, fn($btn) => $btn->primary())
    )
```

### Parent ID in Related Resource

```php
class CommentResource extends ModelResource
{
    use ResourceWithParent;

    protected function getParentResourceClassName(): string
    {
        return PostResource::class;
    }

    protected function getParentRelationName(): string
    {
        return 'post';
    }
}
// Access: $this->getParentId()
```

---

## HasOneThrough

`MoonShine\Laravel\Fields\Relationships\HasOneThrough` - Inherits from HasOne. Same API.

```php
use MoonShine\Laravel\Fields\Relationships\HasOneThrough;

HasOneThrough::make('Car Owner', 'carOwner', resource: OwnerResource::class)
```

---

## HasManyThrough

`MoonShine\Laravel\Fields\Relationships\HasManyThrough` - Inherits from HasMany. Same API.

```php
use MoonShine\Laravel\Fields\Relationships\HasManyThrough;

HasManyThrough::make('Deployments', 'deployments', resource: DeploymentResource::class)
```

---

## MorphOne

`MoonShine\Laravel\Fields\Relationships\MorphOne` - Inherits from HasOne. Same API.

```php
use MoonShine\Laravel\Fields\Relationships\MorphOne;

MorphOne::make('Profile', 'profile', resource: ProfileResource::class)
```

---

## MorphMany

`MoonShine\Laravel\Fields\Relationships\MorphMany` - Inherits from HasMany. Same API.

```php
use MoonShine\Laravel\Fields\Relationships\MorphMany;

MorphMany::make('Comments', 'comments', resource: CommentResource::class)
```

---

## MorphTo

`MoonShine\Laravel\Fields\Relationships\MorphTo` - Inherits from BelongsTo. Requires `types()` method.

```php
use MoonShine\Laravel\Fields\Relationships\MorphTo;

MorphTo::make('Commentable')->types([
    Article::class => 'title',           // display field as string
    Company::class => ['short_name', 'Organization'],  // [display_field, label]
])
```

The `types()` method is **required**. Values:
- Key: `class-string<Model>`
- Value: `string` (field name) or `array` `[field_name, custom_label]`

```php
MorphTo::make('Commentable', resource: PolyCommentResource::class)->types([
    Post::class => 'name',
    Project::class => 'name',
])
```

---

## MorphToMany

`MoonShine\Laravel\Fields\Relationships\MorphToMany` - Inherits from BelongsToMany. Same API.

```php
use MoonShine\Laravel\Fields\Relationships\MorphToMany;

MorphToMany::make('Categories', 'categories', resource: CategoryResource::class)
```

---

## RelationRepeater

`MoonShine\Laravel\Fields\Relationships\RelationRepeater` - Inline editing of HasMany/HasOne inside the main form.

### Basic Usage

```php
use MoonShine\Laravel\Fields\Relationships\RelationRepeater;

RelationRepeater::make('Characteristics', 'characteristics')
    ->fields([
        ID::make(),               // ID is REQUIRED
        Text::make('Name', 'name'),
        Text::make('Value', 'value'),
    ])
    ->creatable(limit: 5)
    ->removable()
```

> The `ID` field is **required** -- otherwise records are always added, never updated.

### All Methods

| Method | Description |
|--------|-------------|
| `fields(iterable $fields)` | Override field set (default: resource form fields) |
| `creatable(?int $limit, ?ActionButton $button)` | Allow adding rows |
| `removable(array $attributes)` | Allow removing rows |
| `vertical()` | Vertical display |
| `reorderable(string $url)` | Drag-and-drop sorting |
| `buttons(array $buttons)` | Override row buttons |
| `modifyTable(Closure)` | Modify TableBuilder |
| `modifyCreateButton(Closure)` | Modify create button |
| `modifyRemoveButton(Closure)` | Modify remove button |

### Example with Resource

```php
RelationRepeater::make('Comments', 'comments', resource: CommentResource::class)
    ->creatable()
    ->removable()
    ->vertical()
```

---

## Common Relationship Patterns

### Async Search (BelongsTo / BelongsToMany)

```php
BelongsTo::make('Category', resource: CategoryResource::class)
    ->asyncSearch('title', limit: 10)
```

### Creatable with Custom Button

```php
BelongsTo::make('Author', resource: AuthorResource::class)
    ->creatable(button: ActionButton::make('New Author', ''))
```

### Values Query Filter

```php
BelongsTo::make('Category', resource: CategoryResource::class)
    ->valuesQuery(fn(Builder $query, Field $field) => $query->where('active', true))
```

### Image in Dropdown

```php
BelongsTo::make('Country', resource: CountryResource::class)
    ->withImage('thumb', 'public', 'countries')
```

### Tab Mode for Multiple Relations

```php
HasMany::make('Comments', resource: CommentResource::class)->tabMode(),
HasMany::make('Reviews', resource: ReviewResource::class)->tabMode()
// Creates a Tabs component with "Comments" and "Reviews" tabs
```
