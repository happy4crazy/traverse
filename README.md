# Traverse

```bash
gem install traverse
```

## Introduction

Traverse is a simple tool that makes it easy to traverse XML and JSON.

Let's say you're messing around with Twitter's XML
[public timeline](http://api.twitter.com/statuses/public_timeline.xml).
Traverse let's you do things like this:
 
```ruby
require 'open-uri'

timeline = Traverse::Document.new(open "http://api.twitter.com/statuses/public_timeline.xml")

timeline.statuses.each do |status|
  puts "#{status.user.name} says: #{status.text}"
end
```

Or, let's say you're foolin' with Spotify's JSON [Search
API](http://ws.spotify.com/search/1/track.json?q=like+a+virgin).

```ruby
search = Traverse::Document.new(open "http://ws.spotify.com/search/1/track.json?q=like+a+virgin")

search.tracks.each do |track|
  puts "Track: #{track.name}"
  puts "Album: #{track.album.name}"
end
```

For a slightly more complicated example, take a look at a
[boxscore](http://gd2.mlb.com/components/game/mlb/year_2011/month_03/day_31/gid_2011_03_31_detmlb_nyamlb_1/boxscore.xml)
pulled from Major League Baseball's public API.

```ruby
url = "http://gd2.mlb.com/components/game/mlb/year_2011/month_03/day_31/gid_2011_03_31_detmlb_nyamlb_1/boxscore.xml"
boxscore = Traverse::Document.new(open url)

boxscore.game_id # => '2011/03/31/detmlb-nyamlb-1'

boxscore.battings[0].batters[1].name_display_first_last # => 'Derek Jeter'

boxscore.battings[0].batters.select do |batter|
  batter.rbi.to_i > 0
end.count # => 4

boxscore.pitchings.find do |pitching|
  pitching.team_flag == 'away'
end.out # => '24'
```

## Traverse's XML API

When in doubt, check the spec file.

### Children

Let's say you're working with the following node of XML:

```xml
<foo>
  <bar ...>
    ...
  </bar>
</foo>
```

Assuming you've wrapped the XML in a `Traverse::Document` named `foo`, you can
traverse to the `bar` node with a simple method call:

```ruby
foo.bar # => returns a representation of the bar node
```

If there are in fact many `bar`s inside `foo`, Traverse will transparently
collect them in an array. You can access that array by pluralizing the name of
the individual nodes:

```ruby
foo.bars # => an array containing all of the bars
foo.bars.first # => grab the first bar
```

Traverse will also do its best to transparently collect singularly-named nodes
inside of pluralized parents. Example:

```xml
<foo>
  <bars>
    <bar ...>
      ...
    </bar>
    ...
    <bar ...>
      ...
    </bar>
  </bars>
</foo>
```
```ruby
foo.bars # => an array of all of the bars
foo.bars.first # => the first bar
```

This won't work if the pluralized parent node has attributes or if its children
aren't all singularized versions of itself! Twitter's timeline is an example;
the parent node's name is `statuses`, and its children are all named
`status`, but the `statuses` node has an attribute.

### Attributes

If your XML node has an attribute named `foo`, you can access that attribute
with a simple method call:

```ruby
my_node.foo # => return the attribute's value
```

Notice that there might be a conflict if your node has both an attribute named
`foo` _and_ a child `foo`. Traverse puts children first, so the attribute `foo`
will get clobbered.

To get around this, Traverse provides a backdoor for when you really want the
attribute:

```ruby
my_node.foo # => grabs the child named foo
my_node['foo'] # => grabs the attribute named foo
```

### Text

If you traverse to a node that has no attributes and contains only text, you
can grab that text with a simple method call. Let's suppose you have the
following XML:

```xml
<foo>
  <bar>
    This bar rocks!
  </bar>
</foo>
```

You can grab the text like this:

```ruby
foo.bar # => "This bar rocks!"
```

What if `bar` does have attributes? Then you'll need to use the `text` method:

```xml
<foo>
  <bar baz="quux">
    This bar rocks!
  </bar>
</foo>
```
```ruby
foo.bar # => grabs a representation of the node, not the text!
foo.bar.text # => "This bar rocks!"
```

I know what you're thinking: what the hell do I do if my node has text inside
it but I want to grab its attribute named `text`???

```ruby
foo.bar['text'] # => grab bar's attribute named 'text'
foo.bar.text # => grab bar's text contents
```

## Traverse's JSON API

Again, when in doubt, check the spec file.

Traverse doesn't really do anything magical with JSON data; under the hood, it
uses `YAJL` to parse JSON into a `Hash`, and hashes are alredy pretty
traversable in Ruby. But, to maintain consistency with the XML API, you can
happily traverse JSON using method calls instead of querying a hash.

## Helper methods
Traverse provides a few helper methods to help with traversal.

If you're traversing some JSON, the ```_keys_``` method will be available. It
returns an array containing the names of the JSON object's keys.

```ruby
search._keys_.count
# =>  2

search.first._keys_
# =>  ["user","favorited","source","id","text","created_at"]
```

If you're looking at XML, the ```_children_``` and ```_attributes_``` methods
will be available. The ```_children_``` method gives you an array of
traversable child nodes, and ```_attributes_``` gives you a hash of
attribute-name/attribute-value pairs.

## Contributors!

Traverse wouldn't be possible without help from friends. Thanks!

- [muffs](https://github.com/muffs)
