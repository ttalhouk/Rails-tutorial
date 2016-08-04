# Notes from Rails course

**Look up Chaining**

`Tweet.where(zombie: "ash").order(:status).limit(10)` will order "ash" zombies, by status, limiting them to the top 10.  

**Updating Entry**

for multiple updates to an entry
```ruby
t = Tweet.find(2)
t.update( t = Tweet.find(2) # t.attribute requires t.save afterwards.
  status: "Can I munch your eyeballs?",
  zombie: "EyeballChomper"
)
```

**Validations**

Some examples of validations
```ruby
validates_presence_of :status
validates_numericality_of :fingers
validates_uniqueness_of :toothmarks
validates_confirmation_of :password
validates_acceptance_of :zombification
validates_length_of :password, minimum: 3
validates_format_of :email, with: /regex/i
validates_inclusion_of :age, in: 21..99
validates_exclusion_of :age, in: 0...21,
                       message: “Sorry you must be over 21”
```

**`link_to`**

Syntax for creating links in Rails

```html
#examples
<%= link_to tweet.zombie.name, zombie_path(tweet.zombie) %>

#or path implied
<%= link_to tweet.zombie.name, tweet.zombie %>

# basic syntax
<%= link_to text_to_show, object_to_show, options %>
```
Check documentation for options

**Strong Parameters**

`@tweet = Tweet.create(params.require(:tweet).permit(:status))`  
Makes sure only :status key is allowed from the :tweet params.  Multiple permits are allowed using array syntax.

**`respond_to`**

For the back end to respond to multiple types of request us the respond_to method
```ruby
class TweetsController < ApplicationController
  def show
    @tweet = Tweet.find(params[:id])
    respond_to do |format|
      format.html # show.html.erb
      format.json { render json: @tweet } #supplies json of @tweet
      format.xml { render xml: @tweet } # <?xml version="1.0" encoding="UTF-8"?> ...
    end
  end
end
```

**Redirects and Flash**

To check if current user is same as the profile being edited.
```ruby
class TweetsController < ApplicationController
  def edit
    @tweet = Tweet.find(params[:id])
    if session[:zombie_id] != @tweet.zombie_id
      flash[:notice] ="Sorry, you can’t edit this tweet" redirect_to(tweets_path )
    end
  end
end

# alt syntax
redirect_to(tweets_path,
  :notice =>"Sorry, you can’t edit this tweet"
)
# for specific tweet
redirect_to tweet_path(@tweet)
```
to display flash messages
```html
<!DOCTYPE html>
 <head>
  <title>Twitter for Zombies</title>
 </head>
 <body>
  <img src="/images/twitter.png" />
  <% if flash[:notice] %>
    <div id="notice"><%= flash[:notice] %></div>
  <% end %>
<%= yield %>
 </body>
</html>
```

**Before Actions**

If there are actions that happen on multiple routes put them in the before_action helper

```ruby
class TweetsController < ApplicationController
  before_action :get_tweet , only: [:edit, :update, :destroy]
  before_action :check_auth , only: [:edit, :update, :destroy]

  def get_tweet
    @tweet = Tweet.find(params[:id])
  end
  def check_auth
    if session[:zombie_id] != @tweet.zombie_id
      flash[:notice] = "Sorry, you can’t edit this tweet"
      redirect_to tweets_path
    end
  end


  def edit
   ...
  end
  def update
  ...
  end

  def destroy
  end
end
```

## Custom named routes

`get '/all' => 'tweets#index', as: 'all_tweets'` can now be used to go to the tweets index by linking to it like this:

`<%= link_to "All Tweets", all_tweets_path %>`

**Redirects**
`get '/all' => redirect('/tweets')` redirect to the /tweets routes

**Special Route Params**

```Ruby
# In tweets controller
def index
 if params[:zipcode]
   @tweets = Tweet.where(zipcode: params[:zipcode])
 else
   @tweets = Tweet.all
 end
  respond_to do |format|
    format.html # index.html.erb
    format.xml  { render xml: @tweets }
  end
end

# in routes

get '/local_tweets/:zipcode'
  => 'tweets#index', as: 'local_tweets'

# linked as
link_to "Tweets in 32828", local_tweets_path(32828)

```

