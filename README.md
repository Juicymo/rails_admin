# Rails Admin 

Rails Admin is a Rails engine that provides an easy-to-use interface for managing your data.

It started as a port of [MerbAdmin][merb-admin] to Rails 3 and was implemented
as a [Ruby Summer of Code project][rubysoc] by [Bogdan Gaza][hurrycane] with
mentors [Erik Michaels-Ober][sferik], [Yehuda Katz][wycats], [Luke van der
Hoeven][plukevdh], and [Rein Henrichs][reinh].

We at Juicymo [Juicymo][juicymo] have just added few more features to this wonderful tool.

[juicymo]: https://github.com/juicymo
[merb-admin]: https://github.com/sferik/merb-admin
[rubysoc]: http://www.rubysoc.org/projects
[hurrycane]: https://github.com/hurrycane
[sferik]: https://github.com/sferik
[wycats]: https://github.com/wycats
[plukevdh]: https://github.com/plukevdh
[reinh]: https://github.com/reinh

***

## Installation

Add this to your `Gemfile`:

```ruby
gem 'rails_admin', :git => 'git://github.com/Juicymo/rails_admin.git'
```

Run:

```bash
$ bundle install
```

Configure your database in the `database.yml` file.

Then run installation script:

```bash
$ rails g rails_admin:install
```

Then finally migrate your database by:

```bash
$ bundle exec rake db:migrate
```

## Usage

Start the server:

```bash
$ rails server
```

