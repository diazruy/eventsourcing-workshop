# Ruby Microservices: Event Sourcing

## Slides

- How to Fail at Microservices ([link](http://bit.ly/msvc-fail), [mirror](https://docs.google.com/presentation/d/1hcg2Q4H6KdFVLNs_nJyi2xfY4UC9GuqPq339zrrtZ3o/pub?start=false&loop=false&delayms=60000&slide=id.p))
- Succeeding with Mircroservices ([link](http://bit.ly/msvc-succeed), [mirror](https://docs.google.com/presentation/d/1hcg2Q4H6KdFVLNs_nJyi2xfY4UC9GuqPq339zrrtZ3o/pub?start=false&loop=false&delayms=60000&slide=id.p))

## Migrating Monolith to microservices

- 1 to 3 year process (after you've stopped making mistakes)

- SOA
  - A lot of logic in the messaging system itself (to ensure one time only message delivery)
  - shields dev from distributed systems
- Microservices: makes dev embrace (messages may be received 0, 1, or multiple times)

- Monolith is not a natural step to microservices

- Distribute data, move it to the function that deals with it

## Pub/Sub
- Send a command
- Handle a command
- Publish an event
- Subscribe to events

- Tell, don't ask (tell something to do something with its data,
don't ask it about it's data and make decisions based on that)

- Event design needs to be done upfront because changing the events
will be costly
- Complete event history is replayable

## CQRS (Command Query Responsibility Segregation)

- Front End (clients) issues commands to
- Services which publish events consumed by
- Data Aggregation which puts together representations to (Transactional Data, Logs)
- View Data (Tables, Graphs, etc.) (reconstructable from the Data Aggregation logs)

## Vocabulary

- Command
- Event
  - Attributes are copied from the command to the event
- Time
  - Business/Effective time (start time)
  - Processed (mechanical) (end time)
- Commands and Events are messages
- Messages are _hanlded_
- Handler - does the work when receiving a message
- Stream - ordered, immutable sequence of Events over time pertaining to a given subject
  - Name: composed of category and ID. E.g. `account-123`
  - Read in order (generally from pos 0 to pos n)
- Entity
  - Data models
  - Corresponds to a single stream
  - Events in a Stream are _projected_ (or sourced) into an Entity
- Projection
  - Data in the messages applied toward an Entity. Mutating the Entity each time
    so at the end you have an Entity that reflects the whole screen
  - This is accomplished via a Reader (or Consumer -this might be a misunderstanding of mine,
    Consumer appears to have a different connotation) that traverses the Events in the Stream
    applying transformations to a new "virgin" Entity object. These tranformations
    are applied methods on the Entity (tell don't ask) that apply the transformations
    from the current Event being read
  - Should set the ID on the Entity on every event since you can't assume that the events will be received
    in the "correct order"
  - Snapshots can be used to "warm" the cache of a Projection so it doesn't need to
    reprocess a complete stream (useful when streams have millions of events)
- Streams can exist for:
  - a particular entity in a class (Entity Stream) E.g. `account-123`
  - all messages for all entities of a given class (Category Stream). E.g. `account`
    - Has two positions:
     - a global position (for the category)
     - the entity position (within it's own Entity Stream)
- Readers - When reaching the end of a stream can:
  - Wait for next message or
  - Terminate
- Readers dispatch messages to Handlers
  - Can keep track of recent position ("snapshot") so that in the event of an unexpected
    service termination, it can recover without returning to the start
  - Before the Reader starts, it reds the last recorded position
- Handlers can (and probably should) be idem-potent so that a recycle (unexpected restart) doesn't re-apply
  an event twice
- Readers can be concurrent (usually by error, not intended. microservices tend to run on a single node)
  - This can circumvent idempotence checks
  - To remediate this, the Stream has a Version, which is the `n` of the last event in the stream
    - Version -1 represents a stream that doesn't exist (is empty)
    - We can pass the expected version _prior_ to a write so that we can raise an error
      if there is a version mismatch
- Consumers (not the same as Reader it turns out!) bind handlers to streams. A Host registers consumers
  that now keep running in the background to process events
  - Account Component is only a service when it's running (via a Consumer)
  - In a separate project, you could package multiple components (as gems), and run them together
    as a service unit
- Reply Streams: Inter service communication. To prevent services having to listen to _every_ event
  another service emits. When issuing a Command, the caller can provide a `reply_stream` address
  - Replying should be done in a separate Handler that only gets triggered when the
    original Eventgets completed
- A Client describes the SDK/API to interact with a service


### Q
- How do you insert to all services atomically?
- As a service, what if you miss a message?
- `method.(x,y)` syntax
  - Shorthand for `method.call(x,y)`
- Versioning withdrawal

### Misc

- Run the service: `script/start`

## Reference

- GTAC 2007 - Ed Keyes - [Sufficiently Advanced Monitoring is Indistinguishable from Tesing](https://www.youtube.com/watch?v=uSo8i1N18oc)
- GOTO 2014 - Martin Fowler - [Microservices](https://www.youtube.com/watch?v=wgdBVIX9ifA)
- GOTO 2016 - Clemens Vasters - [Messaging and Microservices](https://www.youtube.com/watch?v=rXi5CLjIQ9k)
