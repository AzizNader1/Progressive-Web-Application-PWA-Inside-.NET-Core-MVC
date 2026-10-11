# Progressive Web Apps with ASP.NET MVC

## A complete beginner-to-mastery course, built step by step

**Last reviewed:** 2026-10-11  
**Primary path:** ASP.NET Core MVC on .NET 8, .NET 9, or .NET 10  
**Secondary path:** legacy ASP.NET MVC 5 on .NET Framework 4.8  
**Teaching project:** `PwaMvcCourse`, a small task-board application

This document is written as a course, not as a short reference sheet. It assumes that you may know little or nothing about Progressive Web Apps, browsers, JavaScript, HTTP, or ASP.NET MVC. Each chapter introduces the ideas needed by the next chapter. Each implementation chapter adds one working part to the same application.

You should read the chapters in order the first time. When you later use this document at work, jump to the chapter that matches the problem you are solving.

> **The outcome:** by the end, you will understand what a PWA is, why it exists, how an ASP.NET MVC request works, how to create an MVC application, how to add a manifest and service worker, how to design safe offline behavior, how to update and deploy the app, and how to evolve the baseline into a production-grade PWA.

---

# Course map

The course is divided into chapters. The chapters form one journey:

- **Chapter 0 — How to study this guide:** how to read, run, test, and learn from the examples.
- **Chapter 1 — The problem:** what a website, web app, native app, and PWA are.
- **Chapter 2 — History:** how the web gradually acquired PWA capabilities.
- **Chapter 3 — Why use a PWA with MVC:** benefits, costs, limits, and alternatives.
- **Chapter 4 — Vocabulary from zero:** the words you must understand before writing code.
- **Chapter 5 — MVC fundamentals:** Model, View, Controller, routing, Razor, and the request lifecycle.
- **Chapter 6 — Prepare your computer:** tools, SDKs, editors, HTTPS, and project decisions.
- **Chapter 7 — Create the MVC application:** create and run the teaching project from an empty folder.
- **Chapter 8 — Build a small MVC feature:** build a task board so that the PWA is attached to a real application.
- **Chapter 9 — Plan the PWA layer:** decide what should be installed, cached, available offline, or kept online.
- **Chapter 10 — Add the manifest and icons:** make the application describable and installable.
- **Chapter 11 — Add the offline page:** create the safest first offline experience.
- **Chapter 12 — Register a service worker:** connect browser PWA behavior to the MVC layout.
- **Chapter 13 — Write the service worker:** lifecycle, caching, navigation fallback, and line-by-line explanation.
- **Chapter 14 — Test everything:** DevTools, offline testing, update testing, and browser differences.
- **Chapter 15 — Deploy safely:** IIS, reverse proxies, virtual directories, headers, and production verification.
- **Chapter 16 — Legacy MVC 5:** the exact differences when the application is not ASP.NET Core.
- **Chapter 17 — Master caching:** strategies, privacy boundaries, versioning, and invalidation.
- **Chapter 18 — Master offline data:** IndexedDB, outboxes, retries, idempotency, and conflicts.
- **Chapter 19 — Advanced PWA features:** install UX, push, notifications, shortcuts, share, and background work.
- **Chapter 20 — Production engineering:** security, accessibility, performance, observability, and CI/CD.
- **Chapter 21 — Mastery roadmap:** exercises, milestones, and a definition of done.
- **Appendices:** command reference, glossary, troubleshooting, and references.

---

# Chapter 0 — How to study this guide

## 0.1 The rule: type, run, observe, explain

Do not read this guide as if it were a novel. For every implementation step:

1. Read the explanation before the code.
2. Type the command or create the file.
3. Run the application.
4. Open the result in a browser.
5. Change one small thing.
6. Run it again and observe what changed.
7. Return to the line-by-line explanation and explain the code in your own words.

A developer becomes comfortable with PWA work by repeatedly connecting four things:

- **Source code:** what you write.
- **HTTP response:** what the server sends.
- **Browser state:** what the browser stores and controls.
- **User experience:** what the user sees when online, offline, installing, updating, or signing in.

If you skip the browser state, service workers feel mysterious. If you skip the server response, MIME types and HTTPS feel mysterious. If you skip the user experience, it is easy to ship a technically valid but confusing offline application.

## 0.2 The teaching project

We will build `PwaMvcCourse`, a simple task-board application. It will have:

- A normal server-rendered MVC page.
- A model named `TaskItem`.
- A controller named `TasksController`.
- A view that lists tasks and posts a new task.
- A shared layout.
- A manifest.
- An offline page.
- A service worker.
- An install button and an update button.
- Safe default caching that avoids private HTML and state-changing requests.

The teaching application uses an in-memory store first. That keeps the first learning steps focused on MVC and PWA concepts rather than database configuration. Later chapters explain how to replace the in-memory store with a database and how to design offline synchronization.

## 0.3 The two application paths

The main examples use **ASP.NET Core MVC** because it is the current, cross-platform ASP.NET MVC line. The files that run in the browser—HTML, CSS, JavaScript, manifest, and service worker—also work with MVC 5. MVC 5 has different conventions for static files, project files, and IIS deployment, so its adaptation is presented separately in Chapter 16.

If you already have an application, do not create a second production application just to follow the guide. Use the teaching project to learn safely, then apply the same concepts to your existing application after reading Chapter 9.

## 0.4 A note about version numbers

The browser APIs in this guide are web-platform APIs. ASP.NET Core serves the files; it does not replace the browser APIs. The exact way static assets are enabled differs between .NET versions. The guide shows both `UseStaticFiles()` and `MapStaticAssets()` where they differ.

Use the supported .NET SDK installed by your team. The command examples use `net10.0` when creating a new project because the current Microsoft documentation presents .NET 10 as a supported target. If your machine has .NET 8 or .NET 9, replace `net10.0` with the target you actually support.

---

# Chapter 1 — The problem: website, web app, native app, and PWA

## 1.1 What is a website?

A **website** is a collection of pages and resources delivered through web technologies. A user normally opens it with a browser, follows a link, submits a form, and receives another response from a server.

The important characteristic is that the browser is the entry point. The user does not need to install a special executable before opening a URL.

A website can be simple or complex. A news article and an accounting system can both be websites even though their behavior is very different.

## 1.2 What is a web application?

A **web application** is a website that provides interactive functionality. It accepts input, applies rules, stores or retrieves data, and produces a result for the user.

An MVC task board is a web application because it:

- receives a request for a task list;
- obtains task data;
- renders HTML;
- accepts a form submission;
- validates the submitted title;
- stores the task; and
- redirects the browser back to the list.

## 1.3 What is a native application?

A **native application** is software built for a particular operating system or platform family. It may be installed from an app store, appear in the operating system's application launcher, use platform services, and run with a platform-specific user interface.

Native applications can have deep operating-system integration, but usually require a separate build, distribution, and release process for each platform family.

## 1.4 What is a Progressive Web App?

A **Progressive Web App**, usually shortened to **PWA**, is a web application that uses web-platform capabilities to provide a more app-like experience while remaining a website. MDN describes a PWA as an application built with web technologies that can provide an experience similar to a platform-specific app, including installation and offline or background behavior where supported. [1]

The word **progressive** is essential:

- A browser that supports only ordinary HTML should still be able to use the website.
- A browser that supports a manifest may offer installation.
- A browser that supports service workers may provide offline behavior.
- A device that supports notifications or sharing may provide those features.
- The application adds capabilities without requiring every browser to support every capability.

A PWA is therefore not one framework and not one package. It is an architecture built from browser features and good web engineering.

## 1.5 What a PWA adds to an MVC application

An MVC application normally gives you server-rendered pages and server-side behavior. PWA capabilities add several browser-facing features:

- **Installability:** the browser may offer an app-like installation flow.
- **An app identity:** the manifest defines the name, icons, launch URL, display mode, and colors.
- **Offline fallback:** a service worker can respond when a navigation cannot reach the server.
- **Resource caching:** safe static assets can be reused without a network request.
- **Update control:** a new service worker can be installed and activated in a controlled way.
- **Optional operating-system integration:** shortcuts, sharing, notifications, badges, and other features may be available on supported platforms.

The PWA layer does not automatically provide an offline database or offline business process. Those are separate product and engineering decisions.

## 1.6 Website versus PWA: not an either/or choice

A PWA begins as a website. A user can open its URL, search for it, share it, and use it without installing it. Installation is an optional enhancement.

This is different from a system where the user must install an application before seeing any content. The original PWA idea emphasized retaining the web's URLs, accessibility, linkability, and progressive enhancement while adding capabilities such as installability and offline behavior. [2]

---

# Chapter 2 — A short history of PWAs

## 2.1 The web started with documents and links

The early web was primarily a document system. A browser requested a URL, the server returned a document, and the user followed links to other documents.

Over time, the web gained:

- forms and server-side applications;
- JavaScript for behavior in the page;
- asynchronous requests for updating part of a page;
- responsive design for different screen sizes;
- browser storage;
- APIs for device and operating-system capabilities; and
- workers that could run outside the page.

The modern PWA is the result of these capabilities becoming available together.

## 2.2 Responsive web design changed the physical assumption

A page that assumes one desktop screen does not provide a good experience on a phone. Responsive design introduced the expectation that one URL and one codebase could adapt to different screen sizes and input methods.

This matters to PWAs because an installed app may be displayed in a small phone window, a tablet surface, a desktop window, or a browser tab. Responsive design is not an optional decoration; it is part of making one web application usable in many contexts.

## 2.3 Mobile web apps showed that the browser could feel app-like

Mobile browsers introduced home-screen shortcuts and metadata that allowed a website to be launched more like an application. Developers learned that a website could have a full-screen or standalone presentation without becoming a completely separate native codebase.

These earlier techniques were useful, but they did not by themselves solve reliable offline behavior, update control, or broad capability integration.

## 2.4 The term Progressive Web Apps was introduced in 2015

In a June 2015 article, Alex Russell described a new class of web applications and the principles he and Frances Berriman had been discussing. The article used the name **Progressive Apps**, which became widely known as **Progressive Web Apps**. The described principles included responsive behavior, connectivity independence through service workers, app-like interaction, freshness, safety through TLS, discoverability, re-engagement, installability, and linkability. [2]

This history is important because PWA did not begin as a proprietary product that replaced the web. The concept was a way to describe a web application that progressively earned more capabilities while retaining the web's fundamental properties.

## 2.5 Service workers made offline interception practical

A service worker is a special JavaScript worker that can receive lifecycle events and selected network requests. During installation, it can cache resources. During activation, it can remove old caches. During fetch handling, it can choose whether to use the network, a cache, or a fallback response. [5]

That ability to sit between a page and the network is what makes robust offline strategies possible. It also makes mistakes more powerful: a bad worker can serve stale pages, cache private data, or hide server changes. This is why service workers should be introduced conservatively.

## 2.6 The manifest made web applications describable to the operating system

The web app manifest is a JSON document containing application metadata. It gives a supporting browser information such as the application name, icons, start URL, scope, display mode, and colors. [4]

The manifest does not contain your server code and does not install your application by itself. It describes how the browser and operating system should treat the web application.

## 2.7 The modern PWA is a collection of standards

Today, PWA work usually combines:

- HTML for structure;
- CSS for presentation and responsive layout;
- JavaScript for client-side behavior;
- HTTP and HTTPS for transport;
- a manifest for application identity;
- a service worker for lifecycle and request handling;
- Cache Storage for response caching;
- IndexedDB for structured local data when needed; and
- optional APIs for notifications, sharing, background work, and installation.

No single feature is the whole PWA. A manifest without a usable website is not a good PWA. A service worker that caches private data is not a safe PWA. A page that is installable but inaccessible is not a successful application.

---

# Chapter 3 — Why use a PWA with ASP.NET MVC?

## 3.1 The main reason: improve the web experience without abandoning the web

ASP.NET MVC is already a strong model for server-rendered web applications. It provides routing, model binding, validation, dependency injection, filters, Razor views, APIs, and testability. [7]

Adding PWA capabilities lets you improve the browser experience without rewriting the server application as a mobile app or replacing every MVC view with a client-side framework.

