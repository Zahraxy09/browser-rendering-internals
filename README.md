# browser-rendering-internals
A practical guide to browser internals, covering DNS, HTTP, DOM, CSSOM, JavaScript engines, rendering, layout, painting, compositing, browser security, and web performance.
How Web Browsers Work: From URL to Pixels
Introduction

Opening a website feels simple.

You type:

https://example.com

into a browser, press Enter, and a page appears.

But behind that simple action, the browser performs a surprisingly complex sequence of operations.

It may need to:

Parse the URL
Resolve the domain name
Establish a network connection
Negotiate TLS encryption
Send an HTTP request
Download HTML
Parse CSS
Execute JavaScript
Build internal data structures
Calculate page layout
Paint pixels
Composite layers
Display the final result on the screen

A simplified journey looks like:

URL
 ↓
DNS
 ↓
Network Connection
 ↓
HTTP Response
 ↓
HTML + CSS + JavaScript
 ↓
Browser Rendering Engine
 ↓
Pixels

Understanding how browsers work helps developers build faster websites, debug performance problems, understand JavaScript behavior, and make better architectural decisions.

1. What Is a Web Browser?

A web browser is an application that retrieves, interprets, and displays content from the web.

Popular browsers include:

Chrome
Firefox
Safari
Edge

A browser contains several major components:

Browser
├── User Interface
├── Browser Engine
├── Rendering Engine
├── JavaScript Engine
├── Networking Layer
├── Storage Layer
└── Graphics System

Each component performs a different role.

2. The Browser Architecture

A simplified architecture might look like this:

                 User Interface
                       │
                       ↓
                 Browser Engine
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
Rendering Engine   Networking     Data Storage
       │
       ├── HTML Parser
       ├── CSS Parser
       ├── Layout
       └── Paint
       │
       ↓
JavaScript Engine
       │
       ↓
Graphics / GPU

Modern browsers are even more complex, but this gives us a useful mental model.

3. Step One: The User Enters a URL

Suppose the user enters:

https://example.com/products?id=42

A URL can contain several components:

https://example.com:443/products?id=42
│        │         │      │       │
│        │         │      │       └── Query
│        │         │      └────────── Path
│        │         └───────────────── Port
│        └────────────────────────── Host
└─────────────────────────────────── Scheme

The browser first determines how the address should be interpreted.

4. URL Parsing

The browser separates the URL into parts.

For example:

Scheme:   https
Host:     example.com
Port:     443
Path:     /products
Query:    id=42

The scheme tells the browser which protocol should be used.

For HTTPS:

Browser
   ↓
HTTPS
   ↓
TLS
   ↓
Network
5. DNS Resolution

Before connecting to a server, the browser usually needs an IP address.

The browser cannot directly connect to:

example.com

It needs something like:

93.184.216.34

DNS provides that mapping.

example.com
     ↓
DNS Resolver
     ↓
IP Address

The browser may check multiple caches first:

Browser Cache
     ↓
Operating System Cache
     ↓
DNS Resolver

If a valid cached result exists, a new lookup may not be necessary.

6. Establishing a Network Connection

Once the IP address is known, the browser needs to establish communication with the server.

For traditional HTTPS over HTTP/1.1 or HTTP/2, the connection typically involves TCP.

Conceptually:

Browser
   ↓
TCP Connection
   ↓
Server

TCP establishes a reliable byte stream between the two endpoints.

7. The TCP Handshake

TCP traditionally begins with a three-way handshake.

Client                       Server

SYN ------------------------>

    <---------------- SYN-ACK

ACK ------------------------>

After this process, the connection is established.

The browser can then continue with TLS negotiation for HTTPS.

8. TLS Negotiation

HTTPS uses TLS to protect communication.

A simplified process:

Browser
   ↓
TLS Handshake
   ↓
Certificate Verification
   ↓
Shared Encryption Keys
   ↓
Encrypted Connection

The browser verifies the server certificate and establishes cryptographic session keys.

After this, HTTP communication can occur securely.

9. HTTP Request

The browser can now request the page.

A simplified request:

GET /products?id=42 HTTP/1.1
Host: example.com
User-Agent: Browser
Accept: text/html

The server processes the request and sends a response.

10. HTTP Response

A server might respond:

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 5421

followed by HTML:

<!DOCTYPE html>
<html>
<head>
    <title>Store</title>
</head>
<body>
    <h1>Products</h1>
</body>
</html>

The browser now has the first major resource needed to build the page.

11. Streaming the HTML

The browser does not necessarily wait for the entire document before doing anything.

HTML can often be parsed progressively as network data arrives.

Conceptually:

Network
  ↓
HTML Chunk 1
  ↓
Parser Starts

HTML Chunk 2
  ↓
Parser Continues

This enables browsers to begin constructing the page before the complete response has finished downloading.

12. HTML Parsing

The browser parses HTML and builds a tree representation called the DOM.

DOM stands for:

Document Object Model

Consider:

<body>
    <h1>Hello</h1>
    <p>Welcome</p>
</body>

The browser may construct:

Document
   ↓
html
   ↓
body
├── h1
│    └── "Hello"
│
└── p
     └── "Welcome"

This tree becomes the structured representation of the HTML document.

13. What Is the DOM?

The DOM represents elements as nodes.

For example:

<div>
    <button>Buy</button>
</div>

can conceptually become:

DIV
 ↓
BUTTON
 ↓
"Buy"

JavaScript can interact with this structure.

For example:

document.querySelector("button").textContent = "Purchased";

The DOM therefore connects HTML structure with JavaScript behavior.

14. Discovering Additional Resources

While parsing HTML, the browser may discover additional resources.

For example:

<link rel="stylesheet" href="style.css">

<script src="app.js"></script>

<img src="hero.jpg">

The browser may then request:

style.css
app.js
hero.jpg

This means loading a page often involves many network requests rather than a single request.

15. The Preload Scanner

Modern browsers often use optimization techniques to discover resources early.

While the main parser processes HTML, a preload scanner may look ahead for important resources.

HTML
 ↓
Main Parser
 │
 └─────────→ Preload Scanner
                  ↓
               CSS
               JS
               Images

This allows downloads to begin sooner.

16. CSS Parsing

CSS is parsed into another internal structure commonly referred to as the CSSOM.

CSSOM means:

CSS Object Model

Consider:

h1 {
    color: blue;
    font-size: 32px;
}

The browser represents the stylesheet as rules that can be applied to DOM elements.

17. The CSSOM

The CSSOM contains styling information.

Conceptually:

CSS Rules

body
 ├── margin: 0
 └── font-family: sans-serif

h1
 ├── color: blue
 └── font-size: 32px

The browser needs both DOM structure and styling information before it can determine how the page should look.

18. DOM + CSSOM

The browser combines information from:

DOM
 +
CSSOM

to determine which elements need to be visually rendered.

This leads to the creation of the render tree.

DOM
  \
   → Render Tree
  /
CSSOM
19. The Render Tree

The render tree represents visible content and its styles.

Suppose HTML contains:

<p>Hello</p>

<div style="display:none">
    Hidden
</div>

The DOM contains both elements.

But the hidden element does not need a normal visible rendering box.

Conceptually:

DOM

p
div

↓

Render Tree

p

The render tree focuses on what must actually be drawn.

20. Layout

Once the render tree exists, the browser needs to determine the size and position of every element.

This process is commonly called layout.

For example:

Viewport Width: 1200px

Header
x = 0
y = 0
width = 1200
height = 80

Main
x = 100
y = 120
width = 1000
height = 600

The browser calculates geometry based on CSS rules.

21. CSS Layout Systems

Modern browsers support several layout models.

Examples include:

Normal Flow
Flexbox
CSS Grid
Positioning
Floats
Tables

For example:

.container {
    display: flex;
    justify-content: space-between;
}

The layout engine must calculate the positions of child elements according to the Flexbox algorithm.

22. Responsive Layout

Layout also depends on viewport dimensions.

For example:

@media (max-width: 768px) {
    .sidebar {
        display: none;
    }
}

The same page can produce different layouts depending on the device.

Desktop
┌────────────────────────────┐
│ Main          │ Sidebar    │
└────────────────────────────┘

Mobile
┌───────────────┐
│ Main          │
└───────────────┘
23. Painting

After layout determines geometry, the browser needs to determine what should be drawn.

This is the paint stage.

Painting may include:

Text
Colors
Borders
Shadows
Images
Backgrounds

For example:

Paint Background
      ↓
Paint Border
      ↓
Paint Text
      ↓
Paint Image

The browser creates drawing instructions.

24. Paint Records

Instead of immediately drawing every pixel, the browser may create a list of painting operations.

Conceptually:

1. Draw white background
2. Draw blue rectangle
3. Draw heading text
4. Draw image
5. Draw shadow

These instructions can then be rasterized.

25. Rasterization

Rasterization converts drawing instructions into pixels.

Conceptually:

Vector / Paint Instructions
          ↓
      Rasterizer
          ↓
        Pixels

Large pages may be divided into smaller regions or tiles.

This allows the browser to process only the parts that need rendering.

26. Compositing