You should now be able to administer your site at
[http://localhost:3000/admin](http://localhost:3000/admin).

***

## Test

The empty administraiton interface just with the User model is nice, but how to add some more features to it?

Try this:

```bash
$ rails generate model Lecture name:string perex:text
```

```bash
$ rails generate model Room name:string description:text
```

```bash
$ rails generate migration AddSlidesAndRoomToLecture slides:integer room:references
```

Because of limitation in the default rails migration generator when comes to `:references` field type, we need to adjust the
generated migration a bit. Just replace content of file `db/migrate/...add_slides_and_room_to_lecture.rb` with this:

```ruby
class AddSlidesAndRoomToLecture < ActiveRecord::Migration
  def up
    change_table :lectures do |t|
      t.references :room
      t.integer :slides
    end
  end

  def down
    remove_index :lectures, :room_id
    remove_column :lectures, :room_id
    remove_column :lectures, :slides
  end
end
```

Now migrate your database once more by:

```bash
$ bundle exec rake db:migrate
```

Now setup you associations:

To `models/lecture.rb` add:

```ruby
belongs_to :room

attr_accessible :name, :perex, :slides, :room_id
```

To `models/room.rb` add:

```ruby
has_many :lectures

attr_accessible :description, :name, :lecture_ids
```

Now start the server once more:

```bash
$ rails server
```

Now, do you want to have checkboxes instead of the ugly M:N form widget for the lectures field in the add/edit room form? No problem, just add this to `config/initializers/rails_admin.rb`:

```ruby
config.model Room do
  edit do
    group :default do
      field :name
      field :description
      field :lectures do
        partial "form_checkboxes_multiselect"
      end
    end
  end
end
```

***

## New Juicy Features

* Added support for checkbox groups field forms for 1:N (has_many) and M:N (has_and_belongs_to_many) associations.

*Note* 
 * *We are making regular updates to stay up-to-date with the source sferik's [Rails Admin repo][rails-admin]. And yes we will make pull requests to it, but this will take some time to make it in a clean way...*
 * *Rest of this readme below this line is currently just copied from the source [Rails Admin repo][rails-admin]...*

[rails-admin]: https://github.com/sferik/rails_admin

***

## Announcements

* for those with `rake db:migrate` errors, update to master and check that you see the line: "[RailsAdmin] RailsAdmin initialization disabled by default." when you launch the task. If not (or if migrations still don't work), open a ticket with an application on Github that can reproduce the issue.

* `config.models do ... end` is deprecated (note the 's' to models, `config.model(MyModel) do .. end` is fine), for performance reasons (forces early loading of all application's models). Duplicate to each model instead, before next release. If you really need the old behavior:
 
```ruby
config.models.each do |m|
  config.model m do
    # <<<< here goes your code
  end
end
```

## Features

* Display database tables
* Create new data
* Easily update data
* Safely delete data
* Custom actions
* Automatic form validation
* Search and filtering
* Export data to CSV/JSON/XML
* Authentication (via [Devise](https://github.com/plataformatec/devise))
* Authorization (via [Cancan](https://github.com/ryanb/cancan))
* User action history (internally or via [PaperTrail](https://github.com/airblade/paper_trail))
* Supported ORMs
  * ActiveRecord
  * Mongoid [new]

## Demo

Take RailsAdmin for a [test drive][demo] with sample data. ([Source code.][dummy_app])

[demo]: http://rails-admin-tb.herokuapp.com/
[dummy_app]: https://github.com/bbenezech/dummy_app

## Installation

In your `Gemfile`, add the following dependencies:

    gem 'fastercsv' # Only required on Ruby 1.8 and below
    gem 'rails_admin'

Run:

    $ bundle install

And then run:

    $ rails g rails_admin:install

This generator will install RailsAdmin and [Devise](https://github.com/plataformatec/devise) if you
don't already have it installed. [Devise](https://github.com/plataformatec/devise) is strongly
recommended to protect your data from anonymous users. Note: If you do not already have [Devise](https://github.com/plataformatec/devise) 
installed, make sure you remove the registerable module from the generated user model.  


It will modify your `config/routes.rb`, adding:

```ruby
mount RailsAdmin::Engine => '/admin', :as => 'rails_admin' # Feel free to change '/admin' to any namespace you need.
```

Note: Your RailsAdmin namespace cannot match your Devise model name, or you will get an infinite redirect error.
The following will generate infinite redirects.

```ruby
mount RailsAdmin::Engine => '/admin', :as => 'rails_admin'
devise_for :admins
```

Consider renaming your RailsAdmin namespace:

```ruby
mount RailsAdmin::Engine => '/rails_admin', :as => 'rails_admin'
devise_for :admins
```

See [#715](https://github.com/sferik/rails_admin/issues/715) for more details.

It will also add an intializer that will help you getting started. (head for config/initializers/rails_admin.rb)


Finally run:

    $ bundle exec rake db:migrate

Optionally, you may wish to set up [Cancan](https://github.com/ryanb/cancan),
[PaperTrail](https://github.com/airblade/paper_trail), [CKeditor](https://github.com/galetahub/ckeditor), [CodeMirror](https://github.com/fixlr/codemirror-rails)

More on that in the [Wiki](https://github.com/sferik/rails_admin/wiki)

## Configuration

All configuration documentation has moved to the wiki: https://github.com/sferik/rails_admin/wiki

## Screenshots

![Dashboard view](https://github.com/sferik/rails_admin/raw/master/screenshots/dashboard.png "dashboard view")
![Delete view](https://github.com/sferik/rails_admin/raw/master/screenshots/delete.png "delete view")
![List view](https://github.com/sferik/rails_admin/raw/master/screenshots/list.png "list view")
![Nested view](https://github.com/sferik/rails_admin/raw/master/screenshots/nested.png "nested view")
![Polymophic edit view](https://github.com/sferik/rails_admin/raw/master/screenshots/polymorphic.png "polymorphic view")

## Support

If you have a question, please check this README, the wiki, and the [list of
known issues][troubleshoot].

[troubleshoot]: https://github.com/sferik/rails_admin/wiki/Troubleshoot

If you still have a question, you can ask the [official RailsAdmin mailing
list][list].

[list]: http://groups.google.com/group/rails_admin

If you think you found a bug in RailsAdmin, you can [submit an issue][issues].

## Contributing

In the spirit of [free software][free-sw], **everyone** is encouraged to help
improve this project.

[free-sw]: http://www.fsf.org/licensing/essays/free-sw.html

Here are some ways *you* can contribute:

* by using alpha, beta, and prerelease versions
* by reporting bugs
* by suggesting new features
* by writing or editing documentation
* by writing specifications
* by writing code (**no patch is too small**: fix typos, add comments, clean up
  inconsistent whitespace)
* by refactoring code
* by fixing [issues][]
* by reviewing patches
* [financially][pledgie]

[issues]: https://github.com/sferik/rails_admin/issues

## Submitting an Issue
We use the [GitHub issue tracker][issues] to track bugs and features. Before
submitting a bug report or feature request, check to make sure it hasn't
already been submitted. When submitting a bug report, please include a [Gist][]
that includes a stack trace and any details that may be necessary to reproduce
the bug, including your gem version, Ruby version, and operating system.
Ideally, a bug report should include a pull request with failing specs.

[gist]: https://gist.github.com/

## Submitting a Pull Request
1. [Fork the repository.][fork]
2. [Create a topic branch.][branch]
3. Add specs for your unimplemented feature or bug fix.
4. Run `bundle exec rake spec`. If your specs pass, return to step 3.
5. Implement your feature or bug fix.
6. Run `bundle exec rake spec`. If your specs fail, return to step 5.
7. Run `open coverage/index.html`. If your changes are not completely covered
   by your tests, return to step 3.
8. Add, commit, and push your changes.
9. [Submit a pull request.][pr]

[fork]: http://help.github.com/fork-a-repo/
[branch]: http://learn.github.com/p/branching.html
[pr]: http://help.github.com/send-pull-requests/

## Supported Ruby Versions
This library aims to support and is [tested against][travis] the following Ruby implementations:

* Ruby 1.9.2
* Ruby 1.9.3
* [Rubinius][]
* [JRuby][]

[rubinius]: http://rubini.us/
[jruby]: http://jruby.org/
