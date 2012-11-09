---
layout: master
title: Rails 数据库迁移
---

## 参考资料

* [Migrations: Ruby On Rails官方网站](http://guides.rubyonrails.org/migrations.html#creating-a-migration)

## 数据库迁移


### Creating a Migration

* Create a Standalone Migration

    $ rails generate migration AddModeToStandardLogs mode:string

      invoke  active_record
      create    db/migrate/20120813055814_add_mode_to_standard_logs.rb


   $ rails generate migration AddModeToInfoLogs mode:string

    /db/migrate/20120813055814_add_mode_to_standard_logs.rb

      class AddModeToStandardLogs < ActiveRecord::Migration
        def self.up
          add_column :standard_logs, :mode, :string
        end
      
        def self.down
          remove_column :standard_logs, :mode
        end
      end

* Create a Standalone Migration with more columns.

    $ rails generate migration AddDetailsToInfoLogs device_type:string phase:string completion:string

* Set default value.

change the created migration

       def self.up
         add_column :info_logs, :mode, :string, :default => "uploading"
       end



### Running Migrations

$ rake db:migrate VERSION=20120813055814

$ rake db:migrate VERSION=20120813075400

### Rolling Back

$ rake db:rollback

This will run the down method from the latest migration. If you need to undo several migrations you can provide a STEP parameter:

$ rake db:rollback STEP=3

will run the down method from the last 3 migrations.

The db:migrate:redo task is a shortcut for doing a rollback and then migrating back up again. As with the db:rollback task, you can use the STEP parameter if you need to go more than one version back, for example

$ rake db:migrate:redo STEP=3