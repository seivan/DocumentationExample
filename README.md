# Info

## Versioning

### Current Version on staging

> v2

### Current Version on production

> v2


## Environments and URL's

### On heroku we have two environments running

> fone2012-staging

> fone2012


### Url to staging

> http://fone2012-staging.herokuapp.com/api/v2/

### Url to production

> http://fone2012.herokuapp.com/api/v2/


## Authentication rules

### Legend

> API end points that require authentication will be marked with 

```
requires_auth!
``` 

### Authentication params

> For Facebook accounts

```ruby
params = {
  :session => {
    :provider_token => "...",
    :uuid => "..."
  }
}
``` 



> For accounts that do not have a facebook associations, this works fine. 

```ruby
params = {
  :session => {
    :uuid => "..."
  }
}
```

> On Facebook Check their **state** attribute on the user object to see if the user is authenticated.

> The following state exist; 

* Wait - You are being authenticated with Facebook - check POST session below (requires_auth! )

* Invalid - Your account is invalid, need to register again - check POST session below

* Valid - Ready to roll 

> **Facebook Accounts return authorization object with the user embedded**




## Error and Data rules

### Errors

> Always check for an "error" key on your responseObject to see if the object had any resonses.

```ruby
@data = {:error=> {:message => exception.message,
               :params => params},
     :status => error_code }
``` 

> The following exceptions exist and they return different error codes together with a human readable language

```ruby
                  when ActiveRecord::RecordNotFound           then 404
                  when ActionController::RoutingError         then 404
                  when ActionController::MethodNotAllowed     then 501
                  when ActiveRecord::ActiveRecordError        then 501

                  when Authorization::AuthenticateFailedError then 401
                  when Authorization::NoSession               then 401
                  when Authorization::ProviderError           then 401
                  when Authorization::InvalidError            then 401

                  when Prediction::AlreadySubmittedError      then 403
                  when PhotoLike::AlreadyLikedError           then 403
```

> If you hit a path that doesn't exist, you get a **ActionController::RoutingError**

> If you call for an object that doesn't exist you get a **ActiveRecord::RecordNotFound**

### Data

> All the data returned will be embedded in the key named after the controller.
So if you hit 

