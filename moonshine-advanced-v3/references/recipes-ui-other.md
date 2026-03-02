# MoonShine v3 Recipes — UI, Menu, Select Patterns & More

Practical code patterns for UI customization, menu configuration, select field patterns, and other advanced recipes.

---

## UI

### Custom Breadcrumbs from Resource

Override breadcrumbs on any page directly from the resource's `onLoad` method.

```php
class MoonShineUserResource extends ModelResource
{
    protected function onLoad(): void
    {
        parent::onLoad();

        $this->getFormPage()->breadcrumbs([
            '/custom' => 'Custom',
            '#' => $this->getTitle(),
        ]);
    }
}
```

### Relationship Fields in Tabs

Place HasMany/HasOne fields in separate tabs on a form page.

```php
final class ArticleFormPage extends FormPage
{
    protected function fields(): iterable
    {
        return [
            ID::make(),
            // ... main fields ...
            // IMPORTANT: Include relationship fields here so MoonShine can find them
            $this->getCommentsField(),
            $this->getCommentField(),
        ];
    }

    private function getCommentsField(): HasMany
    {
        return HasMany::make('Comments', resource: CommentResource::class)
            ->fillData($this->getResource()->getItem())
            ->async()
            ->creatable();
    }

    private function getCommentField(): HasOne
    {
        return HasOne::make('Comment', resource: CommentResource::class)
            ->fillData($this->getResource()->getItem())
            ->async();
    }

    protected function mainLayer(): array
    {
        return [
            Tabs::make([
                Tab::make('Basics', parent::mainLayer()),
                Tab::make('Comments', [
                    $this->getResource()->getItem()
                        ? $this->getCommentsField()
                        : 'To add comments, save the article',
                ]),
                Tab::make('Comment', [
                    $this->getResource()->getItem()
                        ? $this->getCommentField()
                        : 'To add comments, save the article',
                ]),
                Tab::make('Table', [
                    TableBuilder::make()
                        ->fields([ID::make(), Text::make('Text')])
                        ->cast(new ModelCaster(Comment::class))
                        ->items($this->getResource()->getItem()?->comments ?? [])
                ]),
            ]),
        ];
    }

    // Nullify bottomLayer to avoid duplicating relationship fields
    protected function bottomLayer(): array
    {
        return [];
    }
}
```

Important rules:
1. Do not create forms within forms (leads to conflicts).
2. Always fill fields with data and convert to the required type.
3. Relationship fields must appear in both `fields()` (for system discovery) and in the tabs.

### HasOne via Template Field

Implement a HasOne relationship using the Template field for custom rendering.

```php
protected function formFields(): iterable
{
    return [
        Template::make('Comment')
            ->changeFill(fn (Article $data) => $data->comment)
            ->changePreview(fn($data) => $data?->id ?? '-')
            ->fields(app(CommentResource::class)->getFormFields())
            ->changeRender(function (?Comment $data, Template $field) {
                $fields = $field->getPreparedFields();
                $fields->fill($data?->toArray() ?? []);
                return Components::make($fields);
            })
            ->onAfterApply(function (Article $item, array $value) {
                $item->comment()->updateOrCreate(
                    ['id' => $value['id']],
                    $value
                );
                return $item;
            })
    ];
}
```

### Saving Config to File via Template Field

```php
Template::make('Config', 'config')->fields([
    Text::make('Var'),
    Text::make('Bar'),
])
    ->changeFill(fn(mixed $data) => config('test'))
    ->changeRender(fn(mixed $value, Template $ctx) => FieldsGroup::make($ctx->getPreparedFields())->fill($value))
    ->onApply(function(mixed $item, mixed $value) {
        $content = str_replace(['array (', ')'], ['[', ']'], var_export($value, true));
        file_put_contents(config_path('test.php'), "<?php \n\nreturn $content;");
        return $item;
    })
```

---

## Menu

### Conditional Menu Items (Authorization)

#### Using Gate Facade

```php
use Illuminate\Support\Facades\Gate;
use MoonShine\Laravel\Enums\Ability;

protected function menu(): array
{
    return [
        MenuItem::make('Roles', MoonShineUserRoleResource::class)
            ->canSee(fn() => Gate::check(Ability::VIEW_ANY, MoonshineUserRole::class)),
    ];
}
```

