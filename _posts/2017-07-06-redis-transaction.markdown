---
layout: post
title: "Redis Transaction"
date:   2017-07-6 10:00:01 +0900
categories: redis
layout: post
---

## Transaction

**MULTI , EXEC, DISCARD**
>
Errors inside Transaction
1. general errors **before EXEC**
2. key computational errors **after EXEC**
>
**even when a command fails, all the other commands in the queue are processed**  
(result with both OK and ERR)
>
* Redis does not support Rollback (for performance reason)
>

**WATCH, UNWATCH**
>
* make the EXEC conditional: perform the transaction only if none of the WATCHed keys were modified  
* UNWATCH for flush all watched keys  
* Redis script is used for transaction as well.  
