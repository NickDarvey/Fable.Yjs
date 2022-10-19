## Fable.Yjs.Adaptive

### TODO

1. IndexList starts at 1 is probably why tests are failing

## Fable.Yjs.Adaptive.Elmish

A library so 

### Codec

Our **app schema** describes the structure of our app data, that is, what is in memory while our app is running. Our **state schema** describes our state data, that is, what is persisted in browser storage and synchronized with peers via the app's companion.

Our state schema _must_ be decoupled from our app schema because:
1. Our app schema will change over time as our app changes. The state schema must be protected from breaking changes to the app schema.
   So, we need an anti-corruption layer between the two schemas.
1. Only a subset of app data needs to be persisted.
   So, we need to be able to select what should be persisted.

If we have decoupling, we need an explicit description of how our app schema translates to our state schema and vice versa. We need to be able to _encode_ that app data as state data and _decode_ state data into app data. We need a codec. 

<!-- Changes to data need to flow bi-directionally. That is, changes made to state data need to be made to app data and changes made to app data (such as through interactions with the app) need to be made to state data. [That codec needs to do thingo] -->

Something-something properties:
- At-the-edge-ability


Given an Elmish app and given Yjs state, here we are trying to design a codec for translating between the two.

**Incremental**

If we were to observe a running Elmish loop, we wouldn't be able to capture the operations being applied to the app's model. They're opaque to an observer because they're inside the `update` functions. Instead, we only have access to each successive `'model`, that is, the consequence of operations.

> For example, an Elmish `'model` may contain a list and an interaction may add two items into that list, but from the outside, we only see the new list, not two 'add' operations.

If we observe a Yjs data structure, we'd see all of the operations that occur. It records every operation performed on data and shares it with peers for synchronization.

> For example, a Yjs `Y.Array` and an interaction may add two items to that array and from the outside, we can observe two add operations. (And we can combine all the operations to see the 'current' state of the array.)

If we're to connect the two, we need to bridge these two worlds, that is, we need to be able to go from our changes represented as success models to our changes represented as the operations themselves.

```
'model -> 'model -> 'operations
```

...which looks just like a classic differencing (diffing) function

```
'document -> 'document -> 'delta
```

Lucky for us this is kinda a solved problem, even specifically with our technologies of choice.

['Incremental computation'](https://github.com/fsprojects/Fabulous/issues/258#issue-391515540) has already been needed where people want to build apps that use immutable data structures and performantly update a mutable DOM.

Part of this work has been to [efficiently](https://github.com/fsharp/fslang-suggestions/issues/768) diff two models.






Encoding



Decoding

Updates from Yjs and applying them to Adaptive.

Adaptify would expect a model, doing the diff itself


1. Do we have a symmetric codec or an asymmetric one?
1. What steps do we break the de/encoding into?

   - Yjs schema is the one that can't change easily, so...?

   ```
   App Schema --> Adaptive App Schema --------------------------> Yjs Schema

   App Schema --------------------------> Adaptive Yjs Schema --> Yjs Schema

   App Schema --> Adaptive App Schema --> Adaptive Yjs Schema --> Yjs Schema
                  ^^^^^^^^^^^^^^^^^^^     ^^^^^^^^^^^^^^^^^^^             
                  Shape of app schema     Shape of Yjs schema
   ```

   ```
   App Schema <-- Adaptive App Schema <-------------------------- Yjs Schema
                                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^
   If this is generated by Adaptify, the only way to update is Update(aWholeThing) which is not ideal for an array, for example.
   but does this matter?
   ```

   ```
   App Schema --> Adaptive App Schema -----------encode----------
            ^                                                   v
            ------------decode----------Adaptive Yjs Schema <-- Yjs Schema
   ```


   ```
   // Update an adaptive app schema from an app schema
   type Update1 = 'aschema -> 'schema -> 'aschema

   type Encode = 'aschema -> Y.Doc

   type Decode = Y.Doc -> 'aschema

   type Update2 = 'aschema -> 'schema observable

   App Schema -> Adaptive App Schema
   ```

   if we have an adaptive yjs schema, how do we get from that into an app schema? do we have to observe each property with an 'addcallback'?

   maybe it would be easier to start with something that just uses the .Update on an Adaptify generated aschema?


   next step is to just write the Elmish bit, writing the types for the function building blocks i think i need.!!


Value could be rewritten as

[<Erase>]
type Value =
   | Map of Y.Map<Value>
   | Array of Y.Map<Array>
   | Text of Y.Text
   | String of string
   | Number of ?
   | Object of obj

https://github.com/fable-compiler/Fable/issues/2492


This integration would be simpler if the yjs changes only appeared in the adaptive schema, didn't need to make it back to non-adaptive.
this would complicate the app developers job because they couldn't count on their model being accurate.
maybe that is necessary for perf because replacing the model each keystroke will be expensive.
regardless, that should be dealt with later as a optimization
some data needs to get back into the model otherwise there is no point in having an elmish style app at all

