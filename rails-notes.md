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

**`div_for`**

Creates a div with custom id for each element in a collection


## Rails mailers

`$ rails g mailer ZombieMailer decomp_change lost_brain` generates the following

```
create    app/mailers/zombie_mailer.rb
invoke    erb
create    app/views/zombie_mailer
create    app/views/zombie_mailer/decomp_change.text.erb
create    app/views/zombie_mailer/lost_brain.text.erb
```

This creates two emails

The Mailer Model is where the information from the database is collected for use in the email.
Example:

```ruby
class ZombieMailer < ActionMailer::Base
  default from: "from@example.com"
  def decomp_change(zombie)
    @zombie = zombie
    @last_tweet = @zombie.tweets.last
    attachments['z.pdf'] = File.read("#{Rails.root}/public/zombie.pdf")
    mail to: @zombie.email,
    subject: 'Your decomp stage has changed'
    # other defaults
    # from: my@email.com
    # cc: my@email.com
    # bcc: my@email.com
    # reply_to: my@email.com

  end
...
end
```
**Sending the Email**

In the model that corisponds to the email, call the mailer

```ruby
class Zombie < ActiveRecord::Base
  after_save :decomp_change_notification, if: :decomp_changed?
private
  def decomp_change_notification
    ZombieMailer.decomp_change(self).deliver
  end
end
```
`_changed?` is a built in helper method to detect if property changed

## Custom renders

Based on the status you may want to direct to two different views.  This is done like so.

```ruby
class ZombiesController < ApplicationController
  def show
    @zombie = Zombie.find(params[:id])
    respond_to do |format|
      format.html do
        if @zombie.decomp == 'Dead (again)'
          render :dead_again
        end
        # defaults to show page
      end
      format.json { render json: @zombie }
    end
  end
end
```

For only JSON responses include status
```ruby
class ZombiesController < ApplicationController
  def show
    @zombie = Zombie.find(params[:id])
    render json: @zombie, status: :ok #ok is default
  end
end

# other statuses
#   render json: @zombie.errors, status: :unprocessable_entity
#   render json: @zombie, status: :created, location: @zombie
#
#   :unauthorized #401
#   :processing   #102
#   :not_found    #404
```

## Rails Ajax Requests

```html
<% @zombies.each do |zombie| %>
  <%= div_for zombie do %>
    <%= link_to "Zombie #{zombie.name}", zombie %>
    <div class="actions">
      <%= link_to 'edit', edit_zombie_path(zombie) %>
      <%= link_to 'delete', zombie, method: :delete, remote: true %> </div>
  <% end %>
<% end %>
```
`remote: true` sets up an ajax called

In the controller add the respond to js to accept the remote call

```ruby
class ZombiesController < ApplicationController
  def destroy
    @zombie = Zombie.find(params[:id])
    @zombie.destroy
    respond_to do |format|
      format.html { redirect_to zombies_url }
      format.json { head :ok }
      format.js
    end
  end
end
```

in the javascript file add the handler for the event

```javascript
// app/views/zombies/destroy.js.erb
$('#<%= dom_id(@zombie) %>').fadeOut();
```

## Rails 4 features

Routing concerns allows common nested routes to be placed in concerns for use in other resources

```ruby
concern :sociable do
  resources :comments
  resources :categories
  resources :tags
end
resources :messages, concerns: :sociable
resources :posts,    concerns: :sociable
resources :items,    concerns: :sociable
```

All sociable resources are nested in the other resources that call on them.

**Scope Updates**

```ruby
# old syntax
scope :sold, where(state: 'sold')
default_scope where(state: 'available')

# new syntax using procs
scope :sold, ->{ where(state: 'sold') }

# defauts take proc or block
default_scope { where(state: 'available') }
default_scope ->{ where(state: 'available') }
```

**`.none`**

`.none` creates an empty active record relation which is useful if you want to send an bad recored through validation without erring out for no method error

**`.where.not`**

used for making a `!=` query

**included tables query**

to search a related table you must reference the table using `.references`
```ruby
Post.includes(:comments).
  where("comments.name = 'foo'").references(:comments)

# alternatively
Post.includes(:comments).
  where(comments: {name:  'foo'})

```

## Strong parameters

**Whitelisting Parameter**

when parameters are sent to the controller the params can be evaluated like so...

```ruby
def update
user_params = params.require(:user).permit(:name)
end
```

This requires the params to include a :user hash and within that hash only the :name key is added to user_params

unpermitted params are logged and error can be raised

```ruby
# config/application.rb

# add this line if you want to raise errors for unpermitted params

config.action_controller.action_on_unpermitted_parameters = :raise
```
> For more info, check https://github.com/rails/strong_parameters

Whitelisting is usually done in a private method within the controller like so...

```ruby
def create
  @user = User.new(user_params)
@user.save
  redirect_to @user, notice: 'Created'
end

def update
  @user.update(user_params)
  redirect_to @user, notice: 'Updated'
end

private

def user_params
 params.require(:user).permit(:name) # add whatever is to be whitelisted here
end

```