#### Using Resource Abilities

```php
protected function menu(): array
{
    return [
        MenuItem::make('Roles', MoonShineUserRoleResource::class)
            ->canSee(fn(MenuItem $item) => $item->getFiller()->can(Ability::VIEW_ANY)),
    ];
}
```

#### Without Policy (Role Check)

```php
protected function menu(): array
{
    $menu = [
        MenuItem::make('Articles', ArticleResource::class),
    ];

    if (request()->user()->isSuperUser()) {
        $menu[] = MenuItem::make('Admins', MoonShineUserResource::class);
    }

    return $menu;
}
```

---

## Select Patterns

### Async Select Options

Load options asynchronously via `asyncMethod`:

```php
protected function formFields(): iterable
{
    return [
        Select::make('Select')->async(
            $this->getAsyncMethodUrl('selectOptions'),
        )->asyncOnInit(),
    ];
}

public function selectOptions(): MoonShineJsonResponse
{
    $options = new Options([
        new Option(label: 'Option 1', value: '1', selected: true,
            properties: new OptionProperty(image: 'https://example.com/img.png')),
        new Option(label: 'Option 2', value: '2',
            properties: new OptionProperty(image: 'https://example.com/img2.png')),
    ]);

    return MoonShineJsonResponse::make(data: $options->toArray());
}
```

### Reactive Select

```php
Select::make('Company', 'company')->options([
    1 => 'Laravel',
    2 => 'CutCode',
    3 => 'Symfony',
])->reactive(function (FieldsContract $fields, mixed $value, Select $ctx, array $values): FieldsContract {
    $fields->findByColumn('dynamic_value')?->options(
        (int) $value === 1 ? [4 => 4] : [2 => 2]
    );
    return $fields;
}),

Select::make('Dynamic value', 'dynamic_value')->options([4 => 4])->reactive(),
```

### Select with ShowWhen

```php
Select::make('Company', 'company')->options([
    1 => 'Laravel', 2 => 'CutCode', 3 => 'Symfony',
]),

Select::make('Dynamic value', 'dynamic_value')
    ->setNameAttribute('dynamic_value_1')
    ->showWhen('company', '1')
    ->options([1 => 1, 2 => 2]),

Select::make('Dynamic value', 'dynamic_value')
    ->setNameAttribute('dynamic_value_2')
    ->showWhen('company', '2')
    ->options([3 => 3, 4 => 4]),
```

### Select with onChangeMethod

```php
public function selectValues(): MoonShineJsonResponse
{
    $options = new Options([
        new Option('Option 1', '1', false, new OptionProperty(image: 'https://example.com/img.png')),
        new Option('Option 2', '2', true, new OptionProperty(image: 'https://example.com/img2.png')),
    ]);

    return MoonShineJsonResponse::make()->html(
        (string) Select::make('Next')->options($options)
    );
}

protected function formFields(): iterable
{
    return [
        Select::make('Select')->options([1 => 1, 2 => 2])
            ->onChangeMethod('selectValues', selector: '.next-select'),
        Div::make()->class('next-select'),
    ];
}
```

### Select with Fragments

```php
protected function formFields(): iterable
{
    $selects = [];
    $value = request()->integer('_data.first', 1);

    if ($value === 1) {
        $selects[] = Select::make('Second')->options([1 => 1, 2 => 2]);
    }
    if ($value === 2) {
        $selects[] = Select::make('Third')->options([1 => 1, 2 => 2]);
    }

    return [
        Fragment::make([
            Select::make('First')->options([1 => 1, 2 => 2])
                ->setValue($value)
                ->onChangeEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'selects')),
            ...$selects,
        ])->name('selects'),
    ];
}
```

---

## Other

### Async Remove on Click (Image)

```php
protected function formFields(): iterable
{
    return [
        Image::make('Avatar')
            ->removable(attributes: [
                'data-async-url' => $this->getRouter()->getEndpoints()->method(
                    'removeAvatar',
                    params: ['resourceItem' => $this->getItemID()]
                ),
                '@click.prevent' => <<<'JS'
                    fetch($event.target.closest('button').dataset.asyncUrl)
                        .then(() => $event.target.closest('.x-removeable').remove())
                JS
            ]),
    ];
}

public function removeAvatar(MoonShineRequest $request): void
{
    $item = $request->getResource()?->getItem();
    if (is_null($item)) { return; }
    $item->update(['avatar' => null]);
}
```

