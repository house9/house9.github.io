---
layout: post
title: "ActionView::Template::Error: wrong number of arguments (given 2, expected 1)"
date: 2019-06-19 14:35
comments: true
categories: [rspec, rails]
---

I have recently been checking out rails-6.0.0.rc1. I immediatly ran into an issue when trying to run controller specs with [rspec-rails](https://github.com/rspec/rspec-rails).

Running any controller spec would error with:

```
rspec spec/controllers/home_controller_spec.rb

Failures:

  1) HomeController GET #index returns http success
     Failure/Error: get :index

     ActionView::Template::Error:
       wrong number of arguments (given 2, expected 1)
     # ./spec/controllers/home_controller_spec.rb:7:in `block (3 levels) in <top (required)>'
     # ------------------
     # --- Caused by: ---
     # ArgumentError:
     #   wrong number of arguments (given 2, expected 1)
     #   ./spec/controllers/home_controller_spec.rb:7:in `block (3 levels) in <top (required)>'

Finished in 0.03078 seconds (files took 1.66 seconds to load)
1 example, 1 failure
```

After searching for a bit I found the answer on a github issue, tried to find it again and could not otherwise I would link it here.

My co-worker ran into the same issue so I figured it was time to write a blog post.

The fix: use rspec-rails 4 which is currently in development, modify your Gemfile like so:

```
group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'rspec-core', git: 'https://github.com/rspec/rspec-core'
  gem 'rspec-expectations', git: 'https://github.com/rspec/rspec-expectations'
  gem 'rspec-mocks', git: 'https://github.com/rspec/rspec-mocks'
  gem 'rspec-rails', git: 'https://github.com/rspec/rspec-rails', branch: '4-0-dev'
  gem 'rspec-support', git: 'https://github.com/rspec/rspec-support'
end
```

and then bundle, re-run the specs

```
rspec spec/controllers/home_controller_spec.rb

Finished in 0.01668 seconds (files took 2.14 seconds to load)
1 example, 0 failures
```

Life is good again!
