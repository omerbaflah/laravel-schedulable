![Laravel Schedulable Logo](https://github.com/neelkanthk/repo_logos/blob/master/LaravelSchedulable_small.png?raw=true)

![](https://img.shields.io/github/v/release/neelkanthk/laravel-schedulable?style=for-the-badge)
![](https://img.shields.io/packagist/php-v/neelkanthk/laravel-schedulable.svg?style=for-the-badge)
![](https://img.shields.io/badge/Laravel-%3E%3D6.0-red?style=for-the-badge)
![](https://img.shields.io/badge/Tests-Passing-green?style=for-the-badge)
![](https://img.shields.io/github/issues/neelkanthk/laravel-schedulable?style=for-the-badge)
![](https://img.shields.io/github/license/neelkanthk/laravel-schedulable?style=for-the-badge)

# Laravel Schedulable [![Twitter](https://img.shields.io/twitter/url?style=social&url=https%3A%2F%2Fgithub.com%2Fneelkanthk%2Flaravel-schedulable)](https://twitter.com/intent/tweet?text=Laravel%20Schedulable:&url=https%3A%2F%2Fgithub.com%2Fneelkanthk%2Flaravel-schedulable)

## Salient Features:

1. __Turn any Eloquent Model into a schedulable__ one by using ```Schedulable``` trait in the model.

2. __Schedule Models to a time in future__ and they will be returned in query results at specified date and time.

3. __Reschedule__ and __Unschedule__ at any time using simple methods.

4. Hook into the model's life cycle via __custom model events__ provided by the package.

5. __Override the default column name__ and use your own custom column name.


__*Some example use cases when this package can be useful:*__

1. A Blog like application which allows bloggers to schedule their post to go public on a future date and time.

2. An E-commerce website where the items in the inventory can be added at any time from the admin panel but they can be scheduled to be made available to the customers at a particular date and time.

## Minimum Requirements

1. Laravel 6.0  
2. PHP 7.2

## Installation  

```bash
composer require neelkanthk/laravel-schedulable
```

## Usage

#### 1. Create a migration to add ```schedule_at``` column in any table using package's ```scheduleAt();``` method which creates a column with name ```schedule_at```.  

#### *NOTE:* If you want to use any other column name then simply use the ```$table->timestamp('column_name');``` method as shown below in examples.

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddScheduleAtColumnInPosts extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->scheduleAt(); //Using default schedule_at column
			//or
            $table->timestamp('publish_at', 0); //Using custom column name
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->dropColumn('schedule_at'); //Using default schedule_at column
            //or
            $table->dropColumn('publish_at'); //Using custom column name
        });
    }
}
```

#### 2. Use the ```Neelkanth\Laravel\Schedulable\Traits\Schedulable``` trait in any Model.  

#### *NOTE:* If you have used a custom column name in the migration then you have to specify that column in the Model as shown below.

```php
use Illuminate\Database\Eloquent\Model;
use Neelkanth\Laravel\Schedulable\Traits\Schedulable;

class Post extends Model
{
    use Schedulable;
    
    const SCHEDULE_AT = "publish_at"; //Specify the custom column name
}
```

## Usage

### 1. Scheduling a model

```php
$scheduleAt = Carbon::now()->addDays(10); //Carbon is just an example. You can pass any object which is implementing DateTimeInterface.
$post = new Post();
//Add values to other attributes
$post->scheduleWithoutSaving($scheduleAt); // Modifies the schedule_at attribute and returns the current model object without saving it.
$post->schedule($scheduleAt); //Saves the model in the database and returns boolean true or false
```

### 2. Unscheduling a model

```php
$post = Post::find(1);
$post->unscheduleWithoutSaving(); // Modifies the schedule_at attribute and returns the current model object without saving it.
$post->unschedule(); //Saves the model in the database and returns boolean true or false
```

### 3. Events and Observers

The package provides four model events and Observer methods which the developers can use to hook in the model's lifecycle.

The ```schedule()``` method fires two events namely ```scheduling``` before saving the model and ```scheduled``` after saving the model.

The ```unschedule()``` method fires two events namely ```unscheduling``` before saving the model and ```unscheduled``` after saving the model.

The above events can be caught in the Observer class as follows:

```php
namespace App\Observers;

use App\Post;

class PostObserver
{
    public function scheduling(Post $post)
    {
        //
    }

    public function scheduled(Post $post)
    {
        //
    }

    public function unscheduling(Post $post)
    {
        //
    }

    public function unscheduled(Post $post)
    {
        //
    }
}
```

### 4. Fetching data using queries

We will assume below posts table as reference to the following examples:

| id | title        | created_at          | updated_at | schedule_at         |
|----|--------------|---------------------|------------|---------------------|
| 1  | Toy Story 1  | 2020-06-01 12:15:00 | NULL       | NULL                |
| 2  | Toy Story 2  | 2020-08-02 16:10:12 | NULL       | 2020-08-10 10:10:00 |
| 3  | Toy Story 3  | 2020-10-10 10:00:10 | NULL       | 2020-10-20 00:00:00 |
| 4  | Terminator 2 | 2020-10-12 00:00:00 | NULL       | 2020-10-22 15:10:17 |

Suppose the current timestamp is 2020-10-15 00:00:00.

#### 1. Default

By default all those models are fetched in which the ```schedule_at``` column is having ```NULL``` value or a timestamp less than or equal to the current timestamp.

So a eloquent query 
```php
$posts = Post::get();
``` 
will return Toy Story 1 and Toy Story 2


#### 2. Retrieving scheduled models in addition to the normal.

To retrieve scheduled models in addition to the normal models, use the ```withScheduled()``` method.

```php
$posts = Post::withScheduled()->get();
```

The above query will return all the four rows in the above table.

#### 3. Retrieving only scheduled models without normal.

To retrieve only scheduled models use the ```onlyScheduled()``` method.

```php
$posts = Post::onlyScheduled()->get();
```

The above query will return Toy Story 3 and Terminator 2.


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## Security
If you discover any security-related issues, please email me.neelkanth@gmail.com instead of using the issue tracker.

## Credits

- [Neelkanth Kaushik](https://github.com/username)
- [All Contributors](../../contributors)

## License
[MIT](https://choosealicense.com/licenses/mit/)