## 3.2 Benefits

A PWA can provide:

- a direct URL for every meaningful screen;
- ordinary browser navigation and search-engine discoverability;
- one web codebase for multiple device sizes;
- an optional installed experience;
- faster repeat visits for safe static resources;
- a useful offline explanation instead of a generic browser error;
- controlled update behavior; and
- progressive enhancement for browsers that support additional APIs.

## 3.3 Costs and responsibilities

A PWA also adds responsibilities:

- You must understand service-worker lifecycle and scope.
- You must design cache invalidation.
- You must decide what is safe to store locally.
- You must test online, offline, first visit, repeat visit, update, sign-out, and multiple-tab states.
- You must test the browser and operating systems that matter to your users.
- You must ensure HTTPS and correct deployment headers.
- You must create fallback behavior for unsupported APIs.

The service worker is production code. Treat it with the same review and release discipline as controller, database, and authentication code.

## 3.4 When a PWA is a good choice

A PWA is usually a strong choice when:

- your users already access the application through a browser;
- the same application must work on desktop and mobile;
- installation is useful but a full native application is not required;
- the application benefits from faster repeat visits or limited offline access;
- you want to keep URLs and web discoverability; or
- the team wants to improve an existing MVC application incrementally.

## 3.5 When a PWA may not be enough

Consider additional or alternative technology when you need:

- deep platform integrations that the target browsers cannot provide;
- guaranteed background processing with strict operating-system scheduling;
- complex offline synchronization across many large datasets;
- high-performance graphics or hardware access not exposed by the target browsers;
- app-store distribution requirements that cannot be met by a web wrapper or PWA package; or
- a fully native user experience for one platform.

The right question is not “PWA or native?” in the abstract. The right question is “Which user journeys need which capabilities, and which delivery model gives us the required reliability and cost?”

## 3.6 PWA compared with common alternatives

### Responsive MVC website

A responsive MVC website is the foundation. It can be perfectly adequate when installation and offline behavior are not important. A PWA extends this foundation rather than replacing it.

### Single-page application

A single-page application performs much of its navigation and rendering in the browser. A PWA may be a single-page application, but it does not have to be. The teaching project in this guide remains server-rendered MVC and gains PWA capabilities.

### Native application

A native application may provide stronger platform integration, but it introduces platform-specific development and distribution. A PWA provides a URL-first route and can still be installed where supported.

### Blazor

Blazor is an ASP.NET web UI technology. It is a separate UI approach from traditional MVC views. A Blazor application can also be made into a PWA, but this guide focuses on server-rendered MVC and browser service-worker behavior.

---

# Chapter 4 — Vocabulary from absolute zero

This chapter defines the words used in the rest of the guide. Read it slowly. Understanding the words is more important than memorizing the code.

## 4.1 Computer, operating system, process, and application

A **computer** is the physical or virtual machine running software.

An **operating system** manages the computer and provides services to programs. Windows, macOS, Linux, Android, and iOS are examples.

A **process** is a running instance of a program.

An **application** is a program that performs a useful task for a user. A browser is an application. Your MVC server is also an application when it is running.

## 4.2 Client and server

The **client** is the program that requests a service. In this course, the browser is the client.

The **server** is the program that receives requests and sends responses. In ASP.NET Core MVC, Kestrel, IIS, or a reverse proxy participates in serving the application.

The words do not mean “good” and “bad,” and they do not always mean two different computers. During local development, the client and server may run on the same machine.

## 4.3 Browser

A **browser** is a client application that understands web standards. It can parse HTML, calculate CSS layout, execute JavaScript, store cookies, use Cache Storage, and provide browser APIs such as service workers.

The browser controls whether a PWA can be installed and which optional APIs are available. Your server cannot force a browser to expose an API that it does not support.

## 4.4 URL and URI

A **URL** identifies a resource or endpoint. For example:

```text
https://localhost:5001/Tasks/Index?page=2
```

Break the example into words:

- `https` is the **scheme**. It tells the client to use encrypted HTTP.
- `localhost` is the **host**. It means the current computer.
- `5001` is the **port**. It identifies the network entry point.
- `/Tasks/Index` is the **path**. MVC routing can map it to a controller action.
- `?page=2` is a **query string**. It provides optional parameters.

## 4.5 HTTP

**HTTP** is the protocol used to send requests and responses on the web.

A request commonly contains:

- a method, such as `GET` or `POST`;
- a URL path;
- headers, such as `Accept` and `Cookie`; and
- optionally a body, such as form data or JSON.

A response commonly contains:

- a status code, such as `200`, `302`, `404`, or `500`;
- headers, such as `Content-Type` and `Cache-Control`; and
- a body, such as HTML, JSON, CSS, JavaScript, or an image.

## 4.6 HTTP methods

`GET` normally requests data and should not change server state.

`POST` normally sends data to create or perform an operation.

`PUT` normally replaces a resource.

`PATCH` normally partially updates a resource.

`DELETE` normally removes a resource.

The service-worker baseline in this course handles only safe `GET` requests. It does not intercept writes.

## 4.7 Status codes

A `2xx` status usually means the request succeeded. `200 OK` means a normal successful response.

A `3xx` status usually means redirection. `302 Found` commonly redirects a browser after a form submission.

A `4xx` status usually means the client request is invalid or unauthorized. `404 Not Found` means the requested resource was not found.

A `5xx` status usually means the server failed while processing the request.

For PWA files, a `200` response is not enough. The body must also be the expected file. If `/sw.js` returns an HTML error page with status `200`, the browser cannot execute it as JavaScript.

## 4.8 Origin

An **origin** is the combination of scheme, host, and port. These are different origins:

```text
https://example.com
http://example.com
https://example.com:5001
https://app.example.com
```

Service workers are normally same-origin. A worker served from another origin cannot control your MVC application.

## 4.9 HTML, CSS, and JavaScript

**HTML** describes the structure of a page: headings, links, forms, buttons, and sections.

**CSS** describes presentation: colors, spacing, layout, typography, and responsive behavior.

**JavaScript** adds behavior in the browser: event handlers, fetch requests, install prompts, and service-worker registration.

A PWA requires all three in most real applications, but the service worker is JavaScript and the manifest is JSON.

## 4.10 JSON

**JSON** means JavaScript Object Notation. It is text representing objects and arrays. A manifest is a JSON document.

JSON has strict syntax:

- object keys use double quotes;
- strings use double quotes;
- entries are separated by commas;
- comments are not allowed; and
- a trailing comma is not allowed in standard JSON.

## 4.11 Server-rendered HTML

In a server-rendered MVC application, the server executes controller and view code, produces HTML, and sends the final HTML to the browser.

The browser does not receive a C# Razor file. It receives the HTML produced by that file.

This is why the shared layout is the correct place to add the manifest link: the layout contributes the link to the final HTML of many pages.

## 4.12 MVC

**MVC** means **Model–View–Controller**.

A **Model** represents application data and rules.

A **View** presents data to the user. In ASP.NET Core MVC, views normally use the Razor view engine.

A **Controller** receives a request, coordinates work, and chooses a response, often a view.

Microsoft describes MVC as a separation of models, views, and controllers that helps keep responsibilities clear and applications testable. [7]

## 4.13 Route and action

A **route** is a rule that maps a URL to an endpoint.

An **action** is a public controller method that can handle a request.

For the conventional route:

```text
{controller=Home}/{action=Index}/{id?}
```

this URL:

```text
/Tasks/Index
```

usually maps to:

```text
TasksController.Index()
```

The `id?` portion means the `id` value is optional.

## 4.14 Razor

**Razor** is the view syntax used by ASP.NET Core MVC. A Razor view mixes HTML and C# expressions. A line beginning with `@` usually tells Razor that C# follows.

For example:

```cshtml
<h1>Hello, @Model.Title</h1>
```

The server replaces `@Model.Title` with the encoded value before sending the HTML to the browser.

## 4.15 Static file

A **static file** is served as it exists on disk instead of being generated by a controller on every request. CSS, JavaScript, images, icons, a manifest, and a service worker are static files.

ASP.NET Core normally serves public static files from `wwwroot`. [11]

## 4.16 Manifest

A **manifest** is the JSON description of the web application. It describes the app to the browser and operating system.

The manifest does not execute code and does not cache resources.

## 4.17 Service worker

A **service worker** is a JavaScript worker that can receive lifecycle events and selected fetch events. It is not a controller and it does not have access to the DOM.

The service worker can intercept a request, but it cannot query your server database by magic. It can call a network URL, use Cache Storage, or return a response that the worker constructs.

## 4.18 Cache

A **cache** is stored data that can be reused to avoid repeating work or network access.

There are multiple caches:

- the browser's normal HTTP cache;
- service-worker Cache Storage;
- server response or output caches; and
- CDN or reverse-proxy caches.

A PWA developer must know which cache is serving a response.

## 4.19 Scope

A service-worker **scope** is the URL path the worker can control. A worker at `/sw.js` can naturally control the root path `/`. A worker at `/js/sw.js` naturally controls `/js/` and below unless special headers expand its permitted scope.

For an MVC application, placing the worker at the public root is the simplest arrangement.

## 4.20 Secure context

A **secure context** is a context delivered through a trustworthy security boundary. HTTPS pages are secure contexts. Local loopback addresses such as `localhost` are treated as secure for development. Service workers require secure contexts. [6]

## 4.21 Installability

**Installability** is the browser's decision that a web application can be promoted through an installation flow. The exact criteria and user interface vary by browser and platform.

The manifest, HTTPS, icons, start URL, and display mode are important parts of the installability story, but installation is never a guarantee that every browser will show the same button. [3]

## 4.22 Progressive enhancement

**Progressive enhancement** means starting with a functional baseline and adding improvements when capabilities exist.

In this course:

- the MVC website works without a service worker;
- the manifest adds installation metadata when supported;
- the service worker adds offline behavior when supported; and
- optional features are feature-detected and have fallbacks.

---

# Chapter 5 — ASP.NET MVC fundamentals before PWA work

## 5.1 The MVC request lifecycle in one sentence

A browser sends a request, routing selects a controller action, model binding creates action parameters, the action coordinates application work, and the action returns a response such as a Razor view.

## 5.2 The MVC request lifecycle step by step

Suppose the browser requests:

```text
https://localhost:5001/Tasks/Index
```

The flow is:

1. The browser creates a `GET` request.
2. HTTPS carries the request to the development server.
3. ASP.NET Core receives the request.
4. Middleware processes the request in order.
5. Routing matches `Tasks/Index` to `TasksController.Index`.
6. The controller action executes.
7. The action obtains task data from a store.
8. The action returns a view result.
9. Razor combines the view, layout, and model data into HTML.
10. ASP.NET Core sends the HTML response.
11. The browser parses the HTML, requests CSS and JavaScript, and renders the page.
12. If a service worker controls the page, it may observe selected requests before the browser displays the result.

This explains why the PWA layer belongs at the boundary between the server's public static files and the browser's request handling.

## 5.3 The Model

The model represents data or application state. A model should not be confused with a database table in every case. A model can be:

- a domain object;
- a form input model;
- a view model designed for one screen; or
- a data-transfer object used by an API.

Use separate input and display models when that makes validation and security clearer. Do not bind sensitive fields from a browser just because a database entity contains them.

## 5.4 The View

A view is responsible for presentation. It should display data, create links and forms, and expose validation messages. Business rules should not be hidden in large blocks of Razor code.

Views are normally strongly typed. The `@model` declaration tells Razor which type the `Model` variable contains.

## 5.5 The Controller

A controller handles web interaction. It should be easy to understand what request it accepts and what response it returns.

A controller should not become a database, a template engine, and a service layer at the same time. Move reusable business work into services or domain objects.

## 5.6 Routing

A conventional route such as:

```csharp
pattern: "{controller=Home}/{action=Index}/{id?}"
```

means:

- use `Home` when no controller is specified;
- use `Index` when no action is specified; and
- allow an optional `id` value.

PWA files such as `/sw.js` and `/offline.html` should be served as static files. They should not depend on a controller route.

## 5.7 Model binding

Model binding converts request data into C# parameters or objects. If a form contains an input named `Title`, MVC can bind it to a property called `Title` in a model parameter.

