# Pulse

**WARNING STILL IN DEVELOPMENT**
To see what is ready to use at this stage, read PROGRESS.md

Pulse is an application logic library for reactive Javascript frameworks with support for VueJS, React and React Native.

Created by @jamiepine, @f3ltron and @joaof

Pulse is lightweight, modular and powerful, but most importantly, easy to understand.

## Features

- ⚙️ Modular stucture using "collections"
- ❤ Familiar terminology
- 🔮 Create data relations between collections
- ✨ Automatic data normalization
- 👯 No data repetition
- ⚡ Cached indexes for performance
- 🕰️ History tracking
- 📕 Error logging
- 🔒 Bad data protection
- 🔥 Supports Vue, React and React Native
- Provides a structure for basic application logic (requests, error logging, authentication)

## install

```
npm i pulse-framework --save
```

## Simple example

Manually setting up pulse without a framework

```js
import { PulseClass } from "pulse-framework";

new Pulse({
  collections: {
    channels: {},
    posts: {}
  }
});
```

## Pulse with VueJS

```js
import Pulse from "pulse-framework";

Vue.use(Pulse, {
  collections: {
    channels: {},
    posts: {}
  }
});
```

Now you can access collections and the entire Pulse instance on within Vue

```js
this.$channels;
// or
this.$pulse.channels;
```

## Collections

Pulse provides "collections" as a way to easily save data. They automatically handle normalizing and caching the data. Each collection comes with database-like methods to manipulate data.

Once you've defined a collection, you can begin saving data to it.

```js
// basic api call
let response = await axios.get(`http://datatime.lol/feed_me`);

this.$channels.collect(response.data);
```

Collecting data works like a commit in Vuex or a reducer in Redux, it handles preventing race conditions, saving history for time travel and normalizing the data.

## Primary Keys

Because we need to normalize the data for Pulse collections to work, each piece of data must have a primary key, this has to be unique to avoid data being overwritten.
Lukily if your data has `id` or `_id` as a property, we'll use that automatically, but if not then you must define it in a "model". More on that in the models section below.

## Indexes

You can assign data an "index" as you collect it. This is useful because it creates a cache under that name where you can access that data on your component.

```js
pulse.collect(somedata, "indexName");
```

Inside pulse the index is simply an ordered array of primary keys for which data is built from. This makes it easy and effienct to sort and move and filter data, without moving or copying the original.

If you do not define an index when collecting data there is a default index named 'default' that contains everything that was collected without an index present (coming soon).

You can now get data using `mapCollection()`

```js
// vue component example
import { mapCollection } from "pulse-framework";

export default {
  data() {
    return {
      ...mapCollection("channels", ["indexName", "subscribed"])
    };
  }
};
```

You can also assign custom names for the data properties within the component

```js
...mapCollection("channels", {
  customName: 'indexName'
  sub: 'subscribed'
})
```

## Data

Pulse has the following "forward facing" data types for each collection

- Indexes (cached arrays of data based on the index of primary keys)
- Data (custom data, like state)
- Filters (like getters, these are cached data based on filter functions you define)
- Actions (these )

## Mutating data

Changing data in Pulse is easy, you just change it.

```js
this.$channels.currentlyEditingChannel = true;
```

We don't need mutation functions like "commit" in VueX because we use Proxies to intercept your changes and queue them to prevent race condidtions. Those changes are stored and can be reverted easily.

## Actions

These can happen within actions in your pulse config files, or directly on your component.

```js
// put data by id (or array of IDs) into another index
pulse.$channel.put(2123, "selected");

// move data by id (or array of IDs) into another index
pulse.$channel.move([34, 3], "favorites", "muted");

// change single or multiple properties in your data
pulse.$channel.update(2123, {
  avatar: "url"
});
// replace data (same as adding new data)
pulse.$channel.collect(res.data.channel, "selected");

// removes data via primary key from a collection
pulse.$channel.delete(1234);

// removes any data from a collection that is not currently refrenced in an index
// it also clears the history, so undo will not work after you run clean.
pulse.$channel.clean();

// will undo the last action
pulse.$channel.undo();
```

## Another example

This might help you understand how you could use Pulse in a real world scenario

```js
new Pulse({
  collections: {
    // collection name
    channels: {
      // filters are bound to collection data and cached
      filters: {
        // if you collect data with a name corresponding to one of these, they'll be populated. EG: collect(data, 'suggested')
        suggested: [],
        // or if
        favorites: [],
        //
        current: {}
        // filters can also use our filter API
        recentlyActive: ({ filter }) => {
          return filter({
            from: "subscribed",
            byProperty: "lastActive"
          });
        }
      },

      // a place to define any extra data properties your collection might need
      // they're reactive and accesable directly, just like regular state
      data: {
        channelOpen: false,
        tab: 'favorites'
      },

      // actions aren't required, but can do many things at once
      actions: {
        // first paramater is an object of any functions or collections to use from Pulse, everything after is yours.
        async getSubscribedChannels({ request, collect, posts }, username) {
          // built in request handler
          let res = await request.get(`/channels/subscribed/${username}`);
          // collect data
          collect(res.data.channels, "subscribed");
          // collect to a sister collection
          posts.collect(res.data.posts, [
            // save it under the namespace "subscribed"
            "subscribed",
            // also under the namespace of the username
            res.data.channel.username
          ]);
        }
      }
    },
    // collections don't need any paramaters to function
    posts: {}
  }
});
```

From what we've just created, you can now use mapCollection to access data

```js
...mapCollection('channels', ['subscribed', 'recentlyActive', 'channelOpened'])
```

As you can see we can access indexes, filters and regular data easily in our component, and it's all cached, reactive, and based on our collections internal database.
