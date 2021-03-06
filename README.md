# Repository Design Pattern for Laravel

[![StyleCI](https://styleci.io/repos/60332194/shield?style=flat)](https://styleci.io/repos/60332194)
[![Build Status](https://travis-ci.org/recca0120/laravel-repository.svg)](https://travis-ci.org/recca0120/laravel-repository)
[![Total Downloads](https://poser.pugx.org/recca0120/repository/d/total.svg)](https://packagist.org/packages/recca0120/repository)
[![Latest Stable Version](https://poser.pugx.org/recca0120/repository/v/stable.svg)](https://packagist.org/packages/recca0120/repository)
[![Latest Unstable Version](https://poser.pugx.org/recca0120/repository/v/unstable.svg)](https://packagist.org/packages/recca0120/repository)
[![License](https://poser.pugx.org/recca0120/repository/license.svg)](https://packagist.org/packages/recca0120/repository)
[![Monthly Downloads](https://poser.pugx.org/recca0120/repository/d/monthly)](https://packagist.org/packages/recca0120/repository)
[![Daily Downloads](https://poser.pugx.org/recca0120/repository/d/daily)](https://packagist.org/packages/recca0120/repository)

## Install

To get the latest version of Laravel Exceptions, simply require the project using [Composer](https://getcomposer.org):

```bash
composer require recca0120/laravel-repository
```

Instead, you may of course manually update your require block and run `composer update` if you so choose:

```json
{
    "require": {
        "recca0120/laravel-repository": "~1.2.0"
    }
}
```

## Methods

### Recca0120\Repository\Contracts\Repository

- match($criteria);
- get($criteria = [], $columns = ['*'], $limit = null, $offset = null);
- first($criteria = [], $columns = ['*']);
- count($criteria = []);
- paginate($criteria = [], $perPage = null, $columns = ['*'], $pageName = 'page', $page = null);
- find($id);
- create($attributes, $forceFill = false);
- update($attributes, $id, $forceFill = false);
- delete($id);
- cloneModel();
- newInstance($attributes = []);

## Usage

### Eloquent

#### Create a Model

Create your model normally, but it is important to define the attributes that can be filled from the input form data.

```php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $fillable = [
        'title',
        'author',
     ];
}
```

### Create a Contract

```php

namespace App\Repositories\Contracts;

interface PostRepository
{

}
```

#### Create a Repository

```php

namespace App\Repositories;

use App\Repositories\Contracts\PostRepository as PostRepositoryContract;
use App\Post;
use Recca0120\Repository\EloquentRepository;

class PostRepository extends EloquentRepository implements PostRepositoryContract
{
    public function __construct(Post $model)
    {
        $this->model = $model;
    }
}
```

### Bind

```php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Repositories\Contracts\PostRepository as PostRepositoryContract;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(PostRepositoryContract::class, PostRepository::class);
    }
}

```

#### Controller

```php

namespace App\Http\Controllers;

use App\Repositories\Contracts\PostRepository;

class PostsController extends Controller
{
    protected $repository;

    public function __construct(PostRepository $repository)
    {
        $this->repository = $repository;
    }
}
```

## Methods

Find all results in Repository

```php
$posts = $this->repository->get();
```

Find all results in Repository with pagination

```php
$posts = $this->repository->paginate();
```

Count results in Repository

```php
$posts = $this->repository->count();
```

Create new entry in Repository

```php
$post = $this->repository->create(request()->all());
```

Update entry in Repository

```php
$post = $this->repository->update(request()->all(), $id);
```

Delete entry in Repository

```php
$this->repository->delete($id);
```

New instance

```php
$post = $this->repository->newInstance([
    'author' => 'author'
]);
```

Return Model With Conditions

```
$model = $this->repository->match(Criteria::create()->where('title', '=', 'title'));
```

Find result by id

```php
$post = $this->repository->find($id);
```

### Find by conditions

#### Using the Criteria

Criteria is support all of Eloquent functions

##### Single Criteria

```php

use Recca0120\Repository\Criteria;

$criteria = Criteria::create()
    ->select('*')
    ->where('author', '=', 'author')
    ->orWhere('title', '=', 'title')
    ->orderBy('author', 'asc');

$this->repository->get($criteria);
$this->repository->paginate($criteria);
```

#### Multiple Criteria

```php

use Recca0120\Repository\Criteria;

$criteria = [];

$criteria[] = Criteria::create()
    ->orderBy('author', 'asc');

$criteria[] = Criteria::create()
    ->where('author', '=', 'author')
    ->orWhere('title', '=', 'title');

$this->repository->get($criteria);
// $this->repository->paginate($criteria);
```

##### With

```php

use Recca0120\Repository\Criteria;

$criteria = Criteria::create()
    ->with('author', function(Criteria $criteria) {
        return $criteria->where('author', '=', 'author');
    });

$this->repository->get($criteria);
// $this->repository->paginate($criteria);
```

#### Join

```php

use Recca0120\Repository\Criteria;

$criteria = Criteria::create()
    ->join('author', function (Criteria $criteria) {
        return $criteria->on('posts.author_id', '=', 'author.id');
    });

$this->repository->get($criteria);
// $this->repository->paginate($criteria);
```

#### Expression

```php

use Recca0120\Repository\Criteria;

$criteria = Criteria::create()
    ->where('created_at', '<=', Criteria::expr('NOW()'));

$this->repository->get($criteria);
// $this->repository->paginate($criteria);
```

## Find results by array

```php

use Recca0120\Repository\Criteria;

$posts = $this->repository->get([
    'author' => 'author',
    ['title', '=', 'title'],
    Criteria::create()
        ->where('created_at', '<=', Criteria::expr('NOW()'))
]);
```

#### Custom Criteria

```php

use Recca0120\Repository\Criteria;

class CustomCriteria extends Criteria
{
    public function __construct($id)
    {
        $this->where('id', '=', $id);
    }
}

$this->repository->get(CustomCriteria::create(1));
$this->repository->get((new CustomCriteria::create(2))->where('autor', '=', 'autor'));
```


### Illuminte\Support\Collection

#### Create a Collection

You can use Collection to be your model

```php

namespace App;

use Illuminate\Support\Collection;

class Post extends Collection
{
    public static function make($items = [])
    {
        // it will cast to \Illuminate\Support\Fluent;
        return parent::make([
            [
                'id' => 1,
                'title' => 'title',
                'autor' => 'author'
            ],
            [
                'id' => 2,
                'title' => 'title2',
                'autor' => 'author2'
            ],
        ]);
    }
}
```

#### Support Criteria

- where
- orderBy
