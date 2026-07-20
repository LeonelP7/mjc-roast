# WEB

<details>
<summary>What is URI?</summary>

A URI is a string that identifies a resource - on the web or elsewhere. It's the umbrella category; URLs and URNs are both types of URI. In REST specifically, every resource (any piece of content or entity the API exposes) must be identified by a unique URI, typically shaped like:
`<protocol>://<service-name>/<ResourceType>/<ResourceID>`
e.g. `https://api.example.com/users/42`

* URN (Uniform Resource Name): identifies what something is - a unique, permanent name - without saying where to find it. Think of it like an ISBN for a book; it identifies that exact book forever, regardless of which store or library has a copy. A URN can be resolved into an actual location (a URL) by a "resolver", at which point the resource can be fetched.
e.g. `urn:isbn:0451450523`

* URL (Uniform Resource Locator): identifies where something is and how to retrieve it. It always includes a protocol, a host, and a path - and optionally query parameters.
e.g. `https://amazon.com/index.html?ref=home`

</details>

<details>
<summary>What is HTTP?</summary>

HTTP stands for Hyper Text Transfer Protocol.

It's the primary communication protocol used for interaction between clients and servers. It functions through a request-response pattern.

**Core components:**

* Request:
    * Method/Verb: Indicates the specific action to be performed
    * URI: Uniquely identifies the resource on the server
    * HTTP Version: Specifies the protocol version
    * Request headers: Metadata like client type, expected format, auth token
    * Request body: The actual content or payload sent to the server

* Response:
    * Status code: Represents the outcome of the request
    * Response headers: Metadata such as content type, length, and server info
    * Response body: The actual resource or message returned from the server

</details>

<details>
<summary>What is the HTTP client and HTTP server?</summary>

An HTTP client is a program that establishes a connection to a server to send one or more HTTP request messages.

An HTTP server is a program that accepts connections and serves HTTP requests by sending back HTTP response messages.

</details>

<details>
<summary>What are HTTP methods (verbs)?</summary>

HTTP methods tell the server what action to perform on a resource. They're a core part of REST's "Uniform Interface" constraint - meaning every client talks to every resource using the same standard set of verbs, instead of custom action names.

The main ones:

- GET: Fetch a resource. Read-only.
- POST: Create a new resource.
- PUT: Replace/update an existing resource entirely.
- PATCH: Partially update a resource.
- DELETE: Remove a resource.
- OPTIONS: Retrieves a list of supported operations for a resource on the server.
- HEAD: Identical to GET, but the server must not return a message body. Used to check metadata (existence, size, last-modified) without downloading the actual content.

**Properties:**

* Safe = doesn't change server state -> GET, HEAD, OPTIONS
* Idempotent = calling it N times has the same effect as calling it once -> GET, HEAD, OPTIONS, PUT, DELETE
* POST and PATCH are neither safe nor idempotent

</details>

<details>
<summary>Differentiate between PUT, POST, and PATCH</summary>

All three can be used to "update" data, but they mean different things:

