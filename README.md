
# PHXRouter - A Lightweight Client-Side Router for PHP Developers ✨

PHXRouter is a client-side routing library inspired by React Router, designed to bring SPA-like navigation to your traditional PHP applications without the overhead of complex JavaScript frameworks. Focus on your application logic, and let PHXRouter handle the client-side routing magic\!

## 🌟 Features

  * **Simple Route Management:** Define routes with a familiar and intuitive API.
  * **Dynamic Content Loading (SPA-style):** Load HTML content into a designated root element, creating seamless page transitions.
  * **HTML5 History API Integration:** Utilizes `pushState` and `replaceState` for clean URLs without page reloads.
  * **Background Task Management:** Register and control long-running JavaScript tasks (like `setInterval`) tied to specific routes, ensuring they're cleaned up when navigating away.
  * **Automatic Link Interception:** Automatically intercepts clicks on `<a>` tags with a `data-router` attribute for smooth, in-app navigation.
  * **Dynamic Script Loading:** Inject JavaScript files on demand, perfect for loading route-specific plugins or modules.

## 🚀 Why PHXRouter?

PHXRouter bridges the gap between traditional server-rendered PHP applications and modern single-page application (SPA) experiences. It allows PHP developers to enhance user experience with fluid navigation and dynamic content updates, all while maintaining their preferred PHP backend. It's perfect for projects that need SPA-like features without committing to a full-blown frontend framework like React, Vue, or Angular.

## 📚 Documentation

### Getting Started

#### Installation

To get started, simply include `phxrouter.js` in your project. You can typically place it in your `public/js` directory or similar.

```javascript
// In your main JavaScript file (e.g., app.js)
import PHXRouter from './phxrouter.js'; // Adjust path as needed
```

#### Initialization

Instantiate the router by providing the ID of your root DOM element, which will serve as the container for dynamically loaded content.

```javascript
const router = new PHXRouter('app-container');
// 'app-container' should match the ID of your root DOM element, e.g., <div id="app-container"></div>
```

#### Defining Routes

PHXRouter provides a flexible `on()` method to define your routes and their corresponding actions.

**Style 1: Loading a Specific URL**

This method is straightforward when you know the exact HTML file you want to load for a route.

```javascript
router.on('/', () => {
    router.load('/home.html');
});
```

**Style 2: Loading Based on Current Window Location (Recommended)**

This approach is more dynamic and robust. It uses `router.current` to derive the content URL from the browser's current path, making your routing more declarative.

```javascript
router.on('/', () => {
    router.load(router.current); // The library gets window.location and parses it to match the router call
});
```

**Routes with Dynamic Parameters**

Define routes with dynamic segments using `?`. The `params` argument in the callback provides access to these segments.

```javascript
router.on('/user/?', (params) => {
    console.log(params); // An object containing dynamic segments. E.g., for /user/123 -> { '0': '123' }
    // You can also use router.getParams('name') for query parameters (e.g., /user?id=123)
});
```

#### Navigation

**Programmatic Navigation**

Trigger navigation directly from your JavaScript code.

```javascript
router.navigateTo('/contact');
```

**HTML Link Navigation (Automatic Interception)**

For standard `<a>` tags, add the `data-router` attribute. PHXRouter will automatically intercept clicks on these links, preventing full page reloads and handling the routing internally.

```html
<a href="/about" data-router>About Us</a>
<a href="/dashboard" data-router>Dashboard</a>
```

### Advanced Features

#### Background Task Management

Manage long-running background JavaScript tasks (like `setInterval` or Web Workers) that are specific to certain routes. This ensures that tasks are started when a route is active and automatically stopped when you navigate away, preventing memory leaks and unnecessary processing.

```javascript
// Add a background function
const intervalID = setInterval(() => {
    console.log("Running background task...");
}, 1000);

// Register the task with a unique ID and its reference
router.addbFunc('logger', intervalID);
// 'logger' is the background task name, intervalID is the task reference (e.g., from setInterval, setTimeout)

// Remove later (e.g., when navigating away from the route that needs this task)
router.stopbFunc('logger'); // Stop a specific background function by its name

// Calling stopbFunc() without a name will stop ALL registered background tasks.
router.stopbFunc(); // Good for general cleanup on route changes.

// This is great for "fake async" updates on PHP-rendered pages, allowing you to fetch
// data via server APIs without a full page refresh.
```

