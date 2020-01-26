# CMS9

![CMS9 Logo](https://raw.githubusercontent.com/klikaba/cms9/master/app/assets/images/cms9/cms9_logo_readme.png)

[![Gem Version](https://badge.fury.io/rb/cms9.svg)](https://badge.fury.io/rb/cms9)

A small CMS Admin module for developers.

[Demo Application](https://github.com/klikaba/cms9-demo) running on CMS9.

## What is CMS9

CMS9 is not a standard CMS. CMS9 is targeted towards developers to ease the process of embedding CMS functionality into any site.

CMS9 provides an administrator dashboard where a user can:

* Define different types of forms
* Add user-defined fields to forms
* Manage contents of each form

Currently, we support the following field types:

* Text field
* Text Area (using ckedit, a WYSIWYG text editor)
* Number
* Choose Single
* Choose Multiple
* Date
* Time
* Date & Time
* Image

CMS9 does not have any functionality with regards to displaying content, as its primary task is to provide an administrator dashboard where data can be defined and managed. On the other hand, it exposes a simple interface to access that data.

### Features

* Support for Rails 5 and above
* Automatic form validation
* Event history (records actions like creation, update and deletion of post types and their posts)
* Customizable authentication (via [Devise](https://github.com/plataformatec/devise) or similar frameworks)
* [Dragonfly](https://github.com/markevans/dragonfly) for handling images and other attachments
* [Ckeditor](https://github.com/galetahub/ckeditor) as the default WYSIWYG text editor
* Bootstrap 3 and above with the Admin LTE template

## Getting started

Add CMS9 to your Gemfile:

``` ruby
# Gemfile
gem "cms9"
```

Run the installer:

``` bash
rails generate cms9:install [DEF_ROUTE]
```

`[DEF_ROUTE]` is optional and presents where your CMS9 route will be mounted, by default it is `/cms9`.

The `install` generator will mount the CMS9 route, add the `current_user` configurator initializer and additional configuration for Ckeditor.

Then run:

``` bash
rails db:migrate
```

Finally, open <http://localhost:3000/cms9> to see your new CMS9 dashboard in action.

## Managing content

Content is managed by creating post types, also known as post definitions. A post type represents a form for specific data. In the case of news websites, it can be a `News Form` that has multiple fields for `Title`, `Content`, `CoverImage` and `PublishedStatus`. Each of those fields can be created from the dashboard.

Once a Post Type with a specific name is created and all of the fields are added, a CMS9 administrator can start creating posts based on those post types.

CMS9 does not have any functionality to display content but exposes a simple interface for retrieving data. For example, you can access all of the posts on a news website with:

``` html
<% @CMS9::Post.each do |post| %>
  <%= post.field('Title) %>
<% end %>
```

You can make any kind of layout for your posts and show them however you want. Once you have made a simple layout, you are ready to create as many posts as you want. It's that easy.

## Data interface

All data interface classes are `ActiveRecord` models.

`CMS9::PostDefinition` represents a post definition, like `News`, `Job`, or `Ad`, with each post definition containing multiple fields (`CMS9::PostField`). `CMS9::Post` represents a post, with each post belonging to a post definition. `CMS9::Field` represents the value of a field, with that field belonging to a post.

### CMS9::PostDefinition

There are two methods: `fields()`, which returns a list of fields that are in a post definition, and `field(name)`, which returns a field by its name.

Here's an example. If a user created a post definition called `News` with `Title` and `Content` fields, we can retrieve it by:

`CMS9::PostDefinition.all` - returns all post definitions

`CMS9::PostDefinition.all.first.fields()` - returns all fields for this post type

`CMS9::PostDefinition.all.first.field('Title')` - returns field that describes field named `Title`

### CMS9::PostField

The `CMS9::PostField` class represents the definition of the `Post Type` field. Each class has a name (unique in the context of a single post definition), a type and extra parameters depending on its type.  
Currently, CMS9 supports the following types: `Text Field`, `Text Area` (WYSIWYG text area using Ckedit), `Number`, `Choose Single`, `Choose Multiple`, `Date`, `Time`, `Date & Time` and `Image`.

### CMS9::Post

`field(name)` - returns `CMS9::Field` by name

### CMS9::Field

The `CMS9::Field` class represents the value of a particular field. Depending on the `PostField` type, the `value` column is serialized differently.

`cms9\_field(field)` can be applied to `CMS9::Field` for easier data extraction/display.

## Configuration

Authorization should be added using the `current_user` method. If you pass a block, it will be triggered through a before filter on the first action in CMS9.

To begin with, you may be interested in setting up [Devise](https://github.com/sferik/rails_admin/wiki/Devise) or something similar.

After, in your user model, you need to implement method `cms9_admin?`, which will be used to recognize users and give them permission to access CMS9 dashboard:

``` ruby
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  def cms9_admin?
    return true     # All users have access to CMS9 dashboard
  end
end
```

In `config/initializers/cms9_configurator.rb`, CMS9 passes Devise's `current_user` method (or whatever you use to recognize the user) and `destroy_user_session_path`, which is the default path for deleting user sessions (logging out):

``` ruby
CMS9.configure do |config|
  config.current_user =  :current_user
  config.destroy_user_session =  :destroy_user_session_path
end
```

---

**Note**  
In Devise, the `current_user` method and `destroy_user_session_path` are used by default, so they will work without passing them to CMS9 configuration. If you are using something else, be sure to pass that method and path.

---

### Dependencies

Install ImageMagick for Dragonfly's image processing. If you plan on using Dragonfly's data stores (which are not included in the core of Dragonfly), you will need to include them in the Gemfile. Here are the appropriate links for [Amazon S3](https://github.com/markevans/dragonfly-s3_data_store), [Couch](https://github.com/markevans/dragonfly-couch_data_store) and [Mongo](https://github.com/markevans/dragonfly-mongo_data_store). In case you need a data store which is not listed, you can also [build a custom data store](http://markevans.github.io/dragonfly/data-stores/#building-a-custom-data-store).

### Dragonfly data stores

By default, Dragonfly uses `datastore:file`. If you plan on using any other data store, after including the gem in the Gemfile and installing it, you will need to override Dragonfly's configuration.

For example, if you are going to use Amazon S3 as your default data store, you need to make an initializer (for example, `config/initializers/init_dragonfly_s3.rb`) which will override the default configuration (one for the Dragonfly uploader, and one for the Ckeditor assets uploader):

``` ruby
require 'dragonfly/s3_data_store'

Dragonfly.app.configure do
  ...
  datastore :s3,
    bucket_name: 'mybucket',
    access_key_id: 'my_access_key_id',
    secret_access_key: 'my_secret_access_key'
  ...
end

Dragonfly.app(:ckeditor).configure do
  ...

  datastore :s3,
    bucket_name: 'mybucket',
    access_key_id: 'my_access_key_id',
    secret_access_key: 'my_secret_access_key'

  ...
end
```

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
