---
layout: post
title: "Rails, timezones, javascript and moment.js"
date: 2014-12-13 10:33
comments: true
categories: [rails, emberjs]
---

So you need to display some datetimes in your new ember.js/angular/backbone UI and the times must be displayed for a timezone which is different from the logged on user.

Let us say we have an `Event` class in our rails application with the following attributes:

```ruby
create_table :events do |t|
  t.string :event_name, null: false
  t.text :description
  t.datetime :start_at, null: false
  t.string :time_zone, null: false, default: 'UTC'
end
```

Rails uses the timezone format: `Eastern Time (US & Canada)`

Javascript (and therefore the [moment-timezone](http://momentjs.com/timezone/) library) use [the format](http://en.wikipedia.org/wiki/Tz_database): `America/New_York`

Rails can easily convert this for you and return the proper js format in your json:

```ruby
class Event < ActiveRecord::Base
  # ...

  def time_zone_mapping
    # time_zone => Eastern Time (US & Canada)
    ActiveSupport::TimeZone::MAPPING[self.time_zone] # => America/New_York
  end
end
```

and with [moment.js](http://momentjs.com/) and [moment-timezone](http://momentjs.com/timezone/)

```javascript
// where 'event' is the js representation of your rails model
moment(event.startAt).tz(event.timeZoneMapping).format('dddd, MMMM Do YYYY h:mm a z');
```

You will want to ensure that all json is being rendered with `UTC` as the configured timezone in rails,
i.e. `config.time_zone = 'UTC'` in your application config and/or `Time.zone = 'UTC'` in your api controller.

See also

* [Ruby on Rails: Timezones](/blog/2013/11/15/working-with-timezones-and-ruby-on-rails/)
* [ActiveSupport::TimeZone](http://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html)