Model binding is convenient, but it does not remove the need for validation. A malicious client can submit a request without using your form.

## 5.8 Model validation

Validation attributes such as `[Required]` and `[StringLength]` express constraints. The server must validate the model even if client-side validation also runs.

The PWA does not change this rule. An offline client may be stale or manipulated; the server remains the authority when the request arrives.

## 5.9 Dependency injection

Dependency injection, or **DI**, means that a class receives the services it needs instead of constructing them internally.

For example, `TasksController` receives `ITaskStore` in its constructor. The application configures which concrete store implements that interface.

DI makes the controller easier to test and makes it possible to replace an in-memory store with a database-backed store later.

---

# Chapter 6 — Prepare your computer and choose the starting point

## 6.1 Tools you need

For the ASP.NET Core path, install:

- a supported .NET SDK;
- an editor such as Visual Studio, Visual Studio Code, or another C# editor;
- a modern browser with developer tools; and
- Git if you want version control.

For Visual Studio, select the **ASP.NET and web development** workload. The Microsoft MVC tutorial also uses Visual Studio or Visual Studio Code as the development environment. [8]

For the MVC 5 path, you need Visual Studio with the .NET Framework web-development components and IIS Express or IIS for testing.

## 6.2 Check the .NET SDK

Open PowerShell and run:

```powershell
dotnet --info
dotnet --list-sdks
```

Read the output:

- `dotnet` is the command-line driver.
- `--info` asks for environment details.
- `--list-sdks` lists installed SDK versions.

If the command is not found, install the .NET SDK, not only the runtime. The SDK contains compilers, project templates, restore tooling, and build commands.

## 6.3 Check installed MVC templates

Run:

```powershell
dotnet new list mvc
```

`dotnet new` creates projects and other artifacts from templates. The `mvc` template creates an ASP.NET Core MVC project. The official command supports options such as project name, framework, and output directory. [9]

## 6.4 HTTPS is required for real PWA work

A service worker needs a secure context. Localhost is a permitted development exception, but you still need to run the application through the web server over an HTTPS URL.

Create or trust the .NET development certificate if necessary:

```powershell
dotnet dev-certs https --check --trust
```

If the certificate is missing or untrusted, run:

```powershell
dotnet dev-certs https --trust
```

The `dotnet dev-certs https` command manages a self-signed certificate for local HTTPS development. The `--trust` option trusts it on the local machine. [10]

Do not use `file:///` to open `manifest.webmanifest`, `sw.js`, or an HTML page. The browser must receive these files through the application origin.

## 6.5 Choose the application style

This guide uses:

- C#;
- ASP.NET Core MVC;
- conventional routing;
- Razor views;
- no authentication in the teaching project;
- an in-memory store during the beginner chapters; and
- a conservative service worker.

In a real application, authentication and authorization may already exist. That is exactly why cache design must be explicit. A service worker should not cache private account content merely because it was easy to cache.

---

# Chapter 7 — Create the MVC application from an empty folder

## 7.1 Create the project with the .NET CLI

Choose a parent folder and run:

```powershell
mkdir PwaMvcCourse
cd PwaMvcCourse
dotnet new mvc --name PwaMvcCourse --framework net10.0 --auth None
```

If you are using .NET 8 or .NET 9, replace `net10.0` with `net8.0` or `net9.0`.

### Word-by-word explanation

`mkdir PwaMvcCourse` asks PowerShell to create a directory named `PwaMvcCourse`.

`cd PwaMvcCourse` changes the terminal's current directory. Commands that follow operate from this directory.

`dotnet` invokes the .NET command-line tool.

`new` tells the tool to create something from a template.

`mvc` selects the ASP.NET Core MVC project template.

`--name PwaMvcCourse` gives the generated project and default namespace a name.

`--framework net10.0` selects the target framework moniker. A target framework tells the compiler and runtime which .NET API surface the project targets.

`--auth None` tells the template not to add an authentication system. This keeps the first project small. It does not mean a production application should be unauthenticated.

The template engine creates a working project. Microsoft documents the same MVC template approach in its beginner tutorial. [8] [9]

## 7.2 Inspect the generated files

Run:

```powershell
Get-ChildItem -Recurse -File | Select-Object FullName
```

You should see files similar to:

```text
PwaMvcCourse.csproj
Program.cs
Controllers/HomeController.cs
Models/ErrorViewModel.cs
Views/Home/Index.cshtml
Views/Shared/_Layout.cshtml
Views/Shared/_ValidationScriptsPartial.cshtml
wwwroot/css/site.css
wwwroot/js/site.js
wwwroot/lib/...
```

The exact files can vary slightly with the SDK template version.

## 7.3 Read the project file

Open `PwaMvcCourse.csproj`. A simplified project file resembles:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

Line by line:

- `<Project ...>` begins the MSBuild project document.
- `Sdk="Microsoft.NET.Sdk.Web"` selects the SDK behavior for an ASP.NET Core web project.
- `<PropertyGroup>` groups build properties.
- `<TargetFramework>` tells the SDK which .NET target to compile for.
- `<Nullable>enable</Nullable>` asks the compiler to help detect possible null-reference mistakes.
- `<ImplicitUsings>enable</ImplicitUsings>` automatically imports common namespaces.
- `</Project>` closes the project document.

The project file is not the PWA manifest. It describes how .NET builds the server application. The PWA manifest is a browser-facing JSON file served as a public static asset.

## 7.4 Read `Program.cs`

The modern ASP.NET Core template uses a compact startup file. A simplified version is:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();

app.MapStaticAssets();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

Line by line:

- `var builder = WebApplication.CreateBuilder(args);` creates an application builder. `args` carries command-line arguments into the application.
- `builder.Services` is the dependency-injection service collection.
- `AddControllersWithViews()` registers MVC controllers and Razor views.
- `var app = builder.Build();` turns the configuration into a runnable application.
- `if (!app.Environment.IsDevelopment())` checks whether the app is running outside Development.
- `UseExceptionHandler("/Home/Error")` sends unexpected production errors to an error action rather than exposing implementation details.
- `UseHsts()` asks browsers to remember that the site should use HTTPS. Enable this only when HTTPS is correctly configured.
- `UseHttpsRedirection()` redirects HTTP requests to HTTPS.
- `UseRouting()` enables endpoint routing.
- `UseAuthorization()` applies authorization behavior after routing has selected an endpoint.
- `MapStaticAssets()` maps public static assets in versions that support the static-assets endpoint conventions.
- `MapControllerRoute(...)` creates the conventional MVC route.
- `name: "default"` gives the route a name.
- `pattern: "{controller=Home}/{action=Index}/{id?}"` defines the URL pattern and defaults.
- `app.Run()` starts the server and waits for requests.

For a target where `MapStaticAssets()` is unavailable, use `app.UseStaticFiles()` before routing:

```csharp
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

ASP.NET Core uses `wwwroot` as the default public web root. Static assets should be separated from views, configuration, and private files. [11]

## 7.5 Run the empty application

Run:

```powershell
dotnet run
```

The output contains an HTTPS URL such as:

```text
Now listening on: https://localhost:5001
```

Open that exact HTTPS URL. If the browser warns about a development certificate, trust the certificate only if you understand that it is your local development certificate.

At this point you have an ordinary MVC website. You have not created a PWA yet. That distinction is important: build a functioning website first, then add progressive capabilities.

---

# Chapter 8 — Build a small MVC feature before adding PWA behavior

A PWA is more meaningful when it improves a real user journey. We will create a task board before adding installation or offline behavior.

## 8.1 Create the task model

Create `Models/TaskItem.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace PwaMvcCourse.Models;

public sealed class TaskItem
{
    public int Id { get; set; }

    [Required]
    [StringLength(120)]
    public string Title { get; set; } = string.Empty;

    public bool IsDone { get; set; }
}
```

Line by line:

- `using System.ComponentModel.DataAnnotations;` imports validation attributes such as `Required` and `StringLength`.
- `namespace PwaMvcCourse.Models;` places the type in the Models namespace. The semicolon form is a file-scoped namespace.
- `public sealed class TaskItem` declares a public model class. `sealed` prevents inheritance in this teaching example.
- `public int Id { get; set; }` creates an integer identifier with a getter and setter.
- `[Required]` says that `Title` must contain a value during validation.
- `[StringLength(120)]` limits the title length to 120 characters.
- `public string Title { get; set; } = string.Empty;` declares the title and gives it a non-null default value.
- `public bool IsDone { get; set; }` stores whether the task is completed.

The attributes help MVC validate input, but server validation is still required. A browser can send a request that did not come from your generated form.

## 8.2 Create a page view model

Create `Models/TaskPageViewModel.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace PwaMvcCourse.Models;

public sealed class TaskPageViewModel
{
    public IReadOnlyList<TaskItem> Items { get; set; } = [];

    [Required]
    [StringLength(120)]
    public string NewTitle { get; set; } = string.Empty;
}
```

Line by line:

- The first two lines import validation and define the namespace, just as before.
- `public sealed class TaskPageViewModel` defines a model designed for one page rather than for database storage.
- `Items` contains the tasks to display. `IReadOnlyList` tells the view that it should read the list, not replace it.
- `= []` creates an empty collection by default in modern C#.
- `NewTitle` is the input for the new-task form.
- The validation attributes apply to the submitted form field.

Separating `TaskItem` from `TaskPageViewModel` is useful. The page needs both a list of existing tasks and one input field. A database entity alone is not always the best view model.

## 8.3 Create a store interface

Create `Services/ITaskStore.cs`:

```csharp
using PwaMvcCourse.Models;

namespace PwaMvcCourse.Services;

public interface ITaskStore
{
    IReadOnlyList<TaskItem> GetAll();
    void Add(string title);
}
```

Line by line:

- `using PwaMvcCourse.Models;` lets this file refer to `TaskItem`.
- The namespace identifies service-layer types.
- `public interface ITaskStore` defines a contract rather than an implementation.
- `GetAll()` promises a read operation.
- `Add(string title)` promises a create operation.

The controller will depend on this interface. Later, a database implementation can satisfy the same contract.

## 8.4 Create an in-memory store

Create `Services/InMemoryTaskStore.cs`:

```csharp
using PwaMvcCourse.Models;

namespace PwaMvcCourse.Services;

public sealed class InMemoryTaskStore : ITaskStore
{
    private readonly object _sync = new();
    private readonly List<TaskItem> _items =
    [
        new TaskItem { Id = 1, Title = "Read the MVC request lifecycle" },
        new TaskItem { Id = 2, Title = "Add the PWA manifest" }
    ];

    private int _nextId = 3;

    public IReadOnlyList<TaskItem> GetAll()
    {
        lock (_sync)
        {
            return _items.ToArray();
        }
    }

    public void Add(string title)
    {
        lock (_sync)
        {
            _items.Add(new TaskItem
            {
                Id = _nextId++,
                Title = title.Trim()
            });
        }
    }
}
```

Line by line:

- The `using` directive imports the model.
- `InMemoryTaskStore : ITaskStore` says this class implements the interface contract.
- `_sync` is an object used to protect the list when multiple requests access it at the same time.
- `_items` is a private list. It exists only in server memory.
- The collection expression creates two starting tasks.
- `_nextId` tracks the next identifier for the demo.
- `GetAll()` returns a read-only view of a copied array. The copy prevents callers from modifying the private list directly.
- `lock (_sync)` allows one thread at a time to enter the critical section.
- `Add` trims whitespace and creates a new `TaskItem`.
- `_nextId++` uses the current value and then increments it.

This store is for learning. Data disappears when the process restarts, and multiple server instances would have separate lists. Do not use it as a production database.

## 8.5 Register the store in `Program.cs`

Add this line after `AddControllersWithViews()`:

```csharp
builder.Services.AddSingleton<ITaskStore, InMemoryTaskStore>();
```

Explain the line:

- `builder.Services` is the dependency-injection container configuration.
- `AddSingleton` creates one store instance for the lifetime of the application process.
- `ITaskStore` is the type controllers request.
- `InMemoryTaskStore` is the concrete type the container constructs.

A singleton is appropriate for this small in-memory demonstration because the list must be shared between requests. In a real application, use a database or another durable store and choose a service lifetime appropriate to its dependencies.

## 8.6 Create the controller

Create `Controllers/TasksController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using PwaMvcCourse.Models;
using PwaMvcCourse.Services;

