---
layout: post
title: "Service Class Permutations"
date: "2015-03-04 00:02"
listed: false
published: false
categories:
- Ruby
- Service Classes
---


```ruby
class SendWelcomeEmail
  def initialize(mailer)
    @mailer = mailer
  end

  def call(user)
    @mailer.mail(user.email, subject: "Welcome, #{user.name}")
  end
end

# Later:
SendWelcomeEmail.new(MyApp.mailer).call(user)
```

```ruby
welcomer = SendWelcomeEmail.new(MyApp.mailer).call(user)

welcomer.call(frank)
welcomer.call(kate)
# ... and so on.
```

This could even be a test setup; set it up once beforehand, and then have it send for each case. Awesome.


```
class SendWelcomeEmail
  def self.call(mailer, user)
    mailer.mail(user.email, subject: "Welcome, #{user.name}")
  end
end

SendWelcomeEmail.call(MyApp.mailer, user)

SendWelcomeEmail.call(MyApp.mailer, frank)
SendWelcomeEmail.call(MyApp.mailer, bob)
```

```
class SendWelcomeEmail
  def self.call(mailer, user)
    mailer.mail(user.email, subject: "Welcome, #{user.name}")
  end
end

welcomer = proc { |user|
  SendWelcomeEmail.call(MyApp.mailer, user)
}

welcomer.call(frank)
welcomer.call(kate)
```


TODO: Re: Just using functions.
TODO: Ruby has namespaces, but they're all global namespaces. Functions are in a namespace. Classes are in a namespace.

Why? Well, dumping a bunch of methods into the global namespace is generally frowned upon.


Addendum: how about ActiveRecord::Errors and validations.

Sure, okay, splitting it up into initialize and call means you get to reuse those. If you're not set on using those libraries exactly, then TODO: example of different validations. Returning an object that has same error interface.

TODO: Errors. Exceptions.
