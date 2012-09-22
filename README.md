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

## New Juicy Features

* Added support for checkbox groups field forms for 1:N (has_many) and M:N (has_and_belongs_to_many) associations.

*Note* 
 * *We are making regular updates to stay up-to-date with the source sferik's [Rails Admin repo][rails-admin]. And yes we will make pull requests to it, but this will take some time to make it in a clean way...*

[rails-admin]: https://github.com/sferik/rails_admin

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
