# Fuego

Fuego is a social extensible Web service built on NodeJS and PostgreSQL.
It provides an open database for experimental applications, UIs, schemas, and data-formats.

Currently Fuego has (plans for) APIs for:

 - Forums
 - User accounts
 - Uploading [Dat Archives](http://dat-data.com)

**Fuego is still in early development.**
Everything is subject to change.

## Background

Fuego is a Web service built in concert with a Web browser named [Beaker](https://beakerbrowser.com).

Beaker uses the [Dat peer-to-peer hypermedia protocol](http://dat-data.com) to host websites.
Dat is a bit like BitTorrent.
It lets users fork, share, and deploy applications from the device; you can start a new site by simply generating a keypair, and publishing the site-files on the network.

This p2p deployment network is key to the fun.
By removing the friction of the hack/share/deploy cycle, we let users share their creations freely.

What Fuego provides is a central backend for Dat apps to read/write to.
Its APIs provide generic data models (such as Forums) which can then be extended with custom attributes.
Different "coordination fields" let apps identify the schemas, formats, and modules they use.
It'll be noisy, and it won't always work right, but it will be fun!

## Philosophy

Fuego is a general-purpose backend to applications, with no server-side middleware.
This is known as a [Two-Tier Architecure](https://www.techopedia.com/definition/467/two-tier-architecture), and it's also in line with the [NoBackend](http://nobackend.org/) philosophy.
Parts of Fuego are inspired by the [remoteStorage spec](https://tools.ietf.org/html/draft-dejong-remotestorage-07).

Fuego provides a predefined set of data-structures with extensible schemas.
These predefined structures enable Fuego to provide aggregation queries, caching, and fine-grained permissions, without needing to trust users.

### Why use predefined data-structures?

Any database should provide aggregate knowledge of the current state, such as "number of votes on this post," or "last user to comment on this thread."
SQL databases do this with relational queries, using joins and groupings.
NoSQL databases use computational piplines such as MapReduce.

Neither of these solutions would work in a Web-facing database.
If the query-model is too open or programmable, it'd be possible for users to DoS the database with expensive queries.
Query cost should be predictable so that rate-limiting can be effective, and the data model should be predictable enough for caching to be applied.

Fuego's solution is to create a suite of common application data-structures, such as "Forum Threads," "Calendar Events," and "Polls."
These structures will include fixed properties and relationships which support the most often-needed aggregate knowledge.
Users/Applications can then extend the structures with additional properties.
Basically, Fuego has a fixed approach to [normalization](https://en.wikipedia.org/wiki/Database_normalization) and the resulting queries.

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

## Prior Art

There are several existing platforms/protocols that are similar in
scope to Fuego, but each has limitations that make it unsuitable for
Fuego's vision of providing an extensible database Web service with open schemas
and data formats.

### Open Platforms/Protocols

#### [Hoodie](https://hood.ie)

Hoodie provides an API for building client-side applications without
thinking about the backend. It uses CouchDB to sync the backend with
data stored in the browser, which allows applications to operate in
the face of poor network connectivity. The project's maintainers claim
that ["Hoodies' API is optimized for...being intuitive and
accessible."](http://docs.hood.ie/camp/hoodieverse/hoodie-concepts.html). By
default, Hoodie includes modules for user signup, data storage, data
sharing, and sending emails, but it is
[extendable](http://docs.hood.ie/camp/techdocs/api/client/hoodie.html#extend)
beyond that core functionality.

##### Drawbacks

It does not appear that the platform is suitable for rapid deployment.
By the project maintainers' [own
admission](http://hood.ie/intro/#get-started),
"unfortunately deployments aren't really easy yet."

#### [remoteStorage](https://remotestorage.io)

remoteStorage is an open standard for enabling
[unhosted](https://unhosted.org/) web applications. It intends
to free app developers from the burden of hosting and maintaining a
central database, and to grant users full control over where to store
their application data.

##### Drawbacks

remoteStorage's data model is built on "documents" and "folders",
wherein folders reference a list of and documents documents have the
following properties:

* Version
* Content type
* Content length
* Content

While this is sufficient for many use cases, it isn't conducive to advanced
querying, thus limits the scope of applications that can be built with
the protocol.

### Proprietary Platforms

Much like Hoodie and remoteStorage, [Firebase](https://firebase.google.com),
[deployd](http://deployd.com), [Kinvey](https://kinvey.com), and
[Parse (RIP)](https://parse.com), offer(ed) a platform for building
applications without needing to maintain a backend data storage
service.

While each of these tools is well-received and appreciated by many
developers, they each suffer from a lack of extensibility and
less than convenient data export, which is a central piece of Fuego's
goal to build a platform in which users can seamlessly and freely
import their data into any compatible application.
