---
title: Guide for an experienced developer to add resque to a Rails App
status: 
layout: post
categories: [Resque, Rails]
tags: [Resque, Rails]
published: True
---

#### In gemfile you need

```
gem 'resque', '~> 1.27'
gem 'redis', '~> 3.0'
```

#### In application.rb you need

```
config.active_job.queue_adapter = :resque
```

#### Confirm that redis is running and installed via `brew`

```
brew services list
Name          Status  User          Plist
elasticsearch started sandeeparneja /Users/sandeeparneja/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist
nginx         stopped
postgresql    stopped
redis         started sandeeparneja /Users/sandeeparneja/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

#### Setup resque tasks

Create `resque.rake` in `lib/tasks`.

In this file do  `task resque:setup => :environment ` so it loads rails env for resque and then ut further defines the `work` and `workers` task.

In this task you may also disconnect and connet to AR

In this task also require resque and setup the bath to REDIS 

#### Startup Resuqe Backend Job

```
env TERM_CHILD=1 RESQUE_TERM_TIMEOUT=5 RESQUE_PRE_SHUTDOWN_TIMEOUT=10 VVERBOSE=1 RUN_AT_EXIT_HOOKS=yes bundle exec rake resque:work QUEUE=*
```

- `TERM_CHILD` tells the main process to inform the child process when its about to terminate
- `RESQUE_TERM_TIMEOUT=5` gives 5 seconds in the `Resque::TermException`. Here we can mark that the job has failed and/or call `retry_job`
- `RESQUE_PRE_SHUTDOWN_TIMEOUT=10` gives 10 seconds to the job to finish what its doing before it terminates. this is intensely useful as small running jobs can finish and thus never recieve a `TermException`
- `RUN_AT_EXIT_HOOKS` solved the issue of zombie processes i experienced on my MAC
- `VVERBOSE` makes the logs very verbose

#### Create a Job

```
rails g job EsIndexer
```

#### Add catch for termination and retry

```
require 'resque/errors'

class EsIndexerJob < ApplicationJob
  queue_as :default

  def perform(*args)
    Rails.logger.info "do work here"
  rescue Resque::TermException
    Rails.logger.error "Error in #{self.class}: job asked to terminate while doing work"
    retry_job
  end

end
```

####  Have it run only on record comits

A common issue with resque is when called from a rails callback is that the transaction is not completed and therefore the data is not present in the database while the resque job is running an dlooking for the data in the database. 

The easiest solution would to be only use `after_commit` callbacks and stay away from callbacks like `after_save`, `after_destroy` etc.

This obviously only applies when the job is accessing the DB.
 
```
after_commit :index_on_es, on: [:create, :update]
after_commit :delete_on_es, on: :destroy
```

Now inside the Job, get the record and update ES

## send_later / async
in the examples folder
**coming soon**

## front end
**coming soon**

## scheduler
**coming soon**