### Async Remove on Click (Json)

```php
protected function formFields(): iterable
{
    return [
        Json::make('Data')->fields([
            Text::make('Title'),
        ])->removable(attributes: [
            'data-async-url' => $this->getActivePage()
                ? $this->getRouter()->getEndpoints()->method(
                    'removeJsonData',
                    params: ['resourceItem' => $this->getItemID()]
                ) : null,
            '@click.prevent' => <<<'JS'
                fetch(`${$event.target.closest('a').dataset.asyncUrl}&index=${$event.target.closest('tr').rowIndex}`)
                    .then(() => remove())
            JS
        ]),
    ];
}

public function removeJsonData(MoonShineRequest $request): void
{
    $item = $request->getResource()?->getItem();
    $index = $request->integer('index') - 1;
    if (is_null($item)) { return; }

    $data = $item->data->toArray();
    unset($data[$index]);
    sort($data);
    $item->update(['data' => $data]);
}
```

### Multiple Fragments and Selectors

Update multiple DOM selectors from a single action:

```php
public function multipleSelectors(): MoonShineJsonResponse
{
    return MoonShineJsonResponse::make()->html([
        '.selector1' => 'here 1',
        '.selector2' => 'here 2',
    ]);
}

protected function components(): iterable
{
    return [
        ActionButton::make('Test')->method('multipleSelectors', selector: [
            '.selector1',
            '.selector2',
        ]),
        Div::make([])->class('selector1'),
        Div::make([])->class('selector2'),
    ];
}
```

Async update from multiple fragments:

```php
ActionButton::make('Fragments', $this->getRouter()->getEndpoints()->toPage($this, extra: [
    'fragment' => [
        '.selector1' => '_content1',
        '.selector2' => '_content2',
    ]
]))->async(selector: ['.selector1', '.selector2']),

Div::make([])->class('selector1'),
Div::make([])->class('selector2'),

Fragment::make([time()])->name('_content1'),
Fragment::make([time()])->name('_content2'),
```

### Changing Field Logic on the Fly

Store images in a related table by overriding `onApply`, `onAfterApply`, and `onAfterDestroy`:

```php
Image::make('Images', 'images')
    ->multiple()
    ->removable()
    ->changeFill(function (Model $data, Image $field) {
        return DB::table('images')->pluck('file');
    })
    ->onApply(function (Model $data): Model {
        // Block default onApply
        return $data;
    })
    ->onAfterApply(function (Model $data, false|array $values, Image $field) {
        if ($values !== false) {
            foreach ($values as $value) {
                DB::table('images')->insert([
                    'file' => $field->getApplyClass()->store($field, $value),
                ]);
            }
        }

        foreach ($field->toValue()->diff($field->getRemainingValues()) as $removed) {
            DB::table('images')->where('file', $removed)->delete();
            Storage::disk('public')->delete($removed);
        }

        return $data;
    })
    ->onAfterDestroy(function (Model $data, mixed $values, Image $field) {
        foreach ($values as $value) {
            Storage::disk('public')->delete($value);
        }
        return $data;
    })
```

### Authentication and Profile (MoonShine as User Portal)

Use MoonShine not as an admin panel but as a personal account under the `User` model with login, registration, password recovery, and profile pages. See the full recipe for:

- Custom routes in `routes/web.php` for auth, register, forgot, profile
- Two layouts: `AppLayout` (profile) and `FormLayout` (auth pages)
- Pages: `LoginPage`, `RegisterPage`, `ForgotPage`, `ResetPasswordPage`, `ProfilePage`
- Controllers: `AuthenticateController`, `ForgotController`, `RegisterController`, `ProfileController`
- FormRequest validation classes for each action

Key pattern -- rendering MoonShine pages from standard controllers:

```php
public function form(LoginPage $page): LoginPage
{
    return $page;
}
```

Setting layout per page:

```php
class LoginPage extends Page
{
    protected ?string $layout = FormLayout::class;
}
```