## Sessions


Typically sessions set like so.
```ruby
# controllers/sessions_controller.rb

def create
   ...
   session[:user_id] = user.id
end

# controllers/application_controller.rb

def current_user
  @current_user ||= User.find(session[:user_id])
end
helper_method :current_user

```

This adds secret key to cookie, but should move it to an environment variable if stored on github so the id can't be pulled out of the cookie.

```ruby
# in config/initializers/secret_token.rb
 MyApp::Application.config.secret_key_base = '7014c379b14b6f4(...)'
# secret key for the application created and shown here

# However, it's best to Store and read the secret key from an environment variable
# in config/initializers/secret_token.rb
 MyApp::Application.config.secret_key_base = ENV['SECRET_KEY_BASE']
# add 'SECRET_KEY_BASE' to environment variable lists
```

## views

Collection items for Many to one relationships
```ruby
collection_select(:item, :owner_id, Owner.all, :id, :name)
collection_radio_buttons(:item, :owner_id, Owner.all, :id, :name)
collection_check_boxes(:item, :owner_id, Owner.all, :id, :name)
```

## Etag

```ruby
class ItemsController < ApplicationController
  etag { current_user.id } # Sets etag for user id
  etag { current_user.age } # Sets etag for user age
  def show
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
  def edit
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
  def most_recent
    @item = Item.find(params[:id])
     fresh_when(@item)
  end
end

# same as
# fresh_when([@item, current_user.id, current_user.age])
```

## ActiveController::Live

```ruby
class ItemsController < ApplicationController
  include ActionController::Live
end
```

This allows the following to stream a message
```ruby
def show
  response.headers["Content-Type"] = "text/event-stream"
  # set before the stream
  3.times{
    response.stream.write "Hello, browser!\n"
    sleep 1
  }
  response.stream.close #closes stream
end
```

**Eventsource**

```html
<!-- views/owners/show.html.erb -->

<ul id="items"></ul>

```

```javascript
// assets/javascripts/owners.js
$(document).ready(initialize);
function initialize() {
  var source = new EventSource('/items/events');
  // connects to the path
  source.addEventListener('message', update);
  // calls update everytime message received
};

function update(event) {
  var item = $('<li>').text(event.data);
  $('#items').append(item);
}
```

Used mainly with Redis

```ruby
def events
  response.headers["Content-Type"] = "text/event-stream"
  redis = Redis.new
  redis.subscribe('item.create') do |on|
    on.message do |event, data|
      response.stream.write("data: #{data}\n\n")
    end
  end
  response.stream.close
end
```

**Turbolinks**

updates only the body and title for each link.  However for Jquery to work, you need to add

```ruby
# gemfile
gem 'jquery-turbolinks'

# and update assets in this order
 //= require jquery
 //= require jquery.turbolinks
 //= require jquery_ujs
 //= require turbolinks
```
this will listen for the page load event for jquery

there may be some latency so a loading message may be good to add.

```
-  views/layouts/application.html.erb
<div id="loading">Loading...</div>

- assets/stylesheets/application.css

#loading {
font-size: 24px;
color: #9900FF;
display:none; # hiddent
}
```
in the application js

```javascript
// for slow loading it will show while it is being fetched
$(document).on('page:fetch', function() {
  $('#loading').show();
});

// then hide once the page changed
$(document).on('page:change', function() {
  $('#loading').hide();
});
```

To force full page refresh,

`<%= link_to 'Requests', requests_path, "data-no-turbolink" => true %>`


## Class Methods to clean up controllers

```ruby
self.class.transaction do
  #Multiple Update calls
end
```
If any of the update calls fail the transaction is rolled back to its previous state.


## Decorators

Decorator classes are used to get view logic out of models and into a Decorator class for that model.  These should go into a decorators directory which rails will load automatically

## Serialization

Serializers are used for creating json outside of contollers and active record models.  To use set up the gem file like so.

```ruby
gem 'active_model_serializers', github: 'rails-api/active_model_serializers' #most recent updates
#gem 'jbuilder', '~> 1.2' (comment this out as it will conflict)
```

To create a Serializer

`rails g serializer <Model name>`

Create custom serializer methods.

```ruby
class ItemSerializer < ActiveModel::Serializer
  attributes :id, :name, :url #attributes to serialize in the .to_json format
  def url # custom method
    item_url(object)
  end
end

```

## fine tuning

Using Select in a query

`items = Item.select(:id).where('due_at < ?', 2.days.from_now)` only returns the ID from the object meeting the condition.  These return active record objects.

Using Pluck however only returns an array of values, so `ids = Item.where('due_at < ?', 2.days.from_now).pluck(:id)` which can help perfomance if all you need are values.  Pluck can take multiple arguments.

## Configuring sensitive info for server logs.

```ruby
# in config/application.rb
config.filter_parameters += [:password, :ssn]
# now ssn will show [Filtered]
```

## Replacing WEBrick for production

```ruby
# in gem file

gem 'puma'
```
Puma is faster for critical applications