Modern browsers frequently divide pages into multiple layers.

For example:

Layer 1 → Background
Layer 2 → Page Content
Layer 3 → Fixed Navigation
Layer 4 → Animation

These layers can be composited together.

Layer 4
   +
Layer 3
   +
Layer 2
   +
Layer 1
   ↓
Final Frame

The GPU can help perform this work efficiently.

27. Why Compositing Matters

Suppose an element moves across the screen.

If the browser can move an existing layer without recalculating and repainting large parts of the page, animation may be much smoother.

For example:

transform: translateX(100px);

can often be cheaper than changing layout-affecting properties.

This is one reason transform is commonly recommended for animations.

28. The Critical Rendering Path

The process from receiving resources to displaying pixels is often described using the idea of the critical rendering path.

A simplified version:

HTML
 ↓
DOM
 ↓
        CSS
         ↓
       CSSOM
         ↓
DOM + CSSOM
     ↓
Render Tree
     ↓
Layout
     ↓
Paint
     ↓
Composite
     ↓
Pixels

Optimizing this path can improve page-load performance.

29. JavaScript Enters the Picture

JavaScript can dramatically affect rendering.

For example:

document.body.innerHTML = "<h1>Hello</h1>";

JavaScript can modify:

DOM
Styles
Classes
Attributes
Layout

These changes may cause the browser to perform additional rendering work.

30. JavaScript Engines

Browsers include specialized JavaScript engines.

Examples include:

Chrome / Edge → V8
Firefox       → SpiderMonkey
Safari        → JavaScriptCore

A simplified JavaScript pipeline might look like:

JavaScript Source
       ↓
Parser
       ↓
Internal Representation
       ↓
Interpreter / JIT Compiler
       ↓
Machine Code
       ↓
CPU

Modern engines use sophisticated optimization techniques.

31. JavaScript and the Main Thread

Many important browser operations occur on a main thread.

This may include:

JavaScript Execution
DOM Updates
Style Calculation
Layout
Parts of Rendering Coordination

If JavaScript occupies the main thread for too long:

Long JavaScript Task
        ↓
Main Thread Blocked
        ↓
User Click Delayed
        ↓
Animation Stutters

the page can feel unresponsive.

32. Long Tasks

Imagine:

for (let i = 0; i < 10_000_000_000; i++) {
    // heavy calculation
}

The main thread may remain busy.

During this time, the browser may struggle to process:

User input
Rendering updates
Animation frames

This is why long-running JavaScript should be handled carefully.

33. The Event Loop

JavaScript environments in browsers commonly use an event loop architecture.

Conceptually:

Call Stack
    ↓
Event Loop
    ↓
Task Queue

The browser can schedule work such as:

User Click
Timer
Network Callback
JavaScript Task

The event loop determines when queued tasks can execute.

34. Example Event Loop Behavior

Consider:

console.log("A");

setTimeout(() => {
    console.log("B");
}, 0);

console.log("C");

A typical output is:

A
C
B

The timer callback does not interrupt the currently executing JavaScript.

It becomes eligible to run later through the event loop.

35. Microtasks

Browsers also maintain microtask processing.

Promises commonly schedule microtasks.

Example:

console.log("A");

Promise.resolve().then(() => {
    console.log("B");
});

console.log("C");

Typical output:

A
C
B

Microtasks are processed according to event-loop rules and have important implications for asynchronous JavaScript.

36. Parsing JavaScript

JavaScript source must also be parsed.

For example:

const total = price + tax;

Conceptually:

Source
 ↓
Tokens
 ↓
Parser
 ↓
Syntax Tree
 ↓
Execution / Compilation

Modern JavaScript engines use techniques similar to compilers while also optimizing code dynamically at runtime.

37. JavaScript Can Block HTML Parsing

Consider:

<script src="app.js"></script>

Depending on how the script is loaded, the HTML parser may need to pause.

Conceptually:

HTML Parsing
     ↓
<script>
     ↓
Download JS
     ↓
Execute JS
     ↓
Continue Parsing

This can delay page rendering.

38. Async Scripts

The async attribute allows a script to download independently.

<script src="analytics.js" async></script>

Conceptually:

HTML Parsing ───────────────>

JS Download ───────>

         Execute when ready

Execution timing depends on when the download completes.

39. Defer Scripts

defer also downloads scripts while HTML parsing continues.

<script src="app.js" defer></script>

Deferred scripts generally execute after HTML parsing has completed, before DOMContentLoaded.

Conceptually:

HTML Parsing ─────────────────────>

JS Download ───────────>

                         Execute

For many application scripts, defer provides predictable behavior.

40. Browser Caching

