# Laravel Types Module

This repo provides instrucitons for the addition of a topics module for a Laravel CMS. This new module includes a mant to many relationship with the journal module. 

## Database

Before we start coding the topics list and the entry add files, we need a table to store our journal topics. Creating a table with testing data requires us to create a migration, model, factory, and add some instructions to our seeding script. 

### Migrations

Before we make any files, use the Terminal to change the working directory to your project directory. I am using a Mac and my project folder was on my desktop:

``` sh
cd ~
cd Desktop
cd laravel-blade-cms
```

1. Using the Laravel Artisan tool, create a new migration file:

    ```sh
    php artisan make:migration create_topics_table
    ```
    
    You will now have a new migration file named `<YEAR>_<MONTH>_<DAY>_<SECONDS>_create_topics_table.php`.

2. In the new migration file, change the schema to:

    ```php
    Schema::create('topics', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->timestamps();
    });
    ```
    
3. We also need a table to store the connections between the `topics` and the `entries`. Create one more migration:
 
    ```sh
    php artisan make:migration create_entry_topic_table
    ```

> [!Note]  
> Laravel uses a database naming convention named [Eloquent](https://laravel.com/docs/10.x/eloquent). Eloquent states that the connection table in a many to many relationship is named using the two table names (singular).

4. In the new migration file, change the schema to: 

    ```php
    Schema::create('entry_table', function (Blueprint $table) {
        $table->id();
        $table->foreignId('entry_id');
        $table->foreignId('topic_id');
    });
    ```
    
> [!Note]  
> The foreign keys are named `<TABLENAME>_id`. Also note that we do not need timestamps in this table. 
    
### Model

We need a new model to define the `topics` table relationships and rules. The model scaffolding that Artisan provides is sufficient.

1. Create a new `Topic` model: 

    ```sh
    php artisan make:model Topic
    ```
    
2. You will now have a filed named `Topic.php` in the `/app/Models` folder. No changes needed.

3. We also need to add the relationships to the `Entry.php` and `Topic.php` models. Add this method to the `Entry.php` file just after the `use HasFactory;` line:

    ```php
    public function topics()
    {
        return $this->belongsToMany(Topic::class);
    }
    ```
    
    And this method to the `Topic.php` file:
    
    ```php
    public function entries()
    {
        return $this->belongsToMany(Entry::class);
    }

> [!Note]  
> We do not need a model for the `entry_topic` table.
    
### Factory

We need a factory to give Laravel instructions on how to populate the `topics` table. 

1. Creat a new factory:

    ```sh
    php artisan make:factoryTopicFactory
    ```
    
2. You will now have a filed named `TopicFactory.php` in the `/database/factories` folder. Open this file and change the definiation method return value to:

    ```php
    return [
        'title' => $this->faker->word,
    ];
    ```
    
### Seeding

Lastly we need to give Laravel instructions on how many topics and connections to add. 

1. Open up the `DatabaseSeeder.php` file in the `/database/seeders` folder.

2. At the top of the file add the entries model, add this line below `use App\ModelEntry;`:

    ```php
    use App\Models\Topic;
    ```
    
3. Add some code to remove existing topics when seeding is initiated. Add this below `Project::truncate();`:

    ```php
    Topic::truncate();
    ```
    
    > **Note**  
    > We do not need to seed or truncate the `entry_topic` table.
    
4. Add code to add four fake topics. Add this line before `Project::factory()->count(4)->create();`:

    ```php
    Topic::factory()->count(4)->create();
    ```
    
5. Finally we need to add a few connections to the entries. Change the line of code that adds entries to:

    ```php
    Entry::factory()->count(4)->create()->each(function($entry){
        $topics = Topic::all()->random(rand(1,2) )->pluck('id');
        $entry->topics()->attach($topics);
    });
    ```
    
    This code will add four entries and then attach one to three random topics. 
    
### Execute
    
Lastly we need to execute our migrations and seeding. Using the Terminal (or GitBash on a Windows machine) run this comment:

```sh
php artisan migrate:fresh --seed
```

If you received no errors, there will be tables with data in your database. Open up phpMyAdmin to check!
 
## List

Now that the database is ready and populated, we need to create the topics list and entry add code. Let's start by adding the list.

### Dashboard

Open up `dashboard.blade.php` in the `resources/views/console/` folder, and add link to manage topics. Add this line after entries.

```php
<li><a href="/console/topics/list">Manage Topics</a></li>
```

Open a browser, login to the CMS, and click `Manage Topics Entries`. You should get a `page not found` error. We need to add some new routes.

### Routes

Open the `web.php` file in the routes folder. Import the `TopicsController` by adding a `use` command to the top of your file.

```php
use App\Http\Controllers\TopicsController;
```

Copy and paste one of the list routes from one of the other modules and update for `topics`.

```php
Route::get('/console/topics/list', [TopicsController::class, 'list'])->middleware('auth');
```

Also add a route for the entries add form:

```php
Route::get('/console/entries/add', [EntriesController::class, 'addForm'])->middleware('auth');
Route::post('/console/entries/add', [EntriesController::class, 'add'])->middleware('auth');
```

Refresh your browser, and you will get a new error message that states `Controller TopicsController does not exist`.

### Controller and Method

Use the Laravel Artisan tool to make a new controller.

```sh
php artisan make:controller TopicsController
```

Refresh the browser and you will get a new error that states `Method TopicsController::list does not exist`.

Open up the new `TopicsCopntroller.php` file. Add a `use` command to import the Topic model.

```php
use App\Models\Topic;
```

Add a list method. This can be copied from one of the other modules and then adjusted for entries.

```php
public function list()
{
    return view('topics.list', [
        'topics' => Topic::all()
    ]);
}
```

Refresh the browser and you will get a new error that states `View [entries.list] not found.`.
   
### Views

Lastly we need to create a view. In the `resources/views/` folder, create a new folder named `topics` and copy the `list.blade.php` file from one of the other modules, and adjust for topics. The `types.lisst` view is probably the most similar. 

```php
@extends ('layout.console')

@section ('content')

<section class="w3-padding">

    <h2>Manage Topics</h2>

    <table class="w3-table w3-stripped w3-bordered w3-margin-bottom">
        <tr class="w3-red">
            <th>Name</th>
            <th></th>
            <th></th>
        </tr>
        <?php foreach($topics as $topic): ?>
            <tr>
                <td>{{$topic->title}}</td>
                <td><a href="/console/topics/edit/{{$topic->id}}">Edit</a></td>
                <td><a href="/console/topics/delete/{{$topic->id}}">Delete</a></td>
            </tr>
        <?php endforeach; ?>
    </table>

    <a href="/console/topics/add" class="w3-button w3-green">New Topic</a>

</section>

@endsection
```

Refresh your browser.

## Add

If you haven't already, create a `topics.add` file in your views. Use the `types.add` view as a guide.

> [!Warning]  
> This repo still has a last few steps. Coming soon!
   
***

## Repo Resources

* [laravel-blade-cms](https://github.com/codeadamca/laravel-blade-cms)
* [Laravel](https://laravel.com/)

<a href="https://codeadam.ca">
<img src="https://codeadam.ca/images/code-block.png" width="100">
</a>