| | PUT | POST | PATCH |
|---|---|---|---|
| Purpose | Replace a resource entirely (or create it at a known URI if it doesn't exist) | Create a new resource | Partially update a resource |
| URI targets | A specific resource: `PUT /users/42` | A collection: `POST /users` | A specific resource: `PATCH /users/42` |
| Idempotent | Yes | No | No |
| Payload | Full representation of the resource | New resource data | Only the fields that change |

Quick rule of thumb: if you're sending the *whole* object, it's PUT. If you're sending only *some fields*, it's PATCH. If you don't know the resource's final URI yet (the server assigns it), it's POST.

</details>

<details>
<summary>What are idempotent and safe HTTP methods?</summary>

* **Safe** methods don't change server state at all - GET, HEAD, OPTIONS. They can be cached freely.
* **Idempotent** methods produce the same server state no matter how many times you call them - GET, HEAD, OPTIONS, PUT, DELETE.

Note: idempotent doesn't mean "returns the same response every time" - it means the *state of the resource on the server* ends up the same. Example: calling `DELETE /users/42` twice - the first call returns `204 No Content`, the second returns `404 Not Found` (different responses), but the server state is identical in both cases (the user is gone either way). That's still idempotent.

POST is not idempotent, because each call is intended to create a *new* resource - calling it N times creates N resources.

</details>

<details>
<summary>What is an HTTP status code?</summary>

A 3-digit number in the response that tells the client, at a glance, what kind of outcome occurred - success, client error, server error, etc. - without needing to parse the response body. Good API design uses the status code as the primary signal of *what* happened, and the response body only for supporting *details* (why it happened).

Grouped by first digit:

* 1xx - Informational
* 2xx - Success
* 3xx - Redirection
* 4xx - Client error
* 5xx - Server error

</details>

<details>
<summary>Common status codes</summary>

- 200 OK - success (GET, PUT, PATCH)
- 201 Created - a new resource was created (POST)
- 204 No Content - success, nothing to return (DELETE)
- 301 Moved Permanently - resource has a new permanent URL
- 302 Found - resource temporarily moved
- 304 Not Modified - used with conditional GET to save bandwidth
- 400 Bad Request - validation errors or malformed input
- 401 Unauthorized - client is not authenticated (missing/invalid credentials)
- 403 Forbidden - client is authenticated, but not allowed to do this
- 404 Not Found - resource doesn't exist
- 500 Internal Server Error - unhandled exception on the server
- 502 Bad Gateway - a server acting as a proxy got an invalid response from an upstream server

</details>

<details>
<summary>What's the difference between 401 Unauthorized and 403 Forbidden?</summary>

* **401** = "Who are you?" - the request has no valid authentication (missing token, expired/invalid credentials).
* **403** = "I know who you are, and the answer is no." - the client is authenticated, but lacks permission to access this resource.

Note the naming is a bit misleading: 401 is called "Unauthorized" but it's really about *authentication*, not authorization.

</details>

<details>
<summary>What is Payload? How does it differ from parameters?</summary>

The **payload** is the data in the *body* of an HTTP request or response - typically JSON. Used with POST, PUT, and PATCH to transfer the state of a resource.

**Parameters** (query params) are part of the URL itself, e.g. `?id=101`.

`Payload` = body. `Parameters` = URL. That's the core distinction.

</details>

<details>
<summary>What is an API?</summary>

**General concept:** An API (Application Programming Interface) is a defined set of rules and interfaces that lets two pieces of software communicate, without either one needing to know the other's internal implementation. It specifies *what* you can ask for and *how* to ask for it - not *how* the other side does the work internally. (e.g. a Java library's public methods are an API.)

**In the web context:** A web API applies the same idea to communication between applications over a network, usually over HTTP. Instead of calling a method in the same program, the client sends an HTTP request to an endpoint, and the server responds.

* The "rules" are HTTP methods, URIs, status codes, and data formats (JSON/XML).
* It enables **decoupling**: frontend and backend can evolve independently as long as they honor the contract.
* It acts as a **security boundary**: the client never touches internals directly, only what the API chooses to expose.

</details>

<details>
<summary>What is a Stateless Server?</summary>

In modern web development (like REST), servers are usually stateless, meaning they don't "remember" the client between requests. Every single request must contain all the information necessary (like an auth token) for the server to understand and authorize it. Session-like state, if needed, lives on the client or in a shared store (e.g. Redis) - not in server memory - so any server instance can handle any request.

</details>

<details>
<summary>What are REST and REST API?</summary>

**REST** (REpresentational State Transfer) is an architectural style - a set of constraints for building distributed systems, not a protocol or a technology. Roy Fielding introduced it in his 2000 dissertation. Following its constraints (statelessness, uniform interface, cacheability, etc.) tends to produce systems that are scalable, fast, and easy to integrate with.

**REST API** (or RESTful API) is the *implementation* - a concrete API that follows REST's constraints, typically built over HTTP, letting clients interact with resources exposed by a server.

In a REST API, everything is a **Resource** (a User, an Order, a Product...), each identified by a unique URI. The client sends an HTTP verb (GET, POST...) to a URI; the server responds with a *representation* of the resource's state (usually JSON), not the raw database row.

One-liner: REST is the style/rulebook; a REST API is a real API built following it.

</details>

<details>
<summary>What are the constraints of the RESTful architecture?</summary>

Rules that, when followed, make a service RESTful rather than just "a web service that uses HTTP."

**The 6 constraints:**

1. **Client-Server** - Separation of concerns: client handles UI, server handles data/logic. Each can evolve independently.
2. **Stateless** - The server stores no client context between requests. Every request carries everything needed to process it.
3. **Cacheable** - Responses must indicate whether they're cacheable, to improve performance.
4. **Uniform Interface** - A consistent, standardized way of addressing and manipulating resources (HTTP methods + URIs).
5. **Layered System** - The client can't tell (and shouldn't need to) whether it's talking directly to the server or through intermediaries (load balancer, gateway, cache).
6. **Code on Demand** *(optional)* - The server can temporarily extend client functionality by sending executable code (e.g. JavaScript). This is the only optional constraint.

</details>

<details>
<summary>What is a REST Resource?</summary>

Every piece of content or entity in REST architecture is considered a resource - analogous to an object in OOP. Resources can be text files, HTML pages, images, or dynamic data. Every resource is identified globally by a URI.

