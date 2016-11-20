# Fuego

todo
- dumb services
- prior art
- auth
- remotestorage spec?

Fuego is a Generic Web Service built on NodeJS and PostgreSQL.
It is an open database designed to support a wide variety of applications with extensible schemas and predefined data-structures.

Currently Fuego has APIs for:

 - User accounts
 - Uploading [Dat Archives](http://dat-data.com)
 - Forums

**Fuego is still in early development.**
Everything is subject to change.

## Background

Fuego was created to provide a shared social hacking space.
Users can actively experiment with applications, UI elements, data-schemas, and formats -- while still interacting around a shared dataset.
It is chaotic, messy, and damn fun.

Fuego was built for [Beaker](https://beakerbrowser.com), a browser that uses the [Dat peer-to-peer hypermedia protocol](http://dat-data.com).
Fuego interacts with Beaker's p2p protocol to let users rapidly fork, share, and deploy applications.
This is key to the fun: by removing the friction of the hack/share/deploy cycle, we let everybody share their creations.

## Philosophy

Fuego provides a general-purpose backend to applications with no server-side middleware.
This is known as a [Two-Tier Architecure](https://www.techopedia.com/definition/467/two-tier-architecture), and it's also in line with the [NoBackend](http://nobackend.org/) philosophy.

Fuego provides a predefined set of data-structures with extensible schemas.
These predefined structures enable Fuego to provide aggregation queries, caching, and fine-grained permissions, without needing to trust users.

Fuego is, most importantly, a **dumb service**.

### Dumb Services

Dumb services are TODO
--Why use Two-Tier / NoBackend?--

### Why use predefined data-structures?

Any database should provide aggregate knowledge of the current state, such as "number of votes on this post," or "last user to comment on this thread."
SQL databases do this with relational queries, using joins and groupings.
NoSQL databases use computational piplines such as MapReduce.

Neither of these solutions would work in a Web-facing database.
If the query-model is too open or programmable, it'd be possible for users to DoS the database with expensive queries.
Query cost should be predictable so that rate-limiting can be effective, and the data model should be predictable enough for caching to be applied.

Equally important is the need to provide a flexible, fine-grained permissions model.
These permissions need to be defined in a way that fits the data-structures involved, but without getting bogged down in complex permission-specifications.

Fuego's solution is to create a suite of common application data-structures, such as "Forum Threads," "Calendar Events," and "Polls."
These structures will include fixed properties and relationships which support the most often-needed aggregate knowledge.
Users/Applications can then extend the structures with additional properties.

### What do the structures look like?

This is one example forum definition, for sharing music playlists:

```js
{
  title: "Hot Playlists",
  description: "Share your favorite music playlists!",
  threadSchema: {
    description: "A playlist",
    type: "object",
    properties: {
      title: { type: "string" },
      tracks: {
        type: "array",
        items: { type: "string" }
      },
      comment: { type: "string" }
    },
    "required": ["title"]
  },
  postSchema: {
    description: "A comment",
    properties: {
      comment: { type: "string" }
    }
  }
}
```

### Isn't this a little silly?

Basically, Fuego has a fixed approach to [normalization](https://en.wikipedia.org/wiki/Database_normalization) and the resulting queries.
Making predefined application data-structures is probably going to result in lots of funny opinions creeping into the DB.
It is, in fact, a little silly!
And it isn't fully generic, so Fuego won't work for all cases.

But the way to look at Fuego is as a "Data-Structures Web Service."
It's like any other Web Service, in that it has lots of opinions baked into it, and it doesn't handle all requirements.
But, unlike most other services, it *is* very open and relatively customizable, and that's why it's worth having.

## Data Structure APIs

### Forums

#### GET /forums

Returns:

```
{
  forums: Array of {
    id: String
    owner: UserId
    createdAt: String
    title: String
    description: String
  }
}
```

#### GET /forum/{id}

Returns:

```
{
  id: String
  owner: UserId
  createdAt: String
  title: String
  description: String
  topicSchema: JSONSchema
  postSchema: JSONSchema
}
```

#### POST /forum

Expects:

```
{
  title: String
  description: String
  topicSchema: JSONSchema
  postSchema: JSONSchema
}
```

### Topics

#### GET /topics?{forum,offset,limit,include}

- `forum` ForumID
- `offset` Number
- `limit` Number
- `include` Array of strings, properties to include from the topic schema. Can be '*' to include all.

Returns:

```
{
  topics: Array of {
    id: String
    author: UserID
    createdAt: Date
    title: String
    properties: Object, defined by include
  }
}
```

#### GET /topic/{id}?{include}

- `include` Array of strings, properties to include from the topic schema. Can be '*' to include all.

Returns:

```
{
  id: String
  forum: ForumID
  author: UserId
  createdAt: Date
  title: String
  properties: Object, defined by include
}
```

#### POST /topic

Expects:

```
{
  forum: ForumID
  title: String
  properties: Object, must validate against the forum's topicSchema
}
```

### Posts

#### GET /posts?{topic,offset,limit,include}

- `topic` TopicID
- `offset` Number
- `limit` Number
- `include` Array of strings, properties to include from the post schema. Can be '*' to include all.

Returns:

```
{
  posts: Array of {
    id: String
    author: UserId
    createdAt: Date
    properties: Object, defined by include
  }
}
```

#### GET /post/{id}?{include}

- `include` Array of strings, properties to include from the post schema. Can be '*' to include all.

Returns:

```
{
  id: String
  topic: TopicID
  author: UserId
  createdAt: Date
  properties: Object, defined by include
}
```

#### POST /post

Expects:

```
{
  topic: TopicID
  properties: Object, must validate against the forum's postSchema
}
```

### Users

#### GET /user/{id}

Returns:

```
{
  id: String
  createdAt: Date
}
```