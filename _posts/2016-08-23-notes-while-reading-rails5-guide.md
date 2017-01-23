---
title: My Notes While Reading Rails5 guide
status: true
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

# My Scrap notes while reading official Rails 5 Guide

## ActiveRecord Basics

User.find_by(:name => "a", :age => "b")

User.new do |u|
    u.name="a"
    u.age="b"
end

User.all # activerelation
User.none # activerelation

rails g model Comment body:text user:references
this creates user_id with index and foreign key restriction

table name is plural - UserPaw & model name is plural - user_paws

rails g resource Food # generates model and controller, but no views

## ActiveRecord Migrations

 With reversible, you only need to define the inverse of the instructions that need it, while still enjoying the benefits of Active Record automatically taking care of the rest of them.
https://www.reinteractive.net/posts/178-reversible-migrations-with-active-record

````
def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
````

If the migration name is of the form "AddXXXToYYY" or "RemoveXXXFromYYY" and is followed by a list of column names and types then a migration containing the appropriate add_column and remove_column statements will be created.

rails generate migration AddPartNumberToProducts part_number:string:index user:references

````
def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
    add_reference :products, :user, foreign_key: true
 end
````
 
Specify column details when doing a remove and this will make it reversible
  
rails g migration CreateJoinTableCustomerProduct customer product
 
````
 def change
     create_join_table :customers, :products do |t|
       t.index [:customer_id, :product_id]
       t.index [:product_id, :customer_id]
     end
   end
  ````
  
  create_table :products, :comment => 'This is the products table'
  
  change_table can do what create_table does and more.
  
  change_table :products do |t|
    t.remove name
    t.string title
    t.rename :price, :retail_price
  end
  
  
change_column_null :products, :name, false
change_column_default :products, :approved, from: true, to: false # and this is reversible

add_foreign_key :articles, :authors

Product.connection.execute("UPDATE products SET price = 'free' WHERE 1=1") # returns array

methods supported in a `change_table` block

add_column
add_foreign_key
add_index
add_reference
add_timestamps
change_column_default (must supply a :from and :to option)
change_column_null
create_join_table
create_table
disable_extension
drop_join_table
drop_table (must supply a block)
enable_extension
remove_column (must supply a type)
remove_foreign_key (must supply a second table)
remove_index
remove_reference
remove_timestamps
rename_column
rename_index
rename_table
  
````
create_table :accounts do |t|
  t.belongs_to :supplier, index: true, unique: true, foreign_key: true
end
````  

`alias :belongs_to :references`

`revert MigrationName`
or
`revert do end`
it automatically reverses the code in it. it figure out the opposite command the opposite order.

rails db:setup task will create the database, load the schema and initialize it with the seed data.
db:reset == rails db:drop db:setup

say_with_time do end
say

db/schema.rb - db independent, expressed in ruby. trade off - can't have db specific things
db/structure.sql - db specific, expressed in sql. it can have special things to a db. trade off - is not going to work in a different database

rake db:seed -> write any code in db/seed.rb and it will run. this code be any code. it could call anything, get data and put it in. its like a dedicated rake task for a specific purpose

## ActiveRecord Validations

These methods will trigger a validation check and ones with a !(bang) will raise error upon failure
- create
- create!
- save
- save!
- update
- update!

these skip valiations:
- decrement!
- decrement_counter
- increment!
- increment_counter
- toggle!
- touch
- update_all
- update_attribute
- update_column
- update_columns
- update_counters

also obj.save(:validate => false) will skip validation

## Validations

validates_associated :books
will run validation on the associated records as well

validates :email, confirmation: true
creates virtual field with `_confirmation` and checks for them both to be equal

validates :bla, :exclusion => { :in => %w(a b c} } 
validates :bla, :absence => true

uniqueness does not guarantee against two workers, so do create a unique index

````
validates_each :name do |record, attr, value|
    record.errors.add(attr, 'no good') if value = "bad val"
end
````

Validations by default run on save. but this can be customized by saying :on => :create or :on => :update
it can be also be set to run on a custom context. this custom context will then need to be passed when valid of save is called
:on => :account_setup
obj.valid?(context: account_setup)

they can be made strict so they raise an error rather just adding a validation error to the errors array.

## ActiveRecord Callbacks
    
`after_validation` callback can take `:on` and be run on certain events like `:create`, `update` etc.

You can pass `touch` to a belongs_to. This will touch the parent record when the child is touched, firing the `after_touch` callbacks of both of them 
`belongs_to :company, touch: true`
 
 Queue - model's validations, the registered callbacks, and the database operation.
 
In before callbacks - raise any error or return false to halt the callback issue a rollback
In after callbacks - raise any error to halt and issue rollback

Any other exception besides ActiveRecord::Rollback or ActiveRecord::RecordInvalid will be re-raised breaking the code.

Callback classes. a class can be defined with the methods with same name as of the callback and then an instance of the class can be passed as the parameter to the callback. this will cause the appropriate method in the class to be called and pass in the model instance as a parameter

