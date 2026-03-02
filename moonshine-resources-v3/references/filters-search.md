# Filters, Search, and Query Reference

## Filters

Filters use the same field classes as forms. They are displayed only on the index page.

### Basic Filter Declaration

```php
use MoonShine\UI\Fields\Text;

protected function filters(): iterable
{
    return [
        Text::make('Title', 'title'),
    ];
}
```

If the method returns an empty array (or is absent), no filters are shown.

> Some field types cannot participate in query building and are automatically excluded from filters.

### Custom Filter Logic

Override the filtering behavior with `onApply()` on the field:

```php
use MoonShine\UI\Fields\Text;

protected function filters(): iterable
{
    return [
        Text::make('Title', 'title')
            ->onApply(fn($query, $value) => $query->where('title', 'like', "%{$value}%")),
    ];
}
```

### Cache Filter State

Persist the user's last filter values across page loads:

```php
class PostResource extends ModelResource
{
    protected bool $saveQueryState = true;
}
```

---

## Search

### Basic Search

Specify which model columns participate in the search bar:

```php
protected function search(): array
{
    return ['id', 'title', 'text'];
}
```

If this returns an empty array, the search input is not displayed.

### Full-Text Search

Use the `SearchUsingFullText` attribute. Requires a full-text index on the specified columns.

```php
use MoonShine\Support\Attributes\SearchUsingFullText;

#[SearchUsingFullText(['title', 'text'])]
protected function search(): array
{
    return ['id'];
}
```

### JSON Key Search

For `Json` fields used as `keyValue()`:

```php
protected function search(): array
{
    return ['data->title'];
}
```

For multidimensional `Json` with `fields()`:

```php
protected function search(): array
{
    return ['data->[*]->title'];
}
```

### Relation Search

Search through a related model's fields:

```php
protected function search(): array
{
    return ['category.title'];
}
```

### Custom Search Query

Override `searchQuery()` to customize or extend search behavior:

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

protected function searchQuery(string $terms): void
{
    // Completely override
    $this->newQuery()->where(function (Builder $builder) use ($terms): void {
        $builder->where('custom_field', 'like', "%{$terms}%");
    });
}
```

To extend the default search (add conditions on top of the built-in logic):

```php
protected function searchQuery(string $terms): void
{
    parent::searchQuery($terms);

    $this->newQuery()->where(function (Builder $builder) use ($terms): void {
        $builder->orWhere('extra_field', 'like', "%{$terms}%");
    });
}
```

To completely override including full-text logic:

```php
protected function resolveSearch(string $terms, ?iterable $fullTextColumns = null): static
{
    // Your custom logic
    return $this;
}
```

### Global Search (Laravel Scout)

1. Install the package:

```bash
composer require moonshine/scout
php artisan vendor:publish --provider="MoonShine\Scout\Providers\ScoutServiceProvider"
```

2. Configure `config/moonshine-scout.php`:

```php
'models' => [
    Article::class,
    User::class,
],
```

3. Implement the interface in models:

```php
use Laravel\Scout\Builder;
use Laravel\Scout\Searchable;
use MoonShine\Scout\HasGlobalSearch;
use MoonShine\Scout\SearchableResponse;

class Article extends Model implements HasGlobalSearch
{
    use Searchable;

    public function searchableQuery(Builder $builder): Builder
    {
        return $builder->take(4);
    }

    public function toSearchableResponse(): SearchableResponse
    {
        return new SearchableResponse(
            group: 'Articles',
            title: $this->title,
            url: '/',
            preview: $this->text,
            image: $this->thumbnail,
        );
    }
}
```

4. Replace the `Search` component in Layout:

```php
protected function getSearchComponent(): ComponentContract
{
    return MoonShine\Scout\Components\Search::make();
}
```

---

## Query Tags (Quick Filters)

Query tags display as buttons above the table to apply predefined query scopes.

### Basic Usage

```php
use Illuminate\Database\Eloquent\Builder;
use MoonShine\Laravel\QueryTags\QueryTag;

protected function queryTags(): array
{
    return [
        QueryTag::make(
            'Post with author',
            fn(Builder $query) => $query->whereNotNull('author_id')
        ),
    ];
}
```

### With Icon

```php
QueryTag::make(
    'Post without author',
    fn(Builder $query) => $query->whereNull('author_id')
)->icon('users')
```

### Default Active Tag

```php
QueryTag::make('All posts', fn(Builder $query) => $query)
    ->default()
```

### Conditional Display

```php
QueryTag::make(
    'Post with author',
    fn(Builder $query) => $query->whereNotNull('author_id')
)->canSee(fn() => auth()->user()->moonshine_user_role_id === 1)
```

### Custom Alias

By default, the URL parameter is derived from the label. Override it:

```php
QueryTag::make(
    'Archived Posts',
    fn(Builder $query) => $query->where('is_archived', true)
)->alias('archive')
```

### Display as Dropdown

```php
class PostResource extends ModelResource
{
    protected bool $queryTagsInDropdown = true;
}
```

### Events on Query Tag

Fire custom JS events after data update:

```php
use MoonShine\Support\AlpineJs;
use MoonShine\Support\Enums\JsEvent;

QueryTag::make('QueryTag', fn($q) => $q)->events([
    AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'custom-fragment')
])
```

### Modify Button

```php
use MoonShine\UI\Components\ActionButton;

QueryTag::make('QueryTag', fn($q) => $q)
    ->modifyButton(fn(ActionButton $btn) => $btn)
```

---

## Query Builder Modification

### Modify All Queries

```php
use Illuminate\Contracts\Database\Eloquent\Builder;

protected function modifyQueryBuilder(Builder $builder): Builder
{
    return $builder->where('active', true);
}
```

To completely replace the query builder, override the `newQuery()` method.

### Modify Single Record Query

```php
protected function modifyItemQueryBuilder(Builder $builder): Builder
{
    return $builder->withTrashed();
}
```

To completely replace, override the `findItem()` method.

### Eager Loading

```php
class PostResource extends ModelResource
{
    protected array $with = ['user', 'categories'];
}
```

### Custom Sorting

```php
use Closure;

protected function resolveOrder(string $column, string $direction, ?Closure $callback): static
{
    if ($callback instanceof Closure) {
        $callback($this->newQuery(), $column, $direction);
    } else {
        $this->newQuery()->orderBy($column, $direction);
    }

    return $this;
}
```
