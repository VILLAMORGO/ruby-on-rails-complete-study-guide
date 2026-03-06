# Complete Ruby on Rails Interview & Study Guide

## Comprehensive Technical Reference for Developers

---

## Table of Contents

1. [Foundational Concepts](#part-1-foundational-concepts)
2. [Ruby Fundamentals](#part-2-ruby-fundamentals)
3. [Active Record & Database](#part-3-active-record--database-interactions)
4. [Routing & Controllers](#part-4-routing--controllers)
5. [Security](#part-5-security)
6. [Testing](#part-6-testing)
7. [Advanced Topics](#part-7-advanced-rails-topics)
8. [Design Patterns & Best Practices](#part-8-design-patterns--best-practices)
9. [Performance & Optimization](#part-9-performance--optimization)
10. [Essential Gems](#part-10-essential-rails-gems)
11. [Interview Preparation](#part-11-interview-preparation)

---

## Part 1: Foundational Concepts

### What is Ruby on Rails?

Ruby on Rails (often just called Rails) is an **open-source web application framework** written in Ruby. Created by David Heinemeier Hansson in 2004, it revolutionized web development with two core philosophies:

**Convention over Configuration (CoC)**: Rails makes smart assumptions about naming, file locations, and structure. For example, if you have a `User` model, Rails automatically looks for a `users` table in the database and expects a `users_controller.rb` file in the controllers directory.

**Don't Repeat Yourself (DRY)**: Every piece of knowledge should have a single, unambiguous representation. This reduces redundancy and makes maintenance easier.

### MVC Architecture (Model-View-Controller)

Rails follows the MVC pattern, which separates concerns into three interconnected components:

#### Request-Response Flow

1. **User Action**: User clicks a link, submits a form, or makes an API request
2. **Router**: Routes incoming request to appropriate controller action based on URL and HTTP verb
3. **Controller**: Receives request, acts as traffic cop, coordinates the response
4. **Model**: Controller asks model for data; model handles database operations (CRUD)
5. **Controller Processing**: Receives data from model, performs any business logic
6. **View**: Controller selects view template; view renders HTML/JSON response with data
7. **Response**: Final rendered output sent back to user's browser or API client

#### Component Responsibilities

- **Model**: Represents data and business logic. Handles validations, associations, database queries, and data transformations. Located in `app/models/`

- **View**: Presentation layer that displays data to users. Uses ERB (Embedded Ruby) or other templating engines. Located in `app/views/`

- **Controller**: Handles HTTP requests, processes parameters, interacts with models, and renders appropriate views. Located in `app/controllers/`

#### Practical Example

When a user visits `/articles/5`:

```ruby
# Route (config/routes.rb)
get '/articles/:id', to: 'articles#show'

# Controller (app/controllers/articles_controller.rb)
def show
  @article = Article.find(params[:id])
end

# Model (app/models/article.rb)
class Article < ApplicationRecord
  validates :title, presence: true
  belongs_to :author, class_name: 'User'
end

# View (app/views/articles/show.html.erb)
<h1><%= @article.title %></h1>
<p><%= @article.content %></p>
```

---

## Part 2: Ruby Fundamentals

### Object-Oriented Programming in Ruby

**Everything in Ruby is an object**, even primitive types like integers and booleans. This unified object model makes Ruby elegant and consistent.

#### Core OOP Concepts

- **Classes and Objects**: Classes are blueprints; objects are instances. Use `class ClassName` to define and `ClassName.new` to instantiate.

- **Inheritance**: Ruby supports single inheritance using `class Child < Parent`. Child classes inherit all methods and attributes from parent.

- **Encapsulation**: Control access with `public`, `private`, and `protected` visibility modifiers.

- **Polymorphism**: Objects of different classes can respond to the same message (method call) in different ways.

```ruby
class Animal
  def speak
    'Some sound'
  end
end

class Dog < Animal
  def speak
    'Woof!'
  end
end
```

### Strings vs Symbols

One of the most common Ruby interview questions:

| Aspect | String | Symbol |
|--------|--------|--------|
| Mutability | Mutable (can be changed) | Immutable (cannot be changed) |
| Memory | New object for each instance | Same object reused (stored once) |
| Syntax | `"name"` or `'name'` | `:name` |
| Common Use | User input, text content | Hash keys, identifiers, constants |

💡 **Best Practice**: Use symbols for hash keys and internal identifiers. Use strings for user-facing text and data that might change.

### Modules and Mixins

Ruby only supports **single inheritance** (a class can inherit from only one parent). Modules provide a way to share code across multiple classes without inheritance.

#### Module Keywords

**`include`**: Adds module methods as *instance methods*. These methods are called on objects created from the class.

```ruby
module Swimmable
  def swim
    'Swimming...'
  end
end

class Fish
  include Swimmable  # Instance method
end

fish = Fish.new
fish.swim  # => 'Swimming...'
```

**`extend`**: Adds module methods as *class methods*. These methods are called on the class itself, not on instances.

```ruby
class Fish
  extend Swimmable  # Class method
end

Fish.swim  # => 'Swimming...'
```

**`prepend`**: Like `include`, but inserts the module *before* the class in the method lookup chain. This allows module methods to override class methods.

```ruby
class Fish
  prepend Swimmable  # Module methods checked first
  def swim
    'Fish swimming'
  end
end

fish.swim  # => 'Swimming...' (module method wins)
```

### Blocks, Procs, and Lambdas

Ruby's powerful functional programming features allow you to pass code as objects.

#### Blocks

Chunks of code enclosed in `{ }` or `do...end`. Not objects themselves, but can be yielded to:

```ruby
[1, 2, 3].each { |num| puts num }

def greet
  yield('Hello')  # Pass control to block
end

greet { |msg| puts msg }  # => 'Hello'
```

#### Procs

Blocks wrapped in objects. Can be stored in variables and passed around:

```ruby
my_proc = Proc.new { |x| x * 2 }
my_proc.call(5)  # => 10
```

#### Lambdas

Similar to Procs but with stricter argument checking and different return behavior:

```ruby
my_lambda = lambda { |x| x * 2 }
my_lambda = ->(x) { x * 2 }  # Shorthand syntax
my_lambda.call(5)  # => 10
```

| Feature | Proc | Lambda |
|---------|------|--------|
| Argument checking | Lenient (ignores extras) | Strict (raises ArgumentError) |
| `return` behavior | Returns from enclosing method | Returns from lambda only |

### Important Method Distinctions

#### nil vs empty vs blank

- **`nil?`**: Object doesn't exist or is `nil`
- **`empty?`**: Collection has no elements (works on arrays, strings, hashes)
- **`blank?`**: Rails method - true for `nil`, empty strings, or whitespace-only strings

```ruby
nil.nil?         # => true
''.empty?        # => true
'   '.blank?     # => true (Rails only)
[].empty?        # => true
{}.empty?        # => true
```

#### self Context

The `self` keyword refers to different objects depending on context:

- At the top level: refers to `main`
- Inside a class definition: refers to the class itself
- Inside an instance method: refers to the instance (object)
- Inside a class method: refers to the class

---

## Part 3: Active Record & Database Interactions

### What is Active Record?

Active Record is Rails' **ORM (Object-Relational Mapper)**. It provides a layer between your Ruby code and the database, allowing you to:

- Work with database tables as Ruby objects
- Perform CRUD operations without writing SQL
- Define associations between models
- Validate data before saving
- Handle database migrations

### Database Migrations

Migrations are Ruby classes that allow you to **evolve your database schema over time** in a version-controlled, reversible way.

#### Why Migrations?

- **Version Control**: Track database changes in Git
- **Consistency**: Same schema across dev, test, and production
- **Reversibility**: Roll back changes if needed
- **Team Collaboration**: Multiple developers can work on schema changes

#### Creating a Migration

```ruby
# Command line
rails generate migration AddAgeToUsers age:integer

# Generated migration file
class AddAgeToUsers < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :age, :integer
  end
end

# Run migration
rails db:migrate

# Rollback migration
rails db:rollback
```

### Active Record Associations

Associations define relationships between models, making it easy to work with related data.

#### Main Association Types

**`belongs_to`**: Declares a one-to-one connection with another model. The foreign key is on THIS model.

```ruby
class Post < ApplicationRecord
  belongs_to :user  # posts table has user_id column
end
```

**`has_many`**: Indicates a one-to-many relationship. The foreign key is on the OTHER model.

```ruby
class User < ApplicationRecord
  has_many :posts  # user_id in posts table
end
```

**`has_one`**: Indicates a one-to-one relationship where the foreign key is on the OTHER model.

```ruby
class User < ApplicationRecord
  has_one :profile  # profile_id in profiles table
end
```

**`has_many :through`**: Sets up many-to-many relationship through a join model.

```ruby
class Doctor < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
```

**`has_and_belongs_to_many`**: Direct many-to-many without a join model (simpler but less flexible).

#### Polymorphic Associations

A polymorphic association allows a model to **belong to multiple other models using a single association**.

**Example**: Comments can belong to both Posts AND Pictures:

```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

class Post < ApplicationRecord
  has_many :comments, as: :commentable
end

class Picture < ApplicationRecord
  has_many :comments, as: :commentable
end
```

**Database structure**:
```ruby
# comments table
commentable_id   # References the ID of the parent (post or picture)
commentable_type # Stores the class name ('Post' or 'Picture')
```

### Query Methods

#### Finding Records

| Method | Behavior | Example |
|--------|----------|---------|
| `find` | Raises `ActiveRecord::RecordNotFound` if not found | `User.find(1)` |
| `find_by` | Returns `nil` if not found | `User.find_by(email: 'a@b.com')` |
| `where` | Returns relation (chainable) | `User.where(active: true)` |
| `first`/`last` | Returns first/last record or nil | `User.first` |
| `find_or_create_by` | Finds or creates if not found | `User.find_or_create_by(email: 'a@b.com')` |

### The N+1 Query Problem

One of the **most common performance issues** in Rails applications. Understanding and solving this is crucial for interviews.

#### The Problem

When you load a collection and then access associations in a loop:

```ruby
# BAD: N+1 queries
posts = Post.all  # 1 query
posts.each do |post|
  puts post.author.name  # 1 query per post = N queries
end
# Total: 1 + N queries (if 100 posts = 101 queries!)
```

#### The Solution: Eager Loading

Use **`includes`** to load associations upfront:

```ruby
# GOOD: 2 queries total
posts = Post.includes(:author).all  # 2 queries
posts.each do |post|
  puts post.author.name  # No additional queries
end
```

#### Other Eager Loading Methods

- **`includes`**: Uses separate queries (LEFT OUTER JOIN)
- **`joins`**: INNER JOIN, doesn't load associated records
- **`eager_load`**: Forces LEFT OUTER JOIN in single query
- **`preload`**: Forces separate queries

### Validations

Validations ensure data integrity by checking that data meets certain criteria **before** it's saved to the database.

#### Common Validations

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true
  validates :age, numericality: { greater_than: 0 }
  validates :password, length: { minimum: 8 }
  validates :username, format: { with: /\A[a-zA-Z0-9]+\z/ }
  validates :terms_of_service, acceptance: true
  validates :email, confirmation: true  # Requires email_confirmation field
end
```

#### Custom Validations

```ruby
class User < ApplicationRecord
  validate :email_must_be_company_domain

  private

  def email_must_be_company_domain
    unless email.ends_with?('@company.com')
      errors.add(:email, 'must be a company email')
    end
  end
end
```

⚠️ **validates vs validate**: `validates` uses built-in helpers, `validate` uses custom methods

### Callbacks (Lifecycle Hooks)

Callbacks are methods that get called at **specific points** in an object's lifecycle.

#### Common Callbacks

- **`before_validation`**: Before validations run
- **`after_validation`**: After validations run
- **`before_save`**: Before saving (both create and update)
- **`after_save`**: After saving
- **`before_create`**: Before creating new record
- **`after_create`**: After creating new record
- **`before_update`**: Before updating existing record
- **`after_update`**: After updating existing record
- **`before_destroy`**: Before deleting record
- **`after_destroy`**: After deleting record

#### Example

```ruby
class User < ApplicationRecord
  before_save :normalize_email
  after_create :send_welcome_email

  private

  def normalize_email
    self.email = email.downcase.strip
  end

  def send_welcome_email
    UserMailer.welcome_email(self).deliver_later
  end
end
```

### Delete vs Destroy

**Critical difference** often asked in interviews:

| Aspect | delete | destroy |
|--------|--------|---------|
| Callbacks | Skips ALL callbacks | Runs all callbacks |
| Validations | Skips validations | N/A (already saved) |
| Dependencies | Doesn't delete associated records | Handles `dependent: :destroy` |
| Speed | Faster (direct SQL) | Slower (runs lifecycle) |
| Use when | Speed critical, no cleanup needed | Need callbacks or cleanup (DEFAULT) |

### Database Transactions

Transactions ensure **atomic operations** - either all operations succeed or all fail (rollback).

```ruby
ActiveRecord::Base.transaction do
  user = User.create!(name: 'John')
  profile = Profile.create!(user: user, bio: 'Developer')
  # If profile creation fails, user creation rolls back
end
```

**Why use transactions?**
- **Data Consistency**: Prevent orphaned records
- **All-or-Nothing**: Related operations succeed or fail together
- **Safety**: Protect against partial updates

---

## Part 4: Routing & Controllers

### Routing

The router (`config/routes.rb`) **maps incoming HTTP requests to controller actions**. It's the entry point for all web requests.

#### RESTful Routes

Rails follows REST (Representational State Transfer) conventions:

| HTTP Verb | Path | Action | Purpose |
|-----------|------|--------|---------|
| GET | `/articles` | `index` | List all articles |
| GET | `/articles/:id` | `show` | Show specific article |
| GET | `/articles/new` | `new` | Show form to create |
| POST | `/articles` | `create` | Create new article |
| GET | `/articles/:id/edit` | `edit` | Show form to edit |
| PATCH/PUT | `/articles/:id` | `update` | Update article |
| DELETE | `/articles/:id` | `destroy` | Delete article |

Generate all these routes with one line:
```ruby
resources :articles
```

#### resource vs resources

- **`resources`**: Plural - creates routes with IDs (`/articles/1`)
- **`resource`**: Singular - no ID needed (`/profile` instead of `/profiles/1`)

```ruby
resource :profile  # User has one profile
```

### Controllers

Controllers are the **"traffic cops"** of your Rails application. They:

- Handle incoming HTTP requests
- Process parameters
- Interact with models
- Render views or return JSON
- Handle redirects

#### Example Controller

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]

  def index
    @articles = Article.all
  end

  def show
    # @article set by before_action
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article, notice: 'Article created successfully'
    else
      render :new, status: :unprocessable_entity
    end
  end

  private

  def set_article
    @article = Article.find(params[:id])
  end

  def article_params
    params.require(:article).permit(:title, :content)
  end
end
```

#### render vs redirect_to

| Aspect | render | redirect_to |
|--------|--------|-------------|
| HTTP Request | No new request | New HTTP request |
| URL | URL doesn't change | URL changes |
| Instance Variables | Preserved (@article available) | Lost (new request) |
| Use Case | Show form with errors | After successful create/update |

---

## Part 5: Security

### CSRF Protection

**Cross-Site Request Forgery (CSRF)** is an attack where a malicious site tricks a user into making an unwanted request to your application.

#### How Rails Protects You

- Rails automatically includes an **authenticity token** in all forms
- The token is verified on non-GET requests
- Enabled by default in ApplicationController

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end
```

### Strong Parameters

Strong Parameters prevent **mass assignment vulnerabilities** by requiring you to explicitly whitelist attributes that can be updated.

#### The Problem (Without Strong Parameters)

A malicious user could send unexpected parameters:

```ruby
# User sends: { name: 'John', role: 'admin' }
user.update(params[:user])  # BAD! Sets role to admin
```

#### The Solution

```ruby
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)  # Only permitted attributes
    @user.save
  end

  private

  def user_params
    params.require(:user).permit(:name, :email, :password)
    # 'role' is NOT permitted, so it will be ignored
  end
end
```

### SQL Injection Prevention

Active Record automatically escapes SQL, but you must use it correctly:

```ruby
# VULNERABLE to SQL injection
User.where("name = '#{params[:name]}'")

# SAFE: Uses parameterized queries
User.where("name = ?", params[:name])
User.where(name: params[:name])  # Even better
```

### Authentication & Authorization

#### Common Gems

- **Devise**: Full-featured authentication solution (sign up, log in, password reset, etc.)
- **bcrypt**: Password hashing (use `has_secure_password` in your User model)
- **Pundit**: Authorization - defines what authenticated users can do
- **CanCanCan**: Another popular authorization library

---

## Part 6: Testing

### Testing Frameworks

#### Minitest

Rails default testing framework. Simple and fast.

```ruby
class UserTest < ActiveSupport::TestCase
  test 'user must have email' do
    user = User.new(name: 'John')
    assert_not user.valid?
  end
end
```

#### RSpec

Most popular alternative. More expressive, BDD-style syntax.

```ruby
RSpec.describe User do
  it 'is invalid without an email' do
    user = User.new(name: 'John')
    expect(user).not_to be_valid
  end
end
```

### Test Doubles: Stubs, Mocks, and Spies

Test doubles help **isolate the code under test** by replacing dependencies with controlled objects.

| Type | Purpose | Behavior |
|------|---------|----------|
| **Stub** | Provides predetermined responses | When method called, return this value |
| **Mock** | Sets expectations about calls | Method MUST be called X times, test fails otherwise |
| **Spy** | Records method calls | Verify method was called after the fact |

#### Example

```ruby
# Stub example
allow(User).to receive(:count).and_return(100)

# Mock example
expect(UserMailer).to receive(:welcome_email).once

# Spy example
mailer = spy('mailer')
# ... code that should call mailer ...
expect(mailer).to have_received(:deliver)
```

### Test Types

- **Unit Tests**: Test individual methods/classes in isolation
- **Integration Tests**: Test how multiple components work together
- **System/Feature Tests**: Test from user's perspective (browser automation with Capybara)

---

## Part 7: Advanced Rails Topics

### Service Objects

Service Objects are **Plain Old Ruby Objects (POROs)** that encapsulate a single business operation.

#### Why Use Service Objects?

- **Keep controllers thin**: Move complex logic out of controllers
- **Keep models focused**: Avoid fat models
- **Single Responsibility**: Each service does one thing
- **Testability**: Easy to test in isolation
- **Reusability**: Can be used from multiple places

#### Example

```ruby
# app/services/order_processor.rb
class OrderProcessor
  def initialize(order)
    @order = order
  end

  def process
    return false unless valid?

    ActiveRecord::Base.transaction do
      charge_customer
      update_inventory
      send_confirmation_email
      @order.update!(status: 'completed')
    end

    true
  end

  private

  def valid?
    @order.valid? && @order.items.any?
  end

  # ... other private methods
end

# Usage in controller
def create
  @order = Order.new(order_params)

  if OrderProcessor.new(@order).process
    redirect_to @order, notice: 'Order processed!'
  else
    render :new
  end
end
```

### Background Jobs (Active Job)

Active Job provides a unified interface for **asynchronous job processing**. Move time-consuming tasks out of the request-response cycle.

#### Common Use Cases

- Sending emails
- Processing uploaded files
- Calling external APIs
- Generating reports
- Data cleanup and maintenance

#### Example

```ruby
# app/jobs/send_welcome_email_job.rb
class SendWelcomeEmailJob < ApplicationJob
  queue_as :default

  def perform(user)
    UserMailer.welcome_email(user).deliver_now
  end
end

# Usage
SendWelcomeEmailJob.perform_later(user)  # Enqueue
SendWelcomeEmailJob.perform_now(user)    # Run immediately
SendWelcomeEmailJob.set(wait: 1.hour).perform_later(user)  # Delay
```

#### Background Job Adapters

- **Sidekiq**: Most popular, uses Redis, very fast
- **Delayed Job**: Uses database, simpler setup
- **Resque**: Uses Redis, GitHub's solution

### Caching

Caching stores frequently accessed data to improve performance.

#### Types of Caching

**Fragment Caching**: Cache portions of views (most common)

```erb
<% cache @product do %>
  <%= render @product %>
<% end %>
```

**Low-Level Caching**: Cache arbitrary data

```ruby
Rails.cache.fetch('expensive_calculation') do
  # Expensive operation here
end
```

**SQL Caching**: Automatic caching of queries within a single request

### Action Cable (WebSockets)

Action Cable integrates **WebSockets** with Rails for real-time features.

#### Use Cases

- Chat applications
- Live notifications
- Real-time dashboards
- Collaborative editing
- Live updates (sports scores, stock prices)

### Active Storage

Built-in file upload and attachment management.

#### Features

- Upload to cloud storage (S3, Google Cloud, Azure)
- Upload to local disk
- Image transformation
- Direct uploads (upload straight from browser to cloud)

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
  has_many_attached :photos
end
```

---

## Part 8: Design Patterns & Best Practices

### SOLID Principles

Five fundamental object-oriented design principles:

- **S - Single Responsibility**: A class should have only one reason to change
- **O - Open/Closed**: Open for extension, closed for modification
- **L - Liskov Substitution**: Derived classes must be substitutable for their base classes
- **I - Interface Segregation**: Many specific interfaces better than one general-purpose interface
- **D - Dependency Inversion**: Depend on abstractions, not concretions

### Common Design Patterns

#### Decorator Pattern

Add behavior to objects without modifying their class. Popular gem: Draper

```ruby
class UserDecorator < SimpleDelegator
  def full_name
    "#{first_name} #{last_name}"
  end
end
```

#### Observer Pattern

Objects notify other objects of state changes. Rails callbacks implement this.

#### Factory Pattern

Create objects without specifying exact class. Useful for testing.

### Code Quality Tools

- **RuboCop**: Static code analyzer and formatter. Enforces Ruby style guide.
- **Reek**: Code smell detector. Finds design issues.
- **SimpleCov**: Test coverage analysis.
- **Brakeman**: Security vulnerability scanner.

---

## Part 9: Performance & Optimization

### Database Indexing

Indexes dramatically speed up queries but **slow down writes**. Add indexes to:

- Foreign keys
- Columns used in WHERE clauses
- Columns used in ORDER BY
- Columns used in JOIN conditions

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration[7.0]
  def change
    add_index :users, :email, unique: true
  end
end
```

### Database Query Optimization

**Use select to fetch only needed columns**:
```ruby
User.select(:id, :name)  # Instead of SELECT *
```

**Use pluck for single column**:
```ruby
User.pluck(:email)  # Returns array, not AR objects
```

**Use find_each for large datasets**:
```ruby
User.find_each(batch_size: 1000) do |user|
  # Processes in batches, not all at once
end
```

### Rack

**Rack** is the interface between web servers (Puma, Unicorn) and Ruby frameworks (Rails, Sinatra).

#### Why Rack Matters

- Provides standard way to handle HTTP requests/responses
- Allows middleware (modify requests/responses)
- Makes frameworks server-agnostic

---

## Part 10: Essential Rails Gems

### Must-Know Gems by Category

#### Authentication & Authorization

- **Devise**: Complete authentication solution
- **Pundit**: Simple authorization via Ruby classes
- **CanCanCan**: Authorization with ability classes

#### Background Processing

- **Sidekiq**: Efficient background jobs
- **Delayed Job**: Database-backed background jobs

#### Testing

- **RSpec**: BDD testing framework
- **Factory Bot**: Test data factories
- **Capybara**: Integration testing
- **Faker**: Generate fake data

#### Pagination & Search

- **Kaminari**: Flexible pagination
- **will_paginate**: Simple pagination
- **Ransack**: Advanced search and filtering

#### API Development

- **fast_jsonapi**: Fast JSON serialization
- **Grape**: REST-like API framework
- **rack-cors**: Handle CORS

#### Frontend

- **Webpacker**: Modern JavaScript with Webpack
- **Stimulus**: JavaScript framework for HTML
- **Turbo**: Speed up navigation

---

## Part 11: Interview Preparation

### Common Interview Question Patterns

#### 1. Explain the Difference

Be ready to compare:

- String vs Symbol
- Proc vs Lambda
- include vs extend vs prepend
- belongs_to vs has_many
- delete vs destroy
- render vs redirect_to
- validates vs validate

#### 2. How Would You Optimize...

- Slow database queries? (N+1, indexing, caching)
- A controller with too much logic? (Service objects, concerns)
- Page load times? (Caching, CDN, asset optimization)

#### 3. Design Scenarios

"How would you build..."

- A commenting system (polymorphic associations)
- A subscription service (recurring payments, background jobs)
- A notification system (Action Cable, background jobs)
- An admin dashboard (authorization, scopes)

### Level-Based Expectations

#### Junior Developer

- Understand MVC pattern
- Basic Active Record (CRUD, simple associations)
- Write basic tests
- Follow existing patterns
- Use Git for version control

#### Mid-Level Developer

- Design database schemas
- Optimize queries (N+1, indexing)
- Implement security best practices
- Write comprehensive tests
- Make architectural decisions
- Refactor legacy code

#### Senior Developer

- System design and architecture
- Performance optimization at scale
- Explain trade-offs in technical decisions
- Mentor junior developers
- Lead code reviews
- Contribute to technical strategy

### Behavioral Interview Tips

#### STAR Method

Structure your answers:

- **Situation**: Set the context
- **Task**: Explain the challenge
- **Action**: What you did
- **Result**: The outcome

#### Common Behavioral Questions

- Tell me about a time you debugged a difficult issue
- Describe a technical disagreement with a team member
- How do you stay current with technology?
- Tell me about a project you're proud of

### Study Resources

#### Official Documentation

- Rails Guides (guides.rubyonrails.org)
- Ruby API Documentation (ruby-doc.org)
- Rails API Documentation (api.rubyonrails.org)

#### Practice Platforms

- LeetCode (algorithms)
- HackerRank (coding challenges)
- Exercism (Ruby practice)
- CodeWars (Ruby katas)

#### Recommended Books

- "Agile Web Development with Rails" by Sam Ruby
- "The Rails 7 Way" by Obie Fernandez
- "Effective Testing with RSpec 3" by Myron Marston
- "Design Patterns in Ruby" by Russ Olsen

---

## Final Tips

### Before the Interview

- **Review this guide**: Focus on areas where you feel weak
- **Build something**: Have a Rails project to discuss
- **Practice coding**: Solve problems on LeetCode/HackerRank
- **Review your resume**: Be ready to discuss every project
- **Prepare questions**: Ask about tech stack, team structure, challenges

### During the Interview

- **Think out loud**: Explain your reasoning
- **Ask clarifying questions**: Don't make assumptions
- **Admit when you don't know**: But explain how you'd find out
- **Discuss trade-offs**: Show you think critically
- **Be enthusiastic**: Show genuine interest

### Remember

Interviews are **conversations, not interrogations**. The interviewer wants you to succeed. They're evaluating not just your technical skills, but also:

- How you communicate complex ideas
- How you approach problems
- How you work with others
- Your passion for learning

---

## Good luck with your Ruby on Rails journey!

**Remember: Every expert was once a beginner.**

Keep coding, keep learning, and stay curious!
