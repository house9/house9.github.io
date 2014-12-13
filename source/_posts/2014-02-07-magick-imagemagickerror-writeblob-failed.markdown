---
layout: post
title: "Magick::ImageMagickError: WriteBlob Failed"
date: 2014-02-07 08:55
comments: true
categories: ruby
---

I am using [gruff](https://github.com/topfunky/gruff) to generate line charts in a rails project, putting together a quick proof of concept:

```ruby
class Chart
  def generate_line_graph
    g = Gruff::Line.new(400)
    g.data(:testing, [5, 0, 25, 10)
    g.write("tmp/line.png")
  end
end
```

I quickly ran into the following exception:

<pre>
Magick::ImageMagickError: WriteBlob Failed `/Users/ME/projects/CLIENT/code/RAILS_APP/tmp/line.png' @ error/png.c/MagickPNGErrorHandler/1804
  from /Users/ME/.rvm/gems/ruby-2.1.0/gems/gruff-0.5.1/lib/gruff/base.rb:425:in `write'
  from /Users/ME/.rvm/gems/ruby-2.1.0/gems/gruff-0.5.1/lib/gruff/base.rb:425:in `write'
  from /Users/ME/projects/CLIENT/code/RAILS_APP/app/models/reports/chart.rb:36:in `generate_line_graph'
  from (irb):16
  from /Users/ME/.rvm/gems/ruby-2.1.0/gems/railties-3.2.16/lib/rails/commands/console.rb:47:in `start'
  from /Users/ME/.rvm/gems/ruby-2.1.0/gems/railties-3.2.16/lib/rails/commands/console.rb:8:in `start'
  from /Users/ME/.rvm/gems/ruby-2.1.0/gems/railties-3.2.16/lib/rails/commands.rb:41:in `<top (required)>'
  from script/rails:6:in `require'
  from script/rails:6:in `main'
</pre>

**The issue:** the error is raised when writing to the tmp directory, changing to another directory and it worked as expected?

`g.write("public/line.png")` and `g.write("public/tmp/line.png")` both worked

Turns out this was a simple *mistake on my part*, I just provisioned a new machine and cloned the project repo - I did not have a _tmp_ directory in my project yet, it would be nice if the error message could have just told me so.


