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

    > **Note**  
    > Laravel uses a database naming convention named [Eloquent](https://laravel.com/docs/10.x/eloquent). Eloquent states that the connection table in a many to many relationship is named using the two table names (singular).

4. In the new migration file, change the schema to: 

    ```php
    Schema::create('entry_table', function (Blueprint $table) {
        $table->id();
        $table->foreignId('entry_id');
        $table->foreignId('topic_id');
    });
    ```
    
    > **Note**  
    > The foreign keys are named `<TABLENAME>_id`. Also note that we do not need timestamps in this table. 
    
### Model

We need a new model to define the `entries` table relationships and rules. The model scaffolding that Artisan provides is sufficient.

1. Create a new `Entry` model: 

    ```sh
    php artisan make:model Entry
    ```
    
2. You will now have a filed named `Entry.php` in the `/app/Models` folder. No changes needed.
    
### Factory

We need a factory to give Laravel instructions on how to populate the factory table. 

1. Creat a new factory:

    ```sh
    php artisan make:factory EntryFactory
    ```
    
2. You will now have a filed named `EntryFactory.php` in the `/database/factories` folder. Open this faile and change the definiation method retrn value to:

    ```php
    return [
        'title' => $this->faker->sentence,
        'content' => $this->faker->paragraph,
        'learned_at' => $this->faker->dateTimeThisMonth,
    ];
    ```
    
### Seeding

Lastly we need to give Laravel instructions on how many entries to add. 

1. Open up the `DatabaseSeeder.php` file in the `/database/seeders` folder.

2. At the top of the file add the entries model, add this line below `use App\ModelProject;`:

    ```php
    use App\Models\Entry;
    ```
    
3. Add some code to remove existing records when seeding it initiated. Add this below `Project::truncate();`:

    ```php
    Entry::truncate();
    ```
    
4. Add code to add four fake entries. Add this line under `Project::factory()->count(4)->create();`:

    ```php
    Entry::factory()->count(4)->create();
    ```
    
### Execute
    
Lastly we need to execute our migrations and seeding. Using the Terminal (or GitBash on a Windows machine) run this comment:

```sh
php artisan migrate:fresh --seed
```

![Migrate and Seed](_readme/screenshot-migrate-seed.png)

If you received no errors, there will be tables with data in your database. OPen up phpMyAdmin to check!

![Entries Table](_readme/screenshot-entries.png)


 - Make topics migration
 - Make Topic model
 - Make topic factory
 - Add to seeding
 
 - Make join table migration
 
 - Add belongsToMany methods
 
 - Make join table seeding
 
 
 
 - Add link to dashboard
 - Add routes





 - api routes
 
 
 
 




## List

Now that the database is ready and poulated, we need to create the list, add, edit, and delete code. Let's start by adding the list.

### Dashboard

Open up `dashboard.blade.php` in the `resources/views/console/` folder, and add link to manage entries. Add this line after types.

```php
<li><a href="/console/entries/list">Manage Journal Entries</a></li>
```

Open a browser, login to the CMS, and click `Manage Journal Entries`. You should get a `page not found` error. We need to add some new routes.

![Page Not Found](_readme/screenshot-404.png)

### Routes

Open the `web.php` file in the routes folder. Import the `EntriesController` by adding a `use` command to the top of your file.

```php
use App\Http\Controllers\EntriesController;
```

Copy and paste one of the list routes from one of the other modules and update for `entries`.

```php
Route::get('/console/entries/list', [EntriesController::class, 'list'])->middleware('auth');
```

Refresh your browser, and you will get a new error message that states `Controller EntriesController does not exist`.

![Controller Doees Not Exist](_readme/screenshot-controller.png)

### Controller and MEthod

Use the Laravel Artisan tool to make a new controller.

```sh
php artisan make:controller EntriesController
```

Refresh the browser and you will get a new error that states `Method EntriesController::list does not exist`.

![Method Does Not Exist](_readme/screenshot-method.png)

Open up the new `EntriesCopntroller.php` file. Add a `use` comment too import the Entry model.

```php
use App\Models\Entry;
```

Add a list method. This can be copied from one of the other modules and then adjusted for entries.

```php
public function list()
{
    return view('entries.list', [
        'entries' => Entry::all()
    ]);
}
```

Refresh the browser and you will get a new error that states `View [entries.list] not found.`.

![View Doees Not Exist](_readme/screenshot-view.png)
   
### Views

Lastly we need to create a view. In the `resources/views/` folder, create a new folder named `entries` and copy the `list.blade.php` file from one of the other modules, and adjust for entries.

```php
@extends ('layout.console')

@section ('content')

<section class="w3-padding">

    <h2>Manage Journal Entries</h2>

    <table class="w3-table w3-stripped w3-bordered w3-margin-bottom">
        <tr class="w3-red">
            <th>Title</th>
            <th>Date</th>
            <th></th>
            <th></th>
        </tr>
        @foreach ($entries as $entry)
            <tr>
                <td>{{$entry->title}}</td>
                <td>$entry->learned_at}}</td>
                <td><a href="/console/entries/edit/{{$entry->id}}">Edit</a></td>
                <td><a href="/console/entries/delete/{{$entry->id}}">Delete</a></td>
            </tr>
        @endforeach
    </table>

    <a href="/console/entries/add" class="w3-button w3-green">New Journal Entry</a>

</section>

@endsection
```

Final we can use the Carbon class to format the date. Switch the line that outputs the date.

```php
<td>{{$entry->learned_at}}</td>
```

With this.

```php
<td>{{\Carbon\Carbon::parse($entry->learned_at)->format('d/m/Y h:i A')}}</td>
```

Refresh your browser.

![Completed](_readme/screenshot-complete.png)

## Add, Edit and Delete

Using the projects module as a guide, create the add, edit, and delete pages for the entries module.
   
***

## Repo Resources

* [laravel-blade-cms](https://github.com/codeadamca/laravel-blade-cms)
* [Laravel](https://laravel.com/)

<a href="https://codeadam.ca">
<img src="https://codeadam.ca/images/code-block.png" width="100">
</a>

 
*** 

## Repo Resources

* [Visual Studio Code](https://code.visualstudio.com/)
* [Laravel](https://laravel.com/)

<a href="https://codeadam.ca">
<img src="https://codeadam.ca/images/code-block.png" width="100">
</a>
