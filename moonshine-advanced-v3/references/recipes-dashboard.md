# MoonShine v3 Recipes — Dashboard

Practical code patterns for dashboard customization.

---

### Async Metrics with Date Filters

Use Fragment + FormBuilder to create metrics that update asynchronously when date filters change.

```php
protected function components(): iterable
{
    $startDate = request()->date('_data.start_date');
    $endDate = request()->date('_data.end_date');

    return [
        FormBuilder::make()
            ->dispatchEvent(AlpineJs::event(JsEvent::FRAGMENT_UPDATED, 'metrics'))
            ->fields([
                Flex::make([
                    Date::make('Start date'),
                    Date::make('End date'),
                ]),
            ]),

        Fragment::make([
            FlexibleRender::make("$startDate - $endDate"),

            LineChartMetric::make('Orders')
                ->line([
                    'Profit' => Order::query()
                        ->selectRaw('SUM(price) as sum, DATE_FORMAT(created_at, "%d.%m.%Y") as date')
                        ->whereBetween('created_at', [$startDate, $endDate])
                        ->groupBy('date')
                        ->pluck('sum', 'date')
                        ->toArray(),
                ])
                ->line([
                    'Avg' => Order::query()
                        ->selectRaw('AVG(price) as avg, DATE_FORMAT(created_at, "%d.%m.%Y") as date')
                        ->whereBetween('created_at', [$startDate, $endDate])
                        ->groupBy('date')
                        ->pluck('avg', 'date')
                        ->toArray(),
                ], '#EC4176'),
        ])->name('metrics'),
    ];
}
```

### Dashboard Settings Form

Save settings directly from a dashboard page using `asyncMethod`.

```php
private function getSetting(): Setting
{
    return Setting::query()->find(1);
}

public function store(): MoonShineJsonResponse
{
    $this->form()->apply(fn(Setting $item) => $item->save());

    return MoonShineJsonResponse::make()->toast('Saved');
}

private function form(): FormBuilder
{
    return FormBuilder::make()
        ->asyncMethod('store')
        ->fillCast($this->getSetting(), new ModelCaster(Setting::class))
        ->fields([
            // Your settings fields here
        ]);
}

protected function components(): iterable
{
    yield $this->form();
}
```
