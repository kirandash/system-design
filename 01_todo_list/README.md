## TODO List

## 1 Qn: How to design a todo list application?

## 2 Interview structure

1. Requirements Eng
2. Capacity Estimation
3. Data modeling
4. API design
5. system design
6. design discussion

### 2.1 Requirement engineering

#### Functional requirements

- **Core Features**:

  - User should be able to create/modify/complete/delete TODO items

- **Support Features**:
  - TODO items are private for every user
  - Each todo item will have: title, description and a due date
  - The application should show a list of todo items which are sorted by some metadata ex: due date, created date or title etc

#### Non-functional requirements

- Low latency
- High availability
- High scalability

#### Scale of the system

- 10,000 DAU (daily active users)
- Each user creates around 10 tasks per day
- Each todo item has a size of 50 Kb
- Read/write ratio is 10:1

### 2.2 Capacity Estimation

- three core KPIs(Key Performance Indicator):

  - request (especially write)
  - bandwidth
  - storage

- **Given**
  - 10k DAU
  - 10 items per day
  - Read write ratio: 10:1
  - Each todo item has a size of 50Kb

#### 2.2.1 Calculate Requests

- Calculation:

  - writes:

    - 20k DAU \* 10 = 200K requests per day
    - 200K / 10^5 = 2 RPS (Requests per second)
      - 86400 seconds per day ~ 100000 = 10^5

  - reads:

    - 20 \* 2 RPS = 40 RPS

  - total requests = write + read = 2 + 40 = 42 RPS

#### 2.2.2 Calculate bandwidth

- Calculation:
  - total requests _ resource per todo item = 42 RPS _ 50Kb = 2100 Kbps

#### 2.2.3 Calculate storage

- storage for next 5 years
  - 50Kb _ 2 write RPS _ 5 years
  - 50Kb _ 2 _ 5 _ 365 _ 24 _ 60 _ 60
  - 50Kb _ 2 _ 2 \* 10^8
    - 5 years of seconds: 157680000 ~ 200000000 = 2 \* 10^8
  - 2\* 10^10Kb/10^6GB
    - 20000 GB

### 2.3 Data modeling

- Entities, Attributes & Relations
  - Entities:
    - user, todo item
  - attributes:
    - text, description, due date
  - relation:
    - one to many relation

### 2.4 API design

- createTodoItem(title?, description?)
- updateTodoItem(itemId, description?, dueDate?)
- deleteTodoItem(itemId)
- getTodoItems(userId, sortedBy, pageIndex)

#### 2.4.1 getTodoItems

- parameters:
  - userId: string: uuid
  - sortedBy: string: ASC/DESC
  - pageIndex: number: page number
- Response:

  ```ts
  {
    pageIndex: 3,
    nextPageIndex: 4,
    prevPageIndex: 2,
    items: [
      {
        "itemId": '2334-3344-2221-2345',
        "title": 'task 1',
        "description": 'This is task desc'
      },
      ...
    ],

  }
  ```

### 2.5 System design

#### 2.5.1 Core System components

- **Service**:

  - Backbone of the architecture diagram
  - it serves a specific purpose
    - ex: user service covers all tasks related to user
    - ex: video service, audio service etc

- **Relational or SQL Database**

  - Most widely used type of DB
  - Easy to understand
  - Good fit for most of the use cases

- **Client App**

  - Allows user to interact with the system
  - Ex: mobile, desktop or a web app
  - Representation or UI layer

- **Scaling services**

  - Multiple instances

    - increases the load any system can handle
    - diagram has two layers to represent multiple instances

  - load balancer
    - distribute the network evenly
    - the diagram looks like a router since it helps routing the requests

- **Scaling Relational or SQL Database**:
  - Federation (Functional Partitioning)
    - simple but effective
    - the idea is to reduce the load on individual DBs by splitting up the data by its function
    - this will help us distribute load on multiple databases instead of one

#### 2.5.2 System Design Diagram

- The arrows should point in user flow direction and not data flow
  - data flow will be in opposite direction

### 2.6 Design Discussion

- Design decisions
- Experience
- Tradeoffs
- CS Fundamentals etc
