# MoonShine v3 Recipes — Forms & Tables

Practical code patterns for forms and table customization.

---

## Forms

### Form Events (Table Update + Form Reset)

On successful submission, refresh a table and reset the form.

```php
FormBuilder::make(route('form-table.store'))
    ->fields([
        Text::make('Title')
    ])
    ->name('main-form')
    ->async(events: [
        AlpineJs::event(JsEvent::TABLE_UPDATED, 'main-table'),
        AlpineJs::event(JsEvent::FORM_RESET, 'main-form')
    ]),

TableBuilder::make()
    ->fields([
        ID::make(),
        Text::make('Title'),
        Textarea::make('Body'),
    ])
    ->name('main-table')
    ->async()
```

Custom events from Blade:
```blade
<div x-data=""
     @defineEvent('form_updated', 'my-event', 'alert()')
>
</div>
```

```php
FormBuilder::make(route('form-table.store'))
    ->fields([Text::make('Title')])
    ->name('main-form')
    ->async(events: ['form_updated:my-event'])
```

### Fields Group via Template

Group fields that share common logic (e.g., populating a JSON field).

```php
Template::make('Config', 'config')->fields([
    Text::make('Language'),
    Text::make('Cache'),
])
    ->changeFill(fn(mixed $data) => data_get($data, 'config'))
    ->changeRender(fn(mixed $value, Template $ctx) => FieldsGroup::make($ctx->getPreparedFields())->fill($value))
    ->onApply(function(mixed $item, mixed $value) {
        $item->config = $value;
        return $item;
    })
```

### Mass Edit via Modal

Bulk edit rows using a modal with HiddenIds.

```php
public function massEdit(MoonShineRequest $request): MoonShineJsonResponse
{
    MoonshineUserRole::query()
        ->whereIn('id', $request->array('ids'))
        ->update(['name' => $request->input('name')]);

    return MoonShineJsonResponse::make()->toast('Success', ToastType::SUCCESS);
}

protected function indexButtons(): ListOf
{
    return parent::indexButtons()->add(
        ActionButton::make('')
            ->bulk()
            ->icon('pencil')
            ->inModal(
                'Mass edit',
                fn() => FormBuilder::make()
                    ->name('mass-edit')
                    ->fields([
                        HiddenIds::make($this->getListComponentName()),
                        Text::make('Name')->required(),
                    ])
                    ->asyncMethod('massEdit', events: [
                        AlpineJs::event(JsEvent::TABLE_UPDATED, $this->getListComponentName())
                    ])
                    ->submit('Save'),
            ),
    );
}
```

### HasMany with Parent ID

Use `ResourceWithParent` trait to access parent ID for file storage paths.

```php
use MoonShine\Laravel\Traits\Resource\ResourceWithParent;

class PostImageResource extends ModelResource
{
    use ResourceWithParent;

    protected string $model = PostImage::class;

    protected function getParentResourceClassName(): string
    {
        return PostResource::class;
    }

    protected function getParentRelationName(): string
    {
        return 'post';
    }

    protected function formFields(): iterable
    {
        return [
            ID::make(),
            BelongsTo::make('Post'),
            Image::make('Path')
                ->when(
                    $parentId = $this->getParentId(),
                    static fn(Image $image): string => $image->dir("post_images/$parentId")
                ),
        ];
    }
}
```

---

## Tables

### Custom Paginator for TableBuilder

Use `PaginatorCaster` to paginate data in standalone `TableBuilder` components.

```php
use MoonShine\Laravel\TypeCasts\PaginatorCaster;

protected function components(): iterable
{
    $posts = Post::query()->paginate(); // or ->simplePaginate() or ->cursorPaginate()

    $paginator = (new PaginatorCaster(
        $posts->appends(request()->except('page'))->toArray(),
        $posts->items()
    ))->cast();

    return [
        TableBuilder::make()
            ->fields([Text::make('Name')])
            ->items($paginator)
    ];
}
```

### updateOnPreview with Pivot Fields

Update pivot table values directly from the index page.

```php
protected function formFields(): iterable
{
    return [
        Grid::make([
            Column::make([
                ID::make()->sortable(),
                Text::make('Team title')->required(),
                Number::make('Team number'),
                BelongsTo::make('Tournament')->searchable(),
            ]),
            Column::make([
                BelongsToMany::make('Users')->fields([
                    Switcher::make('Approved')->updateOnPreview(
                        $this->getRouter()->getEndpoints()->method('updatePivot',
                            params: fn($data) => ['parent' => $data->pivot->tournamen_team_id]
                        )
                    ),
                ])->searchable(),
            ])
        ])
    ];
}

public function updatePivot(MoonShineRequest $request): MoonShineJsonResponse
{
    $item = TournamentTeam::query()->findOrFail($request->get('parent'));
    $column = (string) $request->str('field')->remove('pivot.');

    $item->users()->updateExistingPivot($request->get('resourceItem'), [
        $column => $request->get('value'),
    ]);

    return MoonShineJsonResponse::make()->toast('Success');
}
```