namespace PwaMvcCourse.Controllers;

public sealed class TasksController : Controller
{
    private readonly ITaskStore _store;

    public TasksController(ITaskStore store)
    {
        _store = store;
    }

    [HttpGet]
    public IActionResult Index()
    {
        var model = new TaskPageViewModel
        {
            Items = _store.GetAll()
        };

        return View(model);
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(TaskPageViewModel model)
    {
        if (!ModelState.IsValid)
        {
            model.Items = _store.GetAll();
            return View("Index", model);
        }

        _store.Add(model.NewTitle);
        return RedirectToAction(nameof(Index));
    }
}
```

Line by line:

- `using Microsoft.AspNetCore.Mvc;` imports MVC controller types and attributes.
- The next two `using` lines import our models and services.
- `TasksController : Controller` creates a controller with MVC behavior.
- `_store` holds the dependency received through the constructor.
- The constructor parameter `ITaskStore store` is supplied by dependency injection.
- `[HttpGet]` restricts `Index` to GET requests.
- `Index()` creates the page model and fills `Items` from the store.
- `View(model)` asks MVC to render the default view for this action with the model.
- `[HttpPost]` restricts `Create` to POST requests.
- `[ValidateAntiForgeryToken]` protects the form against cross-site request forgery when the form includes the matching token.
- `Create(TaskPageViewModel model)` lets model binding create the page model from submitted fields.
- `ModelState.IsValid` checks the server-side validation results.
- If validation fails, the action reloads the current items and returns the same view so validation messages can be displayed.
- `_store.Add(model.NewTitle)` writes the validated title.
- `RedirectToAction(nameof(Index))` uses the Post/Redirect/Get pattern. The browser follows a redirect with a GET instead of reposting the form on refresh.

The Post/Redirect/Get pattern is especially important in an application that may later be installed. It reduces accidental duplicate submissions when a user reloads a page.

## 8.7 Create the view

Create the directory `Views/Tasks` and file `Views/Tasks/Index.cshtml`:

```cshtml
@model PwaMvcCourse.Models.TaskPageViewModel

@{
    ViewData["Title"] = "Tasks";
}

<h1>Tasks</h1>
<p>These tasks are stored by the MVC server.</p>

<form asp-controller="Tasks" asp-action="Create" method="post">
    @Html.AntiForgeryToken()