#### Dynamic Script Loading

Load additional JavaScript files on demand, which is perfect for route-specific functionalities, third-party plugins, or modularizing your JavaScript.

```javascript
// For regular JS plugins (e.g., a jQuery plugin)
router.injector('/js/custom-plugin.js');

// Dynamic import with import() method is possible for ESM JS modules:
// You can use this within your route definitions for highly optimized loading:

router.on("/office/product/edit/?", async () => {
    // Dynamically import a module for this specific route
    import("./components/product-image.js?v=1.2");

    // Load the HTML content for the route
    await router.load(router.current, true); // true scrolls to top after loading

    // Once content and scripts are loaded, interact with them
    const sw = router.getEl("product-edit"); // Assuming 'product-edit' is an ID in the loaded HTML
    window.ClassicEditor.create(window.router.getEl('textarea')) // Example with a rich text editor
        .then(editorInstance => {
            sw.setEditor(editorInstance); // Store editor instance if needed
        });
});
```

### Methods

#### Content Loading

`load(path, jump = false, data = null)`

Loads HTML content into the router's root container.

  * `path` (String): The URL to fetch HTML content from (e.g., `'/posts.html'`).
  * `jump` (Boolean, default `false`): If `true`, the window will scroll to the top after the content is loaded.
  * `data` (Object, default `null`): Optional data to send with the fetch request (e.g., for POST requests).

<!-- end list -->

```javascript
router.load('/posts.html', true); // Load posts and scroll to top
router.load('/api/dashboard-data', false, { userId: 123 }); // Load with data (e.g., for a POST request)
```

#### Script Injection

`injector(src)`

Dynamically loads a JavaScript file by appending a `<script>` tag to the document's `<head>`.

  * `src` (String): The URL of the JavaScript file (e.g., `'/js/analytics.js'`).

<!-- end list -->

```javascript
router.injector('/js/analytics.js');
```

#### Utility Methods

`getEl(selector, all = false)`

A utility method to get DOM element(s) within the router's root container. This is useful for scoped DOM manipulation after content has been loaded.

  * `selector` (String): The CSS selector for the element(s).
  * `all` (Boolean, default `false`): If `true`, returns a `NodeList` of all matching elements; otherwise, returns the first matching element.

<!-- end list -->

```javascript
const submitButton = router.getEl('.submit-btn');       // Get the first element with class 'submit-btn'
const allButtons = router.getEl('.btn', true);          // Get all elements with class 'btn'
```

`getParams(name)`

Retrieves a query parameter from the current URL's query string.

  * `name` (String): The name of the query parameter (e.g., `'id'` for `?id=123`).

<!-- end list -->

```javascript
// Assuming current URL is: /user-profile?id=123&name=Alice
const userId = router.getParams('id');    // Returns "123"
const userName = router.getParams('name');  // Returns "Alice"
```

`addbFunc(id, data)`

Registers a background function or task reference to be managed by the router.

  * `id` (String): A unique identifier for the background task (e.g., `'statsTimer'`).
  * `data`: The reference to the background task (e.g., the ID returned by `setInterval` or `setTimeout`, or an object representing a worker).

<!-- end list -->

```javascript
const timer = setInterval(() => {
    // ... do something in background
}, 1000);
router.addbFunc('statsTimer', timer);
```

`stopbFunc(name)`

Stops one or all registered background functions.

  * `name` (String, optional): The `id` of the specific background task to stop. If omitted, all registered background tasks will be stopped.

<!-- end list -->

```javascript
router.stopbFunc('statsTimer'); // Stop the specific 'statsTimer'
router.stopbFunc();             // Stop all registered background functions
```

-----


**PHXRouter** is designed to provide a smooth, modern navigation experience for your PHP applications with minimal JavaScript boilerplate. Give it a try\!
