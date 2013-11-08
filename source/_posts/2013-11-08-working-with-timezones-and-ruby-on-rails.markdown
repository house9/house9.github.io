---
layout: post
title: "Working with Timezones and Ruby on Rails"
date: 2013-11-08 10:03
comments: true
categories: rails
---

This article attempts to outline some basic guidelines to follow when making your Ruby on Rails application timezone enabled. Ruby on Rails has great support for timezones, but getting it working correctly can be tricky. Following the techniques below should save you some headache.

* Application Settings - use the default UTC settings
* Set each request to the logged on users timezone
* Display times and dates using the I18n view helpers
* Override user timezone when displaying time aware entity
* Override user timezone when saving time aware entity
* Filter data based on the users timezone

> Is it 'time zone', 'timezone' or 'time-zone'? Both styles end up being used in rails code base `config.time_zone` and `config.active_record.default_timezone`. I use timezone in this article and time_zone in the sample code base. Read more on [stackexchange](http://english.stackexchange.com/questions/3934/time-zone-vs-timezone).

Sample code can be found at: [https://github.com/house9/sample_timez](https://github.com/house9/sample_timez)

The sample code has user profiles with timezone settings, an event model, and a work schedule model. It comes with a small set of sample data as well. To set it up locally, follow the [README](https://github.com/house9/sample_timez/blob/master/README.md) instructions.

## Application Settings

Rails uses the following defaults for a new application

* Application assumes it is running in UTC
* Database assumes it is storing data in UTC

> I recommend you keep these defaults!

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

How you determine the 'current users' timezone will differ from application to application. In the sample code we are storing the value on the user model in a time_zone field. The sample application uses the techniques from this [RailsCast](http://railscasts.com/episodes/106-time-zones-revised) episode. Alternatively you may  want to set it based on the client's browser setting.

Use an `around_filter` or combo of `before_filter` and `after_filter` to set each request's timezone in the `ApplicationController`. If we don't have a current user it will default to UTC.

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

The `I18n` helpers are timezone aware. Aside from rendering the datetimes in the logged on users locale format, they will convert the stored UTC times to the current threads timezone.

* [http://guides.rubyonrails.org/i18n.html#adding-date-time-formats](http://guides.rubyonrails.org/i18n.html#adding-date-time-formats)
* [https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale](https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale)

On the homepage of the sample application there are numerous examples, also check out the `config/locales/en.yml` file. 

Basic example:
 
```
I18n.localize(current_user.created_at)
```

> Beware: I18n.localize does not handle nil values. You will need to guard against nils for nullable columns.

## Override the display of dates

In the sample application we have an Event model and a WorkSchedule model. Each model has its own timezone. Events are based on where the event will actually occur. In this case displaying the dates in the users timezone makes no sense at all, so we must override the display. There are a few techniques that can be employed:

* [in_time_zone](http://api.rubyonrails.org/classes/DateTime.html#method-i-in_time_zone) method (see work schedule in the sample application)
* [Time.use_zone](http://api.rubyonrails.org/classes/Time.html#method-c-use_zone) blocks (see events in the sample application)

`in_time_zone` example: WorkSchedule overrides the start_at and end_at model attributes, therefore no special handling is needed in the views (of course you still need to use I18n.localize).

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

`Time.use_zone` example: The Event model does *not* override start_at or end_at attributes, but uses Time.use_zone blocks in the views.

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

There is odd behavior with Time.use_zone when doing in-memory sorting in ruby (`sort_by`). See the `HomeController` in the sample application. Ordered attributes do not convert in the use_zone block. I am not sure if this a bug or by design.
    
## Override timezone when saving data

Sometimes you want to override saving data. Let's say my profile is set up with 'Pacific' timezone, but I am creating an event that will occur in New York ('Eastern' timezone). I do *not* want rails to convert the times to 'Pacific'. We can reset the current threads timezone to the events timezone before the create and update actions execute. 

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

Alternatively this could be done using `Time.use_zone` blocks in the create and update actions.

## Filter data based on the users timezone

Rails will handle most of this automatically if you don't use raw sql. For example:

```
articles = Article.where("published_at > ?", Time.zone.now)
```

Unlike an Event, the Article does not have its own concept of timezone, so this should work seamlessly. ActiveRecord will do the proper conversions.

However, if we want to query entities that are timezone aware we may have to leverage some of the database's timezone capabilities. Compare the two examples below. Depending on the current users timezone, the events timezone, and the time of day, they could return different results.


TODO: this would work for created_at, updated_at
but when dealing with start_at, end_at we need to use the event.time_zone ????
TODO: put the yesterday example first, re-think other example

```
Event.where("start_at > ?", Time.zone.now).count
# => SELECT COUNT(*) FROM "events" WHERE (start_at > '2013-11-08 19:46:45')
#    count is 5

user_time_zone = Time.zone.now.strftime("%Z")
Event.where("timezone(?, start_at) > ?", user_time_zone, Time.zone.now).count
# => SELECT COUNT(*) FROM "events" WHERE (timezone('FJST', start_at) > '2013-11-08 19:46:45')
#    count is 6
```

The above is postgres only. Use ['timezone' or 'AT TIME ZONE'](http://www.postgresql.org/docs/9.3/static/functions-datetime.html#FUNCTIONS-DATETIME-ZONECONVERT). If you are not using postgres, check your database documentation for an equivalent function.

The next two examples show data created 'yesterday'. The first will work some of the time, but could be off at certain times of the day. The second is more involved but should return correct results.

```
Event.where("DATE(created_at) = ?", Date.current.yesterday).count
# => SELECT COUNT(*) FROM "events" WHERE (DATE(created_at) = '2013-11-08')
#    count is 2
```

Here we use the user's timezone for accurate results:

```
start_at = (Date.current - 1.day).beginning_of_day
end_at = start_at.end_of_day
user_time_zone = Time.zone.now.strftime("%Z")

Event.where("timezone(?, created_at) BETWEEN ? AND ?", user_time_zone, start_at, end_at).count
# => SELECT COUNT(*) FROM "events" WHERE (timezone('FJST', created_at) BETWEEN '2013-11-01 11:00:00' AND '2013-11-08 10:59:59')
#    count is 9
```

Variations of the above examples exist in the `HomeController` (see sample application). It might be worth experimenting with different datasets and user / entity timezone settings. Timezones are very confusing.

> This is not an issue when filtering on date columns (only datetime).


```
# rails will do it for us, Date.current is timezone aware
User.where("birthday_on = ?", Date.current)
# but do NOT do this 
User.where("birthday_on = ?", Date.today)
```

## Conclusion

I hope you found this article helpful. All feedback is welcome - I am sure I have left out some key information or just plain got it wrong.

Additional Reading: [http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails](http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails)

## ETC... (links and resources)

```
rake time:zones:all
```

```
ActiveSupport::TimeZone.zones_map
ActiveSupport::TimeZone.us_zones
```

* API: [DateTime](http://api.rubyonrails.org/classes/DateTime.html)
* API: [time_zone_select](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-time_zone_select)
* Stackoverflow: [Timezone confusion](http://stackoverflow.com/questions/19664652/timezone-confusion-in-ruby-on-rails-4-0-postgres)
* Stackoverflow: [More timezone confusion](http://stackoverflow.com/questions/17818329/rails-3-timezone-confusions/17840938#17840938
)
* [datetime_select preselects wrong times upon edit](https://github.com/rails/rails/issues/9610)

```
Good naming conventions for date and datetime (your milage may vary)

datetime: should end in _at (created_at, updated_at, start_at, end_at)
    date: should end in _on (start_on, end_on, work_on)
```