    <label asp-for="NewTitle">New task</label>
    <input asp-for="NewTitle" autocomplete="off" />
    <span asp-validation-for="NewTitle"></span>
    <button type="submit">Add task</button>
</form>

<ul>
@foreach (var item in Model.Items)
{
    <li>
        <strong>@item.Title</strong>
        <span>(completed: @item.IsDone)</span>
    </li>
}
</ul>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

Line by line:

- `@model ...` declares the view's strongly typed model.
- The Razor block sets the page title used by the layout.
- `<h1>` creates the main heading.
- The paragraph explains where the data comes from.
- `<form ...>` creates a POST form targeting `TasksController.Create`.
- `@Html.AntiForgeryToken()` emits a hidden security token that the server validates.
- `label asp-for` creates a label connected to `NewTitle`.
- `input asp-for` creates an input whose name and validation metadata match `NewTitle`.
- `autocomplete="off"` is a small UX choice for the teaching form; choose autocomplete behavior deliberately in production.
- `span asp-validation-for` displays validation errors for the property.
- The submit button sends the form.
- `@foreach` loops through the model's items on the server.
- `@item.Title` is HTML-encoded by Razor before output.
- `@item.IsDone` displays the completion state.
- The `Scripts` section lets the layout render client-side validation scripts if the template supports them.

## 8.8 Add a link to the task board

Open `Views/Shared/_Layout.cshtml` and add a navigation link inside the navigation element:

```cshtml
<a asp-controller="Tasks" asp-action="Index">Tasks</a>
```

The `asp-controller` and `asp-action` attributes are Tag Helpers. They generate a URL from MVC routing instead of forcing you to hard-code a path.

## 8.9 Run the feature

Run:

```powershell
dotnet run
```

Open:

```text
https://localhost:5001/Tasks/Index
```

Test these behaviors:

1. The two starter tasks appear.
2. Submitting an empty title shows a validation error.
3. Submitting a valid title adds a task.
4. Refreshing the page does not submit the same form again.
5. Restarting the application removes the in-memory tasks you added.

At the end of this chapter, you have a normal MVC application that is worth progressively enhancing.

---

# Chapter 9 — Plan the PWA layer before writing browser code

## 9.1 Define the first release

For the first release, we will provide:

- a manifest;
- icons;
- an HTTPS development and production requirement;
- a root-scoped service worker;
- a self-contained offline page;
- runtime caching for safe static assets; and
- a network-first navigation strategy that falls back to the offline page.

We will **not** cache authenticated task pages or replay POST requests offline in the baseline.

## 9.2 Draw the request categories

Before writing a service worker for your real application, write down the request categories. For `PwaMvcCourse`:

- `/pwa/manifest.webmanifest`: public static metadata; can be precached.
- `/pwa/icons/...`: public static images; can be precached or runtime cached.
- `/css/...`: public static style; can be runtime cached.
- `/js/...`: public static behavior; can be runtime cached.
- `/offline.html`: public fallback; should be precached.
- `/Tasks/Index`: server-rendered HTML; network first in the baseline.
- `/Tasks/Create`: POST mutation; never handled by the baseline cache.
- `/Account/...`: not present in the teaching project, but would be private in a real app.
- `/api/...`: data endpoint; should be network-only until a deliberate data strategy exists.

This list is a cache policy. It is more important than the JavaScript syntax.

## 9.3 Choose the offline promise

A user asking for “offline support” may mean several different things:

- “Show a friendly offline screen.”
- “Open the last public page.”
- “Read my previously downloaded data.”
- “Create records without a connection.”
- “Synchronize edits later.”

These are different products. The baseline in this guide promises the first and provides a safe foundation for the second. It does not promise offline task creation.

## 9.4 Decide what must remain online

Authentication, authorization, database validation, financial calculations, and business rules should remain authoritative on the server.

If a task is created offline in a future version, the server must still validate the user, tenant, operation ID, current version, business constraints, and conflict behavior when synchronization occurs.

## 9.5 Public file layout for the PWA layer

For ASP.NET Core, create:

```text
wwwroot/
├─ sw.js
├─ offline.html
├─ pwa/
│  ├─ manifest.webmanifest
│  └─ icons/
│     ├─ icon-192.png
│     ├─ icon-512.png
│     ├─ icon-192-maskable.png
│     ├─ icon-512-maskable.png
│     └─ apple-touch-icon.png
└─ js/
   └─ pwa-register.js
```

Run:

```powershell
New-Item -ItemType Directory -Force `
  wwwroot\pwa\icons, `
  wwwroot\js | Out-Null
```

The PWA files must be public because the browser needs to request them. They must not contain secrets.

---

# Chapter 10 — Add the manifest and icons

## 10.1 Create the icon files

Use real PNG files. At minimum, provide 192×192 and 512×512 icons for Chromium-based installability checks. A maskable icon should have enough safe padding that its important content survives operating-system cropping. [3]

Place them here:

```text
wwwroot/pwa/icons/icon-192.png
wwwroot/pwa/icons/icon-512.png
wwwroot/pwa/icons/icon-192-maskable.png
wwwroot/pwa/icons/icon-512-maskable.png
wwwroot/pwa/icons/apple-touch-icon.png
```

Do not point the manifest at a file on your development computer. The `src` values must be URLs that the deployed application can return.

## 10.2 Create `manifest.webmanifest`

Create `wwwroot/pwa/manifest.webmanifest`:

```json
{
  "id": "/",
  "name": "PwaMvcCourse Task Board",
  "short_name": "Task Board",
  "description": "A task board used to learn Progressive Web Apps with ASP.NET MVC.",
  "start_url": "/Tasks/Index",
  "scope": "/",
  "display_override": [
    "standalone",
    "browser"
  ],
  "display": "standalone",
  "orientation": "any",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "icons": [
    {
      "src": "/pwa/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/pwa/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/pwa/icons/icon-192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/pwa/icons/icon-512-maskable.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "shortcuts": [
    {
      "name": "Open tasks",
      "short_name": "Tasks",
      "url": "/Tasks/Index",
      "icons": [
        {
          "src": "/pwa/icons/icon-192.png",
          "sizes": "192x192",
          "type": "image/png"
        }
      ]
    }
  ]
}
```

## 10.3 Explain the manifest line by line

`{` starts the JSON object.

`"id": "/"` gives the installed application a stable identifier. Keep it stable unless you intentionally want the browser to treat the app as a different installed application.

`"name"` is the full application name.

`"short_name"` is the compact name used where space is limited.

`"description"` describes the purpose of the application.

`"start_url": "/Tasks/Index"` tells the installed app where to start. It is a normal GET URL, not a form action.

`"scope": "/"` says that URLs under the site root belong to the app experience.

`"display_override"` gives a browser an ordered set of preferred display modes when supported.

`"display": "standalone"` asks for an app-like window without ordinary browser chrome. A browser can fall back when it does not support the mode.

`"orientation": "any"` allows the device to choose orientation.

`"background_color"` helps define launch or splash presentation on supporting platforms.

`"theme_color"` can influence browser and operating-system UI colors.

`"icons"` is an array of icon descriptions.

Each icon's `src` is the public URL of an image, `sizes` describes its dimensions, `type` describes its MIME type, and `purpose` tells the browser how the icon is intended to be used.

`"shortcuts"` is optional operating-system integration. It provides a safe link to a normal GET page. It is not required for the baseline.

The manifest is JSON. Do not add comments, use single quotes, or leave trailing commas.

## 10.4 Link the manifest in the layout

Open `Views/Shared/_Layout.cshtml`. Add this inside `<head>`:

```cshtml
@{
    var manifestUrl = Url.Content("~/pwa/manifest.webmanifest");
    var serviceWorkerUrl = Url.Content("~/sw.js");
    var appleIconUrl = Url.Content("~/pwa/icons/apple-touch-icon.png");
}

<link rel="manifest" href="@manifestUrl" />
<link rel="apple-touch-icon" sizes="180x180" href="@appleIconUrl" />
<link rel="icon" href="@Url.Content("~/favicon.ico")" />
<meta name="theme-color" content="#2563eb" />
<meta name="mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<meta name="apple-mobile-web-app-title" content="Task Board" />
<meta name="pwa-service-worker" content="@serviceWorkerUrl" />
```

Line by line:

- The Razor block computes application-relative URLs.
- `Url.Content("~/...")` turns `~/` into the application's public base path.
- The manifest `<link>` tells the browser where the manifest is.
- The Apple touch icon is a platform-specific fallback.
- The favicon is the normal browser-tab icon.
- `theme-color` provides a browser UI color hint.
- The mobile metadata supports platforms that use these legacy or platform-specific hints.
- The custom `pwa-service-worker` meta tag gives the registration script a path without hard-coding the deployment root.

The manifest should be linked from the shared layout so every page that uses the layout exposes it. A page that does not use the layout must link the manifest separately if it is part of the installable application. [3] [4]

## 10.5 Confirm the manifest response

Run the application and open:

```text
https://localhost:5001/pwa/manifest.webmanifest
```

Then use:

```powershell
curl.exe -I https://localhost:5001/pwa/manifest.webmanifest
```

Check:

- status is `200 OK`;
- the response is JSON, not an HTML error page;
- the icon URLs are reachable; and
- the content type is preferably `application/manifest+json`.

If the file returns 404, check that it is under `wwwroot` and that static-file delivery is enabled.

---

# Chapter 11 — Add a self-contained offline page

## 11.1 Why start with an offline page?

A complete offline application is complex. A self-contained offline page is a small, safe first promise.

It gives the user a clear answer when the server cannot be reached. It does not pretend that private data is available offline and does not encourage you to cache every MVC page immediately.

## 11.2 Create `offline.html`

Create `wwwroot/offline.html`:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#2563eb">
  <title>Offline | Task Board</title>
  <style>
    :root {
      color-scheme: light dark;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    body {
      display: grid;
      min-height: 100vh;
      place-items: center;
      margin: 0;
      padding: 1.5rem;
      background: #eff6ff;
      color: #172033;
    }

    main {
      width: min(100%, 38rem);
      padding: 2rem;
      border-radius: 1rem;
      background: #ffffff;
      box-shadow: 0 0.75rem 2rem rgb(23 32 51 / 12%);
      text-align: center;
    }

    button {
      padding: 0.75rem 1rem;
      border: 0;
      border-radius: 0.5rem;
      background: #2563eb;
      color: #ffffff;
      font: inherit;
      cursor: pointer;
    }

    @media (prefers-color-scheme: dark) {
      body {
        background: #111827;
        color: #e5e7eb;
      }

      main {
        background: #1f2937;
      }
    }
  </style>
</head>
<body>
  <main>
    <h1>You are offline</h1>
    <p>The server could not be reached. Check your connection and try again.</p>
    <button type="button" onclick="location.reload()">Try again</button>
  </main>
  <script>
    window.addEventListener("online", function () {
      location.reload();
    });
  </script>
</body>
</html>
```

## 11.3 Explain the offline page line by line

`<!doctype html>` tells the browser to use modern HTML parsing.

`<html lang="en">` begins the document and identifies its language for accessibility tools.

`<head>` contains metadata and resources that describe the page.

`charset="utf-8"` selects a Unicode-compatible character encoding.

`viewport` tells mobile browsers how to size the page.

`theme-color` provides an optional browser UI color.

The `<title>` appears in a browser tab or standalone window title.

The `<style>` element contains CSS. The page uses inline CSS so that it has no dependency on a stylesheet that might not be cached.

`color-scheme: light dark` tells the browser that the page can adapt to light and dark preferences.

`body` uses grid layout to center the `main` element.

`min-height: 100vh` makes the body at least as tall as the viewport.

`padding` prevents the card from touching the screen edge.

`main` is the content card.

The `@media` rule changes colors when the operating system prefers dark mode.

The `<h1>` gives the page a clear primary heading.

The paragraph explains the condition without claiming that the application data is available.

The button calls `location.reload()` when clicked. Reloading lets the service worker retry the original navigation.

The `online` event fires when the browser believes connectivity has returned. The handler reloads the page so the MVC server can respond again.

## 11.4 What this page does not do

It does not:

- display a user's private task list;
- create a task while offline;
- validate authentication offline;
- guarantee that the network is actually usable; or
- replace conflict-resolution design.

It is intentionally simple and safe.

---

# Chapter 12 — Register the service worker from MVC

## 12.1 What registration means

Registration tells the browser to fetch a JavaScript file and install it as a service worker for a scope.

The page that registers the worker and the worker script must be same-origin. The script must be served through HTTPS or a permitted local origin. The path determines the natural scope. [5] [6]

## 12.2 Add the install and update buttons

In `_Layout.cshtml`, add these controls somewhere inside `<body>`:

```cshtml
<div class="pwa-actions" aria-live="polite">
    <button type="button" data-pwa-install hidden>
        Install this app
    </button>
    <button type="button" data-pwa-update hidden>
        Update available
    </button>
</div>
```

The `hidden` attribute keeps the controls invisible until JavaScript has a reason to show them.

`aria-live="polite"` tells assistive technology that changes in this region can be announced without interrupting the user.

The `data-pwa-install` and `data-pwa-update` attributes are hooks for JavaScript. They are not standard HTML elements; they are custom data attributes.

## 12.3 Create `pwa-register.js`

Create `wwwroot/js/pwa-register.js`:

```javascript
(() => {
  "use strict";

  const workerUrl = document.querySelector("meta[name='pwa-service-worker']")?.content;
  const installButton = document.querySelector("[data-pwa-install]");
  const updateButton = document.querySelector("[data-pwa-update]");

  let deferredInstallPrompt = null;
  let waitingWorker = null;
  let isRefreshing = false;

  if (installButton) {
    window.addEventListener("beforeinstallprompt", (event) => {
      event.preventDefault();
      deferredInstallPrompt = event;
      installButton.hidden = false;
    });

    installButton.addEventListener("click", async () => {
      if (!deferredInstallPrompt) return;

      deferredInstallPrompt.prompt();
      await deferredInstallPrompt.userChoice;
      deferredInstallPrompt = null;
      installButton.hidden = true;
    });

    window.addEventListener("appinstalled", () => {
      deferredInstallPrompt = null;
      installButton.hidden = true;
    });
  }

  if (!workerUrl || !("serviceWorker" in navigator)) {
    return;
  }

  const showUpdate = (worker) => {
    waitingWorker = worker;

    if (!updateButton) return;

    updateButton.hidden = false;
    updateButton.addEventListener("click", () => {
      waitingWorker?.postMessage({ type: "SKIP_WAITING" });
      updateButton.disabled = true;
    }, { once: true });
  };

  const register = async () => {
    try {
      const worker = new URL(workerUrl, window.location.href);
      const scope = new URL("./", worker).pathname;
      const registration = await navigator.serviceWorker.register(worker, { scope });

      if (registration.waiting) {
        showUpdate(registration.waiting);
      }

      registration.addEventListener("updatefound", () => {
        const installingWorker = registration.installing;
        if (!installingWorker) return;

        installingWorker.addEventListener("statechange", () => {
          if (
            installingWorker.state === "installed" &&
            navigator.serviceWorker.controller
          ) {
            showUpdate(installingWorker);
          }
        });
      });

      await registration.update();
    } catch (error) {
      console.error("Service-worker registration failed:", error);
    }
  };

  navigator.serviceWorker.addEventListener("controllerchange", () => {
    if (isRefreshing) return;

    isRefreshing = true;
    window.location.reload();
  });

  window.addEventListener("load", register, { once: true });
})();
```

## 12.4 Explain the registration file line by line

`(() => {` starts an immediately invoked function expression. It creates a private scope so variables do not become accidental global variables.

`"use strict";` asks JavaScript to apply stricter parsing and runtime rules.

`document.querySelector(...)` finds the meta tag that the Razor layout rendered.

`?.content` is optional chaining. If the meta element does not exist, the expression becomes `undefined` instead of throwing immediately.

The next two constants find the install and update buttons.

`deferredInstallPrompt` stores the browser's install event until the user clicks the application button.

`waitingWorker` stores a new service worker that has finished installing but is waiting to become active.

`isRefreshing` prevents multiple `controllerchange` events from causing repeated reloads.

The `if (installButton)` block runs only when the layout contains an install button.

`beforeinstallprompt` is a browser event. The script calls `preventDefault()` so the browser does not immediately show its own prompt. It stores the event and reveals the custom button.

The click handler calls `prompt()` only after a user action. It waits for `userChoice`, then clears the event and hides the button.

The `appinstalled` event hides the button after successful installation.

The feature-detection guard returns when there is no worker URL or no service-worker API. This is progressive enhancement: the normal MVC site continues to work.

`showUpdate` receives a waiting worker and reveals the update button. The button sends a message to the waiting worker asking it to skip the waiting phase. The `{ once: true }` option removes the click listener after the first click.

`register` is an asynchronous function because registration and update checks return promises.

`new URL(workerUrl, window.location.href)` resolves an application-relative URL against the current page.

`new URL("./", worker).pathname` calculates the directory scope. If the app is deployed under `/BackOffice/`, this calculation can produce `/BackOffice/` instead of incorrectly forcing `/`.

`navigator.serviceWorker.register(worker, { scope })` asks the browser to register the script and scope.

`registration.waiting` indicates that an updated worker is waiting to control pages.

`updatefound` fires when the browser finds a new worker.

`registration.installing` is the new worker while it moves through its lifecycle.

`statechange` observes lifecycle changes.

The `installed` and `navigator.serviceWorker.controller` check distinguishes a first installation from a later update. A first installation does not need to show an update button.

`registration.update()` asks the browser to check the worker script. HTTP cache headers still influence when the browser can discover a changed script, so production headers matter.

The `catch` block logs registration failures without breaking the website.

`controllerchange` fires when a new worker takes control. The page reloads once so old and new application assets are not mixed.

The final line waits until the page has loaded before registering. The service worker does not need to block the initial HTML response.

## 12.5 Add the script to the layout

Near the end of `_Layout.cshtml`, add:

```cshtml
<script src="@Url.Content("~/js/pwa-register.js")" defer></script>
```

`src` points to the public JavaScript file.

`Url.Content("~/...")` respects the application base path.

`defer` lets the browser download the script without blocking HTML parsing and runs it after the document has been parsed.

---

# Chapter 13 — Write and understand the service worker

## 13.1 The service-worker lifecycle

A service worker moves through stages:

1. **Registration:** the page asks the browser to register the script.
2. **Installation:** the browser downloads and evaluates the script; the `install` event can populate caches.
3. **Waiting:** an updated worker may wait while an older worker controls existing pages.
4. **Activation:** the `activate` event can delete old caches and claim clients.
5. **Control:** the worker receives fetch events for pages within its scope.

The first service worker may not control a page that was already open. `clients.claim()` can help it claim clients after activation, but a reload is still a normal part of service-worker development. [5]

## 13.2 Create `sw.js`

Create `wwwroot/sw.js`:

```javascript
const CACHE_PREFIX = "pwa-mvc-course";
const STATIC_CACHE = `${CACHE_PREFIX}-static-v1`;
const RUNTIME_CACHE = `${CACHE_PREFIX}-runtime-v1`;

const appRoot = new URL("./", self.registration.scope).pathname;
const pathFromRoot = (relativePath) =>
  `${appRoot}${relativePath.replace(/^\/+/, "")}`;

const OFFLINE_URL = pathFromRoot("offline.html");
const PRECACHE_URLS = [
  OFFLINE_URL,
  pathFromRoot("pwa/manifest.webmanifest")
];

const PRIVATE_ROUTE_NAMES = [
  "account",
  "manage",
  "signin",
  "signup",
  "logout",
  "api"
];

const STATIC_ASSET_PATTERN =
  /\.(?:css|js|mjs|png|jpg|jpeg|gif|svg|webp|ico|woff2?|ttf|otf)$/i;

function isSameOrigin(url) {
  return url.origin === self.location.origin;
}

function isPrivateRoute(pathname) {
  const normalized = pathname.toLowerCase();
  const root = appRoot.toLowerCase();

  if (!normalized.startsWith(root)) return true;

  const relative = normalized.slice(root.length).replace(/^\/+/, "");
  return PRIVATE_ROUTE_NAMES.some((name) =>
    relative === name || relative.startsWith(`${name}/`)
  );
}

function isCacheableStaticRequest(request, url) {
  return (
    request.method === "GET" &&
    isSameOrigin(url) &&
    !isPrivateRoute(url.pathname) &&
    !request.headers.has("authorization") &&
    STATIC_ASSET_PATTERN.test(url.pathname)
  );
}

self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE).then((cache) => cache.addAll(PRECACHE_URLS))
  );
});

self.addEventListener("activate", (event) => {
  const currentCaches = new Set([STATIC_CACHE, RUNTIME_CACHE]);

  event.waitUntil(
    caches.keys().then((cacheNames) =>
      Promise.all(
        cacheNames
          .filter(
            (cacheName) =>
              cacheName.startsWith(`${CACHE_PREFIX}-`) &&
              !currentCaches.has(cacheName)
          )
          .map((cacheName) => caches.delete(cacheName))
      )
    ).then(() => self.clients.claim())
  );
});

self.addEventListener("message", (event) => {
  if (event.data?.type === "SKIP_WAITING") {
    self.skipWaiting();
  }
});

async function networkFirstNavigation(request) {
  try {
    return await fetch(request);
  } catch {
    const cachedPage = await caches.match(request);
    return cachedPage || caches.match(OFFLINE_URL);
  }
}