The REST server provides access to resources; the REST client consumes (accesses/modifies) them.

</details>

<details>
<summary>What are best practices for naming REST URIs?</summary>

- Use **plural nouns** for resources: `/users`, not `/user`.
- Use hyphens or underscores for multi-word resources, not spaces: `/authorized-users`.
- Keep URIs lowercase.
- **Don't put verbs in the URI** - the HTTP method already expresses the action. Use `GET /books/{id}`, not `GET /getBook`.
- Use `/` to express hierarchy between resources and sub-resources: `/books/{id}/authors`.

</details>

<details>
<summary>Can you tell the disadvantages of RESTful web services?</summary>

The most significant is the over-fetching/under-fetching problem, where clients often receive more data than they need or have to make multiple round-trips to get a complete dataset. Additionally, because REST is stateless, it places a heavier burden on the client to manage session state. Finally, for real-time communication, REST's request-response nature is less efficient than modern alternatives like WebSockets or gRPC.

</details>

<details>
<summary>What is a Servlet?</summary>

A Java class that implements the `Servlet` interface, designed to handle requests and generate responses within a web container, acting as the entry point where an incomming HTTP request lands inside your Java code, in a form the container already parsed for you. lives inside a web container (like Tomcat). Its job:

1. Receive an HTTP request.
2. Process some logic (e.g. read from a database).
3. Generate an HTTP response (HTML or JSON).

In old-style Java web apps, you had to write a separate servlet per page/action (`LoginServlet`, `RegisterServlet`...), which became hard to manage - this is the problem the DispatcherServlet solves.

When working with `@RestController`

The flow is:
Web Container (Tomcat)
   ↓ receives raw HTTP request
DispatcherServlet (the ONE Servlet — implements HttpServlet)
   ↓ looks at the URI + method, asks "which controller handles this?"
   ↓ (uses a HandlerMapping internally to figure this out)
Your @RestController method
   ↓ runs your business logic, returns data
DispatcherServlet
   ↓ converts your return value into JSON (via HttpMessageConverter)
   ↓ writes it to the response
Web Container
   ↓ sends the HTTP response back to the client

</details>

<details>
<summary>What is a Web Container?</summary>

A web container is the runtime enviroment that manages servlets - it's the piece of software that actually runs your web application inside a server. Examples: Tomcat, Jetty, Undertow

Its responsabilities:

* Lifecycle management - creates, initializes and destroys servlets instances (calls `init()`, `service()`, `destroy()` on them)

* Request handling - listens on a port, accepts incoming HTTP connections, parses raw HTTP requests into Java objects (`HttpServletRequest`), and routes each one to the right servlet.

* Response building - takes what your servlet produces and turns it into a valid HTTP response (`HttpServletResponse`) sent back over the socket.

* Concurrency - manages thread pool so multiple requests can be handled at once without you writting any threading code.

* Security & session management - handles things like sessions ID, cookies and basic auth enforcement, if configured.

</details>

<details>
<summary>What is the DispatcherServlet?</summary>

The single entry point for all incoming requests in Spring MVC. It routes each request to the appropriate controller (based on `@RequestMapping`-style annotations), manages the request lifecycle, and ensures the correct response format is sent back.

In Spring Boot, auto-configuration detects you're building a web app and registers this bean for you automatically - you don't configure it by hand.

</details>

<details>
<summary>What's the difference between @Controller and @RestController?</summary>

`@RestController` is `@Controller` with `@ResponseBody` applied to every method by default - so instead of resolving the return value to a view template, the DispatcherServlet writes it directly to the response body as data, usually JSON.

| @Controller | @RestController |
|---|---|
| Used in traditional Spring MVC, where a view (HTML/JSP) needs to be rendered | Used for RESTful services that return data directly |
| Needs `@ResponseBody` on methods/class to write raw data to the response | `@ResponseBody` is implied - it's `@Controller` + `@ResponseBody` combined |
| Returns a view name by default | Writes the return value directly to the response body (typically JSON) |

</details>

<details>
<summary>What's the relationship between @RequestMapping and @GetMapping / @PostMapping / etc?</summary>

`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, and `@PatchMapping` are specialized, composed versions of `@RequestMapping` - each is `@RequestMapping` pre-set to a specific HTTP method.

`@RequestMapping` can be used at the class level (to define a base path for the whole controller) or the method level (for any HTTP method, specified via an attribute). The specialized annotations are almost always used at the method level, since they make the code more expressive and reduce boilerplate.

</details>

<details>
<summary>What does @PathVariable do?</summary>

Binds a value from the URI path to a method parameter. Used to extract dynamic segments of a URI, e.g.:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }
```

Here `{id}` in the path maps directly to the `id` parameter.

</details>