## Create a Rails App

```Bash
$ rails new AppName

create
create  Gemfile
create  app
create  app/controllers/application_controller.rb
create  app/mailers
create  app/models
create  app/views/layouts/application.html.erb
create  config
create  log
create  public/index.html
create  script/rails
   run  bundle install

```

**Scaffolding**

`$ rails g scaffold zombie name:string bio:text age:integer`

Creates models, views, tests, controllers for Zombie using the parameters inputed.

**Migrations and table Updates**

*Adding columns*
```
$ rails g migration AddEmailAndRottingToZombies email:string rotting:boolean

#results in the following migration

class AddEmailAndRottingToZombies < ActiveRecord::Migration
  def change
    add_column :zombies, :email, :string
    add_column :zombies, :rotting, :boolean, default: false
  end
end

# syntax
"Add<columns>To<Table>"
# default adds default value
```

*Removing columns*
```
$ rails g migration RemoveAgeFromZombies age:integer

#results in the following migration

class RemoveAgeFromZombies < ActiveRecord::Migration
  def up
    remove_column :zombies, :age
    end

# RUN WITH ROLLBACK
  def down
    add_column :zombies, :age, :integer
  end
end

```

## Named Scoping

For reusability, queries can be scoped to use in the controllers

```ruby
# in contoller
class RottingZombiesController < ApplicationController
  def index
    @rotting_zombies = Zombie.where(rotting: true) # commonly used query
  end
end

# can be scoped to the model so that

#model
class Zombie < ActiveRecord::Base
  scope :rotting, where(rotting: true)
  # other examples
  scope :rotting, where(rotting: true)
  scope :fresh, where("age < 20")
  scope :recent, order("created_at desc").limit(3) # can chain  
end

#controllers
class RottingZombiesController < ApplicationController
  def index
    @rotting_zombies = Zombie.rotting # performs where query

    #chaning scopes
    Zombie.recent.fresh.rotting

  end
end

```

**Before actions in Model**

If we want to give an attribute based on other attributes, we can perform a before save method that can check what it should be.  Example:
```ruby
class Zombie < ActiveRecord::Base
  before_save :make_rotting
  def make_rotting
    if age > 20
      self.rotting = true # need self to be called for this instance to know it is an attribute
    end
  end
end
```

Other actions are before and after validation, create, save, update, destroy

**dependent destroy**

For some relationships we want to destroy any related objects when the base object is destroyed. To do this use the dependent destroy when specifing the relationship.

```ruby
class Zombie
  has_one :brain, dependent: :destroy
end
```

**Includes Option**

```ruby
def index
  @zombies = Zombie.includes(:brain).all
```
will query all zombies and associated brains instead of querying each brain seperate

## Links for REST actions

```
<%= link_to 'show', zombie %>
<%= link_to 'create', zombie, method: :post %>
<%= link_to 'update', zombie, method: :put %>
<%= link_to 'delete', zombie, method: :delete %>
```

**Rails Forms**

```
<%= form_for(@zombie) do |f| %>
   <%= f.label :name %><br />
    <%= f.text_field :name %>
  <%= f.submit %>
<% end %>
```
Determines if @zombie is in the data base and will create the form method accordingly.  It also updates the submit button to customize update vs create

common fields...

```
# text field
<%= f.text_area :bio %>

#check box
<%= f.check_box :rotting %>

# radio buttons
<%= f.radio_button :decomp, 'fresh', checked: true %>
<%= f.radio_button :decomp, 'rotting' %>
<%= f.radio_button :decomp, 'stale' %>

# selector with value
<%= f.select :decomp, [['fresh', 1], ['rotting', 2], ['stale', 3]] %>

# types password
<%= f.password_field :password %>

# number fields
<%= f.number_field :price %>

# range fields
<%= f.range_field :quantity %>

# email fields
<%= f.email_field :email %>

# url fields
 <%= f.url_field :website %>

# phone fields
 <%= f.telephone_field :mobile %>


```
