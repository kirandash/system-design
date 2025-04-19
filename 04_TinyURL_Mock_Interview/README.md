# TinyURL Mock Interview

## 1 Question

- https://tinyurl.com/
- Founder: https://en.wikipedia.org/wiki/Kevin_Gilbertson
- long urls are difficult to memorize and hard to spell
- some use cases:
  - url in printables ex: books
  - url in twitter post where max characters is limited
  - to track web analytics on 3rd party URLs ex: clicks
- ex: https://docs.google.com/spreadsheets/d/1U1gNCkeUv6gkpOGnhc8ycAfxWOWlp8IwEQfZcdVP-ms/edit?pli=1&gid=0#gid=0
  - short: https://tinyurl.com/729uv8ck
- alternatives: https://alternativeto.net/software/tinyurl/?sort=likes

## 2 Answer structure

1. Requirements Engg
2. Capacity Estimation
3. Data Modeling
4. API design
5. System Design
6. Design discussion

### 2.1 Requirements Engg

- Functional requirement
  - core
  - support
- Non functional requirement
- Expected scale of the system

- To answer the above questions better, lets do **system analysis**

  - Read heavy or write heavy?
    - Read heavy (less people create new url, but more people click on the tiny url)
      - Note: if we add some analytics then read write might be 1:1
  - Single server or distributed architecture?
    - Distributed system
  - What's important? Availability or Data consistency?
    - Availability

- To define the **Functional requirements** lets answer:

  - core features of tinyURL
    - The tinyurl service should be able to generate a unique and short link for any given URL
    - Users should be able to click on the short link and get redirected to the original link
  - support features (convincing yet simple to cover)
    - Link expiry: links should expire after the mentioned expiry time. Default expiry time = 2 years
    - Link History: list of all the short links ever created by a user
    - Analytics: we can track click events (Optional)

- Define **Non Functional Requirements**:

  - Highly Available
  - Scalable
    - redirection speed should not decrease if system has more users
  - Low latency
    - redirect should not take too much time

- Assume and also ask the interviewer about the **Expected scale of the system**:

  - How many DAU TinyURL has?
    - 100 million daily active users
  - Are there any events that lead to spikes in traffic?
    - No, highly unlikely to have spikes
  - How often do users interact with the app per day?
    - 1 click per user per day
  - What is the read/write ratio?
    - 10:1
  - How long are the URLs on avg that are supposed to be shortened
    - 200bytes
    - https://mothereff.in/byte-counter
  - What's the replication factor?
    - 3 standard or ignore data replication

- Final Summary:
  - Functional requirement
    - core
      - Unique and short links
      - Redirect
    - support
      - link history
      - link expiration
      - link analytics (Optional)
  - Non functional requirement
    - High Availability
    - Low latency
    - Scalable
  - Expected scale of the system
    - 100M DAU
    - No spikes
    - 5-10 click per user per day
    - 10:1 Read to write ratio
    - request size: 200bytes
    - ignore data replication

### 2.2 Capacity Estimation

- three core KPIs(Key Performance Indicator):

  - request (especially write)
  - bandwidth
  - storage

- Given:
  - 100M DAU
  - 1 click per user per day
  - 10:1 read/write ratio

#### 2.2.1 Calculation:

- Requests:

  - Read RPS: 10 clicks per day \* 100,000,000 DAU = 10^9 Read per day = 10^9 / 10^5 = 10^4 RPS
  - Write RPS: 10^4 RPS / 10 = 10^3 RPS

- bandwidth:

  - Read Bandwidth = 200bytes _ 10^4RPS = 2 _ 10^6 Bytes / sec = 2 \* 10^6 / 10^3 Kbps = 2000 KBps
  - Write bandwidth = 2000 KBps / 10 = 200 KBps

- storage: (for the next 5 years) ie 2 \* 10^8
  - Write bandwidth _ 2 _ 10 ^ 8 = 200 _ 2 _ 10^8 KBps = 4 _ 10^10KBps = 4 _ 10^10KBps / 10^6 GB = 4\*10^4 GB = 40000 GB = 40TB

### 2.3 Data Modeling

- Entities, Attributes, Relations
- Decide these based on core and support features

- For DB:
  - for users: go with relational DB. This is true in most apps for users since it may have complex relationships
    - Although not important in this use case but relational DB also has high data consistency
  - for links:
    - we want to be able to query url based on key
    - it should have low latency
    - we also don't need any complex relationships
    - hence we will go for key value store
  - for key range:
    - The most important: data consistency
    - we also want to filter keys by status which is user or not used ie filtering based on value
    - Hence we will use relational DB

### 2.5 API Design

- createLink(originalUrl)
- getLinks(userId, sorting)
- lookUpUrl(shortUrl)

#### 2.5.1 createLink

- param: originalUrl: string
- response:
  ```json
  {
    status: 200
    shortUrl: "https://tinyurl.com/729uv8ck"
  }
  ```

#### 2.5.2 getLinks

- params:
  - userId: string, uuid
  - sorting: string, ASC/DESC
- response:
  ```json
  {
    status: 200,
    data: {
      links: [
        {},
        ...
      ],
      order: "ASC"
    }
  }
  ```

#### 2.5.3 lookUpUrl

- param: shortUrl: string
- response:
  ```json
  {
    status: 200
    originalUrl: "https://docs.google.com/spreadsheets/d/1U1gNCkeUv6gkpOGnhc8ycAfxWOWlp8IwEQfZcdVP-ms/edit?pli=1&gid=0#gid=0"
  }
  ```

### 2.5 System Design

### 2.6 Design discussion

- Qn: What is your strategy for expiration? How are you going to expire? any cron job or anything else?
  - Daily cron job
    - Run a service periodically to the links db and filter all the links that are expired and delete them
      - and mark the corresponding keys in key range db as not used
  - Expire when you hit
  - Expire when you hit + Weekly cron job ✅
- Are you going to store the used keys? and what is the strategy for generation ?
  - Yes. Since we are expiring as well, We can reuse them. That would save a lot in terms of space.
- 68 billion keys will not take that much space (To calculate later). Do you really need to reprocess the key?
  - Yes! Because using 5 digits (1 B keys) still helps us with unique keys and saves a lot of space
- TODO: since there is no lock mechanism in key range db. how are you going to avoid data concurrency? 🚨
- TODO: "Why not nosql". Asking this because you considered AP approach. 🚨
- TODO: I think you can avoid key concurrency issue by pre-generating keys. your thoughts? 🚨

- **Justification Questions**:

  - What happens if the available key ranges are getting exhausted?
  - What if the short url never expire?
  - The key generator service in current design is a single point of failure. How will you improve this?

- **Extension questions**:
  - Basic link analytics:
    - User should be able to see how many times a short link has been clicked
  - Custom expiration date:
    - User should be able to enter a custom expiration day
  - Custom Link / alias
    - User should be able to create a custom short link for their URL
