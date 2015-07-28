# command-oriented-programming
Trying to distill everything I know about web application architecture into some kind of useful philosophy

Principles of command-oriented programming

+ Don't Deny the DB
  + A web app is powered by a database. Don't pretend its not there, don't abstract it away. Harness it. Take advantage of it.
  + Let SQL drive Ruby, not vice versa
  + Models are abstractions over DB tables, not real-world objects
  + Relations represent joins, not real world relationships
  + Isolate the query logic (use of models)
  + Explicitly open and close transactions in command objects
  + No arbitrary Ruby code running between SQL commands causing transactions to remain open longer than they should
    + Scalability issues
  + Force devs to think of the request lifecycle as:
    + GET
      + Query -> Business Logic
    + POST (PATCH, PUT)
      + Business Logic -> Query
  +Keep very close understanding of what queries are run
+ RPC over REST
  + Program using verbs, not nouns
  + Your classes represent commands, such as ConfirmUser, FetchCompany, UpdateOrder
  + RPC
     + Speaks in verbs. E.G. ConfirmUser, UpdateUser, OpenAccount 
  +  Command-oriented development is not RESTful, but it can be used alongside REST, where REST is the appropriate paradigm
     + Requests where we're truly just  doing CRUD on resources
     +  Less than 50% of where people use REST is really best represented by REST
     +  90% of all statistics are made up
+ Test at the extremes
   + Separate the queries from the business logic
   + Unit Testing
     + Tests one function only
       + Ok, we don't have to be extremists, we can let them test more than one in special cases, but try to make them test one function only
100% unit test coverage on the business logic
     + No stubbing in unit tests
       + This is only difficult if you have a query that only runs under certain conditions (business rules)
Then pass in a lambda that returns the data :-)
     + Don't unit test the queries (this is not possible by the strictest definition of 'Unit Test' anyways)
     + Test none of the side effects
  + Functional Test the whole call 
    + including reading, writing to the DB
    + Test only the side effects
    + No stubbing except external API calls
    + You don't have to cover every corner case, because the unit tests do that
    + Test the glue that holds it altogether
+ Command / Query separation
  + Class names indicate command vs. query, I.E Write vs. Read, FooCommand vs. FooService
  + A class represents an action, not an object
  + base classes represents what kind of action
    + Read (service)
    + Write (command)

Pedagogical
+ Abstractions make things harder, not easier
+ Devs need to understand the DB. There's no alternative
+ Hiding the db from beginners is idiotic
+ Teach relational model first.
+ Treat objects as icing, not cake
+ Try to teach testing using The Rails Way? It's impossible. Futile
+ Command / Query separation is how the web works.
  + GET
  + POST -> redirect to GET
Class design
+ MVC
  + Laughably simple
  + Leads to bloated models
  + Not reusable business logic (E.G. from a Rake task)
+ MVCCSOVM - gotta think of something better
  + Model / View / Controller / Command / Service / Operation / Viewmodel
  + Models
    + Represent database tables not real world objects
    + Relationships represent JOINs in queries, not real-world relationships
    + Instances only deal with 
      + persistence
      + field validation (before save)
        + OR use ONLY use db level validation? :-)
          + Why do we need model validation?
    + Static methods are only scopes
    + Really the repository pattern would be more command-oriented than the active record pattern so model classes wouldn't have to worry about querying
    + Should never reference other models
      + So the calling class can manage transactions
      + Because which model does it belong in?
      + No async callbacks in a single-threaded web request please
      + Because why would one db table need to know about another one?
  + Views
    + Go to the extreme making them as dumb as possible
    + By feeding them complex view models
      + Trees (for hierarchical views)
      + Dictionaries (with pre-computed values)
      + Projections of the table data
      + In SQL powered web apps, tables were never supposed to be 1-to-1 with views. Views display table data after it has gone through 
        + JOINs,
        + complex object-relational mapping, 
        + fields derived using business logic
     + JS frameworks already do this using simplistic templating languages (handlebars etc.)
     + NEVER trigger a query from a view
     + Pass only one viewmodel to a view
     + need extra stuff? Make the viewm model a hash
     + The API version can just JSONify the viewmodel
     + Presenters are ok, but only for reusable view logic NOT business logic
     + Model decorators?
  + Controllers
    + Cleansing and casting params
    + POST
      + Call command object
    + GET
      + Call service
    + Catching errors and returning error responses
    + Redirecting
    + Only conditionals should be based on response of command (POST)
      + Usual just success or fail
      + Can be more complex if multiple possible results
    + Base controller class should be able to handle uncaught exceptions and return correct status codes based on exception type and response type
    + Call policies to check security permissions
+ Commands
  + Concerned with write operations
  + Ok if we need to read first before we write
  + If complex read, command calls a service
  + Executes queries, passes resultant data to Operation
  + Transaction management
  + Sending emails
  + Make external API calls
  + Return only simple responses with minimum data controller needs to correctly redirect
    + Usually boolean
    + Or thrown exception vs. not
    + Maybe success message or updated / created model
  + Can call other commands for composability / reusability
  + Can call services if they need to query before performing operation
  + Can can call operations if business logic needs to be performed to calculate what to write
  + Redirects to GET
+ Services
  + Concerned with read operations
  + Perform object-relational mapping needed to format table data for view
  + Can call other services for composability / reusability
  + Or operations
  + Low level caching
  + Returns viewmodel
+ Operations
Pure business logic
Pure classes / functions
No side effects ever
Can only call other operations for composability / reusability
Can / should have 100% unit test coverage
Encouraged to use functional style
ViewModel
Dumb struct with data that matches the hierarchy of the view / API response
Other patterns that fit well into Command-Oriented Development
Query Objects
For reusable SQL
boolean params that control sorting
int params for paging etc.
Presenter / Decorators
But only for reusing or encapsulating view logic, not business logic
Policies
Raise exception if a user isn't authorized to access service / command
Builders
Factories
Domain objects?
Certainly less useful in command-oriented programming
Keep separate from command-oriented objects
Separate folder in app
Treat as isolated set of classes
For simulating real world
Called from commands or services
Most applications wouldn't need any at all

"The wrong set of small objects is the worst case scenario. Really big objects are bad, but not that bad" - @sarahmei

It's not about what a class represents, it's about the action it performs
