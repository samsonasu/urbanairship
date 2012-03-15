# Why does this fork exist

So you can do: 


    Urbanairship.push {
      :aliases => %W(big long list of aliases), 
      :message => "This message will be delivered to both android and ios devices"
    }


Instead of this 

    Urbanairship.push {
      :aliases => %W(big long list of aliases), 
      :aps => "This goes to ios",
      :android => "This goes to android"
    }


Urbanairship is a Ruby library for interacting with the [Urban Airship API](http://urbanairship.com).

Installation
============
    gem install urbanairship

Note: if you are using Ruby 1.8, you should also install the ```system_timer``` gem for more reliable timeout behaviour. See http://ph7spot.com/musings/system-timer for more information.

Configuration
=============
```ruby
Urbanairship.application_key = 'application-key'
Urbanairship.application_secret = 'application-secret'
Urbanairship.master_secret = 'master-secret'
Urbanairship.logger = Rails.logger
Urbanairship.request_timeout = 5 # default
```

Usage
=====

Registering a device token
--------------------------
```ruby
Urbanairship.register_device('DEVICE-TOKEN')
```

Unregistering a device token
----------------------------
```ruby
Urbanairship.unregister_device('DEVICE-TOKEN')
```

Sending a push notification
---------------------------
```ruby
notification = {
  :schedule_for => [1.hour.from_now],
  :device_tokens => ['DEVICE-TOKEN-ONE', 'DEVICE-TOKEN-TWO'],
  :aps => {:alert => 'You have a new message!', :badge => 1}
}

Urbanairship.push(notification) # =>
# {
#   "scheduled_notifications" => ["https://go.urbanairship.com/api/push/scheduled/123456"]
# }
```

Batching push notification sends
--------------------------------
```ruby
notifications = [
  {
    :schedule_for => [{ :alias => 'deadbeef', :scheduled_time => 1.hour.from_now }],
    :device_tokens => ['DEVICE-TOKEN-ONE', 'DEVICE-TOKEN-TWO'],
    :aps => {:alert => 'You have a new message!', :badge => 1}
  },
  {
    :schedule_for => [3.hours.from_now],
    :device_tokens => ['DEVICE-TOKEN-THREE'],
    :aps => {:alert => 'You have a new message!', :badge => 1}
  }
]

Urbanairship.batch_push(notifications)
```


Sending broadcast notifications
-------------------------------
Urban Airship allows you to send a broadcast notification to all active registered device tokens for your app.

```ruby
notification = {
  :schedule_for => [1.hour.from_now],
  :aps => {:alert => 'Important announcement!', :badge => 1}
}

Urbanairship.broadcast_push(notification)
```

Polling the feedback API
------------------------
The first time you attempt to send a push notification to a device that has uninstalled your app (or has opted-out of notifications), both Apple and Urban Airship will register that token in their feedback API. Urban Airship will prevent further attempted notification sends to that device, but it's a good practice to periodically poll Urban Airship's feedback API and mark those tokens as inactive in your own system as well.

```ruby
# find all device tokens deactivated in the past 24 hours
Urbanairship.feedback(24.hours.ago) # =>
# [
#   {
#     "marked_inactive_on"=>"2011-06-03 22:53:23",
#     "alias"=>nil,
#     "device_token"=>"DEVICE-TOKEN-ONE"
#   },
#   {
#     "marked_inactive_on"=>"2011-06-03 22:53:23",
#     "alias"=>nil,
#     "device_token"=>"DEVICE-TOKEN-TWO"
#   }
# ]
```

Deleting scheduled notifications
--------------------------------

If you know the alias or id of a scheduled push notification then you can delete it from Urban Airship's queue and it will not be delivered.

```ruby
Urbanairship.delete_scheduled_push("123456789")
Urbanairship.delete_scheduled_push(123456789)
Urbanairship.delete_scheduled_push(:alias => "deadbeef")
```

-----------------------------

Note: all public library methods will return either an array or a hash, depending on the response from the Urban Airship API. In addition, you can inspect these objects to find out if they were successful or not, and what the http response code from Urban Airship was.

```ruby
response = Urbanairship.push(payload)
response.success? # => true
response.code # => '200'
response.inspect # => "{\"scheduled_notifications\"=>[\"https://go.urbanairship.com/api/push/scheduled/123456\"]}"
```

If the call to Urban Airship times out, you'll get a response object with a '503' code.

```ruby
response = Urbanairship.feedback(1.year.ago)
response.success? # => false
response.code # => '503'
response.inspect # => "{\"error\"=>\"Request timeout\"}"
```
