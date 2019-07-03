# Выполнить сортировку(упорядочивание) по связующему полю Eloquent

Вы могли бы сделать сортировку в [ `whereHas` ](https://laravel.com/docs/5.5/eloquent-relationships#querying-relationship-existence) 

```php
// Retrieve all products with at least one variant ordered by price

$query = Product::whereHas('variants', function ($query) use ($request) {
    if ($request->input('sort') == 'price') {
        $query->orderBy('variants.price', 'ASC');
    }
})
->with('reviews')
```

По сути, ваш подзапрос будет: `select * from variants where product_id in (....) order by price` , и это не то, что вы хотите, верно?

```php
<?php 
// ...

$order = $request->sort;

$products = Product::whereHas('variants')->with(['reviews',  'variants' => function($query) use ($order) {
  if ($order == 'price') {
    $query->orderBy('price');
  }
}])->paginate(20);

```

Если вы хотите отсортировать товар + / или вариант, вам нужно использовать join.

```php
$query = Product::select([
          'products.*',
          'variants.price',
          'variants.product_id'
        ])->join('variants', 'products.id', '=', 'variants.product_id');

if ($order === 'new') {
    $query->orderBy('products.created_at', 'DESC');
}

if ($order === 'price') {
    $query->orderBy('variants.price');
}

return $query->paginate(20);
```

**********
[Eloquent](/tags/Eloquent.md)