Browsers cache many downloaded resources.

Examples:

CSS
JavaScript
Images
Fonts
API Responses

Instead of downloading the same resource again:

Browser
   ↓
Cache
   ↓
Resource

This can significantly improve repeat visits.

41. Memory Cache vs Disk Cache

Browsers can use different types of caches.

Conceptually:

Browser Cache

├── Memory Cache
│      ↓
│   Very Fast
│
└── Disk Cache
       ↓
    Persistent

The exact caching behavior depends on browser implementation and HTTP caching headers.

42. Local Storage

Web applications can also store data locally.

One option is localStorage.

Example:

localStorage.setItem(
    "theme",
    "dark"
);

Retrieve it:

const theme = localStorage.getItem("theme");

Local storage persists across browser sessions until removed.

43. Session Storage

sessionStorage is similar but typically belongs to a particular page session.

sessionStorage.setItem(
    "step",
    "checkout"
);

It can be useful for temporary state associated with a tab or session.

44. Cookies

Cookies are small pieces of data associated with websites.

Example:

Set-Cookie: session=abc123

A browser may later send:

Cookie: session=abc123

Cookies are commonly used for:

Sessions
Authentication
Preferences
Tracking

Security attributes such as:

Secure
HttpOnly
SameSite

are important when using cookies.

45. IndexedDB

For more complex browser-side data, applications can use IndexedDB.

It provides a browser database suitable for structured data.

Potential uses include:

Offline Applications
Large Client-Side Data
Cached Application State
Progressive Web Apps

IndexedDB supports significantly more sophisticated storage than simple key-value APIs such as localStorage.

46. Browser Processes

Modern browsers often use multi-process architectures.

A simplified design:

Browser Process
      │
      ├── Renderer Process
      ├── Renderer Process
      ├── GPU Process
      └── Network Process

Different tabs or sites may be isolated into separate processes depending on the browser.

This improves security and stability.

47. Why Browsers Use Multiple Processes

Imagine one webpage crashes.

With a single-process architecture:

Page Crash
    ↓
Entire Browser Crash

With process isolation:

Renderer Crash
      ↓
Affected Tab

The rest of the browser may continue running.

Process separation can also provide stronger security boundaries.

48. Browser Sandboxing

Renderer processes may operate inside a restricted sandbox.

Conceptually:

Untrusted Website Code
        ↓
Renderer Sandbox
        ↓
Restricted System Access

Even if malicious code exploits a bug in the renderer, the sandbox attempts to limit what that code can access.

Browser security relies on multiple layers rather than a single mechanism.

49. Same-Origin Policy

Browsers enforce an important security model called the Same-Origin Policy.

An origin is typically defined by:

Scheme + Host + Port

For example:

https://example.com:443

JavaScript from one origin is generally restricted from freely accessing sensitive resources from another origin.

50. CORS

Sometimes cross-origin requests are necessary.

Cross-Origin Resource Sharing (CORS) provides a controlled mechanism.

A server may return:

Access-Control-Allow-Origin: https://app.example.com

This tells the browser that a particular origin is allowed under the relevant CORS rules.

CORS is enforced by browsers; it is not an authentication system.

51. Reflow and Layout Changes

Some DOM or style changes require layout to be recalculated.

For example:

element.style.width = "500px";

may affect the position of surrounding elements.

Conceptually:

DOM Change
   ↓
Style Recalculation
   ↓
Layout
   ↓
Paint
   ↓
Composite

Repeated unnecessary layout changes can hurt performance.

52. Repaint

Some visual changes may require repainting but not major layout recalculation.

For example:

color: red;

might affect pixels without changing element geometry.

Conceptually:

Style Change
    ↓
Paint
    ↓
Composite

The actual rendering work depends on the property and browser implementation.

53. Layout Thrashing

A performance problem called layout thrashing can happen when JavaScript repeatedly alternates between reading layout information and changing styles.

For example:

element.style.width = "100px";

console.log(element.offsetWidth);

element.style.width = "200px";

console.log(element.offsetWidth);

The browser may be forced to repeatedly calculate layout.

A better strategy is often to batch reads and writes.

54. Browser Rendering and 60 FPS

Smooth animations often target approximately:

60 frames per second

At 60 FPS, each frame has roughly:

1000 ms / 60 ≈ 16.7 ms

available.

Within that time, the browser may need to perform:

JavaScript
Style
Layout
Paint
Composite

Long work can cause dropped frames.

55. Web Workers

Heavy JavaScript computation can sometimes be moved away from the main thread using Web Workers.

Architecture:

Main Thread
    │
    ├── UI
    ├── DOM
    └── Events
         │
         ↓
      Worker
         ↓