async function cacheFirstStaticAsset(request) {
  const cached = await caches.match(request);
  if (cached) return cached;

  const response = await fetch(request);
  if (response.ok) {
    const cache = await caches.open(RUNTIME_CACHE);
    await cache.put(request, response.clone());
  }

  return response;
}

self.addEventListener("fetch", (event) => {
  const request = event.request;
  const url = new URL(request.url);

  if (!isSameOrigin(url)) return;

  if (request.mode === "navigate" && request.method === "GET") {
    event.respondWith(networkFirstNavigation(request));
    return;
  }

  if (isCacheableStaticRequest(request, url)) {
    event.respondWith(cacheFirstStaticAsset(request));
  }
});
```

## 13.3 Explain the service worker line by line

### Constants and paths

`const CACHE_PREFIX = "pwa-mvc-course";` defines a readable name prefix for this application's caches.

`STATIC_CACHE` names the cache for resources that are known during installation.

`RUNTIME_CACHE` names the cache for resources learned during normal browsing.

The `-v1` suffix is a manual version. Change it when the worker's caching contract changes.

`self` is the service-worker global object. It is not the page's `window` object.

`self.registration.scope` is the active registration scope.

`new URL("./", self.registration.scope).pathname` computes the path root of the registration. This makes the worker more tolerant of an application deployed under a virtual directory.

`pathFromRoot` appends a relative public path to the calculated application root.

`OFFLINE_URL` becomes `/offline.html` at the domain root or `/BackOffice/offline.html` under a virtual directory.

`PRECACHE_URLS` lists files that must be available after installation. The list is intentionally small.

### Private route rules

`PRIVATE_ROUTE_NAMES` is an application-specific deny list. Add your real private route prefixes here.

This list is not a replacement for security. It is a cache safety rule. The server must still enforce authorization.

`STATIC_ASSET_PATTERN` identifies common static-file extensions. It does not match MVC HTML pages, so those pages are not runtime-cached by the asset strategy.

### Same-origin and privacy checks

`isSameOrigin` compares the request origin with the worker's origin. It prevents this worker from trying to manage another site's resources.

`isPrivateRoute` normalizes the path to lower case, confirms that it is under the application root, removes the root prefix, and checks whether it begins with a configured private route name.

`isCacheableStaticRequest` requires all of the following:

- the method is `GET`;
- the URL is same-origin;
- the path is not private;
- the request has no `Authorization` header; and
- the path looks like a static asset.

This is deliberately conservative. Add a route to a cache only after you understand its privacy and freshness requirements.

### Installation

`self.addEventListener("install", ...)` listens for the installation lifecycle event.

`event.waitUntil(...)` tells the browser that installation is not complete until the promise resolves.

`caches.open(STATIC_CACHE)` opens or creates the named Cache Storage cache.

`cache.addAll(PRECACHE_URLS)` fetches each URL and stores successful responses. If a required URL fails, installation can fail. Keep the precache list small and test every URL.

### Activation and old-cache cleanup

`self.addEventListener("activate", ...)` listens for activation.

`currentCaches` lists cache names that the current worker is allowed to keep.

`caches.keys()` lists existing cache names for this origin.

The `filter` keeps only old caches belonging to this application prefix and excludes current cache names.

`caches.delete(cacheName)` removes an old cache.

`self.clients.claim()` asks the active worker to claim eligible open clients. It returns a promise, so it is chained into the activation work.

### Controlled updates

The `message` event receives messages from controlled pages.

If the message type is `SKIP_WAITING`, `self.skipWaiting()` asks the waiting worker to activate immediately. The page only sends this after the user chooses the update button.

### Navigation strategy

`networkFirstNavigation` tries the server first. This keeps normal MVC pages fresh and respects server authorization and current data.

If `fetch(request)` fails because the device is offline, the function looks for an exact cached page. The baseline does not populate ordinary MVC pages, so the final fallback is `OFFLINE_URL`.

### Static-asset strategy

`cacheFirstStaticAsset` looks in Cache Storage first. If an asset is already cached, it returns it immediately.

If it is not cached, the worker requests it from the network. A successful response is cloned and placed in the runtime cache. Responses are streams; cloning allows one copy to be returned while another is stored.

### Fetch routing

The `fetch` event runs for requests within the worker's scope.

The worker constructs a `URL` object so it can inspect origin and path.

Cross-origin requests are ignored. The browser handles them normally.

Navigation GET requests use the network-first function.

Static asset requests use the cache-first function.

All other requests are ignored. In particular, the baseline does not intercept POST, PUT, PATCH, or DELETE requests.

## 13.4 Why the service worker does not cache task HTML

`/Tasks/Index` may look public in the teaching project, but in a real application an HTML response may contain:

- the authenticated user's name;
- role-specific links;
- tenant-specific data;
- permissions;
- time-sensitive values; or
- a token that should not be reused.

The safe default is to request such pages from the server and show the offline page when the server cannot be reached. If you later cache a public page, explicitly prove that it is public and non-personalized.

## 13.5 Add the worker script to the layout

The meta tag was already added in Chapter 10:

```cshtml
<meta name="pwa-service-worker" content="@serviceWorkerUrl" />
```

The registration script reads that value. The worker itself is not included with a normal `<script>` tag because service workers are registered through the browser's `navigator.serviceWorker` API.

## 13.6 Confirm registration

Run the application over HTTPS and open DevTools:

1. Select **Application**.
2. Select **Service Workers**.
3. Confirm that `/sw.js` is registered.
4. Confirm that it is activated.
5. Reload the page.
6. Check that `navigator.serviceWorker.controller` is not `null` after reload.

Open the worker URL directly:

```text
https://localhost:5001/sw.js
```

It must display JavaScript source, not an MVC error page.

---

# Chapter 14 — Test the PWA from first visit to offline use

## 14.1 Test the normal online site first

Before enabling offline mode:

1. Open `/Tasks/Index`.
2. Add a valid task.
3. Submit an empty title and confirm server validation.
4. Refresh the page.
5. Navigate between Home and Tasks.
6. Open the browser console and check for errors.

A PWA cannot repair a broken MVC site. Fix normal navigation and forms first.

## 14.2 Inspect the manifest

In Chromium DevTools:

1. Open **Application**.
2. Select **Manifest**.
3. Verify the application name.
4. Verify `start_url`.
5. Verify the display mode.
6. Verify the scope.
7. Verify that both required icon sizes are loaded.
8. Read every installability warning.

The browser's install menu and installability warnings are more reliable than assuming that a custom button should always appear.

## 14.3 Inspect Cache Storage

In **Application → Cache Storage**, look for:

```text
pwa-mvc-course-static-v1
pwa-mvc-course-runtime-v1
```

The static cache should contain the offline page and manifest. The runtime cache should contain only safe static assets you requested.

If you find private HTML, API responses, authentication pages, or mutation responses, stop and fix the worker before continuing.

## 14.4 Simulate offline mode

Use DevTools Network conditions:

1. Load the site online.
2. Confirm the worker is active.
3. Enable **Offline**.
4. Navigate to a new MVC URL.
5. Confirm that the offline page appears.
6. Turn the network back on.
7. Click **Try again** or reload.
8. Confirm that the MVC page returns.

The first visit while offline cannot be rescued if the browser has never installed the worker and never cached the offline page.

## 14.5 Test installation

Installation behavior varies by browser and platform. Supporting Chromium browsers can promote installable PWAs. Safari and iOS use their own flows, and iOS does not expose the same `beforeinstallprompt` event. [3]

Test:

- the browser's own install menu;
- the custom install button when available;
- the installed app's icon;
- the installed app's launch URL;
- the standalone window appearance; and
- uninstalling and reinstalling from a clean state.

## 14.6 Test an update

Change:

```javascript
const STATIC_CACHE = `${CACHE_PREFIX}-static-v2`;
const RUNTIME_CACHE = `${CACHE_PREFIX}-runtime-v2`;
```

Also change a visible string in the worker, such as a comment. Deploy or save the worker and reload.

Expected flow:

1. The browser finds a new worker.
2. The new worker installs.
3. The new worker waits while the old worker controls the page.
4. The update button appears.
5. Clicking the button sends `SKIP_WAITING`.
6. `controllerchange` fires.
7. The page reloads.
8. The old caches are deleted during activation.

If the update is delayed, inspect the HTTP cache headers for `/sw.js`.

## 14.7 Test form safety

With the network disabled:

1. Open the task page if it is already available.
2. Submit the Add Task form.
3. Confirm that the request is not silently stored by the worker.
4. Turn the network on.
5. Decide whether the form should be retried manually or whether the product needs an offline outbox.

The baseline should fail visibly rather than claim success for a task that was never accepted by the server.

## 14.8 Console commands

Run these commands in the page console:

```javascript
await navigator.serviceWorker.getRegistrations();
```

This lists service-worker registrations.

```javascript
navigator.serviceWorker.controller;
```

This shows the worker currently controlling the page, or `null` if none controls it.

```javascript
await caches.keys();
```

This lists Cache Storage cache names.

```javascript
await caches.delete("pwa-mvc-course-runtime-v1");
```

This deletes one development cache. Do not use arbitrary cache deletion in production without understanding the user's local data.

## 14.9 Use a clean browser state

Service workers persist beyond a normal page refresh. When testing a new worker:

- use DevTools **Unregister**;
- use **Clear storage**;
- remove application data; or
- use a fresh browser profile.

Do not interpret a stale worker as evidence that your new source code is wrong until you have checked registration, update, and cache state.

---

# Chapter 15 — Deploy safely to production

## 15.1 Production requirements

Every non-local environment should use HTTPS. Service workers are restricted to secure contexts, and browser installation requires secure delivery except for permitted local development origins. [3] [6]

Production must also provide:

- correct public URLs;
- correct static-file MIME types;
- a service worker at the intended scope;
- a valid manifest;
- accessible icons;
- a self-contained offline page;
- a cache policy that protects private data; and
- a service-worker update policy.

## 15.2 Publish ASP.NET Core

Run:

```powershell
dotnet publish .\PwaMvcCourse.csproj `
  --configuration Release `
  --output .\publish
```

Line by line:

- `dotnet` invokes the .NET CLI.
- `publish` produces deployable application output.
- the project path identifies the project.
- `--configuration Release` uses release build settings.
- `--output .\publish` writes the result to a publish directory.

ASP.NET Core's publish process generates a `web.config` for IIS hosting. Do not remove it from the published output. IIS uses the ASP.NET Core Module to start or forward requests to the app. [12]

## 15.3 IIS deployment

For IIS:

1. Install the matching .NET Hosting Bundle on the server.
2. Create a folder for the published application.
3. Create an IIS site with an explicit host name and HTTPS binding.
4. Set the physical path to the published folder.
5. Give the application pool identity read access.
6. Deploy the contents of `publish`.
7. Open the public HTTPS URL.
8. Test the PWA files from a separate client machine.

Microsoft warns against top-level wildcard IIS bindings such as `http://*:80/` for production because they can create security vulnerabilities. Use explicit host names. [12]

## 15.4 Verify production response headers

Run:

```powershell
$base = "https://www.example.com"

curl.exe -I "$base/sw.js"
curl.exe -I "$base/offline.html"
curl.exe -I "$base/pwa/manifest.webmanifest"
curl.exe -I "$base/pwa/icons/icon-192.png"
```

Expected results:

- `/sw.js`: JavaScript content and a short-lived or `no-cache` policy.
- `/offline.html`: HTML content and a successful response.
- `/pwa/manifest.webmanifest`: JSON content and a successful response.
- icons: image content and successful responses.

Do not make `sw.js` effectively immutable forever. The browser needs to discover worker changes. Ordinary fingerprinted assets can have longer cache lifetimes because their names change when their content changes.

## 15.5 Reverse proxies and forwarded HTTPS

If a reverse proxy terminates TLS and forwards traffic to Kestrel over HTTP, configure forwarded headers correctly. The application must understand the external scheme when generating URLs and applying HTTPS behavior.

If the application thinks the request is HTTP while the browser used HTTPS, you may see:

- redirect loops;
- insecure cookie behavior;
- incorrect absolute URLs;
- mixed-content errors; or
- installability failures.

Do not trust arbitrary forwarded headers from untrusted clients. Configure trusted proxies according to your hosting environment.

