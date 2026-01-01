# Load-Testing-Web-Server

## Purpose

The **Load Testing Web Server (LTS)** is a high-performance, correct, and observable HTTP 1.1 server implemented in Java. The goal of this project is to build a server that is resistant to common failure modes and demonstrate its scalability using a high-concurrency load testing client. This project explores the transition from basic socket handling to advanced features like **Persistent Connections (Keep-Alive)** and **Virtual Threads** for massive concurrency.

---

## 🏗 System Architecture

The project is designed to be implemented in three distinct phases, evolving from a simple file server to a high-concurrency benchmarking tool.

### Phase 1: Correctness Baseline

* **Static File Serving:** Serves assets from the `public/` directory.
* **Security:** Implements path normalization and prevents directory traversal attacks (403 Forbidden).
* **HTTP Standards:** Proper status lines (200, 404, 405), Content-Type detection via file extensions, and mandatory `Content-Length` headers.
* **Logging:** Real-time monitoring of method, path, status, and latency.

### Phase 2: Persistent Connections

* **Keep-Alive:** Implements the `Connection: keep-alive` header to allow multiple requests per TCP connection, reducing handshake overhead.
* **Idle Timeouts:** Prevents resource leakage by automatically closing inactive sockets after a configured period.
* **State Management:** Manages a per-connection read loop to handle sequential requests accurately.

### Phase 3: Scalable Concurrency & Echo

* **Echo Endpoint:** Implementation of `GET /echo/<size>` for synthetic workload generation. It generates deterministic bytes and provides an `X-Payload-Hash` header (SHA-256) for integrity verification.
* **Virtual Threads:** Utilizing Java's Project Loom (Virtual Threads) to handle thousands of concurrent connections with a "one-thread-per-connection" model without the overhead of platform threads.

---

## 🛠 Build and Run

### Prerequisites

* Java 21 or higher (required for Virtual Thread support).
* The provided `distrib.zip` files: `lts.java`, `ltc.java`, and the `public/` folder.

### Server Usage

```bash
java lts.java [options] [port]

```

**Options:**

* `-t`: Enable **Virtual Thread** handling for connections.
* `-k [seconds]`: Enable **HTTP Keep-Alive** with a specific idle timeout.
* `-q`: **Quiet mode** (disables logging for cleaner load test results).
* `port`: Defaults to `8080`.

### Client Usage (Load Tester)

The `ltc.java` client drives multiple connections to stress the server.

```bash
java ltc.java [options]

```

---

## 🧪 Testing Workflow

### 1. Manual Smoke Test

Open your browser to `http://localhost:8080` and navigate the built-in tutorials. Verify that static assets (CSS/JS) load correctly.

### 2. Protocol Verification

Use `curl` to verify persistent connections:

```bash
# Verify Keep-Alive
curl --http1.1 -v --header "Connection: keep-alive" http://localhost:8080/index.html --next http://localhost:8080/index.html

```

### 3. Load Testing

Target the echo endpoint to stress the server’s bandwidth and concurrency:

```bash
# Test 100 concurrent connections against a 1MB echo
java ltc.java -c 100 -s 1048576

```

---

## 📋 HTTP Details & Constraints

| Feature | Requirement |
| --- | --- |
| **Status Codes** | 200, 400 (Bad Request), 403 (Forbidden), 404 (Not Found), 405 (Method Not Allowed), 500 (Internal Error). |
| **Headers** | Case-insensitive names, terminated by CRLF, single values only. |
| **Security** | Traversal protection, maximum echo size bounds, and header size limits. |
| **Stability** | No indefinite reads; all I/O must respect the configured timeouts. |

---

## 📂 Project Structure

* `lts.java`: The server implementation (Main entry point).
* `ltc.java`: The load testing client.
* `public/`: A directory containing tutorial pages, static assets, and test targets. **Read `public/index.html` first!**

---

## 🎓 Learning Goals

* Understanding the **wire format** of HTTP 1.1.
* Mastering **TCP socket programming** in Java.
* Implementing **concurrent programming** via Virtual Threads.
* Designing for **observability** (logging) and **robustness** (error handling).
