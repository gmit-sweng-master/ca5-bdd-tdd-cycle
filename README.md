Acceptance-Unit Test Cycle
===

In this assignment you will use a combination of Acceptance and Units tests with the Cucumber and RSpec tools to add a "find movies with same director" feature to RottenPotatoes.


Learning Goals
--------------
After you complete this assignment, you should be able to:
* Create and run simple Cucumber scenarios to test a new feature
* Use RSpec to create unit tests that drive the creation of app code that lets the Cucumber scenario pass
* Understand where to modify a Rails app to implement the various parts of a new feature, since a new feature often touches the database schema, model(s), view(s), and controller(s)


Introduction and Setup
----
1) Change into the rottenpotatoes directory: `cd hw-acceptance-unit-test-cycle/rottenpotatoes`  
2) Run `bundle install` to make sure all gems are properly installed.    
3) Run 

```shell
rake db:migrate
rake db:test:prepare
```

to apply database migrations.

5) You can double-check if everything was installed by running the tasks `rspec` and `cucumber`. Running `cucumber` should show a number of scenarios passing, some failing, and some undefined:

```shell
7 scenarios (3 failed, 4 passed)
39 steps (3 failed, 13 skipped, 1 undefined, 22 passed)
0m0.546s
```

Running `rspec` should show no examples (specs or tests) found:

```shell
gitpod /workspace/ca5-bdd-tdd-cycle/rottenpotatoes $ rspec
No examples found.


Finished in 0.00045 seconds (files took 0.1064 seconds to load)
0 examples, 0 failures
```

**Part 1: add a Director field to Movies**
First we need to create and apply a migration that adds the Director field to the movies table.  Use rails to auto-generate the migration:

```bash
rails generate migration add_director_to_movies director:string
```

Now run this new migration on the test database:

```shell
rake db:migrate
rake db:test:prepare
```
Remember to add `:director` to the list of movie attributes in the `def movie_params` method in `movies_controller.rb`.


**Part 2: use Acceptance and Unit tests to get new scenarios passing**

Three Cucumber scenarios to drive creation of the happy path of Search for Movies by Director are 
provided in `features/director.feature`.
The first lets you add director info to an existing movie, 
and doesn't require creating any new views or controller actions 
(but does require modifying existing views, and will require creating a new step definition and possibly adding a line
or two to `features/support/paths.rb`).

Three Scenarios:

```
Feature: search for movies by director

  As a movie buff
  So that I can find movies with my favorite director
  I want to include and serach on director information in movies I enter

Background: movies in database

  Given the following movies exist:
  | title        | rating | director     | release_date |
  | Star Wars    | PG     | George Lucas |   1977-05-25 |
  | Blade Runner | PG     | Ridley Scott |   1982-06-25 |
  | Alien        | R      |              |   1979-05-25 |
  | THX-1138     | R      | George Lucas |   1971-03-11 |

Scenario: add director to existing movie
  When I go to the edit page for "Alien"
  And  I fill in "Director" with "Ridley Scott"
  And  I press "Update Movie Info"
  Then the director of "Alien" should be "Ridley Scott"

Scenario: find movie with same director
  Given I am on the details page for "Star Wars"
  When  I follow "Find Movies With Same Director"
  Then  I should be on the Similar Movies page for "Star Wars"
  And   I should see "THX-1138"
  But   I should not see "Blade Runner"

Scenario: can't find similar movies if we don't know director (sad path)
  Given I am on the details page for "Alien"
  Then  I should not see "Ridley Scott"
  When  I follow "Find Movies With Same Director"
  Then  I should be on the home page
  And   I should see "'Alien' has no director info"
```

The second lets you click a new link on a movie details page "Find Movies With Same Director", 
and shows all movies that share the same director as the displayed movie.  
For this you'll have to modify the existing Show Movie view, and you'll have to add a route, 
view and controller method for Find With Same Director.  

The third handles the sad path, when the current movie has no director info but we try 
to do "Find with same director" anyway.

Going one Cucumber step at a time, use RSpec to create the appropriate
controller and model specs to drive the creation of the new controller
and model methods.  At the least, you will need to write tests to drive
the creation of: 

+ a RESTful route for finding other movies by the same director 

+ a controller method to receive the click
on "Find With Same Director", and grab the id (for example) of the movie
that is the subject of the match (i.e. the one we're trying to find
movies similar to) 

+ a model method in the Movie model to find movies
whose director matches that of the current movie. Note: This implies that you should write at least 2 specs for your controller: 1) When the specified movie has a director, it should...  2) When the specified movie has no director, it should ... and 2 for your model: 1) it should find movies by the same director and 2) it should not find movies by different directors.

It's up to you to decide whether you want to handle the sad path of "no director" in the
controller method or in the model method, but you must provide a test
for whichever one you do. Remember to include the line 
`require 'rails_helper'` at the top of your *_spec.rb files.

### Code coverage
The percentage of the code covered by tests is being reported by the `simplecov` gem.
When you run `rspec` or `cucumber`, SimpleCov will generate a report in a directory named
`coverage/`, as well as reporting coverage on the command line. 

```shell
Coverage report generated for Cucumber Features to /workspace/ca5-bdd-tdd-cycle/rottenpotatoes/coverage. 2 / 64 LOC (3.13%) covered.
```

Since both RSpec and Cucumber are so widely used, SimpleCov can intelligently merge the results, so running the tests for Rspec does
not overwrite the coverage results from SimpleCov and vice versa.
To see the results, open /coverage/index.html. You will see the code, but click the Run button at the top. This will spin up a web server with a link in the console you can click to see your coverage report.

**The autograder will check the percentage of code covered by tests. You should improve your test coverage by adding unit tests for untested or undertested code.**


**Submission:**

Here are the instructions for submitting your assignment for grading. Submit a zip file containing the following files and directories of your app:

* app/
* config/
* db/migrate
* features/
* spec/
* Gemfile
* Gemfile.lock

If you modified any other files, please include them too. If you are on a *nix based system, navigate to the root directory for this assignment and run

```sh
$ cd ..
$ zip -r acceptance-tests.zip rottenpotatoes/app/ rottenpotatoes/config/ rottenpotatoes/db/migrate rottenpotatoes/features/ rottenpotatoes/spec/ rottenpotatoes/Gemfile rottenpotatoes/Gemfile.lock
```

This will create the file `acceptance-tests.zip`, which you will submit.

IMPORTANT NOTE: Your submission must be zipped inside a rottenpotatoes/ folder so that it looks like so:

```
$ tree
.
└── rottenpotatoes
    ├── Gemfile
    ├── Gemfile.lock
    ├── app
    ...
```