## 15.6 Virtual directories

If the application is deployed at:

```text
https://www.example.com/BackOffice/
```

then use:

```json
{
  "id": "/BackOffice/",
  "start_url": "/BackOffice/Tasks/Index",
  "scope": "/BackOffice/"
}
```

Use `Url.Content("~/...")` in Razor rather than hard-coded root paths. Register the worker at `/BackOffice/sw.js` with `/BackOffice/` scope. The worker sample computes its path root from the registration scope.

## 15.7 Deployment smoke test

Use this PowerShell script after each deployment:

```powershell
$base = "https://www.example.com"
$urls = @(
  "/",
  "/sw.js",
  "/offline.html",
  "/pwa/manifest.webmanifest",
  "/pwa/icons/icon-192.png",
  "/pwa/icons/icon-512.png"
)

foreach ($url in $urls) {
  $response = Invoke-WebRequest -Uri ($base + $url) -Method Head

  [pscustomobject]@{
    Url = $url
    Status = $response.StatusCode
    ContentType = $response.Headers["Content-Type"]
    CacheControl = $response.Headers["Cache-Control"]
  }
}
```

The script checks status and headers, but a browser-level test should also inspect the response body, register the worker, go offline, and verify the fallback.

---

# Chapter 16 — Adapt the guide to legacy ASP.NET MVC 5

## 16.1 What remains the same

The browser-side files remain conceptually the same:

- `manifest.webmanifest`;
- icons;
- `offline.html`;
- `pwa-register.js`; and
- `sw.js`.

The manifest and worker still need HTTPS, same-origin delivery, and correct paths.

## 16.2 What changes

MVC 5 normally uses:

- `Global.asax` instead of modern `Program.cs` startup;
- `Web.config` for IIS configuration;
- `Content` and `Scripts` folders for public files;
- IIS or IIS Express static-file behavior; and
- older Razor and bundling conventions.

Place the service worker at the application root:

```text
sw.js
```

Place the manifest and icons under a public content path:

```text
Content/Pwa/manifest.webmanifest
Content/Pwa/icons/icon-192.png
Content/Pwa/icons/icon-512.png
```

Place the registration script under:

```text
Scripts/pwa-register.js
```

## 16.3 MVC 5 layout links

In `Views/Shared/_Layout.cshtml`, use:

```cshtml
<link rel="manifest" href="@Url.Content("~/Content/Pwa/manifest.webmanifest")" />
<link rel="apple-touch-icon" sizes="180x180"
      href="@Url.Content("~/Content/Pwa/icons/apple-touch-icon.png")" />
<meta name="theme-color" content="#2563eb" />
<meta name="pwa-service-worker" content="@Url.Content("~/sw.js")" />
<script src="@Url.Content("~/Scripts/pwa-register.js")" defer></script>
```

The paths differ from the ASP.NET Core paths because the files live under `Content` and `Scripts`.

The worker's precache list must also use the MVC 5 paths:

```javascript
const PRECACHE_URLS = [
  pathFromRoot("offline.html"),
  pathFromRoot("Content/Pwa/manifest.webmanifest")
];
```

Update the icon paths in the manifest to use `/Content/Pwa/icons/...`.

## 16.4 MVC 5 MIME mapping

Merge this configuration into the existing `Web.config`:

```xml
<configuration>
  <system.webServer>
    <staticContent>
      <remove fileExtension=".webmanifest" />
      <mimeMap fileExtension=".webmanifest" mimeType="application/manifest+json" />
    </staticContent>
  </system.webServer>
</configuration>
```

Line by line:

- `<configuration>` is the root configuration element.
- `<system.webServer>` contains IIS web-server configuration.
- `<staticContent>` configures static-file content types.
- `<remove ... />` prevents a duplicate mapping if the server already defines one.
- `<mimeMap ... />` maps `.webmanifest` to the manifest media type.
- Closing tags end each configuration section.

Do not replace a production `Web.config` blindly. Merge the required setting with existing URL rewrite, authentication, compression, and application configuration.

## 16.5 MVC 5 content properties

In Visual Studio, set the build action of these files to **Content**:

- `sw.js`;
- `offline.html`;
- `manifest.webmanifest`;
- `pwa-register.js`; and
- icon files.

Confirm that publishing copies them to the IIS application's physical directory. Open the final URLs directly. If `/sw.js` returns an MVC error view, fix static-file delivery before debugging JavaScript.

## 16.6 MVC 5 deployment checklist

- HTTPS binding is configured.
- IIS Static Content is installed.
- The application pool can read the files.
- `.webmanifest` has the correct MIME type.
- `sw.js` is physically at the application root.
- the manifest paths include `Content/Pwa`.
- the registration script path includes `Scripts`.
- the worker precache paths match the deployed URLs.
- the application base path matches the manifest scope.
- `sw.js` is not given a permanent cache policy.

---

# Chapter 17 — Master caching strategies

## 17.1 Caching is a product decision

Caching is not simply a performance trick. A cached response can be old, unauthorized, incomplete, or associated with the wrong user.

Ask these questions for every response:

1. Is this response public?
2. Can it safely be reused by another visit?
3. How stale may it be?
4. What happens when the user is offline?
5. How is it invalidated?
6. Does it contain a token or user-specific data?
7. What happens after sign-out or tenant switching?

## 17.2 Cache-first

**Cache-first** checks the cache and uses the network only when the resource is not cached.

It is appropriate for immutable or versioned static assets such as:

- icons;
- fingerprinted JavaScript;
- fingerprinted CSS;
- fonts; and
- images.

The risk is staleness. If the URL remains the same while the content changes, the user may see the old file.

## 17.3 Network-first

**Network-first** asks the server first and uses a cached response only when the network fails.

It is useful for public HTML where freshness matters and offline fallback is still useful. It is safer than cache-first for MVC pages, but it still requires a privacy review before caching any page.

The baseline navigation strategy is network-first but does not cache ordinary MVC HTML by default.

## 17.4 Stale-while-revalidate

**Stale-while-revalidate** returns a cached response immediately and then requests a fresh response in the background.

It can make a public page feel fast, but users may see stale data briefly. Do not use it for private or highly time-sensitive content without a strong design.

## 17.5 Network-only

**Network-only** always goes to the server. Use it for:

- authenticated APIs;
- mutations;
- payment or financial operations;
- sign-in and sign-out;
- security-sensitive pages; and
- data that must be current.

The baseline worker effectively uses network-only for non-navigation requests that are not safe static assets.

## 17.6 Cache-only

**Cache-only** serves only from the cache. It is useful for a known precached offline shell, but dangerous if used for resources that may not yet exist in the cache.

## 17.7 HTTP cache versus service-worker cache

The browser HTTP cache is controlled largely by HTTP headers. Cache Storage is controlled by service-worker JavaScript. ASP.NET Core response caching and output caching are server-side mechanisms. CDN caches are another layer.

A response can be fresh in one cache and stale in another. Debug by inspecting the actual response source and headers.

Microsoft distinguishes response caching, which follows HTTP cache headers, from output caching, which is server-controlled. [13]

## 17.8 Cache invalidation

The simplest reliable service-worker invalidation pattern is:

1. change the worker code or cache version;
2. install the new worker into new cache names;
3. activate the new worker;
4. delete older cache names; and
5. notify or reload controlled pages at an appropriate time.

Never delete every cache belonging to every application on the origin. Use a unique prefix for your application.

## 17.9 Avoid caching sensitive responses

Never cache by default:

- sign-in, sign-out, and password pages;
- account and management pages;
- private API responses;
- pages containing user names, roles, or tenant data;
- mutation responses;
- responses with access tokens or security headers that must not be replayed; and
- pages whose content changes based on cookies unless explicitly designed.

The service worker is not an authorization layer. A cached response may be returned without a new server authorization check.

## 17.10 Static asset fingerprinting

A fingerprinted asset includes a content-derived identifier in its URL, for example:

```text
site.93b1a1f2.css
```

When the file changes, the URL changes. That makes long-lived caching safer. Do not apply the same immutable strategy to the service-worker script itself, because the browser must discover its changes.

---

# Chapter 18 — Master offline data and synchronization

## 18.1 Offline reads and offline writes are different

Offline reads can sometimes use a local copy of previously downloaded data.

Offline writes create a command that the server has not yet accepted. That requires a queue, retry behavior, server validation, and conflict handling.

Do not implement offline writes by simply placing POST requests into an arbitrary JavaScript array.

## 18.2 IndexedDB

**IndexedDB** is a browser database for structured data. It is more suitable than `localStorage` for larger structured offline data, but it is still local browser storage and should not be treated as a secure vault.

Store only data that the product has decided to keep locally. Document retention and sign-out cleanup.

## 18.3 The outbox pattern

An **outbox** stores commands that are waiting to be sent to the server.

A command should contain:

```json
{
  "operationId": "7dbfcf31-9dc7-4ac4-a0e5-0f9b5e5c5f4a",
  "entityId": "task-123",
  "operation": "create-task",
  "clientCreatedAt": "2026-10-11T10:00:00Z",
  "payload": {
    "title": "Review offline synchronization"
  }
}
```

The `operationId` is an idempotency key. If the browser retries after a timeout, the server can recognize that it already processed the command instead of creating duplicates.

## 18.4 Synchronization steps

A safe synchronization flow is:

1. The user creates a local command.
2. The UI marks it as pending.
3. The application tries to send it when online.
4. The server authenticates and authorizes the request.
5. The server checks the idempotency key.
6. The server validates the operation and current entity version.
7. The server accepts, rejects, or reports a conflict.
8. The client marks the command as succeeded, failed, or conflicted.
9. The UI explains what happened.

## 18.5 Conflicts

A conflict occurs when the local assumption is no longer true. For example, the user edits a task offline while another user changes it online.

Possible policies include:

- last write wins;
- server wins;
- client wins;
- field-level merge; or
- ask the user to resolve it.

Choose a policy based on the data's business meaning. Last write wins can be unacceptable for financial or operational records.

## 18.6 Authentication and offline mode

Offline mode cannot prove that a session is still valid with the server. Do not treat the presence of a cookie as current authorization.

The safest first offline design is:

- allow non-sensitive locally stored data only;
- require the server for protected data;
- require fresh authorization for synchronization; and
- clear local data on sign-out or tenant change.

## 18.7 Background Sync

Background Sync can help a supported browser retry work later, but it is not a universal guarantee that every operating system will run arbitrary work. Provide a visible foreground retry path as well.

The user should be able to understand whether a command is pending, succeeded, failed, or conflicted.

---

# Chapter 19 — Advanced PWA capabilities

## 19.1 Custom installation UX

The registration script uses `beforeinstallprompt` when the browser exposes it. Do not show an install message immediately to every first-time visitor. Explain the value first, then offer installation at a useful moment.

On platforms that do not expose this event, provide platform-specific instructions or let the browser's own menu handle installation.

## 19.2 Notifications

Notifications require user permission. Ask after a clear explanation of the benefit, not automatically on first page load.

A production notification system needs:

- permission handling;
- a way to disable notifications;
- subscription storage;
- expiration and revocation handling;
- server authorization for sending; and
- a review of sensitive payload content.

## 19.3 Push

Push notifications usually involve:

1. the browser creates a push subscription;
2. the client sends the subscription to the server;
3. the server stores it for the authenticated user;
4. the server sends a push request through a push service; and
5. the service worker receives a push event and displays a notification.

Treat subscriptions and private keys as protected data. Do not put confidential information into a notification body merely because it is convenient.

## 19.4 Shortcuts

Manifest shortcuts provide quick links from an installed app's operating-system menu. Point them to normal, safe GET pages. Never encode destructive actions as a shortcut URL.

## 19.5 Share target

A share target lets other applications send text or files to your PWA where supported. Treat the receiving MVC endpoint like an upload endpoint:

- require authorization when appropriate;
- validate size and content type;
- scan files;
- prevent path traversal;
- avoid trusting file names; and
- provide a normal browser fallback.

## 19.6 App badges

Badges can communicate counts without interrupting the user. Feature-detect the API and provide an ordinary in-app count when it is unavailable.

## 19.7 Store packaging

