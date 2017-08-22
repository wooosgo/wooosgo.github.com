---
layout: post
title: "MongoDB modeling"
date:   2017-07-6 10:00:01 +0900
categories: mongodb
layout: post
---

## modeling 1:N

* consideration :

- Will the entities on the “N” side of the One-to-N ever need to stand alone?  
- What is the cardinality of the relationship: is it one-to-few; one-to-many; or one-to-squillions?  

Based on these factors, you can pick one of the three basic One-to-N schema designs:  

1. Embed the N side if the cardinality is one-to-few and there is no need to access the embedded object outside the context of the parent object  **embedding**  
2. Use an array of references to the N-side objects if the cardinality is one-to-many or if the N-side objects should stand alone for any reasons  **child-referencing**  
3. Use a reference to the One-side in the N-side objects if the cardinality is one-to-squillions  **parent-referencing**  


* Two way referencing & Denormalization

You can use bi-directional referencing if it optimizes your schema, and if you are willing to pay the price of not having atomic updates
If you are referencing, you can denormalize data either from the “One” side into the “N” side, or from the “N” side into the “One” side

When deciding whether or not to denormalize, consider the following factors:

1. You cannot perform an atomic update on denormalized data
2. Denormalization only makes sense when you have a high read to write ratio

Consider Denormalization for the sake of
- Read / Write ratio (when Read is much higher than write)
- Reduce application level joins
- but, having more expensive updates

# Rule of thumb #
1. favor embedding (
2. do not embed when accessing the object itself
3. do not embed when high cardinality arrays are on many side
    - 2~300 documents on many side ; no embed
    - 2~3,000 documents on many side ; no array of objectID references

4. Application level join is fast enough (with correct index)
5. Mostly read and seldom write ; good candidate for Denormalizing
6. design your database schema to match the needs of your application
