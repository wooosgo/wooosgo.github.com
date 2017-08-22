---
layout: post
title: "mongodb basics - shell"
date:   2017-07-20 10:00:01 +0900
categories: mongodb
layout: post
---

## mongodb
db.col.findOne() equals to db.col.find().limit(1)

projection
db.col.find( {}, {title: true} );

projection into the embedded document
db.col.find( {}, {item:1, "size.uom":1} )

accessing arrays -- only 1 last item from the array
db.col.find( {}, {item:1, instock:{$slice: -1}} )

null column
--item field has null
db.col.find({ item : null});
--items does not contain the item field
db.col.find({ item: {$exists: false} } )
