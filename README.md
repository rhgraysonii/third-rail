third-rail
==========

##voltrb on rails

The goal is to provide access to voltrb from within existing rails apps via addition of a third-rail gem.

Any thoughts or discussion is welcome at https://gitter.im/catprintlabs/third-rail


##Initial thoughts (sort of stream of conscience random ideas)

rails and volt will coexist with volt being a separate rack app.  Communication between the two will be via mongodb.

add mongoid or mongomapper to active record models ala https://github.com/hayesdavis/active-expando (very old but provides a starting point)  similar concept here: http://britg.com/2012/01/07/forging-forgecraft-a-hybrid-sql-mongodb-data-solution/

this should provide the basic communication mechanism.

##syncing the two databases

###Brute Force:
Syncronize every change between the databases using activerecord and mongos builtin monitoring...

###More Intelligent - but more work:
Describe mapping in the active record models to volt models 

###Harder still - but probably how you would do it from scratch:
Map volt models to rails, and manually connect

###Forget Mongo + Rails...  
use an automatically generated API to talk between the volt app and rails.

##Different Approach altogether

think of a typical migration.  You have some pages that are rails views, but you want to start using volt.
What you would like initially is something like this:
* request the page, 
* controller does normal stuff
* then makes sure that any "volt" models are synced to the rails data.
* then does a "special" render operation that hands control over to volt, which delivers the page


## Potential model

the model, here

```RUBY
require 'volt/models/synchronizer' # Arbitrary, could be modular

class Comment < ActiveRecord::Model
  attr_accessor :text
  volt_sync, :true
end
In rails controller:
```

would only need one line added, the volt sync option.

The Controller,

```RUBY
class CommentController < ApplicationController
  def index
    @comments = Comment.all
  end
end
```

would become

```RUBY
require 'volt' # arbitrary, could also be specfic to avoid module pattern
               # exhibited, ie `require volt/application_controller/rails_synchonizer'

class CommentController < ApplicationController
  def index
    @comments = Volt::ApplicationController::Synchronizer.new(Comment.all)
  end
end
```

And the corresponding comments view in rails, originally

```RUBY
...
  <div class='col-md-8'>
    <% @comments.each do |c| %>
      <p><%= @comment.text %></p>
    <% end %>
  </div>
...
```

becomes

```RUBY
<div class='col-md-8'>
  {{ @comments.each do |c| }}
  <p>{{ comment._text }}</p>
  {{ end }}
</div>
```

and theoretically, if a comment is updated or added it will now sync using the new layer.

This could be powered by requiring synchronization of permissions, attributes, and a mapping of
mongo id to rails' db id. The synchronization could be triggered by having an event fired upon
the change of a field in the model on the frontend provided from the controller. This could be
managed with a task queue and done with unique UPDATE queries rather than inserting the same values
again, and updating the entire model with duplicate fields.

NOTE: this is pseudocode for a good chunk and really just me thinking out loud