A PWA can sometimes be packaged for app stores, but packaging does not remove the need for web security, deployment, responsive design, and service-worker testing. Treat store distribution as a separate release target.

---

# Chapter 20 — Production engineering and mastery

## 20.1 Security

Before production:

- enforce HTTPS;
- protect cookies with appropriate secure settings;
- keep authorization on the server;
- keep antiforgery protection for state-changing MVC forms;
- do not store credentials or tokens in Cache Storage or `localStorage`;
- exclude private routes from the service-worker cache;
- do not serve unknown file types broadly;
- keep private uploads outside the public static web root;
- disable directory browsing;
- limit notification and push permissions; and
- review sign-out, tenant switching, and shared-device behavior.

A service worker can make a private response persist locally. That makes cache review part of the security review.

## 20.2 Accessibility

A PWA is still a website. It should work with keyboard navigation, screen readers, zoom, high contrast, reduced motion, and narrow viewports.

Use:

- semantic headings;
- labels associated with form controls;
- visible focus states;
- sufficient color contrast;
- meaningful button text;
- status messages that assistive technology can perceive; and
- an offline page that explains what happened and what the user can do.

The install button is not useful if the user cannot discover or activate it with a keyboard.

## 20.3 Performance

Measure before optimizing. Common improvements include:

- reducing image dimensions and file sizes;
- using modern image formats where appropriate;
- minimizing JavaScript shipped to the browser;
- serving compressed CSS and JavaScript;
- using content-based asset fingerprinting;
- avoiding unnecessary synchronous work on the main thread;
- making the first server response fast; and
- caching only resources that are safe and useful.

A service worker can make a slow or badly designed first visit appear faster on repeat visits, but it cannot fix server latency for the first visit.

## 20.4 Observability

Track enough information to diagnose PWA problems without recording sensitive content:

- service-worker registration failures;
- worker version;
- update available and update activated events;
- offline fallback frequency;
- failed synchronization commands;
- browser and operating-system categories; and
- deployment version.

Do not log cookies, authorization headers, passwords, private response bodies, or complete personal payloads.

## 20.5 Automated testing

Add tests at several levels:

### Server tests

Test controllers, validation, authorization, and service behavior without a browser.

### Static-file smoke tests

Verify that manifest, worker, offline page, and icons return the expected status and content type.

### Browser tests

Verify that:

- the manifest is linked;
- the worker becomes active;
- the offline page appears when a navigation fails;
- POST requests are not silently treated as successful offline operations;
- a worker update can be activated; and
- private routes do not enter Cache Storage.

### Deployment tests

Run against a staging HTTPS host from a client machine. A test run only against `localhost` does not prove that IIS, a CDN, a reverse proxy, or a virtual directory is configured correctly.

## 20.6 Continuous delivery

A useful deployment pipeline is:

1. restore dependencies;
2. build the application;
3. run server tests;
4. publish the application;
5. run static-file checks against the publish directory or staging host;
6. deploy to staging;
7. run browser smoke tests;
8. inspect worker and manifest responses;
9. approve production deployment; and
10. monitor service-worker errors and offline fallback behavior.

Every service-worker change should be reviewed like a migration. Ask what old cache entries remain, what data might be served, and whether an existing page can be updated safely.

## 20.7 The production cache checklist

Before release, answer these questions in writing:

- What is precached?
- What is runtime-cached?
- What is network-only?
- Which routes are private?
- What happens after sign-out?
- How do cache versions change?
- How do old caches get deleted?
- How does a user receive an update?
- What is the offline promise?
- Which features are browser-dependent?

If the team cannot answer these questions, the PWA policy is not finished.

---

# Chapter 21 — Mastery roadmap and exercises

## 21.1 Beginner milestone

You are at the beginner milestone when you can:

- explain client, server, URL, HTTP, HTML, CSS, and JavaScript;
- explain Model, View, and Controller;
- create and run an MVC application;
- add a model, controller, service, and view;
- explain why the layout renders on multiple pages; and
- explain the difference between a static file and a controller response.

## 21.2 PWA foundation milestone

You are at the foundation milestone when you can:

- explain manifest versus service worker;
- explain secure context and HTTPS;
- register a worker from an MVC layout;
- explain install, activate, fetch, and scope;
- show a self-contained offline page; and
- inspect registration and Cache Storage in DevTools.

## 21.3 Safe caching milestone

You are at the safe-caching milestone when you can:

- choose cache-first or network-first intentionally;
- distinguish HTTP cache from Cache Storage;
- explain why private HTML is not cached by the baseline;
- version and remove old caches;
- test worker updates; and
- identify a response that must remain network-only.

## 21.4 Offline-data milestone

You are at the offline-data milestone when you can:

- model local data ownership;
- use IndexedDB for structured data;
- design an outbox;
- create idempotent server commands;
- handle retries and conflicts; and
- explain what sign-out does to local data.

## 21.5 Production milestone

You are at the production milestone when you can:

- deploy under HTTPS;
- configure IIS or a reverse proxy;
- support a virtual directory;
- verify MIME types and cache headers;
- deploy a worker update safely;
- test installation across supported browsers; and
- review the PWA cache policy as part of a security review.

## 21.6 Exercises

### Exercise 1: change the application identity

Change the name, short name, theme color, description, and icons. Reopen DevTools and verify each value.

### Exercise 2: add a public help page

Create a public, non-personalized help view. Add it to `PRECACHE_URLS`. Test that it opens offline. Explain why it is safe for this specific page and not automatically safe for `/Tasks/Index` in a real authenticated system.

### Exercise 3: add a second cache version

Change the worker version, deploy it, inspect the waiting state, activate the update, and verify that the old cache is deleted.

### Exercise 4: create a deliberate failure

Temporarily add a nonexistent file to `PRECACHE_URLS`. Observe the installation failure. Remove the file and explain why `cache.addAll` made installation fail.

### Exercise 5: build a network-only API

Add a JSON endpoint that returns task data. Confirm that the worker does not cache it. Then write down what additional design would be needed before caching it offline.

### Exercise 6: move the app under a virtual directory

Host the app under `/BackOffice/`. Update the manifest, layout, worker registration, and precache paths. Verify that the scope is `/BackOffice/`.

### Exercise 7: design offline create-task behavior

Do not implement it immediately. First write the operation format, idempotency behavior, conflict rule, sign-out behavior, and user-visible states. Only then implement the browser queue and server endpoint.

## 21.7 Final definition of done

The teaching application is complete when:

- the MVC task board works online;
- the manifest is valid and linked;
- icons return successfully;
- the worker registers over HTTPS;
- the offline page is precached;
- safe static assets can be reused;
- normal MVC navigation remains network-first;
- non-GET requests are not silently cached;
- the update button can activate a new worker;
- old cache versions are deleted;
- the application works when the PWA APIs are unavailable; and
- the production checklist has been reviewed.

A production application is complete only when its own authentication, tenant, data, deployment, privacy, and offline requirements have been reviewed separately.

---

# Appendix A — The complete baseline file list

For ASP.NET Core:

```text
PwaMvcCourse/
├─ Controllers/
│  └─ TasksController.cs
├─ Models/
│  ├─ TaskItem.cs
│  └─ TaskPageViewModel.cs
├─ Services/
│  ├─ ITaskStore.cs
│  └─ InMemoryTaskStore.cs
├─ Views/
│  ├─ Tasks/
│  │  └─ Index.cshtml
│  └─ Shared/
│     └─ _Layout.cshtml
├─ wwwroot/
│  ├─ offline.html
│  ├─ sw.js
│  ├─ js/
│  │  └─ pwa-register.js
│  └─ pwa/
│     ├─ manifest.webmanifest
│     └─ icons/
│        ├─ icon-192.png
│        ├─ icon-512.png
│        ├─ icon-192-maskable.png
│        ├─ icon-512-maskable.png
│        └─ apple-touch-icon.png
├─ PwaMvcCourse.csproj
└─ Program.cs
```

Remember that the layout, static-file configuration, and `Program.cs` also contain changes described in the chapters.

---

# Appendix B — Essential command reference

```powershell
# Show installed SDK information.
dotnet --info

# List installed SDKs.
dotnet --list-sdks

# List MVC templates.
dotnet new list mvc

# Create an MVC project.
dotnet new mvc --name PwaMvcCourse --framework net10.0 --auth None

# Restore dependencies explicitly.
dotnet restore

# Build the application.
dotnet build

# Run the application.
dotnet run

# Run with hot reload during development.
dotnet watch

# Run tests.
dotnet test

# Publish for deployment.
dotnet publish --configuration Release --output .\publish

# Check and trust the local HTTPS certificate.
dotnet dev-certs https --check --trust

# Create and trust a local HTTPS certificate.
dotnet dev-certs https --trust

# Inspect an HTTP response.
curl.exe -I https://localhost:5001/sw.js
```

---

# Appendix C — Troubleshooting

## The project will not run

Check the SDK version, target framework, restore output, and the exact URL printed by `dotnet run`. If the project targets `net10.0` but only an older SDK is installed, either install the target SDK or change the project target to a supported version.

## The browser says the certificate is not trusted

Run `dotnet dev-certs https --check --trust`. If the certificate store is confused, use cleanup only when you understand that it removes local development certificates, then create and trust a new one.

## The manifest is 404

Check that it is under `wwwroot/pwa/manifest.webmanifest`, that static-file delivery is enabled, and that the final HTML points to the correct URL.

## The manifest is shown as HTML

The URL is probably being handled by an MVC fallback or error route. A manifest must be returned as JSON. Inspect the response body and content type.

## The service worker says it has a syntax error

Open `/sw.js` directly. If the response is an HTML error page, fix routing or static-file delivery. If it is JavaScript, run the source through a syntax checker and inspect the browser console line number.

## The service worker will not register

Check HTTPS, same-origin delivery, the exact path, scope permissions, and browser private-mode restrictions. A worker in a nested directory does not automatically control the application root. [5] [6]

## The service worker does not seem to change

Unregister it, clear storage, inspect `registration.waiting`, increment cache versions, call `registration.update()`, and inspect `Cache-Control` on `/sw.js`.

## The offline page does not appear

Confirm that the worker is active, that `/offline.html` is in the static cache, that the navigation is within scope, and that the fetch handler catches the failed navigation.

## Private content appears offline

Treat this as a security defect. Clear local caches, remove the caching rule, review all cache layers, and confirm that the worker does not cache responses based on authenticated cookies.

## The app works at `/` but not under `/BackOffice/`

Use `Url.Content("~/...")`, update manifest paths, calculate the worker scope from its registration, and test all final public URLs under the virtual directory.

## POST appears to succeed offline

The baseline worker should not intercept POST. Check whether another service worker, browser extension, page script, or application queue is handling the request. Do not claim success until the server confirms the mutation.

---

# Appendix D — References

[1]: https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps "Progressive web apps - MDN Web Docs"

[2]: https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/ "Progressive Web Apps: Escaping Tabs Without Losing Our Soul - Alex Russell"

[3]: https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable "Making PWAs installable - MDN Web Docs"

[4]: https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest "Web application manifest - MDN Web Docs"

[5]: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers "Using Service Workers - MDN Web Docs"

[6]: https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts "Secure contexts - MDN Web Docs"

[7]: https://learn.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-10.0 "Overview of ASP.NET Core MVC - Microsoft Learn"

[8]: https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc?view=aspnetcore-10.0 "Get started with ASP.NET Core MVC - Microsoft Learn"

[9]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new "dotnet new - Microsoft Learn"

[10]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-dev-certs "dotnet dev-certs - Microsoft Learn"

[11]: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files?view=aspnetcore-10.0 "Serve static files in ASP.NET Core apps - Microsoft Learn"

[12]: https://learn.microsoft.com/en-us/aspnet/core/tutorials/publish-to-iis?view=aspnetcore-10.0 "Publish an ASP.NET Core app to IIS - Microsoft Learn"

[13]: https://learn.microsoft.com/en-us/aspnet/core/performance/caching/overview?view=aspnetcore-10.0 "Caching overview in ASP.NET Core - Microsoft Learn"

[14]: https://web.dev/learn/pwa "Learn PWA - web.dev"
