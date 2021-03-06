## Short introduction to Ruby on Rails (4.x)

This text is suppose to give a short introduction to publishing a Ruby on Rails (RoR) web application. Ruby is a program language and Rails is a framework and together they are some of the most popular web application frameworks used today (when analyzing [Github](https://www.github.com) and [Stackoverflow](http://stackoverflow.com/)). This text isn´t for those that are trying to learn the programming language Ruby or how to build applications with Ruby on Rails. There are good resources you can check up for that:

* [Ruby on Rails documentation](http://rubyonrails.org/)
* [Rails for Zombies](http://railsforzombies.org/)
* [Codecademy](https://www.codecademy.com/learn/learn-rails)

This text will give some hint for you that are trying to publish a RoR-application. Of course it is not a full coverage of this subject but should count as a start point. Things like; what commands to use, how the RoR-application works in development vs. production and so on.

### Ruby gems
The Ruby platform gives the developers the opportunity to use and develop smaller modules to be used in a application. These "modules" is called Gems. A RoR-application is using gems when building the application, for example the gem named "rails". All the gems is listed in a so called Gemfile which should be in the application root. The Gemfile lists all the gems that are used in the application, some just in the development phase (like gems specified for testing) or in the production.

When downloading a RoR-application the first thing to du is to install all the gems the application is dependent of. This is dune by the bundle program. Type, in your terminal in the application root, this command:

`bundle install`

This will install all gems needed. This could also be a point of failure when running a rails application on linux distributions. Some gems required specific linux packages that is installed on the system through apt-get.

### RoR and databases
A dynamic web application usually use a database to store the data. The RoR framework tries to simplify this for the developer and make it more easy to change the database manager. In most cases the application will have a so called [Relational Database](https://en.wikipedia.org/wiki/Relational_database), it could be a mySQL-, a SQLite or a postgreSQL-database.

RoR is a [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) framework where the M stands for Model which is the code where all the logic bind to the database is stored. RoR handle much of the code that makes all the "magic" between the application and the DB and as a developer you define your models through defining the database tables and their relations. This is done in migration-files which can sees as smaller scriptfiles you can call to build the database. The idea is that you only configure which database manager you should use and then run the migrations and the database is created for you.

### Where to find the DB-configuration
RoR uses Convention over configuration meaning that there will be a naming convention that minimize the need for writing configuration files. But you will find a file called database.yml in every RoR-applications. By convention the file should be placed in the config-folder in the RoR-application.
The file may look like this:

```
development: &default
  adapter: sqlite3
  database: db/db_dev_db.sqlite3
  pool: 5
  timeout: 5000

test:
  <<: *default
  database: development_db

production:
  <<: *default
  adapter: postgresql
  database: production_db
  encoding: utf8
  username: db_user
  password: ENV[DB_USER_POSTGRES_PASSWORD']
  ```

As you can see in this case the application runs a simple sqlite3 database manager in the development and test environment and a postgreSQL in production. The migrations-script files should work and adopt in all cases. (This may not be the best practice and maybe you should run the same db-manager in all three environments for minimize the risk of database dependent bugs).

The migration files is placed in the db/migrate folder and there are usually several files so that the developer can specify each step in the creation of the database and also do rollbacks and have some kind of version control.

The whole creation process is executed by one command:

`rake db:migrate`

This command is executed in your terminal when standing in the applications root-folder. This will take all the migration files and run them in order and create the database for the application. It will also produce content in the db/schema.rb file where the whole database scheme is presented. You will also find a file db/seed.rb where the developer could specify code that populates the database with start values in the database. The most common command to set up the database is

`bin/rake db:setup`

(in rails version 5 this is replaced by `bin/rails db:migrate`) which recreated the database (if created before), run all migrations and also run the code in seed.rb.

### Running the application
When the database is in place the application can be taken for a test run. The application an, as you seen, run in different environments. This is by default development, test and production. The different environments have different databases that could have different configurations. The environment is defined by the [environment variable](https://en.wikipedia.org/wiki/Environment_variable) named RAILS_ENV which should be set du the preferred type like;

`RAILS_ENV=production`

The environment variable could be set on the production server or just when running a specific process like:

`RAILS_ENV=production rake db:migrate`

which migrates the database defined for production. Environment variables is a good thing to know about and use to define many configurations for the application.

One file that the default RoR-application needs is the config/secret.yml. That is a file that you never should push to your repository if it is public. If you are using git this file should be in your .gitignore. In this file you define the secret token used for sensitive data hashing on the server. Things like sessions, password hashing and stuff like that. Here is an example:

```
development:
  secret_key_base: 5aa4039b1cf66eeacaa4a6647635ea94e936d9e531d3646920d978cbf54e19db35b6c583637caeefec05c830ff308b553d055536b8e43a78e8aa0789fd46cc5d

test:
  secret_key_base: 7b8989830ef3df52eca605fea07251e5e4b48b0632d360317eb055ba6ee85b0297a88936c4092d117cff076f31f37aab6fb0fa390d8ed2a8bcee97dc25ea9bd5

# Do not keep production secrets in the repository,
# instead read values from the environment.
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

As you can see that the token is different in different environments. In production the token is defined in a environment variable called "SECRET_KEY_BASE"

When running a RoR application taken from a open repository you will probably end up creating this file in the application or handling this in some way.

The rails framework comes with a built in web server called webrick. This server is OK for handling smaller applications but in many cases you want to run a dedicated Ruby server. There are many to choose from. Here is a list:

* [Passenger](https://www.phusionpassenger.com/)
* [Unicorn](https://unicorn.bogomips.org/)
* [Thin](http://code.macournoyer.com/thin/)

In this text we are using the built in server. You start the server with the command

`bin/rails server -p 8080`

, in this case we define the server running on port 8080 (3000 is default). Wen not defining a environment the server starts in development and if all thing is ok you should be able to get the startpage through your browser.

## When going to production
There are some thing to know about when going to production with a rails application. As mention before your maybe should thing about using a dedicated ruby server and not the built-in server. If you do you start it in production

`RAILS_ENV=production rails server -p 80`

This will optimize the server and make it ready for production. Of course you should do a migration to the production database before you start the application. You will probably get some errors to your static asserts (images/client scripts...). RoR wants you to pre-compile all this files (minifies and combine different files). This is done by running:

`rake assets:precompile RAILS_ENV=production`

Hopefully this should get your application running in production. Of course there will be more to do to get your application running in a more scalable way. Mayby you dont want to handle the static resources by the framework itself and just have a simple web server for this (like nginx). You maybe want to put a reversed proxy with https installed (like nginx - again) in front of the application(s), using one or more database servers and handling the web cache with a cache server (like memcached) and so on...