* /api/v2/**events**.json
  * You will get your event objects inside {:**events** => [...]}

* /api/v2/**session**.json
  * You will get your session object inside {:**session** => [...]}

### Deletion

> The server will do it's best to always return objects that are published

> If an object has is_deleted as true, then the clients will do their best to delete it from the local persistant store

> All objects will have a is_deleted boolean to delete from the clients. News, Events and Photos 


## Default scopes

### Event default_scope order(:start_date)

### News default_scope order(:published_at)


# Current endpoints


## Events

> GET    /api/v2/events(.:format)  

> JSONP support with ?callback=

> returns

```ruby
    api_accessible :v1_public do |t|
      t.add :id
      t.add :title
      t.add :body
      t.add :latitude
      t.add :longitude
      t.add :start_date
      t.add :end_date
      t.add :venue
      t.add :thumbnail_url
      t.add :event_url
      t.add :venue_address
      t.add :venue_postal_code
      t.add :pricing
      t.add :ticket_url
      t.add :updated_at
      t.add :is_published
      t.add :is_deleted
    end
```


> Takes an optional limit parameter (defaults to 4)

> Limited to 4 event objects as per request

> GET   /api/v2/events(.:format)?callback=myCallback&today=true

or

> GET   /api/v2/events(.:format)?callback=myCallback&date="2012-08-21"

> returns

```ruby
    api_accessible :v1_public do |t|
      t.add :id
      t.add :title
      t.add :start_date
    end
```



## Session

> POST   /api/v2/session(.:format)

#### For Facebook

```ruby 
params = {
  :session => {
    :provider_token => "...",
    :uuid => "..."
  }
}
```

> returns for with facebook

```ruby


    api_accessible :v1_private do |t|
      t.add :provider_id
      t.add :provider_token
      t.add :user
      t.add :state
      t.add :error_message
      t.add :provider_name
      t.add :user
          api_accessible :v1_private do |t|
            t.add :id
            t.add :name
            t.add :email
            t.add :device_token
            t.add :uuid
            t.add :nric
            t.add :phone_number
            t.add :created_at
            t.add :updated_at
           end
        end
    end
```






## Predictions

### GET    /api/v2/predictions/:id(.:format)

> pre-end Prediction requests before end_date will only return answers and questions

> post-end Prediction requests after end_date will also review data

> There will only be two rounds. :id => 1, :id => 2

> post-end is not yet implemented
  
> returns

```ruby
    api_accessible :v1_public do |t|
      t.add :id
      t.add :title
      t.add :start_date_unixtime
      t.add :end_date_unixtime
      t.add :round 
      t.add :questions
          api_accessible :v1_public do |t|
            t.add :id
            t.add :title
            t.add :answers
                api_accessible :v1_public do |t|
                  t.add :id
                  t.add :title
                end
          end
    end
``` 

### POST   /api/v2/predictions/:prediction_id/submission(.:format)

> requires_auth! 

> Send an array of chosen answer id's

```ruby
{ :answers => [12,34,21,54,12,23,12], 
  :options =>  {:phone_number => "...", :nric => "...", :is_sharing_on_facebook => true} 
};
```

> returns

```ruby
    api_accessible :v1_public do |t|
      t.add :id
      t.add :title
      t.add :start_date_unixtime
      t.add :end_date_unixtime
      t.add :round 
      t.add :questions
          api_accessible :v1_public do |t|
            t.add :id
            t.add :title
            t.add :answers
                api_accessible :v1_public do |t|
                  t.add :id
                  t.add :title
                  t.add :users_count
                  t.add :users_percentage
                  t.add :is_correct
                end
          end
    end
``` 
 
## Predictions

### POST   /api/v2/sharings/(.:format)

> requires_auth! 

> Requires Facebook


```ruby
{ 
  :options =>  {:message => "..."} 
}
```

> returns

```ruby
    api_accessible :v1_public do |t|
      t.add :id
      t.add :title
      t.add :start_date_unixtime
      t.add :end_date_unixtime
      t.add :round 
      t.add :questions
          api_accessible :v1_public do |t|
            t.add :id
            t.add :title
            t.add :answers
                api_accessible :v1_public do |t|
                  t.add :id
                  t.add :title
                  t.add :users_count
                  t.add :users_percentage
                  t.add :is_correct
                end
          end
    end
```  

## Photos

### GET   /api/v2/events(.:format)?albums=true

> returns

```ruby
    api_accessible :v1_public_album do |t|
      t.add :id
      t.add :title
      t.add :photos
      t.add :is_deleted

      api_accessible :v1_public do |t|
        t.add :id
        t.add :created_at
        t.add :updated_at
        t.add :event_id
        t.add :user_id
        t.add :photo_likes_count
        t.add :thumb_url
        t.add :normal_url
        t.add :is_deleted
    end

  end
```  


> Photo per event

### GET   /api/v2/events/:event_id/photos(.:format)

> --------

> Current users photo 

> requires_auth!

### GET   /api/v2/session/photos(.:format)

> --------

> Most liked photo limited by 10

### GET   /api/v2/likes/photos(.:format)


> returns

```ruby
    api_accessible :v1_public do |t|
        t.add :id
        t.add :created_at
        t.add :updated_at
        t.add :event_id
        t.add :user_id
        t.add :photo_likes_count
        t.add :thumb_url
        t.add :normal_url
        t.add :is_deleted
    end
```  


## POST /api/v2/events/:event_id/photos(.:format)

> requires_auth!

> Send a multipart request with the photo

```ruby
 {:image => YOURPHOTO},
  :options =>  {:is_sharing_on_facebook => true} }
```

> Can only share if it has a facebook session 

> The event_id needs to exist and needs to be in the past or 1 hour in the future. 

> returns

```ruby
    api_accessible :v1_public do |t|
        t.add :id
        t.add :created_at
        t.add :updated_at
        t.add :event_id
        t.add :user_id
        t.add :photo_likes_count
        t.add :thumb_url
        t.add :normal_url
        t.add :is_deleted
    end
```

## Likes

### POST /api/v2/events/:event_id/photos/:photo_id/likes(.:format)

> returns

```ruby
    api_accessible :v1_public do |t|
        t.add :id
        t.add :created_at
        t.add :updated_at
        t.add :event_id
        t.add :user_id
        t.add :photo_likes_count
        t.add :thumb_url
        t.add :normal_url
        t.add :is_deleted
    end
```


## News


### GET /api/v2/news(.:format)

> Use If-Modified-Since header with the timestamp that is returned from a 200 response.
> Make sure to save the timestamp! 

> Example
```
curl -I  http://fone2012-staging.herokuapp.com/api/v2/news.json --header 'If-Modified-Since: Thu, 06 Sep 2012 06:55:04 GMT'
```

> If up to date, it will return a 304

> If NOT up to date, will return news from that timestamp and above in a 200 response - save the new timestamp!

> If you don't have a timestamp, you'll get all the news. But make sure to save the timestamp again!


> body attribute contains html for attributed strings

> published_at is just a date and not DateTime

> value objects inside properties are not going to be strings. You will have to parse them from json manually. 

> example; 

```ruby
x.properties => {"X"=>"[\"LOL\", \"X\"]"} 
# X is the key
# the value of X is a string typed array. Your json parser will make it into an array you can use. 
```

> returns

```ruby
    api_accessible :v2_public do |t|
      t.add :id
      t.add :title
      t.add :sub_title
      t.add :author
      t.add :published_at
      t.add :race_name
      t.add :body
      t.add :image_url 
      t.add :properties
      t.add :is_deleted 
      t.add :created_at
    end
```