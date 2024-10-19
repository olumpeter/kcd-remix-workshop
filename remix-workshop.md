# Remix Workshop By Kent C. Dodds

## 01-Routing

### 🤓 Background

- Almost everything with a URL has routing requirements. Whether it's a web app, or a series of API endpoints, routing is an important part. Remix has built-in routing. There are three options for routing in a Remix app:
  1. [File-system](https://remix.run/docs/en/main/start/v2#file-system-route-convention) based routes (most common)
  2. [`remix.config.js`](https://remix.run/docs/en/main/guides/migrating-react-router-app#remixconfigjs) based routes (less common)
  3. Runtime defined routes (primarily used for [migrations](https://remix.run/docs/en/main/guides/client-data#migration))



- When you place a file in `app/routes` Remix creates a route for that file. You can [read about the filename convention here](https://remix.run/docs/en/main/file-conventions/remix-config#file-name-conventions). The most important thing for you to know right now is that the file should have a component as the `default` `export` which will be rendered for the part of the UI the file represents:

```jsx
export default function ExampleRoute() {
  return (
    <div>
      <h1>Example</h1>
      <p>I am a good example</p>
    </div>
  );
}
```

- Every Remix app starts with a root route found in `app/root.tsx`. This will render an `<Outlet />` component which determines where the direct children go. If those child routes have child routes of their own then they will also render an `<Outlet />`, but we'll get into that in more detail later.


### 