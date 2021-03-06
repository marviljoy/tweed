# Tweed

jQuery plugin for displaying a simple feed of recent tweets.

[See the DEMO](http://darsain.github.com/tweed)


## Calling

```js
$ul.tweed( query [, options ] );
```
### query

Type of query that determines what kind of API request will be made. Three types of queries are supported:

+ `@JohnDoe` - Author name prefixed with `@` creates a user timeline API request. In this case: recent tweets from `@JohnDoe` account.
+ `@JohnDoe/coolpeople` - This creates a list API request for `coolpeople` list name that belongs to `@JohnDoe` twitter account.
+ `#justinbieber` - Anything else than upper 2 cases creates a search API request. In this case: recent tweets containing `#justinbieber` hash tag will be displayed,
but you can use any Search Operator from [Twitter Search API](https://dev.twitter.com/docs/using-search).

### [options]

**limit:** `default: 5` how many tweets to display

**retweets:** `default: 1` Whether to include retweets in author timelines.

**replies:** `default: 0` Whether to include replies in author timelines.

**amend:** `default: 10` How many additional tweets to request when `replies` or `retweets` options are disabled.
This is because when you'll request 10 tweets without replies, and 2 of the latest tweets are replies, you'll get
a response array with 8 tweets. Yes, this is retarded. Amend options is however just trying to ease the problem.
If `replies` or `retweets` are disabled, `limit = 5`, `amend = 10`, and the last 15 tweets on an account are replies
or retweets, response object will return 0 tweets...

**linkify:** `default: 1` Turns URLs, @author references, and #hashes into links.

**refreshInterval:** `default: 0` Refresh interval in seconds, leave `0` to never refresh.

**pauseOnHover:** `default: 1` When **refreshInterval** is enabled, pause it when mouse hovers over tweets container.

**template:**  HTML template using mustache-like tags `{{...}}` that will be replaced with tweet data.

Default template is:

```html
<li>
	<span class="text">{{tweet}}</span><br>
	<a href="{{author_url}}" class="author">{{author}}</a>
	<a href="{{tweet_url}}" class="time">{{time}}</a>
</li>
```

Available template data keys:

+ **author** account name, e.g. `@JohnDoe`
+ **name** account owner name, e.g. `John`
+ **author_url** account owners website, e.g. `http://johndoe.com`
+ **tweet** tweet, i.e. 140 characters long message
+ **tweet_url** url to this specific tweet, e.g. `http://twitter.com/TheChloeLamb/status/176591251674640384`
+ **date** UNIX timestamp of tweet post, e.g. `1331417055`
+ **time** human readable relative time of tweet post, e.g. `3 minutes ago`
+ **avatar_url** default avatar url, e.g. `http://a0.twimg.com/profile_images/1543025697/vader_hoth_normal.jpg`
+ **avatar_mini_url** smaller avatar url, e.g. `http://a0.twimg.com/profile_images/1543025697/vader_hoth_mini.jpg`
+ **avatar_bigger_url** bigger avatar url, e.g. `http://a0.twimg.com/profile_images/1543025697/vader_hoth_bigger.jpg`
+ **avatar** avatar in prepared img tag, e.g. `<img src="[avatar_url]" alt="JohnDoe's avatar" />`
+ **avatar_mini** img tag with smaller avatar, e.g. `<img src="[avatar_mini_url]" alt="JohnDoe's avatar" />`
+ **avatar_bigger** img tag with bigger avatar, e.g. `<img src="[avatar_bigger_url]" alt="JohnDoe's avatar" />`
+ also all keys of an original tweet object from API response are available

## Methods

Tweed has a few useful methods available.

#### Refresh

```js
$ul.tweed( 'refresh' );
```

Refreshes the tweets. If there are any new ones, they will be prepended into container, removing the oldest tweets to not overflow over options limit.
Old tweets will be updated with new relative time.

#### Pause

```js
$ul.tweed( 'pause' );
```

Pause refresh interval when enabled. Otherwise it does nothing.

#### Resume

```js
$ul.tweed( 'resume' );
```

Un-pause refresh interval. When refresh interval is disabled or it is not paused, it has no effect.

#### Destroy

```js
$ul.tweed( 'destroy' );
// or alias
$ul.tweed( false );
```

Destroys the plugin instance, removes attached event listeners, and replaces tweets with whatever content was in the container before the first load.


## Custom events

All events are triggered on tweets container.

#### tweed:load

Event triggered after each load, i.e. after tweets from successful response has been placed into container.

```js
$ul.on( 'tweed:load', function( event, $tweets, $newTweets, author ){ ... } );
```

Arguments passed:

+ **$tweets** - jQuery object with all tweets
+ **$newTweets** - jQuery object with only new tweets. if this is the first load event, all tweets are new.
+ **author** - object with tweets author data. this argument is available only in author timeline query (`@JohnDoe`). Object contains this keys:
`username`, `name`, `url`, `description`, `website`, `followers`, `avatar_url`, `avatar_mini_url`, `avatar_bigger_url`, `avatar`, `avatar_mini`,
`avatar_bigger`, `last_tweet`, `location`, `tweets`, `following`

#### tweed:fetch

Event triggered on fetch, i.e. when plugin requests a response from twitter API.

```js
$ul.on( 'tweed:fetch', function( event ){ ... } );
```

When invalid response has been received, there will be more than one `tweed:fetch` event triggered before plugin will fire `tweed:load` event.


## Example of a call with all default options

```js
$ul.tweed( '@JohnDoe', {
	limit:           5,
	retweets:        1,
	linkify:         1,
	refreshInterval: 0,
	pauseOnHover:    0,
	template: '<li><span class="text">{{tweet}}</span><br><a href="{{author_url}}" class="author">{{author}}</a> <a href="{{tweet_url}}" class="time">{{time}}</a></li>'
});
```

## Notable behaviors

+ When twitter API returns some error, or anything other than valid response, it is considered that twitter is temporarily unresponsive/overloaded.
In this case `tweed` will be sending request 1 second after every bad response until it receives something valid.

+ Tweed will replace anything inside a container, so if you want to have a simple "tweets loading" message, just place it inside of a container, and Tweed will
than replace it with loaded tweets.

+ Twitter API responses are sometimes really weird, especially for search type query. Sometimes it returns different number of tweets than requested, sometimes it returns the same request
as a few seconds ago, but 3rd tweet is missing, ... just weird ...


## Credits

Tweed was created after I've tried using [seaofclouds/tweet](https://github.com/seaofclouds/tweet) plugin, but found it to do a lot of redundant stuff in a non-simple way,
yet didn't gave me a lot of power over it's output. Nevertheless, I've used its function for parsing twitter date, so thx :)