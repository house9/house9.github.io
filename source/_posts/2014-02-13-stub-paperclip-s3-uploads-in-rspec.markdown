---
layout: post
title: "stub paperclip s3 uploads in rspec"
date: 2014-02-13 14:22
comments: true
categories: rails
---

I was searching around stackoverflow trying to figure how to stub out s3 uploads with paperclip when running unit tests; came across a few that looked promising, along the lines of:

```ruby
Model.any_instance.stubs(:save_attached_files).returns(true)
```

But that was not working with [paperclip](https://github.com/thoughtbot/paperclip) version 4, a quick look at the stacktrace reveled the method I wanted to mock.

<pre>
Failure/Error: report.save
AWS::S3::Errors::InvalidAccessKeyId:
  The AWS Access Key Id you provided does not exist in our records.
/Users/me/.rvm/gems/ruby-2.1.0/gems/aws-sdk-1.33.0/lib/aws/core/client.rb:374:in `return_or_raise'
/Users/me/.rvm/gems/ruby-2.1.0/gems/aws-sdk-1.33.0/lib/aws/core/client.rb:475:in `client_request'
(eval):3:in `put_object'
/Users/me/.rvm/gems/ruby-2.1.0/gems/aws-sdk-1.33.0/lib/aws/s3/s3_object.rb:1751:in `write_with_put_object'
/Users/me/.rvm/gems/ruby-2.1.0/gems/aws-sdk-1.33.0/lib/aws/s3/s3_object.rb:607:in `write'
/Users/me/.rvm/gems/ruby-2.1.0/gems/paperclip-4.0.0/lib/paperclip/storage/s3.rb:337:in `block in flush_writes'
/Users/me/.rvm/gems/ruby-2.1.0/gems/paperclip-4.0.0/lib/paperclip/storage/s3.rb:314:in `each'
/Users/me/.rvm/gems/ruby-2.1.0/gems/paperclip-4.0.0/lib/paperclip/storage/s3.rb:314:in `flush_writes'
/Users/me/.rvm/gems/ruby-2.1.0/gems/paperclip-4.0.0/lib/paperclip/attachment.rb:239:in `save'
/Users/me/.rvm/gems/ruby-2.1.0/gems/paperclip-4.0.0/lib/paperclip/has_attached_file.rb:87:in `block in add_active_record_callbacks'
/Users/me/.rvm/gems/ruby-2.1.0/gems/activesupport-3.2.16/lib/active_support/callbacks.rb:460:in `_run__2863894366922569793__save__4567171711544756900__callbacks'
/Users/me/.rvm/gems/ruby-2.1.0/gems/activesupport-3.2.16/lib/active_support/callbacks.rb:405:in `__run_callback'
/Users/me/.rvm/gems/ruby-2.1.0/gems/activesupport-3.2.16/lib/active_support/callbacks.rb:385:in `_run_save_callbacks'
# ...
</pre>

after double checking the code on github, a working stub:

```ruby
Paperclip::Attachment.any_instance.stub(:save).and_return(true)
```