````
class Picture
    before_save PictureCallbacks.new 
end

class PictureCallbacks

    def before_save(rec)
        puts rec.name 
        if name = "blank"
            return false
        end
    end

end
````

Use transaction callbacks like `after_commit` and `after_rollback` to run code which communicates with an external system and should not be part of the current transaction.

## ActiveRecord Associations

with polymorphic relationships a model can belong to more than one model. all it needs to have is a type and an id.

````
create_table :pictures do |t|
  t.string  :name
  # t.integer :imageable_id
  # t.string  :imageable_type
  t.references :imageable, polymorphic: true, index: true
  t.timestamps
end
# add_index :pictures, [:imageable_type, :imageable_id]
````

````
class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end
 
class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end
````

Self Joins 

````
class Employee < ApplicationRecord
  has_many :subordinates, class_name: "Employee",
                          foreign_key: "manager_id"
 
  belongs_to :manager, class_name: "Employee"
end
````

````
create_table :employees do |t|
  t.references :manager, index: true
  t.timestamps
end
````

Using `inverse_of` ensures that we don't load multiple copies of the same object. 
u = User.first
v = u.visitors.first
v.user  == u

Immediate associations do not need to be included as they are automatically loaded.

Association callbacks - before save etc.

````
has_many :books, before_add: :check_credit_limit
 
  def check_credit_limit(book)
    ...
  end
 ````
  
 Association Extensions
 
 STI migration help 
 rails generate model car --parent=Vehicle
 
 Client.find([1, 10])
 SELECT * FROM clients WHERE (clients.id IN (1,10))

`find_by` is equivalent to `where` with a `first`

`find_each` works on relations but applies order.
If an order is present in the receiver the behaviour depends on the flag config.active_record.error_on_ignored_order. If true, ArgumentError is raised, otherwise the order is ignored and a warning issued, which is the default.

`where.not`

remove the uniqueness constraint:
````
query = Client.select(:name).distinct
# => Returns unique names
 
query.distinct(false)
# => Returns all names, even if there are duplicates
z
````

`unscope`

````
Article.where('id > 10').limit(20).order('id asc').unscope(:order)`
Article.where(id: 10, trashed: false).unscope(where: :id)

````

`only`

 override conditions using the only method. For example:

Article.where('id > 10').limit(20).order('id desc').only(:order, :where)

`reorder`

`reverse_order`

`rewhere`

`Article.none`

`readonly`

Optimistic locking - add `lock_version` coloumn, it will check out data with value of type column incremented, and when updating if the value you have is lower than what it has, it will reject the update.

Pessimistic locking - it used mechanisms provided by the database. used in a transaction to prevent deadlocks. Using the `lock` gets an exclusive lock on the rows for update.

````

Item.transaction do
  i = Item.lock.where(:id => 1) # select * from items where id=1 for update
  i.name = 'Jones'
  i.save!
end
````

`joins` - used for inner joins
`left_outer_joins`

return all authors with their count of posts, whether or not they have any posts at all"
`Author.left_outer_joins(:posts).distinct.select('authors.*, COUNT(posts.*) AS posts_count').group('authors.id')`

`Client.includes(:address)`
we use plural `includes` and specify associations as singular

Even though Active Record lets you specify conditions on the eager loaded associations just like joins, the recommended way is to use joins instead.

use `references` to force joined tables:
Article.includes(:comments).where("comments.visible = true").references(:comments)

We can mix and match scope and where conditions and the final sql will have all conditions joined with AND.
If we do want the last where clause to win then Relation#merge can be used.
`User.active.merge(User.inactive)`

default_scope will be prepended in scope and where conditions.

`unscoped`

`Enum`
vales map to integer in db but names in AR queries

````
class Conversation < ActiveRecord::Base
  enum status: { active: 0, archived: 1 }
end

c = Conversation.create(:status => :archived)
c.status # 1 or archived
c.archived? #true
c.active? #false
c.active! #true
c.active? #true
````

`find_or_create_by` method checks whether a record with the specified attributes exists. If it doesn't, then create is called.
User `create_with` to specify things you want to set in the record but don't want to query on it.
`User.create_with(:name => "s").find_or_create_by(:age => 25)`
Here we look up on `age` but when its come to creating we set `name` and `age`

The `find_by_sql` method will return an `array` of objects of **instantiated** `model` objects, even if the underlying query returns just a single record.
`select_all` is similar to `find_by_sql` but returns an array of hashes rather than instantiated model objects.

`pluck` returns an array with values to be the column plucked. it only queries the columns needed and never converts to a model instance, both making it much more efficient. When plucking multiple columns it will give an array of arrays. PS: cant chain these.

`ids`, `any?`, `many?`, `exists?`

`include ActiveModel::AttributeMethods` can be used to define dynamic methods for multiple attributes on regular objects
`include ActiveModel::Callbacks` to setup callbacks on regular objects
`include ActiveModel::Dirty`
`include ActiveModel::Validations`
`extend ActiveModel::Naming`
`include ActiveModel::Serialization`





















