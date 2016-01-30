##Validations 

An operations lets you define a form object, or contract. Internally, this simply creates a Reform
form class.
```
class Create < Trailblazer::Operation
 contract do
  property :body, validates: {presence: true}
  property :author, validates: {association: true}
 end
 def process(params)
  # ..
end
```

As soon as you define a contract for an operation you can use it for deserializing the incoming
parameters and validate the object graph that was created. This sounds crazy but is extremely simple.
Check out the #process method now.( page 28 )

In order to do a update, just do this:
```
1 class Update < Create
2   action :update
3
4   contract do
5     property :author, readonly: true # make author read-only for update.
6   end
7 end
```

## Builders

Keeping that knowledge in the operation and out of the caller environment like the controller allows
to use your domain virtually everywhere without having to replicate logic.
```
1 class Create < Trailblazer::Operation
2  builds ->(params) do
3   Moderated if params[:current_user].needs_moderation?
4  end
5
6  class Moderated < self
7   # ..
8  end
9 end
```


Just like the operation, the preceding builder has access to the params passed into the operation call.
It’s up to you what authorization framework you use here. Use cancan, rolify, pundit, or write your
own, everything is allowed (or denied). We’ll check out several authorization gems while building
our app with this book."

## Rendering Forms

To retrieve the form object from the Operation you can use its ::present method. In this example,
I access the operation’s form object in a controller action to render it.


While it might be sufficient for simple operations to find the model (or multiple models) in
the process method, it is always better to do just that in a separate place. The operation will
automatically run your retrieval logic when you put it into #model!, which makes it reusable when
presenting the operation’s form, too.

```
1 class Comment::Update < Create
2 # inherit contract
3
4  def process(params)
5   @model = Comment.find(params[:id]) # The method model replace that
6
7   validate do |f|
8  # and so on..
9   end
10 end
11end
```

```
1 class Comment::Update < Create
2  def process(params)
3   validate do |f|
4    # and so on..
5   end
6 end
7
8 private
9  def model!(params)
10  Comment.find(params[:id])
11 end
12end
```
## Representers
When defining properties in a contract Reform internally uses the Representable gem to create a representer for the form.

## Controllers
```
1 class ThingsController < ApplicationController
2  def create
3    run Thing::Create
4  end
5 end
```
This runs the operation and then returns flow control back to the controller (line 3). It’s probably
easier to understand when I show you what happens inside #run. Here’s what basically happens
when you use run in a controller.

```
Thing::Create[params]
```

```
1 class Create < Trailblazer::Operation
2  include CRUD
 ### This below enables me to remove thing = Thing.new on process, which changes validate to validate(params[:thing]) . Instead of validate(params[:thing], thing)
3  model Thing, :create 
4
5  contract do
6  property :name
7  property :description
8
9  validates :name, presence: true
10 validates :description, length: {in: 4..160}, allow_blank: true
11 end
12
13  def process(params)
14   validate(params[:thing]) do |f|
15    f.save
16   end
17  end
18 end
```


```
## form 
1class ThingsController < ApplicationController
2 def new
3  form Thing::Create
4 end
```
Calling #form in the controller will instruct the operation to only instantiate its model without
running any processing code. The operation will create or find the respective model for you as this
reuses the same mechanics from the creating process we discussed earlier.

## Update

```
1 class Update < Create
2  action :update # This tells me that he's going to find a object model, not create one.
3
4  contract do
5   property :name, writeable: false # He put writeable false, because he's going to use name as url, so it can't change
6  end
7 end
```

## Cell ( view models )

```
 concept("thing/cell", Thing.last)
``` 
 This will result in a class lookup for Thing::Cell. It then instantiates this cell and passes in the
 remaining arguments and invokes the cell’s show method. So, the concept call basically gets translated to something along this.

 1 Thing::Cell.new(Thing.last).show
 
 This is not 100% what happens, but it helps understanding the workflow when rendering a cell. We’ll
 learn more about different call styles in later chapters.
 
 Example of cell class:
 
```
 1 class Thing::Cell < Cell::Concept
 2  def show
 3   render
 4  end
``` 
 This is a fundamental change to the way Rails handles views. In Cells, the concept of “helpers” does
 not exist anymore. The handy Rails helpers are still available, but these are all instance methods of
 the cell class.
 
``` 
 1 class Thing::Cell < Cell::Concept
 2  def show
 3   render
 4  end
 5
 6 private
 7 def name_link
 8   link_to model.name, thing_path(model)
 9 end
 10
 11 def created_at #This can be remove if you use property, example below.
 12   model.created_at
 13 end
``` 

