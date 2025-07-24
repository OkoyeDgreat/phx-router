### **First version** 
 _***Basic usage***_ 
Create and include your router file in the main html document:
 ` <script type"module" src=""path/your-router-file.js"></script>`

**inside your-router-file.js**

```
import PHXRouter from "./../cssjs/phx-router.js?v=1.0";
//also import other plugins  here 
```


//Initializing the Router

```
const router = new PHXRouter('app-root');
```

> app-root can be any valid html element selector example : #app for id or .app for class or even the element tag just app for <app>

**Defining Routes**
```

router.on('/', () => {
    router.load('/pages/home.html');
});

or

router.on('/about', () => {
    router.load(router.current,true); //to load router matched url pattern  and scroll to top after loading if the true second arg is passed
});

for url with params
router.on('/about/?', () => {
    router.load(router.current); //to load router matched url pattern 
});
//this will match /about/?item="first"

```

**Navigation**

You can trigger navigation programmatically using 
```

router.navigateTo('/about');

```

To avoid double rendering of data from server since php sends server render html to use 
wrap section of your elements your don't to be re-render after first render like header,footer,link and script

<phx-element data-name="name-for-the-element"></phx-element>

**inside your-router-file.js**
 before you start defining routes
add this line 
```
`
router.phxElement = ["name-for-the-element","and more if any"];  
`

```

**Don't forget to include .htaccess rule 
to redirect to home incase a url is not found** 
```


without apache
ErrorDocument 404 /

with  mod_rewrite
//RewriteEngine On

# If the request is not for a valid file (e.g., .css, .js, images)
RewriteCond %{REQUEST_FILENAME} !-f
# If the request is not for a valid directory
RewriteCond %{REQUEST_FILENAME} !-d
# Redirect to root
RewriteRule ^ / [L,R=301]



```
**Accessing Elements within the Root**
```
router.getEl("selector"); //will return first element that matches 
router.getEl("selector",true); //will return all element that matches 

**This to note so be able to enjoy working with this library**
- Error Handling: Implement robust error handling in your load() callbacks to gracefully manage cases where content fetching fails. Although some hint will be outputted on the console.log
- Loading Indicators: Provide visual feedback (e.g., loading spinners) while content is being fetched.
- Security: Be mindful of XSS vulnerabilities when injecting dynamic HTML content. Sanitize any user-generated content
- 


```
