# 🚀 Distributed Cache System

A **distributed caching system** built using **C++, Redis, Drogon, Nginx, PostgreSQL, Docker, and multithreading** to provide faster data retrieval and reduce repeated database access.

The system uses **multiple Redis cache nodes** and **consistent hashing** to distribute cached data efficiently. It also implements **LRU eviction** to manage limited cache space and supports concurrent request processing using multithreading.

---

## 📌 About Project

In a normal application, every request for data may directly reach the database.

For frequently requested data, this can result in:

* Repeated database queries
* Higher database load
* Increased response time

This project adds a **distributed cache layer** between the application and the database.

Frequently accessed data is stored in **Redis**, which allows subsequent requests to retrieve the data from memory instead of repeatedly querying PostgreSQL.

The main idea is simple:

```text
Client
   ↓
Nginx
   ↓
Drogon Backend
   ↓
Redis Cache
   ↓
PostgreSQL (if cache miss)
```

---

## 🏗️ System Architecture

```text
                         Client
                           │
                           ▼
                      ┌─────────┐
                      │  Nginx  │
                      └────┬────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Drogon API  │
                    │    C++      │
                    └──────┬──────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Consistent Hashing│
                  └────────┬─────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌─────────┐  ┌─────────┐  ┌─────────┐
         │ Redis 1 │  │ Redis 2 │  │ Redis 3 │
         └─────────┘  └─────────┘  └─────────┘
                           │
                      Cache Miss
                           │
                           ▼
                    ┌────────────┐
                    │ PostgreSQL │
                    └────────────┘
```

---

## ⚙️ How the System Works

When a client sends a request, it first reaches **Nginx**, which acts as the entry point for the system.

The request is then handled by the **Drogon-based C++ backend**.

The backend determines which Redis node should contain the requested key using **consistent hashing**.

The selected Redis node is then checked for the requested data.

### ✅ Cache Hit

If the requested data is already present:

```text
Request
   ↓
Redis
   ↓
Data Found
   ↓
Return Response
```

The database does not need to be accessed.

This is the main reason caching can significantly reduce data retrieval latency for frequently requested information.

### ❌ Cache Miss

If the requested key is not present in Redis:

```text
Request
   ↓
Redis
   ↓
Cache Miss
   ↓
PostgreSQL
   ↓
Fetch Data
   ↓
Store in Redis
   ↓
Return Response
```

The data is retrieved from **PostgreSQL**, returned to the client, and stored in Redis so that future requests can potentially be served directly from the cache.

---

## 🔄 Consistent Hashing

One of the important parts of the system is **consistent hashing**.

Since there are multiple Redis nodes, the system needs a way to decide **which node should store a particular key**.

Instead of randomly selecting a Redis server, a hash function is used to map keys to cache nodes.

For example:

```text
                Hash Ring

             ┌─────────────┐
          ┌──┘             └──┐
       Redis 1              Redis 2
          │                     │
          │    Key A            │
          │    Key B            │
          │                     │
          └────── Redis 3 ──────┘
```

The major advantage is that when a Redis node is **added or removed**, the system does not need to redistribute every existing key.

Only the affected portion of the keys needs to be remapped.

This makes consistent hashing useful for maintaining a more stable distribution of cached data as the number of cache nodes changes.

---

## 🗑️ LRU Cache Eviction

Cache memory is limited, so the system needs a strategy for deciding which data should be removed when the cache becomes full.

The project uses **LRU — Least Recently Used** eviction.

The basic idea is:

> **Keep recently accessed data and remove data that has not been used for the longest time.**

For example:

```text
Cache:

A → B → C → D

If D has not been accessed for the longest time,
D becomes the first candidate for removal.
```

This helps keep frequently requested data available while making room for new entries.

---

## 🧵 Concurrent Request Processing

The backend uses **multithreading** to handle multiple requests and cache operations concurrently.

Instead of processing every request one after another:

```text
Request 1 → Finish
Request 2 → Finish
Request 3 → Finish
```

the system can work with multiple requests concurrently:

```text
          ┌── Request 1
          │
Backend ──┼── Request 2
          │
          └── Request 3
```

This is particularly useful for a caching system where multiple clients may request data at the same time.

---

## 🌐 Role of Nginx

**Nginx** is used as the front-facing layer of the system.

It receives incoming requests and helps route them toward the available backend/cache services.

A simplified flow is:

```text
Client
   ↓
Nginx
   ↓
Backend / Cache Services
```

Using Nginx keeps the request-routing layer separate from the core C++ application logic.

---

## 🐳 Dockerized Environment

The different components of the system are run using **Docker**.

This makes it easier to run multiple services independently and maintain a consistent development environment.

For example, the architecture can contain separate containers for:

```text
Drogon Backend
      │
      ├── Redis Node 1
      ├── Redis Node 2
      ├── Redis Node 3
      │
      ├── PostgreSQL
      │
      └── Nginx
```

Docker also makes it easier to reproduce the same environment without manually installing every dependency.

---

## 🛠️ Technology Stack

### **C++**

Used for the backend implementation and concurrent request processing.

### **Drogon**

A high-performance **C++ web framework** used to build the backend API and handle HTTP requests.

### **Redis**

Used as the **in-memory cache** for storing frequently accessed data.

### **PostgreSQL**

Used as the persistent database and acts as the source of data when a cache miss occurs.

### **Nginx**

Used for handling incoming requests and routing traffic.

### **Docker**

Used to containerize the services and run multiple components in an isolated environment.

---

## 📈 Performance

The caching approach helped achieve approximately **40% lower data retrieval latency for frequently accessed data** by reducing repeated database access.

The main performance improvement comes from serving frequently requested data from **Redis instead of repeatedly querying PostgreSQL**.

```text
Without Cache:

Client → Backend → PostgreSQL → Response


With Cache:

Client → Backend → Redis → Response
                     ↑
                Much faster path
```

---

## 🔑 Key Concepts

This project covers several important **backend and distributed-system concepts**:

* **Distributed caching**
* **Redis**
* **Consistent hashing**
* **LRU eviction**
* **Cache hit / cache miss**
* **Multithreading**
* **Concurrent request processing**
* **Nginx request routing**
* **Database fallback**
* **Docker containerization**
* **C++ backend development**

---

## 💡 What I Learned

This project helped me understand how a caching layer can be designed to reduce database dependency and improve response time.

I also gained practical understanding of how **multiple cache nodes can work together**, how **consistent hashing** helps distribute keys, how **LRU** manages limited cache space, and how **multithreading** can be used to handle concurrent requests.

The project also gave me hands-on exposure to combining **C++ backend development, Redis, PostgreSQL, Nginx, and Docker** into a single distributed-system architecture.

---

## 🚀 Project Highlights

* ⚡ Reduced retrieval latency by approximately **40%** for frequently accessed data
* 🔄 Implemented **consistent hashing** across multiple Redis nodes
* 🗑️ Added **LRU-based cache eviction**
* 🧵 Supported **concurrent request processing using multithreading**
* 🌐 Used **Nginx** for request routing
* 🐳 Containerized services using **Docker**
* 💾 Used **PostgreSQL** as the persistent data source
* ⚙️ Built the backend using **C++ and Drogon**

---

## 🎯 Project Goal

The goal of this project is to demonstrate how a **distributed caching layer** can be designed to improve application performance by reducing unnecessary database operations and efficiently distributing cached data across multiple nodes.

**Core idea:**

> **Cache frequently used data → reduce database requests → improve retrieval speed → handle multiple requests efficiently.**
> 
**Developed with ♥️ by :** [Tulsi Singh](https://github.com/tulsi-singh4)
