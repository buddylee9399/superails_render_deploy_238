# #238 Deploy Rails 8 to Render.com. Solid Trifecta on Postgres
- https://www.youtube.com/watch?v=eTgE5ClYK9s

## creating app
- rails new superails_render_deploy_R8 -d postgresql --css=tailwind
- updated layout/app
```
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

    <%# Includes all stylesheet files in app/assets/stylesheets %>
    <%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>
```

- rails g controller pages index contact pricing, and copied tailwind code to it
- using this website about solid queue to make 1 database with rails 8 not 4
- https://andyatkinson.com/solid-queue-mission-control-rails-postgresql
- using solid queue github for single database configuration
- https://github.com/rails/solid_queue?tab=readme-ov-file
- rails g migration SolidQueue - and copy in from the corresponding schemas
- rails g migration SolidCable - and copy in from the corresponding schemas
- rails g migration SolidCache - and copy in from the corresponding schemas
- update database.ymml
```
production:
  primary: &primary_production
    <<: *default
    database: superails_render_deploy_r8_production
    username: superails_render_deploy_r8
    password: <%= ENV["SUPERAILS_RENDER_DEPLOY_R8_DATABASE_PASSWORD"] %>
  # cache:
  #   <<: *primary_production
  #   database: superails_render_deploy_r8_production_cache
  #   migrations_paths: db/cache_migrate
  # queue:
  #   <<: *primary_production
  #   database: superails_render_deploy_r8_production_queue
  #   migrations_paths: db/queue_migrate
  # cable:
  #   <<: *primary_production
  #   database: superails_render_deploy_r8_production_cable
  #   migrations_paths: db/cable_migrate
  

```

- update cable.yml - change writing cable to primary, which is database.yml production: primary
```
# Async adapter only works within the same process, so for manually triggering cable updates from a console,
# and seeing results in the browser, you must do so from the web console (running inside the dev process),
# not a terminal started via bin/rails console! Add "console" to any action or any ERB template view
# to make the web console appear.
development:
  adapter: async

test:
  adapter: test

production:
  adapter: solid_cable
  connects_to:
    database:
      writing: primary
  polling_interval: 0.1.seconds
  message_retention: 1.day
  
```
- update cache.yml
```
production:
  database: primary
  <<: *default  
```

- no need to update queue.yml
- update production.rb - taking out the connects to
```
  config.active_job.queue_adapter = :solid_queue
  # config.solid_queue.connects_to = { database: { writing: :queue } }
```

- add to development.rb
```
  config.active_job.queue_adapter = :solid_queue

```
- to use solid queue in development

## solid queue and mission control
- trying to trigger a job
- rails g job HelloWorld
- update the job
```
class HelloWorldJob < ApplicationJob
  queue_as :default

  def perform(*args)
    # Do something later
    puts "Hello, world"
  end
end  

```

- in rails c
```
HelloWorldJob.perform_now - it worked
HelloWorldJob.perform_later
HelloWorldJob.set(wait: 1.week).perform_later  
```

- to see that we have the jobs queued
- install mission control
- bundle add mission_control-jobs
- add to routes
```
    mount MissionControl::Jobs::Engine, at: "/jobs"
```

- go to localhous/jobs - didnt work
- add to development (from the githubhttps://github.com/rails/mission_control-jobs)
```
  config.mission_control.jobs.http_basic_auth_enabled = false
```
- restart server, 
- go to localhost/jobs- it worked
### starting solid queue for the jobs to start running
- in a terminal window:  bin/rails solid_queue:start
- to not have to run that in development, update puma.rb
```
  plugin :solid_queue if ENV["SOLID_QUEUE_IN_PUMA"] || Rails.env.development?
```

- restart server
- stop solid queue
- rails c and create a new perform later
- IT WORKED

## CREATING POSTS
- rails g scaffold post title body:text




