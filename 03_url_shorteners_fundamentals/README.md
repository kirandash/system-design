# URL shortener

## 1 Question

- long urls are difficult to memorize and hard to spell
- some use cases:
  - url in printables ex: books
  - url in twitter post where max characters is limited
  - to track web analytics on 3rd party URLs ex: clicks
- ex: https://medium.com/@Pure_Soul25/thank-you-donald-trump-afa7e20311b6
  - short: k.ds/a12434

## 2 How encoding work?

- Requirement: long urls should be converted to short urls with no collision
- Agenda of encoding
  - Idea
  - Explore possible techniques to implement the idea
  - Translate the technique into a System design Architecture

### 2.1 Idea

- Parameters
  - length of the key
  - range of allowed characters
  - constraint: the characters should be URL safe characters
- Available encoding schemas:
  - base 36: [a-z][0-9]
  - base 62: [a-z][A-Z][0-9]
  - base 64: [a-z][A-Z][0-9],/,+
- If we go with a length of 6 then we can generate upto 68B unique URLS!
- Note: Base 64 encoding will not make the url shorter but only encode it by converting all 3 bits into 4 bits and eventually making the url longer

### 2.2 Techniques

To shorten it, before encoding we must do some additional step:

- **MD5 hash fn:**
  - pros:
    - quick, and takes low computational power
    - it will helps us hash the URL into a fixed length 128bit string
    - use the initial 36 bits as our unique key
  - cons:
    - Algo is not full proof, uniqueness not guaranteed
    - Just using the first 36 bits may not be enough and might create collision
    - we will have to add additional complex logic to avoid the collision
- **URL independent encoded counter**:
  - The counter increases every time a new short link is created
  - we can use base 64 encoding to generate unique short keys
  - pros:
    - simple
    - very scalable
    - no DB query is required to create the short url
  - cons:
    - no fixed length
    - slowly increasing length
- **Key Range**:
  - Generate all 6 digit keys (68B entries) before hand and store in DB
  - All keys are unique and can be used for URLs
  - Cons:
    - We need to store 68B entries in DB which is resource intensive
- **Key Range for 5 digit**:
  - Generate all 5 digit keys (1B entries) before hand
  - store all the keys in DB
  - all keys will be unique hence no collision
  - 6th digit we will add on the fly
  - Better solution compared to rest ✅

### 2.3 System design Architecture

- Pre generate all 5 figure keys (1 billion)
- Store it in a DB
- System retrieve 5 figure keys from DB
- for the 6th key we will use 1 out of the 64 characters ([a-z][A-Z][0-9],/,+) from base 64
- No risk of collision

### 2.4 Summary

- Problem: Create unique short URL keys
- Challenge: Guarantee uniqueness, Guarantee fixed 6 length, shortness and support billions of entries
- Design choice:
  - base 64 encoding
  - pre-generated 5 figure keys (1 billion)
  - 1 figure (1 out of 64) appended on fly
- Architecture:
  - 2 DBs: to store key-ranges and final short keys or urls

## 3 Key-value store

- Relational DB
  - This is useful when we want to store data in multiple linked tables and thereby reducing duplicate data and also helps us write better flexible queries
- In our use case, we only need to have a key, pass to DB and retrieve the long URL
  - we will use the simplest NoSQL DB ie key value store
- Key Value Store
  - Storing data
    - Basic: key, value (string, number, booleans)
    - Advanced: key, value (dictionaries, arrays)
  - Retrieve data
    - It is recommended to name the keys systematically so that the keys allow for sorting
    - use cases:
      - start with a certain letter
      - are within a range of numbers
      - are < or > a certain number
      - are within a certain period of time if the key is timestamp
  - Retrieving or querying data by value is not possible
  - worse case: iterate over values
    - because it will be highly inefficient and we will have to query all the values
    - this is why key value stores don't come with SQL like query language
- How to interact?
  - get(key), delete(key), put(key, value)
- Non functional features:
  - Abstracts away all the implementation details of deployment and scale
  - Allows to focus on writing code with high business impact
  - Excellent performance and has almost instantaneous access
    - absence of complex queries
    - in-memory data structure
      - key value store will keep most frequently accessed data in an in-memory store and hence RAM can read it from there instead of reading from the disk.
      - Extreme use case:
        - Some key value store implementations take access speed to an extreme by storing data only in-memory
        - used to build cache layer on top of other databases
- Normal use case of key-value store:

  - Handle lots of small continuous reads or writes
  - examples:
    - to store key ranges for url shortener
    - store session details
    - ecommerce shopping carts
    - user preference for a user
    - personalized product recommendations

- Popular key value implementations:
  - https://github.com/google/leveldb
    - OS
  - https://github.com/facebook/rocksdb
    - OS
  - https://engineering.fb.com/2021/08/06/core-infra/zippydb/
    - abstraction on top of rocksdb
  - https://aws.amazon.com/dynamodb/
    - Most widely used key-value store
    - exclusively hosted on AWS
    - one of the first key-value store
    - not opensource

### 3.1 Key-value store summary

- Pros
  - Fast response time
  - Easy to store values
  - Highly scalable due to the no sql properties
- Cons
  - only look-up by key
  - no standard query language
  - no data normalization