Heavy Computation

Workers cannot directly manipulate the DOM, but they can communicate using messages.

56. Web Worker Example

Main script:

const worker = new Worker("worker.js");

worker.postMessage({
    numbers: [1, 2, 3]
});

worker.onmessage = event => {
    console.log(event.data);
};

Worker:

self.onmessage = event => {
    const result = event.data.numbers.reduce(
        (a, b) => a + b,
        0
    );

    self.postMessage(result);
};

This can keep heavy computation from blocking the UI thread.

57. GPU Acceleration

Browsers can use GPUs for certain rendering operations.

The GPU is especially good at massively parallel graphics tasks.

Conceptually:

Browser
   ↓
Compositor
   ↓
GPU
   ↓
Screen

Operations involving transforms and compositing can often benefit from GPU acceleration.

58. Images and Rendering Performance

Images are often among the largest resources on a webpage.

For example:

HTML       50 KB
CSS        80 KB
JavaScript 300 KB
Images     4 MB

Optimizing images can dramatically improve load performance.

Common techniques include:

Compression
Responsive Images
Modern Formats
Lazy Loading
CDNs
59. Lazy Loading

Images below the visible area may not need to load immediately.

HTML supports:

<img
    src="photo.jpg"
    loading="lazy"
    alt="Example"
/>

Conceptually:

Page Loads
   ↓
Visible Images Download
   ↓
User Scrolls
   ↓
Additional Images Load

This can reduce initial network usage.

60. Browser Performance Metrics

Modern web performance often focuses on user-centered metrics.

Important concepts include:

Largest Contentful Paint
Interaction responsiveness
Layout stability
Initial server response
Resource-loading behavior

These metrics help developers understand how quickly and smoothly users experience a page.

61. The Complete Browser Journey

Let's combine everything.

The user enters:

https://example.com

Then:

URL
 ↓
Parse URL
 ↓
DNS Lookup
 ↓
TCP / QUIC Connection
 ↓
TLS
 ↓
HTTP Request
 ↓
HTML Response
 ↓
Parse HTML
 ↓
Build DOM
 ↓
Download CSS / JS / Images
 ↓
Build CSSOM
 ↓
Execute JavaScript
 ↓
Create Render Tree
 ↓
Calculate Layout
 ↓
Paint
 ↓
Rasterize
 ↓
Composite Layers
 ↓
GPU
 ↓
Pixels on Screen

All of this can happen in a fraction of a second on a well-optimized website.

62. Browser Performance Best Practices

Developers can improve browser performance by following several principles.

Reduce unnecessary JavaScript.

Less JavaScript
      ↓
Less Parsing
      ↓
Less Execution
      ↓
More Responsive Page

Optimize critical CSS.

Avoid unnecessarily blocking resources.

Use:

async
defer
lazy loading
caching
compression
CDN delivery

when appropriate.

Avoid excessive DOM manipulation.

Prefer batching changes instead of repeatedly modifying the page.

63. Why Understanding Browsers Matters

Frontend development becomes much easier to reason about once you understand the browser itself.

Problems such as:

Slow Rendering
Layout Shifts
JavaScript Blocking
Slow Network Requests
Animation Stutters
Caching Bugs
CORS Errors

are no longer mysterious.

They can be connected to specific stages of the browser architecture.

64. Browser vs Web Application

It is important to remember that your application does not directly control the screen.

Your code interacts with the browser.

Application Code
      ↓
Browser APIs
      ↓
Rendering Engine
      ↓
Operating System
      ↓
GPU
      ↓
Display

The browser acts as a sophisticated runtime between web applications and the underlying computer.

Conclusion

Displaying a webpage is one of the most complex everyday operations performed by modern computers.

What appears to the user as:

Type URL
   ↓
See Website

actually involves:

DNS
Networking
TLS
HTTP
HTML Parsing
CSS Parsing
JavaScript Execution
DOM
CSSOM
Layout
Painting
Rasterization
Compositing
GPU Rendering

Browsers combine networking systems, compiler technology, operating-system concepts, security models, graphics pipelines, databases, and JavaScript runtimes inside a single application.

The complete journey can be summarized as:

URL
 ↓
Network
 ↓
Resources
 ↓
Browser Engine
 ↓
DOM + CSSOM
 ↓
Layout
 ↓
Paint
 ↓
Composite
 ↓
Pixels

Understanding this pipeline helps developers build applications that are not only functional, but also fast, responsive, secure, and efficient.

A browser may look like a simple window into the internet, but internally it is one of the most sophisticated pieces of software running on a modern computer.
