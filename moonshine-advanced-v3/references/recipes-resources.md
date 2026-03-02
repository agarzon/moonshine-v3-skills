# MoonShine v3 Recipes — Resources

Practical code patterns for resource customization.

---

### Reorderable Resource

Drag-and-drop row reordering. Only suitable for small datasets without pagination.

```php
use MoonShine\Contracts\UI\ComponentContract;
use MoonShine\Laravel\MoonShineRequest;

protected string $sortColumn = 'position';
protected SortDirection $sortDirection = SortDirection::ASC;

/**
 * @param TableBuilder $component
 */
public function modifyListComponent(ComponentContract $component): ComponentContract
{
    return $component->reorderable(
        $this->getAsyncMethodUrl('reorder')
    );
}

public function reorder(MoonShineRequest $request): void
{
    if ($request->str('data')->isNotEmpty()) {
        $request->str('data')->explode(',')->each(
            fn($id, $position) => $this->getModel()
                ->where('id', $id)
                ->update(['position' => $position + 1]),
        );
    }
}
```

With a drag handle column:

```php
protected function fields(): iterable
{
    return [
        Preview::make(
            column: '__handle',
            formatted: static fn () => Icon::make('bars-4'),
        )->customWrapperAttributes(['class' => 'handle', 'style' => 'cursor: move']),
        // ... other fields
    ];
}

public function modifyListComponent(ComponentContract $component): ComponentContract
{
    return $component->reorderable(
        $this->getAsyncMethodUrl('reorder')
    )->customAttributes([
        'data-handle' => '.handle',
    ]);
}
```

### Soft Deletes

Full soft delete support with restore, force delete, and query tags.

Model setup:
```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Article extends Model
{
    use SoftDeletes;
}
```

Resource implementation:
```php
use MoonShine\Laravel\QueryTags\QueryTag;

protected function indexButtons(): ListOf
{
    return parent::indexButtons()
        ->prepend(
            ActionButton::make('Restore')
                ->method('restore', events: [$this->getListEventName()])
                ->canSee(fn(Article $model) => $model->trashed()),

            ActionButton::make('Force delete')
                ->method('forceDelete', events: [$this->getListEventName()])
                ->canSee(fn(Article $model) => $model->trashed()),
        );
}

protected function queryTags(): array
{
    return [
        QueryTag::make('Deleted', static fn(Builder $q) => $q->onlyTrashed())
    ];
}

protected function modifyItemQueryBuilder(Builder $builder): Builder
{
    return $builder->withTrashed();
}

public function restore(MoonShineRequest $request): MoonShineJsonResponse
{
    $item = $request->getResource()->getItem();
    $item->restore();
    return MoonShineJsonResponse::make()->toast('Success');
}

public function forceDelete(MoonShineRequest $request): MoonShineJsonResponse
{
    $item = $request->getResource()->getItem();
    $item->forceDelete();
    return MoonShineJsonResponse::make()->toast('Success');
}

protected function modifyDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->canSee(fn(Article $model) => !$model->trashed());
}

protected function modifyMassDeleteButton(ActionButtonContract $button): ActionButtonContract
{
    return $button->canSee(fn() => request()->input('query-tag') !== 'deleted');
}
```

### Index Page as Cards (CardsBuilder)

Replace the table on the index page with cards.

```php
class MoonShineUserResource extends ModelResource
{
    public function getListEventName(?string $name = null, array $params = []): string
    {
        $name ??= $this->getListComponentName();
        return AlpineJs::event(JsEvent::CARDS_UPDATED, $name, $params);
    }

    public function modifyListComponent(ComponentContract $component): ComponentContract
    {
        return CardsBuilder::make($this->getItems(), $this->getIndexFields())
            ->cast($this->getCaster())
            ->name($this->getListComponentName())
            ->async()
            ->overlay()
            ->title('email')
            ->subtitle('name')
            ->url(fn ($user) => $this->getFormPageUrl($user->getKey()))
            ->thumbnail(fn ($user) => asset($user->avatar))
            ->buttons($this->getIndexButtons());
    }
}
```
