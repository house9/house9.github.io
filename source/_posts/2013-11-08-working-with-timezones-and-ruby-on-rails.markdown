---
layout: post
title: "Working with Timezones and Ruby on Rails"
date: 2013-11-08 10:03
comments: true
categories: rails
---

This article attempts to outline some basic guidelines to follow when making your Ruby on Rails application timezone enabled. Ruby on Rails has great support for timezones, but getting it working correctly can be tricky, following the below techniques will hopefully save you some headaches.

* Application Settings - use the default UTC settings
* Set each request to the logged on users timezone
* Display times and dates using the I18n view helpers
* Override user timezone when displaying time aware entity
* Override user timezone when saving time aware entity
* Filter data based on the users timezone

> Is it 'time zone', 'timezone' or 'time-zone'? Both styles end up being used in rails code base `config.time_zone` and `config.active_record.default_timezone`, I use timezone in this article and time_zone in the sample code base. Read more on [stackexchange](http://english.stackexchange.com/questions/3934/time-zone-vs-timezone).

Sample code can be found at: [https://github.com/house9/sample_timez](https://github.com/house9/sample_timez)

The sample code has user profiles with timezone settings, an event model and a work schedule model. It comes with a small set of sample data as well. To set it up locally, follow the [README](https://github.com/house9/sample_timez/blob/master/README.md) instructions.

## Application Settings

Rails uses the following defaults for a new application

* Application assumes it is running in UTC
* Database assumes it is storing data in UTC

> I recommended you stick with these defaults!

#### in rails console (default settings)

```
Time.zone.name
# => "UTC"

ActiveRecord::Base.default_timezone
# => :utc
```

Both *can be* overriden in your application configuration file (config/application.rb) - *don't do it!*

```
# Do NOT do this!!!
config.time_zone = 'Central Time (US & Canada)'
config.active_record.default_timezone = :local
``` 

## Set the timezone for each request

How you determine the 'current users' timezone is going to differ from application to application, in the sample code we are storing the value on the user model in a time_zone field. The sample application uses the techniques from this [RailsCast](http://railscasts.com/episodes/106-time-zones-revised) episode. Alternatively you may  want to set it based on clients browser setting.

Use an `around_filter` or combo of `before_filter` and `after_filter` to set each requests timezone in the `ApplicationController`. If we don't have a current user it will default to UTC.

```
class ApplicationController < ActionController::Base
  # ...
  around_filter :user_time_zone, if: :current_user

  private

  def user_time_zone(&block)
    Time.use_zone(current_user.time_zone, &block)
  end
end
```

## Display times and dates using the I18n view helpers

The `I18n` helpers are timezone aware, besides rendering the datetimes in the logged on users locale format they will convert the stored UTC times to the current threads timezone.

* [http://guides.rubyonrails.org/i18n.html#adding-date-time-formats](http://guides.rubyonrails.org/i18n.html#adding-date-time-formats)
* [https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale](https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale)

On the homepage of the sample application there are numerous examples, also check out the `config/locales/en.yml` file. 

Basic example:
 
```
I18n.localize(current_user.created_at)
```

> Beware: I18n.localize does not handle nil values, so you might need to conditionally guard them

## Override the display of dates

In the sample application we have an event model and a work schedule model, each model has its own timezone. Events are based on where the event is actually going to occur, this is one of those cases where displaying the dates in the users timezone makes no sense at all, so we must override the display. There are a few techniques that can be employeed:

* [in_time_zone](http://api.rubyonrails.org/classes/DateTime.html#method-i-in_time_zone) method (see work schedule in the sample application)
* [Time.use_zone](http://api.rubyonrails.org/classes/Time.html#method-c-use_zone) blocks (see events in the sample application)

Work Schedule overrides at the model level and no speical handling is needed in the views (of course you still need to use I18n.localize).

```
class WorkSchedule < ActiveRecord::Base
  def start_at
    super.in_time_zone(time_zone) if super && time_zone
  end

  def end_at
    super.in_time_zone(time_zone) if super && time_zone
  end
end
```

The event model does not override its start_at or end_at but uses Time.use_zone blocks in the views


    <!-- displayed times in the events timezone -->
    <% Time.use_zone(@event.time_zone) do %>
      <%= l(@event.start_at) %>
      <%= l(@event.end_at) %>
    <% end %>
    <!-- now back to the logged on users timezone -->
    <p>
      Created at <%= l(@event.created_at) %>
      Updated at <%= l(@event.updated_at) %>
    </p>

There is odd behaviour with Time.use_zone if you do any in memory sorting in ruby vs sorting at the database level. See the `HomeController` in the sample application - ordered attributes do not convert in use_zone block, not sure if it a bug or by design.
    
## Override timezone when saving data

Sometimes you want to override saving data, lets say my profile is setup with 'Pacific' timezone but I am creating an event that is going to occur in New York ('Eastern' timezone) I do NOT want rails to convert the times to 'Pacific'. We can reset the current threads timezone to the events timezone before the create or update actions execute. 

```
class EventsController < ApplicationController
  before_action :set_event_time_zone, only: [:create, :update]

  # ... all action code (unchanged)

  private

  def set_event_time_zone
    if params[:event]
      Time.zone = params[:event][:time_zone]
    end
  end
end
```

Alternativly this could be done using `Time.use_zone` blocks in the create and update actions.

## Filter data based on the users timezone

Rails will handle most of this automatically if you don't use raw sql, for example:

```
articles = Article.where("published_at > ?", Time.zone.now)
```

Unlike an Event, the Article does not have its own concept of timezone so this should work seamlessly. ActiveRecord will do the proper conversions.

However if we want to query entities that are timezone aware we may have to leverage some of the databases timezone capabilities. Compare the two examples below, depending on the current user and events timezones and the time of day they could return different results.

```
Event.where("start_at > ?", Time.zone.now).count
# => SELECT COUNT(*) FROM "events" WHERE (start_at > '2013-11-08 19:46:45')
#    count is 5

user_time_zone = Time.zone.now.strftime("%Z")
Event.where("timezone(?, start_at) > ?", user_time_zone, Time.zone.now).count
# => SELECT COUNT(*) FROM "events" WHERE (timezone('FJST', start_at) > '2013-11-08 19:46:45')
#    count is 6
```

The above is postgres only, use ('timezone' or 'AT TIME ZONE') - see [http://www.postgresql.org/docs/9.3/static/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT](http://www.postgresql.org/docs/9.3/static/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT) - you are using postgres right?

Another example, show some data created 'yesterday', the first example will work sometimes, but might be off at certain times of the day. 

```
Event.where("DATE(created_at) = ?", Date.current.yesterday).count
# => SELECT COUNT(*) FROM "events" WHERE (DATE(created_at) = '2013-11-08')
#    count is 2
```

This is more involved but should return correct results

```
start_at = (Date.current - 1.day).beginning_of_day
end_at = start_at.end_of_day
user_time_zone = Time.zone.now.strftime("%Z")

Event.where("timezone(?, created_at) BETWEEN ? AND ?", user_time_zone, start_at, end_at).count
# => SELECT COUNT(*) FROM "events" WHERE (timezone('FJST', created_at) BETWEEN '2013-11-01 11:00:00' AND '2013-11-08 10:59:59')
#    count is 9
```

Variations of the above examples exist in the `HomeController` in the sample application, it might be worth experimenting with different datasets and user / entity timezone settings, timezones are very confusing.

> When filtering on a date column instead of a datetime this would not be an issue.

```
# rails will do it for us, Date.current is timezone aware
User.where("birth_day = ?", Date.current)
# but do NOT do this 
User.where("birth_day = ?", Date.today)
```

## Conculsion

Hopefully you find the article helpful, all feedback is welcome - I am sure I have left out some key information or plain got some of the above wrong.

Additional Reading: [http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails](http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails)

## ETC... (links and resources)

```
rake time:zones:all
```

```
ActiveSupport::TimeZone.zones_map
ActiveSupport::TimeZone.us_zones
```

* [http://api.rubyonrails.org/classes/DateTime.html](http://api.rubyonrails.org/classes/DateTime.html)
* [http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-time_zone_select](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-time_zone_select)
* [http://stackoverflow.com/questions/19664652/timezone-confusion-in-ruby-on-rails-4-0-postgres](http://stackoverflow.com/questions/19664652/timezone-confusion-in-ruby-on-rails-4-0-postgres)
* [https://github.com/rails/rails/issues/9610](https://github.com/rails/rails/issues/9610)
* [http://stackoverflow.com/questions/17818329/rails-3-timezone-confusions/17840938#17840938](http://stackoverflow.com/questions/17818329/rails-3-timezone-confusions/17840938#17840938
)



