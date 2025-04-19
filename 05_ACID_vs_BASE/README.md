# ACID vs BASE

## 1 Intro

- SQL vs NoSQL
  - so far we have discussed the differences around functional features
    - flexible queries ex: query by value etc then use relational DB
    - need key value look up ex: url look up in tiny url we use key value store
  - now we will discuss about **non functional features**

## 2 Relational Databases (ACID)

- CRUD operations

  - operations are the foundations of a DB
  - CRUD operation allows for interaction with DB
  - CRUD operations together helps us with persisting data

- Transactions and Error handling:
  - Transaction simplify error handling
  - No need to worry about returning to prev state if some operations succeed and other fail
  - Txn guarantees for easy reasoning about DB state before and after a txn
  - These safety guarantees are described as ACID

### 2.1 ACID (Atomicity, Consistency, Isolation, Durability)

#### 2.1.1 Atomicity

- Something that can not be broken down into further smaller operations
  - Txn can't be broken down into individual patterns
  - Allows us to abort an error and discard all other operations in a txn
  - This prevents a disastrous state in DB

#### 2.1.2 Database Consistency

- Previously we talked about Availability vs Data consistency. Database Consistency is different than that

- All data points within the DB must align in order to be properly read or accepted
- If any data does not meet the conditions and tries to enter the DB, then it will result in a consistency error.
- ex:
  - For govt id if it allows only 3alphabet + 5 digits, adding 4 alphabet + 5 digit will throw consistency error

#### 2.1.3 Isolation

- Problem statement:
  - DBs are typically accessed by multiple clients at the same time
  - This may cause concurrency problems also known as race conditions
- Solution: isolation
  - Concurrently executing transactions are **isolated** from each other
  - Isolation allows txns to operate as if they are the only operation running on the DB
  - This ensures that the final result is consistent, even when transactions are run concurrently
  - Txns can commit without interfering with each others operations

#### 2.1.4 Durability

- This ensures that once a txn is committed to the DB, it is permanently preserved
- in the event of a failure, mechanisms viz backup and tx logs are in place to restore committed txns.

### 2.2 When ACID or Relational DB is a good fit?

- When data is structured
- When use case requires lots of complex querying
- When you don't want to show any inconsistent data to the user

### 2.3 Limitation of Relational DB or ACID?

- DO NOT SAY "Relational DBs don't scale horizontally" ❌

  - This was true 30 years before but not anymore
  - Relational DBs are scalable but extremely complex to scale and also to deploy because of the challenges ACID brings
  - https://www.mysql.com/products/cluster/

- It is difficult to scale relational DBs horizontally because of the ACID challenges in distributed system

#### 2.3.1 ACID challenges in distributed system

- Txs handling is complicated when there are multiple DB nodes to commit and abort all txns
- one of the common protocol to achieve atomic and consistent txns in distributed system is 2PC
  - 2PC (Two phase commit protocol)
    - This protocol requires a careful coordination b/w the nodes, which happens in two phases
      - **prepare**
        - this asks each node if it's able to promise to carry out the txn
      - **commit**
        - this carries out the txn if a promise was made
- **Blocking in Distributed systems**:
  - Records may be blocked for a short time during a txn, but network factors may significantly increase the blocking time
  - records may remain blocked until all the nodes agree to execute the txn
  - network latency can cause delays in communication b/w nodes
  - large part of the app may become unavailable till the txn is committed

## 3 NonRelational DBs (BASE)

- started becoming popular after 2000s to fix the challenges we had from ACID
  - Transactions != Scalability
  - to solve this dynamoDB was invented as one of the first SQL killer
- Types of Non-relational DB:
  - Key value store
  - Document store
  - Wide-column store
  - Graph store
- So all these types above have the BASE property and is opposite of ACID
- Diff b/w ACID vs BASE
  - ACID
    - Enforces consistency
    - Pessimistic
  - BASE
    - Optimistic
    - Eventual consistency
    - Data in non-relational DB is always in a state of flux
- BASE:
  - Basically Available
  - Soft state
  - Eventual consistency

### 3.1 BASE in detail:

#### 3.1.1 Basically Available

- Guarantees it's always possible to read and write data, even though data might not be consistent
  - u might read outdated data
  - ur writes might not be persisted after conflicts are reconciled

#### 3.1.2 Soft state

- Lack of consistency can lead to changes in data state without any current interactions with the app
- But with eventual consistency, the system will create consistency and better state over time

#### 3.1.3 Eventual consistency

- The system uses a loose consistency model, in which data will eventually become consistent once the input stops
- This allows for high availability and scalability even for distributed systems
- In eventual consistency, it is possible for the system to have a temporary inconsistent data

### 3.2 Benefits of BASE

- Since we don't have atomicity, overheads like 2pc becomes obsolete
- Without the need to enforce consistency across all nodes, the DB can be scaled much easier
- Without isolation, there is no need for blocking behaviour and this allows for data to be always available.

### 3.3 When to use BASE?

- You have a large amount of data that isn't easy to fit in a tabular form
- When use case requires high availability and performance and can live with a lack of consistency across nodes

### 3.4 Limitations

- BASE DBs were supposed to be SQL killers but SQL dbs are definitely still one of the most popular ones
- Temporary inconsistency can not be hidden from the user, then we can not use BASE
  - ex: fintech apps can not use BASE
- BASE lacks standardization compared to ACID. BASE comes in many different formats and rules
  - no ACID guarantees
