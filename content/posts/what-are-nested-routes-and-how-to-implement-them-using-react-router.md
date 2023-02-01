---
title: "What Are Nested Routes and How to Implement Them Using React Router"
date: 2023-01-31T20:19:09-05:00
draft: false
tags:
  - JavaScript
  - React
  - React Router
---

Officially, **Nested Routing** is the general idea of coupling segments of the URL to component hierarchy and data. In my own words, a nested route is a region within a page layout that reacts to route changes. 

For example, in a single-page application, when navigating from one URL to another, you do not need to render the entire page, but only those regions within the page that are dependent on that URL change.
A navigation menu is a great example of this. 

> React Router's nested routes were inspired by the routing system in Ember.js circa 2014. 

The Ember team realized that in nearly every case, segments of the URL determine:

- The layouts to render on the page
- The data dependencies of those layouts

**React Router** embraces this convention with APIs for creating nested layouts coupled to URL segments and data.

Example: 

```jsx
// Use plain objects
createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "contact",
        element: <Contact />,
      },
      {
        path: "dashboard",
        element: <Dashboard />,
        loader: ({ request }) =>
          fetch("/api/dashboard.json", {
            signal: request.signal,
          }),
      },
      {
        element: <AuthLayout />,
        children: [
          {
            path: "login",
            element: <Login />,
            loader: redirectIfUser,
          },
          {
            path: "logout",
            action: logoutUser,
          },
        ],
      },
    ],
  },
]);
```

This [visualization](https://remix.run/_docs/routing) might be helpful.