+++
title="Recreate ElasticSearch index for integration testing"
date=2013-10-07
+++

I fought against this for most of last week so now that I solved it I figured I
could share it with the rest of the world (not that I had much fun running tons
of Jenkins' builds to see if it was fixed...).

So, we have a Rails app that uses ElasticSearch for a few features. There's a
single index that we query, and for integration test purposes we create a fake
test index so we can go through the whole stack. We are using
[Tire](https://github.com/karmi/retire) with its `Persistence` module, so in our
`spec_helper.rb` (asuming we have a model called `Book`) we had something along
the lines of:

```ruby
before(:each) do
  Book.index.delete
  Book.create_elasticsearch_index
end
```

There was some more unrelated stuff in there (like deleting the index after the
whole suite was completed, or using
[Webmock](https://github.com/bblimke/webmock) to ensure that we are not making
any unwanted HTTP requests), the only detail that I want to mention is that you
might want to wait for a
[yellow status](https://github.com/karmi/retire/issues/537#issuecomment-11124205)
before each test to avoid "No active shards" errors.

Back to the problem at hand, nothing seems wrong with this, but then we started
having random 404 errors because the index was missing during the examples. But
it should be there, right? It should be created right after it was deleted.

I enabled debugging on Tire's config, and I found something like the following:

```sh
# 2013-10-04 09:25:05:839 [DELETE] ("test_index")
#
curl -X DELETE http://some-server:9200/test_index

# 2013-10-04 09:25:05:840 [200]
#
# {
#   "ok": true,
#   "acknowledged": true
# }

# 2013-10-04 09:25:05:852 [HEAD] ("test_index")
#
curl -I "http://some-server:9200/test_index"

# 2013-10-04 09:25:05:852 [200]
```

So, right after the `DELETE` request, there's a `HEAD` request against the same
index, which returns 200.

What.

First of all, the `HEAD` request comes from Tire doing
[an existence check before creating the index](http://rubydoc.info/github/karmi/tire/master/Tire/Model/Indexing/ClassMethods#create_elasticsearch_index-instance_method).
Makes sense. But why would it return 200 if the `DELETE` request that came just
before that one returned a 200 ok everything is perfect response?

Well, help comes from
[the great people at StackOverflow](http://stackoverflow.com/questions/19182682/elasticsearch-async-delete-200-just-after-deleting-index-in-rails-app/19224515).
First comment: turns out
[the entire ES HTTP API is asynchronous](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-http.html).
So yeah, I get the 200 for the `DELETE` request but the index wasn't necessarily
deleted yet. So, what do we do? Follow the suggestion at the accepted answer for
that question: poll ES until we are sure that the index was deleted.

So, in our Tire initializer, I added:

```ruby
Tire::Index.class_eval do
  def ensure_deleted
    5.times do
      return true unless exists?
    end

    raise "The ElasticSearch index wasn't successfully deleted."
  end
end
```

And then modified the hooks to look like:

```ruby
before(:each) do
  Book.index.ensure_deleted
  Book.create_elasticsearch_index
end

after(:each) do
  Book.index.delete
end
```

So basically, we check five times to see if the index was deleted. I didn't show
the whole log, but in all the failures only one request returned the fake 200
after the `DELETE`, the next one always returned 404 correctly, so limiting it
to 5 tries made sense.

That's it! I hope this can save someone some time and the anger against the
world that I went through.
