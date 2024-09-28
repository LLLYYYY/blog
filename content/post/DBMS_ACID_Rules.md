---
title: Database Management System ACID Rules
date: 2024-09-28
categories:
- General
tags:
- random_notes
---

> ACID means `Atomicity`, `Consistency`, `Isolation` and `Durability`. 

## A: `Atomicity`:
One transaction being atomic meaning it either succeeds completely or fails completely. If any of the statements constituting a transaction fails to complete, the entire transaction fails and the database is left unchanged. This is guaranteed in all situation, even in power failure, crashes etc. 

## C: `Consistency`:
The DB data must be changed in allowed ways. The DB data must satisfy all defined rules (database constraints), including [constraints](https://en.wikipedia.org/wiki/Integrity_constraints "Integrity constraints"), [cascades](https://en.wikipedia.org/wiki/Cascading_rollback "Cascading rollback"), [triggers](https://en.wikipedia.org/wiki/Database_trigger "Database trigger"), and any combination thereof. This is not directly equals to data corruption? But only consider the defined rules as database constraints:  including [constraints](https://en.wikipedia.org/wiki/Integrity_constraints "Integrity constraints"), [cascades](https://en.wikipedia.org/wiki/Cascading_rollback "Cascading rollback"), [triggers](https://en.wikipedia.org/wiki/Database_trigger "Database trigger"). 


## I: `Isolation`:
The DB data must be modified the same when multiple concurrent read/write transactions on it as if the transactions are sequencially executed. This can be implemented in different isolation ways:

### Serializable:
The highest isolation level. It requires read and write locks (acquired on selected data) to be released at the end of the transaction. Also _range-locks_ must be acquired when a [SELECT](https://en.wikipedia.org/wiki/Select_(SQL) "Select (SQL)") query uses a ranged _WHERE_clause, especially to avoid the _**[phantom reads](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Phantom_reads)**_ phenomenon. Strong isolation guaranteed. We can assume the concurrent transaction as if it is executed sequentially. 

### Repeatable reads:
Similar to `Serializable`, but could have `phantom reads` and `write screw` phenomena. 

- Phantom Read:
	- Official explanation: A _phantom read_ occurs when a transaction retrieves a set of rows twice and new rows are inserted into or removed from that set by another transaction that is committed in between.
	- Simpler explanation: When a transaction has begun, the `SELECT` results can still be affected by another transactions' committed write. Therefore, two `SELECT`s queries in the same transaction can output different results given the execution of the `SELECT`, some other transactions had modified the data. 
- Write Screw:
	- If two write happens on the **same column** in the same table, resulting in the column having data that is a mix of the two transactions.


### Read committed:
Not only have `phantom reads`, but also `non-repeatable reads`. This isolation level only keeps read locks before [SELECT](https://en.wikipedia.org/wiki/Select_(SQL) "Select (SQL)") operation is finished. Meaning that it only guarantee that it won't read `uncommitted` data. But if another transaction commited the data modification, this transaction would read the updated data after the transaction already starts. 

- Non-repeatable Reads: This is different to `Phantom Read` as the `Phantom Read` means the two SELECT operations in the same transaction could output different number of rows (as *new data* are inserted), but old data in-place modification could be isolated. While the `Non-repeatable Reads` means not only the returned row number, but the modified in-placed rows could also be different. 

### Read uncommitted:
The lowest isolation level. The read in transaction can read `UNCOMMITTED` data from other transaction, named `dirty reads`. What's worse, the other transaction could `rollback` their transaction, which means the data `dirty read` from the `SELECT` may not be actually saved to the DB. 

## D: `Durability`:
Durability guarantees that once a transaction has been committed (returning success), it will remain committed even in the case of a system failure (e.g., power outage or crash).