# Event Ordering

Let's consider a pure event based system where events consist of a unique **identifier**, 
a **timestamp** and the actual payload which is not used for ordering events.

In addition to the above, an event may causally dependent on other events. For example,
an event which deletes a row would only make sense if the event that inserted it was
applied before it.  In effect, every event also has a **parents** property to track all the
dependencies.  Even in distributed systems, most events happen in a linear chain and so
a single parent is sufficient to describe the event dependnecies.  Multiple parents do
occur when two separate event chains are being _merged_ (i.e. their effects are being 
combined). 

## Distributed Clock

Ideally, ordering events by timestamp should guarantee that all the parents of an event
precede the event.  But due to the nature of distributed clock skew, this may not be 
sufficient.

## Perfect Ordering

Let's add a new **effective timestamp** property to each event, defined as follows:

```
effective timestamp of event = max(timestamp of event, effective timestamp of parents)
```

We can now use the tuple **(effective timestmap, timestamp, identifier)** to sort all
events.  When we do this, we get the following nice properties:

1. The sorting is **deterministic**.  It does not depend on the order of the input events.
2. The sorting is **causally consistent**. All the parents of an event precede the event.
3. The sorting is **mostly chronological**.  The chronological order is only broken for
events whose local timestamps are earlier than some parent or ancestor event.  But because
the second element of the tuple is the actual timestamp, ordering within these out-of-order
events is still locally chronologoical.
4. The sorting can be efficiently performed incrementally.  It does require maintaining the
effective timestamp (either as part of the sorting state or preferably within the event
itself as the max computation can be cheaply performed at each event generation time).

## Generalizations

The problem can be generalized in some intereseting ways to compute a full order with
least entropy given a specific partial order.  That will be a separate note.
