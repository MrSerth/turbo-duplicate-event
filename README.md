# turbo-duplicate-event

This repository is a minimal reproduction of the issue described in [hotwired/turbo#1413](https://github.com/hotwired/turbo/issues/1413).

<hr>

We are currently in the process of migrating from Turbolinks to Turbo. While doing so, we noticed an unusual behavior for pages with a turbo-visit-control directive. Specifically, Turbo dispatches an event with the previous URL, not the target URL (as Turbolinks did).

**Example:**  
- I have two pages, A and B with links between them.
- I am currently on page A.
- Page B has a meta tag:
  ```html
  <meta name="turbolinks-visit-control" content="reload">
  ```
- Therefore, clicking on the link to page B will force a full reload.

**Current Behavior with Turbo:**  
The `turbo:load` event is dispateched two times:

1. A first occurrence is issued *before* the full page reload happens. The `event.detail.url` still refers to page **A** (!).
2. A second occurrence is issued *after* the full page reload happends. The `event.detail.url` refers to page **B**.

**Expected Behavior:**  
Ideally, only one `turbo:load` event is triggered (after the full page reload).

Otherwise, we would prefer to get the previous behavior from Turbolinks, where both events contained the target URL:

1. A first occurrence is issued *before* the full page reload happens. The `event.data.url` refers to page **B**.
2. A second occurrence is issued *after* the full page reload happends. The `event.data.url` refers to page **B**.

**Reasons/Explanations:**  
For us, the current behavior of Turbo is problematic, since it causes errornous behavior and makes distinguishing between actual page loads and these "intermediate" page loads before full page reloads rather difficult. In our use case, we combine a trigger for `turbo:load` with another one for `turbo:before-visit` to perform some unload actions. With the current behavior, unloading is "useless", since another `page:load` event is triggered, just milliseconds before navigating the page.

Since the first occurrence of the event does not represent a "real" page load (but rather some intermediate state before the navigation occurs), we would prefer to drop this event entirely. Otherwise, the previous Turbolinks implementation worked for us, since it included the target URL, allowing some custom filtering of the false event.

**Reproduction Example:**  
I've created a minimal reproduction example showing the behavior. Please ensure to open the browser's development console, enable "Preserve log" and disable "Group similar messages in console" to see all events:

<img width="902" alt="Image" src="https://github.com/user-attachments/assets/72d967bc-ca18-4ac0-b41d-0a30133997d2" /> 

Turbo: https://mrserth.github.io/turbo-duplicate-event/turbo/page_a.html  
Turbolinks: https://mrserth.github.io/turbo-duplicate-event/turbolinks/page_a.html

**Environment:**  
Turbo Version: 8.0.13  
Turbolinks Version: 5.2.0  
Browser: Chrome 137.0.7151.120  
Operating System: macOS 15.5 (24F74)
