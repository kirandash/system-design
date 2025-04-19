# Fundamentals

## 3 Requirements Engineering

### 3.1 Functional Requirements

- What does the system do?

  - Which problem does the system solve?
  - Which features are needed to solve the problem?

#### 3.1.1 Which problem does the system solve?

- focus on the general problem user wants to solve
- Purpose of the app also helps us define functional and nonfunctional requirement
- Ex:
  - Purpose of TODO: The purpose was to help users become more productive with the daily tasks
  - Features: Simple list of todo items
  - Non Functional requirement: Low latency, High availability, High scalability
- Ex:
  - Purpose of Banking app: The purpose is to perform transactions securely
  - Non Functional requirement: High Latency

#### 3.1.2 Which features are needed to solve the problem?

- Core Features
- Support Features

- Core Features

  - Bare minimum feature that is needed to solve users problem
  - Ex: TODO: Create, update, delete, get list

- Support Features

  - Features that help user to solve problem in a better or more easier way
  - Ex: Priority tags for todo item, due date

- We should ask interviewer about the non functional requirement they care about.
  - based on that we should decide the system tradeoffs, user experience, the system attributes

### 3.2 System Analysis

#### 3.2.1 Read heavy or write heavy?

ex: streaming like netflix is read heavy
ex: analytics platform like google analytics or posthog is write heavy

- **Impact on architecture**:
  - Priority of System Attribute:
    - if it is read heavy then it is ok to fail during write than to read
  - Design requirement:
    - Scale DBs
    - Cache efficiency
    - Maintain multiple instances of services
      - ex: we will maintain multiple instances for read but not for write if the system is read heavy
- Note: Most apps are read heavy

#### 3.2.2 Single server or distributed architecture?

- Monolithic/Centralized system: This system will keep all our components on a single physical server
- Distributed/No-share system: The components are on many independent machines
- How to pick an architecture?
  - Depends on the interview qn requirement. For large scale systems ex: Uber, google, amazon, netflix etc choose distribute arch
  - for small scale systems ex: web crawler etc: use single server or monolithic system

#### 3.2.3 What's important? Availability or Data consistency?

- Network leads to more complexity
- Impact of a network interruption

  - Option A (More available system)
    - State of data might become inconsistent
    - Inconsistencies may be solved later
    - Multiple write nodes will be there to ensure availability
  - Option B (More Data consistent system)
    - Some nodes are not accessible or not available
    - Reduced availability
    - State of data becomes consistent

- Note: It's very important to understand which is more favourable for our app.

  - Ex: online document editing like google docs with live collaboration
    - data consistency is more important than availability
  - Ex: For netflix feed with all tv shows or movies, availability is more important than data consistency
    - because it's fine if few items are missing
  - Ex: for facebook home feed, availability is more important than data consistency

##### 3.2.3.1 Availability

- Availability describes how long the system is up and running per year

  - Five nines: 99.999% per year
  - Down time: Ex: OpenAI was down for 4.34 minutes a year in 2024

- To make a system more available focus on:
  - reliability
  - redundancy
  - fault tolerance for all components

##### 3.2.3.2 Data consistency

- Data is consistent if it appears to be same in all the nodes at the same time
- Data consistency = Linearlizability

#### 3.2.4 Scalability

- Scalability describes the ability of a system to handle fast growing user base and occasional/temporary load spikes
- Most common in all interview qns

- There are two types of scalability
  - Vertical scalability
  - Horizontal

##### 3.2.4.1 Vertical scalability

- straightforward approach to scale
- we add additional resources on the same system

- Pros:

  - Faster to scale
  - Faster for the internal component to do inter-process communication
  - It helps with data consistency
    - since it has a single database in the same system

- Cons:
  - Single point of failure
  - Economical limit to scalability
    - cheaper till certain extent but when user grows significantly the additional cost will be very expensive
    - cost scales linearly till a point and then becomes exponential

##### 3.2.4.2 Horizontal scalability

- No single server but a cluster of regular servers
- Scale by adding more servers to the cluster

- Pros:

  - Higher availability
  - Cost Scales linearly
    - price will always scale linearly

- Cons:
  - Data inconsistency
  - complex architecture

#### 3.2.5 Latency

- Latency is the amount of time in ms the system takes for a msg to be delivered
- Latency experienced by User = Network + System Latency
- Impact of latency on design choices 🔥:
  - Database and data model will significantly impact the DB query performance. hence will impact latency
  - Caching can also be leveraged to achieve lower latency

#### 3.2.6 Compatibility

- Compatibility describes the ability of a system to operate seamlessly with other SW and HW.

#### 3.2.7 Security

- Very vast and complex
- Only add it to your list of NFR if you are confident
- Maybe, add standard security measures in your design
  - API keys with rate limiting
  - TLS encrypted network traffic etc...

#### 3.2.8 Absolute expected scale

##### 3.2.8.1 DAU

- key metric to describe their scale
- MAU is useful too

##### 3.2.8.2 Peak Active Users

- Triggered by some events
  - ex: release of movie on friday will cause a peak compared to other days
- will cause an x times avg DAU
- A good System design should handle peaks

##### 3.2.8.3 Interactions per user

- Figure out about the kind of operation (read or write)
- use read-write ratio for a better understanding
  - ex: 10:1 read, write ratio
- Focus for read-write ratio only for the core feature

- Also we should ask for number of interactions per day
  ex: 10 reads per day

##### 3.2.8.5 Request size

- size of the payload of the requests
  - ex: the size of the JSON payload object ie 50Kb
  - ex: size of payload for image for a FB post might be higher ex: MBs

##### 3.2.8.6 Replication factor

- We can replicate data in Databases across different servers to increase scalability
- Repl factor is how many times the same time is duplicated
  - the standard factor is 3 which means the data will be replicated thrice
- Cons
  - Creates redundancy
  - Takes up more storage capacity

## Summary - Requirements Engg

- FR
  - What the system is supposed to do?
    - Define the core feature and support features?
    - Which problem does the system solve for the users?
    - Which features are essential to solve the problem?
- NFR
  - How the system is supposed to work?
    - Read or write heavy?
    - Is it a small or a large scale system?
    - Availability or data consistency?
    - Absolute scale of the system?