``` 
 1 class Thing::Cell < Cell::Concept
 2  property :name
 3  property :created_at
```
 Using property will create the delegation for us and saves typing. Internally, property simply
 defines the method created_at in the cell class almost exactly as we did it manually a few minutes
 ago.
 
 Rendering collections
``` 
 1 %h3
 2 Welcome to Gemgem!
 3
 4 .row
 5 = concept("thing/cell", collection: Thing.latest)
```

## Additional optional

1 concept("thing/cell", thing, border: 1, show_image: true) #Last two parameters says which are my options
  We can now access the additional options in the cell via the options method.
```
1 def show
2   options #=> {border: 1, show_image: true}
3 end 
```
----------------------------------------------------------------------------------------------------------------------------

## Nested Forms

He stated that he hates nested forme, and found an away of getting out of this.

"To compute the URL for our form, we need a reference to the outer “resource”, too. One of the many
problems nested resources create is dependencies. Even though this is purely UI-related logic, and
we only need this instance variable in the view for URL generation, this feels wrong. Anyway, bear
with it for this chapter, we will fix it very soon."

```
1 class Comment < ActiveRecord::Base
2  class Create < Trailblazer::Operation
3   include CRUD
4   model Comment, :create
5
6  def process(params)
7   validate(params[:comment]) do |f|
8    f.save
9   end
10 end
11
12 private
13  def setup_model!(params)
14   model.thing = Thing.find_by_id(params[:thing_id])
15  end
```


The setup_model! method is the perfect place to prepare your model before it gets validated. This
hook is run directly before process is invoked, but after the generic model was created, making it
the ideal location to add or change attributes of the CRUD model. In case you’re interested, check
out Operation#setup! where all the prepping happens.


## Nested form contract
```
1 contract do
2  property :body
3  property :weight
4  property :thing
5
6  validates :body, length: { in: 6..160 }
7  validates :weight, inclusion: { in: ["0", "1"] }
8  validates :thing, :user, presence: true
9
10  property :user do
11   property :email
12   validates :email, presence: true, email: true
13  end
14 end
```
He says that, when he tried to creted nested form, the user was not render, cause @form object was empty - he doesnt know what to get,
so i have to prepopulate it.
```
1 property :user, prepopulator:      ->(*) { self.user = User.new },
2                 populate_if_empty: ->(*) { User.new } do
3  property :email
4 end
```
In :populate_if_empty, you simply return a model instance from the block. Reform will automatically
wrap the model with a nested form for you and attach it to the parent form. This is because the
:populate_if_emtpy option is run per missing form, Reform knows that is has to wrap the return
value with a nested form.

## Composed views

The present code is almost identical to form, which we used in chapter 3 to retrieve the form object of the operation.
present only run update with readonly params, it doesnt run process.

As you can see, the controller present helper really just forwards the params hash to the operation’s
same-named method, which will solely find the model and then return itself (line 2).

```
1 def present(*) 
2  op = Thing::Update.present(params)
3
4  @model = op.model
5  @thing = @model
6 end
```

## Trailblazer:
  ---> Models becomes only: Retrieving and writing data to a database
  ---> You have a concept for every high-level domain, which is operation which server-side do to delivery data to client.
  ----> Controller only gets http questions, and passes its business-logic to operation.
  ----> To retrieve the form object from Operation you can use its ::present method.
  
  Contract works as validators, they do not touch on model at all.
  Builders works permission handling.
  Representers do Rendering and parsing documents in Trailblazer.
  
  
  About process method:
  "While it might be sufficient for simple operations to find the model (or multiple models) in
the process method, it is always better to do just that in a separate place. The operation will
automatically run your retrieval logic when you put it into #model!, which makes it reusable when
presenting the operation’s form, too." 
  
  
  Redenring views:
  "Running the operation will yield the instance to the block
  which is only executed for a successful validation. The #run call also assigns @model and @operation
  instance variables in case you need them in your view."
  
  Cells:
   ---> It's kind of object of the view.
  "cells allow you to define properties of the wrapped model. A
  property in cell is automatically exposed as a reader to the view."
  

## Important
  Every operation validates its input using a form object.
  
  Representers can be used to parse incoming documents and to render API representations.
  Since form objects internally use representers they can be used for that directly without having
  to specify a representer Controller only knows http layer.
  
  
