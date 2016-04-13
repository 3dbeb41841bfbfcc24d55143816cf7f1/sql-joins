# SQL Joins

## Learning Objectives.
- Define SQL joins.
- Explain what SQL joins and how to use them.
- Use the different type of joins to display records.
- Shopping datbase example


## Opening Framing (5 / 5)
An SQL JOIN clause is used to combine rows from two or more tables, based on a common field between them.

The most common type of join is: SQL INNER JOIN (simple join). An SQL INNER JOIN returns all rows from multiple tables where the join condition is met.

### Information Dive (5 / 10)
For the next 5 minutes, research SQL joins are

[SQL Joins](http://www.w3schools.com/sql/sql_join.asp)

![SQL joins diagram](Visual_SQL_JOINS_orig.jpg))

### T&T (5 / 15)
Now, turn & talk to your neighbor and discuss:

1. At a high level, what are ORM's and how might they be useful?
2. What is the importance of interfacing the server with the database?

## ORM's & Active Record (10 / 25)
- *Official* wikipedia definition. A programming technique for converting data between incompatible type systems in object-oriented programming languages.

> what does that really mean?

We need a way to encapsulate our databases into objects that we can talk to on our server. ORM's serve that purpose. Remember those tables we created in SQL? Well, its an object represented on our server now. That's what ORM's do.

More concretely ORM's:
- 'Map' (translate) objects to rows in our DB (and vice versa)
- **Conventions**:
  - 1 table per Model/Class/Entity
  - table name is model name pluralized
  - each column is an attribute for that model
- Table associations are handled using foreign keys

It just so happens you will be learning one of the best ORM's on the market. It has some of the best documentation and best syntax(because ruby is awesome) the industry has to offer. This ORM is Active Record.

> Active Record is the M in MVC - the model - which is the layer of the system responsible for representing business data and logic. Active Record facilitates the creation and use of business objects whose data requires persistent storage to a database. It is an implementation of the Active Record pattern which itself is a description of an Object Relational Mapping system. - Taken from AR docs

## Active Record
### Convention Over configuration (ST-WG - 10 / 35)
Before we get started with code, I want to highlight a reoccurring theme with Active Record and Rails in general. You'll often here us say Convention over Configuration.

**Question:**  Without getting into the specifics of AR, what do you think we mean by convention over configuration?

Basically Active Record and Rails, and other frameworks have a whole bunch of conventions that they follow so that you do not have to mess with different configuration details later. These conventions exist because developers agree on best practices, and therefore allows us to spend less time trying to configure, when there all ready is an accepted way to do things. Some of the common ones we will encounter are naming conventions such as: plural vs single, capital or lower, camel or snake.

**BOARD:** Throughout this lesson, I will write on the board Active Record's conventions so we can list them as we go.

In a nutshell, if you don't follow the conventions, you're going to have a bad time.

Alright! Let's get started with some code!

### Setup SQL - WDI (I Do - 5 / 40)
> Throughout the day, I'll be doing some code simulating a "wdi application" then you will code along with our Tunr
applications.

Let's go over our domain model for both applications.

![ERDs](./active-record.png)




I want to be able to do CRUD for these models with Active Record. We'll be going into greater detail about how we are going to use Active Record as an interface between our server and our database, but to start, the first thing that I want to do is create/setup a database.

First let's create our database and create our schema file in the terminal:

```bash
# in ~/wdi/sandbox
$ mkdir ar_wdi
$ cd ar_wdi
$ mkdir db
$ touch db/wdi_schema.sql
$ createdb wdi_db
```

> All we did here was create a database in PSQL. If you ever want to drop a database the command is `$ dropdb <database_name>` (VERY DANGEROUS)

Next, I want to update the schema file and then load a table for our model into the database:

in `db/wdi_schema.sql` file:

```sql
DROP TABLE IF EXISTS instructors;
DROP TABLE IF EXISTS students;

CREATE TABLE instructors (
  id SERIAL PRIMARY KEY,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  age INT NOT NULL
);

CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  age INT NOT NULL,
  job TEXT,
  instructor_id INT
);
```

now lets run our `wdi_schema.sql` file in the terminal:

```bash
$ psql -d wdi_db < db/wdi_schema.sql
```

**Question (ST-WG):** Why did we do this? Why not just go into psql and create the tables in postgres?

It'll be nice going forward with your application that we package the schema up so that its modular. We can have others quickly pick up our code and have our exact database setup.

### Setup SQL - Tunr (You Do - 10 / 50)


### Setup Ruby - WDI (I Do - 10 / 60)
Great, now we have a table loaded into our database we're now ready to get started on the ruby side.
Let's first create all the directories/files we're going to need in the terminal:

```bash
$ touch app.rb
$ touch Gemfile
$ mkdir models
$ touch models/student.rb
```

> The `app.rb` file is our main application file. This is where a lot of the main program logic will live.

> The `Gemfile` will contain all the dependencies for our program.

> The `models/student.rb` file will contain the class definition for the Student class that will represent the students table in SQL

> by convention we always name our model file names singular

In the `Gemfile`:

```ruby
source 'https://rubygems.org'

gem "pg"  # this gem allows ruby to talk to postgres
gem "activerecord"  # this gem provides a connection between your ruby classes to relational database tables
gem "pry"  # this gem allows access to REPL
```

Then I'm going to run `$ bundle install` in the terminal.

### Break (10 / 80)

### Functionality - WDI (I Do - 20 / 100)

In the `models/student.rb` file, let's define our student model:

```ruby
class Student < ActiveRecord::Base
  # AR classes are singular and capitalized by convention
end
```

> In this ruby file, we create a class of Student that inherits from `ActiveRecord::Base`. Essentially, when we inherit from `ActiveRecord::Base`, it gives this class a whole bunch of functionality that maps the ruby `Student` class to the `students` table in postgres.

Now, lets create a file that handles the connection between our application and our database:

```bash
# Add a connection.rb file to our db directory
$ touch db/connection.rb
```

In `db/connection.rb`:
```ruby
ActiveRecord::Base.establish_connection(
  :adapter => "postgresql",
  :database => "wdi_db"
)
```

Finally, let's build out the functionality of the `app.rb` file(which in this case is just dropping us into pry):

```ruby
require "bundler/setup" # require all the gems we'll be using for this app from the Gemfile. Obviates the need for `bundle exec`
require "pg" # postrgres db library
require "active_record" # the ORM
require "pry" # for debugging

require_relative "db/connection" # require the db connection file that connects us to PSQL, prior to loading models
require_relative "models/student" # require the Student class definition that we defined in the models/student.rb file

# This will put us into a state of the pry REPL, in which we've established a connection
# with the wdi database and have defined the Student Class as an
# ActiveRecord::Base class.
binding.pry

```

> note the difference between `require` and `require_relative`. With `require` we are getting gems and `require_relative` we are getting files relative to the location of the file we wrote `require_relative` in


### Methods - WDI (I Do - 30 / 140)

Great! We've got everything done that we need to get setup with single model CRUD in our application. Let's run it in the terminal:

```bash
$ ruby app.rb
```

When we run this app, we can see that it drops us into pry. Let's write some code in pry to update our database... **IN REALTIME!!!**

**Board:** I want to come up with an ongoing list of class and instance methods that we can add to as we use them

Let's create an instance of the Student object on the ruby side, but that does not save originally:

> **Note** the syntax for creating a new instance.

```ruby
george = Student.new(first_name: "George", last_name: "Washington", age: 270, job: "Prez")
```

To save our instance to the database we use `.save`:

```ruby
george.save
```

If we want to initialize an instance of an object AND save it to the database we use `.create`:

```ruby
abe = Student.create(first_name: "Abe", last_name: "Lincoln", age: 150, job: "Prez")
```

One really handy feature we get from an Active Record inherited class is that all of the attribute columns of our model are now `attr_accessor`'s as well. So we can do things like:

```ruby
george.first_name
# gets the first name of george
george.first_name = "Jorge"
# sets the first name to "Jorge"
george.save
# saves the model to the database
```

> **Note:** Should be noted that when we set the attribute using a setter, it doesn't automatically save to the database, its not until we call `.save` on the object that it saves to the database

To get all of the students we use `.all`:

```ruby
Student.all
```

We can also find a student by its ID using `.find`:

```ruby
Student.find(1)
```

Additionally we can also find a student by an attribute using `.find_by`:

```ruby
Student.find_by(last_name: "Washington")
```

> Note that find_by only returns the first object that meets the requirements of the arguments

If you want all students that meet a certain query then we use `.where`:

```ruby
Student.where(last_name: "Washington")
```

We can also update attributes and save at the same time using the `.update` method:

```ruby
george = Student.find_by(last_name: "Washington")
george.update(job: "actor", first_name: "Denzel")
```

Finally we can also just destroy/delete rows in our database with the `.destroy` method:

```ruby
george = Student.find_by(last_name: "Washington")
george.destroy
# goodbye george you're gone forever
```

> This is exciting stuff by the way, imagine, while we do these things, that our students model is instead a post on facebook, or a comment on facebook. So the next time you comment on someone's facebook page you have an idea now of whats happening on the database layer. Maybe not the whole picture, but you have an idea. We're going to build on that idea in the coming week and half, and thats really exciting.


