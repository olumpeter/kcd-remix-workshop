# Remix Workshop Notes By Kent C. Dodds

Source: [Kent C. Dodds Remix Workshop](https://github.com/kentcdodds/remix-workshop)

## 01. Routing

### 📝 Notes

### 🤓 Background

Almost everything with a URL has routing requirements. Whether it's a web app, or a series of API endpoints, routing is an important part. Remix has built-in routing. There are three options for routing in a Remix app:

1. [File-system](https://remix.run/docs/en/main/start/v2#file-system-route-convention) based routes (most common)
2. [`remix.config.js`](https://remix.run/docs/en/main/guides/migrating-react-router-app#remixconfigjs) based routes (less common)
3. Runtime defined routes (primarily used for [migrations](https://remix.run/docs/en/main/guides/client-data#migration))

When you place a file in `app/routes` Remix creates a route for that file. You can [read about the filename convention here](https://remix.run/docs/en/main/file-conventions/remix-config#file-name-conventions). The most important thing for you to know right now is that the file should have a component as the `default` `export` which will be rendered for the part of the UI the file represents:

```tsx
export default function ExampleRoute() {
    return (
        <div>
            <h1>Example</h1>
            <p>I am a good example</p>
        </div>
    );
}
```

Every Remix app starts with a root route found in `app/root.tsx`. This will render an `<Outlet />` component which determines where the direct children go. If those child routes have child routes of their own then they will also render an `<Outlet />`, but we'll get into that in more detail later.

To navigate between URLs on the web, we use `<a>` (anchor) tags with an `href`. This triggers a full-page reload between the pages. This isn't the best experience, so Remix supports client-side navigations by preventing this default behavior and interacting directly with the browser's history API. To do this, we'll be using the `<Link />` component and the `to` prop:

```html
<a href="/puppies">Full-page reload happiness</a>
<Link to="/puppies">Client-side navigation happiness</Link>
```

Those two elements will lead to the same place, but the `<Link />` will be a better user experience.

📜 [Remix Routing Docs.](https://remix.run/docs/en/main/discussion/routes)
📜 [Blog Tutorial: Your First Route](https://remix.run/docs/en/main/tutorials/blog)

### 💪 Exercise

Our goal for this exercise is to add a link to a new route at `/posts` and to create a file to be rendered when the user visits that route.

Here's a nice and styled link for you to use if you like:

```tsx
<div className="mx-auto mt-16 max-w-7xl text-center">
    <Link to="/posts" className="text-xl text-blue-600 underline">
        Blog Posts
    </Link>
</div>
```

And here's what you can use as the default export of the posts route:

```tsx
<main>
    <h1>Posts</h1>
</main>
```

### 🗃 Files

-   `app/routes/index.tsx`
-   `app/routes/posts/index.tsx` <-- you create this file

## 02. Data Loading

### 📝 Notes

Data loading is built into Remix.

If your web dev background is primarily in the last few years, you're probably used to creating two things here: an API route to provide data and a frontend component that consumes it. In Remix your frontend component is also its own API route and it already knows how to talk to itself on the server from the browser. That is, you don't have to fetch it.

If your background is a bit farther back than that with MVC web frameworks like Rails, then you can think of your Remix routes as backend views using React for templating, but then they know how to seamlessly hydrate in the browser to add some flair instead of writing detached jQuery code to dress up the user interactions. It's progressive enhancement realized in its fullest. Additionally, your routes are their own controller.

The way this works is through a `loader` function that you export from your route module. That goes hand-in-hand with a `useLoaderData` hook in your component. For example:

```tsx
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader({ request }) {
    const userId = await requireUserId(request);
    const flights = await getUserFlights(userId);
    return json({ flights }); // <-- send the data from your backend
}

export default function FlightsRoute() {
    const { flights } = useLoaderData(); // <-- get the data into your UI
    return (
        <div>
            <h3>Flights</h3>
            {flights.map((flight) => (
                <div key={flight.id}>{flight.number}</div>
            ))}
        </div>
    );
}
```

There's more to do for that to make TypeScript more helpful, but that's the gist.

One other tip is that the responsibility of the `loader` is to return a `Response` object and the `json` function is simply a helper for creating a Response object. It's effectively this:

```tsx
return new Response(JSON.stringify(data), {
    status: 200,
    headers: {
        "Content-Type": "application/json",
    },
});
```

### 💪 Exercise

We're going to start simple by putting our blog posts directly in the loader to start. Here are the blog posts we need to get from our server to the UI:t

```ts
const data = {
    posts: [
        {
            slug: "my-first-post",
            title: "My First Post",
        },
        {
            slug: "90s-mixtape",
            title: "A Mixtape I Made Just For You",
        },
    ],
};
```

Put that in your loader and then get that from the loader to the component using json and useLoaderData. You can render the posts using this JSX:

```tsx
<main>
    <h1>Posts</h1>
    <ul>
        {posts.map((post) => (
            <li key={post.slug}>
                <Link
                    to={post.slug}
                    className="text-blue-600 underline"
                >
                    {post.title}
                </Link>
            </li>
        ))}
    </ul>
</main>
```
### 🗃 Files

-   `app/routes/posts/index.tsx`

### 💯 Extra Credit

#### 1. Help TypeScript help us

-   The `json` utility is a generic function and accepts a type which you can use to make sure the types you're using are consistent. We also have a `LoaderFunctionArgs` type which we can use to get nice auto-complete on the loader function arguments. Here's the same example as above but with types.

```tsx
import type { LoaderFunctionArgs } from "@remix-run/node";
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader({ request }: LoaderFunctionArgs) {
    const userId = await requireUserId(request);
    const flights = await getUserFlights(userId);
    return json({ flights }); // <-- send the data from your backend
}

export default function FlightsRoute() {
    const { flights } = useLoaderData<typeof loader>(); // <-- get the data into your UI
    return (
        <div>
            <h3>Flights</h3>
            {flights.map((flight) => (
                <div key={flight.id}>{flight.number}</div>
            ))}
        </div>
    );
}
```

For this extra credit add `typeof` loader to the `useLoaderData` call (so far we don't need arguments to our loader, but keep LoaderArgs in mind for the future).

#### 🗃 Files

-   `app/routes/posts/index.tsx`

#### 2. Switch to prisma for data

This workshop isn't really about prisma. Remix works with whatever data loading mechanism you need. So you can just as easily make a `fetch` request directly in your `loader` to another CMS vs interact directly to your database.

In this workshop we're going to use SQLite with prisma. If you have some extra time, go ahead and swap our hard-coded data with some calls into `prisma`.

As an extra tip, it's pretty common practice to put interaction with database models in a special module responsible for that specifically, so you can create a file at `app/models/posts.server.ts` and put a `getPosts` function in that. We'll be adding more functions to that module soon.

#### 🗃 Files:

-   `app/models/post.server.ts`
-   `app/routes/posts/index.tsx`

#### 3. Optimize data loading

One cool thing about running on the server is we can trim down the data that our `loader` sends to only the stuff we want. Whether it's coming from a third party API we're hitting with `fetch` or a database call, we can fine-tune the data we return from the `loader` to only the stuff we need in our component.

Let's use prisma's querying API to limit what we get from the database to just the data our UI needs. In fact, we can rename getPosts to getPostListItems to more accurately reflect what it's doing.

```tsx
prisma.dogo.findMany({ select: { id: true, name: true } });
```

See if you can reduce the amount of data over the wire when you navigate to the post listing page.

#### 🗃 Files:

-   `app/models/post.server.ts`
-   `app/routes/posts/index.tsx`

## 03. Dynamic Params

### 📝 Notes

### 🤓 Background

Checkout this link to a tweet:

```
https://twitter.com/kentcdodds/status/1509892236032413702
```

That last long number is a unique identifier for the tweet. It tells twitter which tweet we actually want to view. If we change that identifier, we'll get a different tweet. So you could look at that URL more generically as:

```
https://twitter.com/kentcdodds/status/:tweetId
```

Where `:tweetId` is like a parameter we pass to the twitter status "function". Somewhere on twitter's server is some generic code that handles loading the tweet information based on the `:tweetId`

In fact, the username in there could be a parameter as well:

```
https://twitter.com/:username/status/:tweetId
```

In Remix, we call `:username` and `:tweetId` a "param" (short for parameters) and there is a filename convention you can follow to place generic code for handling those. For example, if we wanted to create the nested routing structure to handle the tweet, then we would create the following file structure:

```
.
└── app
    ├── entry.client.tsx
    ├── entry.server.tsx
    ├── root.tsx
    └── routes
        ├── $username
        │   └── status
        │       └── $tweetId.tsx
        └── $username.tsx
```

```
./app/routes/$username.status.$tweetId
./app/routes/$username
```

In the Remix convention, a file with a `$` prefix in the filename indicates a param. In the `$tweetId.tsx` file, you can access the params in the loader via the loader's arguments:

```tsx
import type { LoaderFunctionArgs } from "@remix-run/node";
import { json } from "@remix-run/node";
import { getTweet } from "~/models/tweets";

export async function loader({ params }: LoaderFunctionArgs) {
    return json({
        tweet: await getTweet(params.username, params.tweetId),
    });
}
```

You can also access the params in your components via the useParams hook:

```tsx
import { useLoaderData, useParams } from "@remix-run/react";

export default function Tweet() {
    const { username, tweetId } = useParams();
    const data = useLoaderData<typeof loader>();

    return <div>{/* render tweet */}</div>;
}
```

We'll learn more about nested routing in a future exercise and then we can talk about what would be in some of the other files.

### 💪 Exercise

Let's make the blog post page so we can read this sweet sweet content. Create a file in `app/routes/posts/$slug.tsx` (our param will be called slug).

In that file you'll need a `loader` that uses the `slug` param to find the blog post by its slug (you'll create a new function for that in the `post.server.ts` file).

From there, your default export component can be something simple like this:

```tsx
<main className="mx-auto max-w-4xl">
    <h1 className="my-6 border-b-2 text-center text-3xl">
        {post.title}
    </h1>
</main>
```

### 🗃 Files

-   app/models/post.server.ts
-   app/routes/posts/$slug.tsx <-- you create this file

### 💯 Extra Credit

#### 1. Add post content

Our post content is written in [markdown](https://www.markdownguide.org/). Let's convert that into HTML so we can have links and things rendered properly.

We're going to use a simple tool for this called `[marked](https://npm.im/marked)`.

It's already installed, all you need to do is use it. Here's a quick example of how to use it:

```tsx
// import marked
import { marked } from "marked";

// use it with some HTML

const html = marked(`
# Hello world

This is some **bold** text!
`);

// use that HTML in a React component

function MyComponent({ html }: { html: string }) {
    return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

You may be concerned about that `dangerously` part of `dangerouslySetInnerHTML`. React does cross-site scripting (XSS) mitigation for you automatically. This ensures that you're safe when you do something like this `<div>{userSubmittedContent}</div>`. For example, if React did not do this and `userSubmittedContent` were something other users of our app can modify and someone submitted a string of text like:` <script>alert("gotcha!")</script>` then we could be in trouble.

However, in our case, we're the author of our own blog posts so we're safe to use this escape hatch, so no worries!

### 🗃 Files:

-   app/models/post.server.ts
-   app/routes/posts/$slug.tsx

#### 2. Help TypeScript help us

We've got a few spots where TypeScript is upset at us. One of them is the use of `params.slug`. The way that's typed by Remix says it could be `string | undefined`. This is correct. Even though Remix will ensure that `params.slug` will be a string because our file is called `$slug.tsx`, TypeScript doesn't understand that convention. So we need to help it a bit. You can use regular type guards like this:

```tsx
const { slug } = params;
if (!slug) throw new Error("This should be impossible");
// now slug is a string only
```

That works fine, but I want to show you another package called `[tiny-invariant](https://npm.im/tiny-invariant)` which can be used to do runtime type checking like that. Checkout the docs for that package (it's already installed) and make TypeScript happy about things.

We'll also want to do the same for if no post is found by the given slug in the database. We'll deal with a proper `404` status code and error page later.

### 🗃 Files:

-   `app/models/post.server.ts`
-   `app/routes/posts/$slug.tsx`

## 04. Nested Routing

### 📝 Notes

### 🤓 Background

Most web applications have multiple route "segments". You might think of a URL as something similar to a filepath on a filesystem:

```
# filesystem
/Users/yourname/photos/mountains/timpanogos.jpg
# URL
https://yourname.com/photos/mountains/timpanogos
```

And just like a filesystem where files are contained within folders which are contained within folders, UIs often resemble this same relationship. For a deeper look at this idea, checkout 📜[What is nested Routing](https://remix.run/docs/en/v1/guides/routing#what-is-nested-routing) in the Remix documentation.

What makes nested layout routes so great is that it allows you to de-couple the parent routes from their children. This means that teams can work independently and Remix will stitch the application together at runtime. The team in charge of the header/footer can work without worrying about the data the various other product teams need, and those teams don't need to worry about what the header/footer team needs either. This eliminates one of the major challenges that lead organizations to desire a complicated micro-frontend architecture.

Even though the parent and child don't need to communicate, the parent is required to specify where the child should be rendered. This is where the `<Outlet />` component comes in. Here's how you use it:

```tsx
import { Outlet } from "@remix-run/react";

export default function PhotosRoute() {
    return (
        <div>
            <h1>Your Photos</h1>
            <Outlet /> {/* 👈 This is where any children will be rendered */}
        </div>
    );
}
```

```tsx
export default function CategoryRoute() {
    return (
        <div>
            {/* Remix will make sure that I'm rendered within the <Outlet /> above */}
        </div>
    );
}
```

Another neat feature of nested routing comes in the form of relative route calculation for the `<Link />` `to` prop. So if I render a `<Link />` component from within the route `/photos` and I want it to navigate to `/photos/mountains` then my `to` prop can be simply `to="mountains"` rather than `to="/photos/mountains"`. This makes the code much more portable and just generally easier to deal with (that said, you can still do an absolute path if you like, you can even do `../` paths!).

### 💪 Exercise

It's about time we start writing blog posts! Let's add a route to `/posts/admin` where we can see a list of our posts and edit them at `/posts/admin/:slug` and create new ones at `/posts/admin/new`.

For this exercise, all you're expected to do is create the files and export the components. We'll get to implementing the features in the next exercise once we have the routes all configured.

🦉 TIP! At any time, you can `cd` into the project directory and run `npx remix routes` and Remix will print out the route configuration that will be generated based on your filesystem. For example, before this exercise your route configuration will look something like this:

```jsx
<Routes>
    <Route file="root.tsx">
        <Route path="posts/:slug" file="routes/posts/$slug.tsx" />
        <Route path="posts" index file="routes/posts/index.tsx" />
        <Route path="logout" file="routes/logout.tsx" />
        <Route index file="routes/index.tsx" />
        <Route path="login" file="routes/login.tsx" />
    </Route>
</Routes>
```

After you're finished, it should look something like this:

```jsx
<Routes>
    <Route file="root.tsx">
        <Route path="posts/:slug" file="routes/posts/$slug.tsx" />
        <Route path="posts/admin" file="routes/posts/admin.tsx">
            <Route index file="routes/posts/admin/index.tsx" />
            <Route path="new" file="routes/posts/admin/new.tsx" />
        </Route>
        <Route path="posts" index file="routes/posts/index.tsx" />
        <Route path="logout" file="routes/logout.tsx" />
        <Route index file="routes/index.tsx" />
        <Route path="login" file="routes/login.tsx" />
    </Route>
</Routes>
```

💰 Because this is a Remix workshop (not a React one), I'm going to give you some JSX you can use in some of these new files you'll create:

```jsx
<div className="mx-auto max-w-4xl">
    <h1 className="my-6 mb-2 border-b-2 text-center text-3xl">
        Blog Admin
    </h1>
    <div className="grid grid-cols-4 gap-6">
        <nav className="col-span-4 md:col-span-1">
            <ul>
                {posts.map((post) => (
                    <li key={post.slug}>
                        <Link
                            to={post.slug}
                            className="text-blue-600 underline"
                        >
                            {post.title}
                        </Link>
                    </li>
                ))}
                <li>
                    <Link
                        to="new"
                        className="text-blue-600 underline"
                    >
                        ➕ Create New Post
                    </Link>
                </li>
            </ul>
        </nav>
        <main className="col-span-4 md:col-span-3">
            {/* 🐨 your job is to add an Outlet component here */}
        </main>
    </div>
</div>
```

```jsx
<h2>New Post</h2>
```

```jsx
<p>
  <Link to="new" className="text-blue-600 underline">
    Create a New Post
  </Link>
</p>
```

(We'll expand on these 👆 in the next exercise)

### 🗃 Files

- app/routes/posts/index.tsx
- app/routes/posts/admin.tsx <-- new file
- app/routes/posts/admin/new.tsx <-- new file
- app/routes/posts/admin/index.tsx <-- new file


## 05. Mutations

### 📝 Notes

### 🤓 Background

Getting data onto the page is the easy part. Remix has great APIs and conventions for it. For a very small number of small websites, this is enough. But most web applications require mutations of some kind. The user wants to be able to change the data. "All you have to do" is create a `<form />`, add a `onSubmit` with event.`preventDefault()` and off you go, right? Nah... This is where things typically get pretty complicated.

The problem is you can't just make a network request to change the data. You also have to update the data you already loaded onto the page. For example, what if you're building a chat app and there's a list of unread messages as well as a total at the top. You can't just mark a message as read. You also have to update the total.

If you've been working in the frontend in the last few years, you're probably starting to think about bringing in your favorite application state management library. Redux, MobX, Apollo, React Query, etc. These each come with their own set of trade-offs and challenges. It's a complicated problem (as evidenced by the number of options).

In Remix, we like to take steps back... A lot of steps. Some of us have been around long enough to remember when web development wasn't so difficult. Back in early days of the web, when the user made a mutation (using a `<form />` *without* `event.preventDefault())`, the browser would serialize the user inputs and submit it automatically. Then the backend would process it and either send back the new (up-to-date) HTML or tell the browser where to go next (to get up-to-date) HTML.

The mental model was so simple. The problem is replacing your HTML wholesale like that is not the best UX. Can you imagine getting a full-page refresh every time you favorited a tweet? But maybe there's a way that we can have the old, simpler mental model with the modern UX our users have grown accustomed to? Kinda like a **remix** of the old with the new 😏.

This is one of the main things that makes Remix so special. In Remix, you don't actually need a state management library to handle these things. Remix handles your mutations and ensures your data is up-to-date after mutations are finished doing their mutating.

Here's a simple example of how to use the conventional API for this:

```js
import type { ActionFunctionArgs } from "@remix-run/node";
import { json, redirect } from "@remix-run/node";
import { Form } from "@remix-run/react";
import { createNewDogo } from "~/models/dogo.server";

export async function action({ request, params }: ActionFunctionArgs) {
  const formData = await request.formData(); // <-- 📜 learn more https://mdn.io/formData
  const name = formData.get("dogo-name");
  const newDogo = await createNewDogo({ name });
  return redirect(`/dogo/${newDogo.id}`);
}

export default function NewDogoRoute() {
  return (
    <Form method="post">
      <label>
        Name: <input name="dogo-name" />
      </label>
      <button type="submit">Create Dogo</button>
    </Form>
  );
}
```

There is a tiny bit more to it once you want to start thinking about pending UI (which we'll get to next), but that's the basic idea. You've got a `Form` with `method="post"` and some form elements and when it's submitted, Remix will call your action with the request which you can use to get the form data.

> 🦉 Note that without the `method="post"`, the HTTP method will be a "GET" which will cause your `loader` to be called instead. There are use cases for this, but that's outside the scope of this workshop.

### 💪 Exercise

We want to be able to create blog posts. So let's flesh out the `new.tsx` file a bit. Again, this isn't a React workshop and I don't want you wasting your time writing out `JSX`, so I've given you the starting point for the form.

You'll need to add the action and create a createPost function in your `post.server.ts` file that accepts the `title`, `slug`, and `markdown`.

### 🗃 Files

- app/models/post.server.ts
- app/routes/posts/admin/new.tsx

### 💯 Extra Credit

#### 1. Error handling

Sometimes users make errors. Before we try to submit their data to the database, we should probably make sure their data is error-free so we can send back a handy error message. To communicate back from the `action` to the form, we use a hook called `useActionData`. If we add error handling to our earlier example it'd be something like this:

```jsx
import type { ActionFunctionArgs } from "@remix-run/node";
import { json, redirect } from "@remix-run/node";
import { useActionData } from "@remix-run/react";
import { Form } from "@remix-run/react";
import { createNewDogo } from "~/models/dogo.server";

export async function action({ request, params }: ActionFunctionArgs) {
  const formData = await request.formData(); // <-- 📜 learn more https://mdn.io/formData
  const name = formData.get("dogo-name");
  if (typeof name !== "string" || !name) {
    return json({ name: "Dogo name is required" });
  }
  if (name.length < 2 || name.length > 12) {
    return json({ name: "Dogo name must be between 2 and 12 characters" });
  }
  const newDogo = await createNewDogo({ name });
  return redirect(`/dogo/${newDogo.id}`);
}

export default function NewDogoRoute() {
  const errors = useActionData();
  return (
    <Form method="post">
      <label>
        Name: <input name="dogo-name" />
        {errors?.name ? <em className="text-red-600">{errors.name}</em> : null}
      </label>
      <button type="submit">Create Dogo</button>
    </Form>
  );
}
```

Go ahead and handle errors for our little app. All three fields are required. If any of them has an error, we should display an error for the user.

#### 2. Help TypeScript help us

Let's take a second to let TypeScript help us out here. First, let's fix createPost to accept the right types for `slug`, `title`, and `markdown` (strings), then you'll need to make sure that our action asserts those values from the `formData` are all `string`s.

Similar to the `loader` function, the `action` export's arguments can be typed via `ActionFunctionArgs` which you can get from `@remix-run/node`.

Finally, the `useActionData` is a generic similar to `useLoaderData`, so you can use it like: `useActionData<typeof action>` and it'll give you some nice type inference.

### 06. Progressive Enhancement

### 📝 Notes

### 🤓 Background

I've got a surprise for you... Did you know that we can completely disable client-side JavaScript on this app and it'll still work? Give it a try! Go to `app/root.tsx` and comment out the `<Scripts />` component. Try creating a post and check it out

How does this black magic work?! The answer is simple: Progressive Enhancement!

Turns out the browser actually is pretty capable without client-side JS to help it out. It knows how to encode user input and make a POST request with that as the post body. So what Remix did was say "hey, let's just make sure when the browser does its thing, we'll handle it the same way we do when there is client-side JS."

Remix will handle a browser's POST request and route it to the appropriate action. The action then sends back a redirect and the browser will follow that, or the action responds with JSON and Remix will run your app as if a new request has been made and respond with the new HTML which the browser will then render.

```jsx
import { useNavigation } from "@remix-run/react";

export default function NewDogoRoute() {
  const navigation = useNavigation();
  const isSubmitting = Boolean(navigation.state === "submitting");
  return (
    <Form method="post">
      <label>
        Name: <input name="dogo-name" />
      </label>
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Creating Dogo..." : "Create Dogo"}
      </button>
    </Form>
  );
}
```

In this example we're simply asking Remix about the current transition. If there's a `submission` on it then we know that the user has submitted a form and we can display Creating Dogo... in the submit button and disable it. You could also render a loading spinner or whatever else you'd like based on that information. If the request errors, then the submission goes away and we re-render with `isSubmitting` as `false` so the user can try again.

Another cool thing we can do with JavaScript is pre-fetch data/code/other assets so it's ready when the user actually navigates to different routes. One challenge relatively unique to the web is that we don't download the entire application upfront. Instead we split the application code into chunks resulting in a much faster initial load.

However, we also like to co-locate our data requirements with the code that requires it (makes it much easier to maintain long-term). Traditionally this has meant that we have to download the code first, and then the code will execute to download the data. That's a waterfall effect which results in a slower application.

Remix changes this because with the structure of a Remix app, we're able to know code and data requirements just by looking at the URL. Remix doesn't have to execute any of your code to know which bit of it is needed. This makes it trivial to pre-fetch code and data as the user is using the site. Seriously, this is all you need to do:

```diff
- <Link to={dogo.id}>{dogo.name}</Link>
+ <Link to={dogo.id} prefetch="intent">{dogo.name}</Link>
```

With that, Remix will automatically prefetch code, data, and other needed assets as soon as the user hovers or focuses the link. This sometimes makes the app feel instant. Like it was all downloaded from the start!

There are lots of other cool things we can do to enhance the user's experience with JavaScript. For a long time the status quo has been that JavaScript enables the experience. The cool thing about Remix is that the baseline is a functional app experience and we simply enhance the experience with JavaScript. There are two cool things about this:

1. The basic functions of the app work before the JS has started loading.
2. It's evidence that you don't need to think about application state management. 🤯

That's enough for now...

P.S. Make sure you add the `<Scripts />` back before doing the exercise because we kinda need JavaScript for this improved UX 😅

### 💪 Exercise

Let's start by adding a loading state to the create button in our form. Once the user submits the form, the button should change to "Creating..." and we should disable the button

### 🗃 Files

- `app/routes/posts/admin/new.tsx`

### 💯 Extra Credit

#### 1. Prefetch Posts

When users are reading our blog posts, we want them to load as quickly as possible. Let's add a `prefetch` to the links on our post listing page (`app/routes/posts/index.js`) and we may as well do the same thing for our admin page as well (`app/routes/posts/admin.tsx`).

## 07. Multiple Forms

### 📝 Notes

### 🤓 Background

Checkout this form:

```jsx
<form method="post">
  <label>
    Username: <input name="username" />
  </label>
  <label>
    Password: <input name="password" type="password" />
  </label>
  <button type="submit">Login</button>
  <button type="submit">Sign Up</button>
</form>
```

We have two buttons for a single form. Even if these were two different forms on the same page, they're going to post to the same server code. How does our server code know which one is being submitted?!

The answer is simple, `<button>`s can have a name and a value just like other form elements! No joke! Check this out:

```jsx
<button type="submit" name="intent" value="login">
  Login
</button>
<button type="submit" name="intent" value="signup">
  Sign Up
</button>
```

When the user clicks the "Sign Up" button, the form will get submitted with an `intent` value set to `signup`!

In a Remix context, it's important for you to know also that the transition.submission has all the form data as well, so we can use that to know which button was used to submit the form on the client as well:

```jsx
const navigation = useNavigation();
const isLoggingIn = navigation.formData.get("intent") === "login";
```

### 💪 Exercise

Let's add a "delete" button to our post creation page. We'll also change the "create" button for an "update" button for when we're editing an existing post.

Our backend already has the proper `deletePost` and `updatePost` functions so we just need to know when to call those.

We'll also want to make sure that the pending states are correct based on which button was clicked.

> Note: we moved `app/routes/posts/admin/new.tsx` to `app/routes/posts/admin/$slug.tsx` to reuse the creation form for editing posts. There's not too much that's Remix-specific here, so I did that for you. If you want to see a diff, run this:


```diff
git diff --no-index final/06-progressive-enhancement.extra-01-prefetch/app/routes/posts/admin/new.tsx exercise/07-multiple-forms/app/routes/posts/admin/\$slug.tsx
```

### 🗃 Files

- `app/routes/posts/admin/$slug.tsx`

## 08. Error handling

### 📝 Notes

### 🤓 Background

No matter how hard you try, you will experience errors in production. Not only is it inevitable (systems are imperfect), but we actually expect it to happen. For example, what if we were building Airbnb and someone went to a house listing that was deleted? They should get a 404 error in that case.

So the secret to errors is not only to try and prevent them from happening, but to make sure your code is resilient to them when they do happen.

In Remix, thanks to nested routing, we can have nice contextual errors because Remix allows us to say: "When this route or its children experience an error, render this instead!" And we communicate this via another export called `ErrorBoundary`. For example:

```jsx
export default function UhOhRoute() {
  // Oh no! "candy" is not defined! Should've used TypeScript 🙃
  return <div>{candy.eat}</div>;
}

export function ErrorBoundary({ error }: { error: Error }) {
  console.error(error);

  return <div>An unexpected error occurred: {error.message}</div>;
}
```

An important note is that Remix Error Boundaries bubble up to parent routes as well, so if an error happens in a child route but that route doesn't export an error boundary, then Remix will look for the closest ancestor that does and render that one.

And that's all there is to it. You might be familiar with the concept of Error Boundaries in React. This is similar with an important difference: it works on the server render as well.

Error Boundaries like this will handle unexpected errors, but what about expected errors like the `404` case? Well, Remix has another handy feature where you can `throw` a Response in your loader or action. When this happens, Remix will catch that and render the nearest `CatchBoundary` component.

```jsx
import type { LoaderFunctionArgs } from "@remix-run/node";

export async function loader({ params }: LoaderFunctionArgs) {
  const dogo = await getDogo(params.dogoId);
  if (!dogo) {
    throw new Response("not found", { status: 404 });
  }
  // ... etc...
}

export function CatchBoundary() {
  const caught = useCatch();

  // "caught" is the response that was through
  if (caught.status === 404) {
    return <div>Not found</div>;
  }

  throw new Error(`Unexpected caught response with status: ${caught.status}`);
}
```

Another cool bit of this feature is you can throw a redirect for situations where a user's not supposed to be somewhere at all. We'll get into doing this later when we get into auth:

```jsx
throw redirect("/login");
```

### 💪 Exercise

Let's handle errors more contextually for the `/posts/:slug`, `/posts/admin/:slug`, and the `/posts` routes by adding an `ErrorBoundary` to those files.

One tricky bit about this is that we have a "hidden" parent route. It's the parent of all the routes under `/posts`. You'll notice we don't have a `app/routes/posts.tsx` file, even though we have children under that directory. We're doing that because we don't need a parent to wrap that UI, so we're getting the default which is effectively:

```jsx
import { Outlet } from "@remix-run/react";
export default function HiddenParentRoute() {
  return <Outlet />;
}
```

So, if we want to add an error boundary to handle all unhandled errors under `/posts`, we'll need to create a `/app/routes/posts.tsx` file with that in it (to get the same functionality) and then we can add the `ErrorBoundary`.

### 🗃 Files

- app/routes/posts.tsx <-- you'll create this file
- app/routes/posts/$slug.tsx
- app/routes/posts/admin/$slug.tsx

### 💯 Extra Credit

#### 1. Catch Boundaries

We've handled unexpected errors. Now let's handle expected errors. If someone navigates to a post that doesn't exist, we should show a more helpful error message explaining what went wrong.

Add a `CatchBoundary` to both `/app/routes/posts/$slug.tsx` and `/app/routes/posts/admin/$slug.tsx`. Each has the following code:

```jsx
invariant(post, `Post not found: ${params.slug}`);
```

Swap that with:

```jsx
throw new Response("not found", { status: 404 });
```

The `CatchBoundary` should be looking for caught responses that are status `404` and display a helpful error message in that case.


#### 🗃 Files

app/routes/posts/$slug.tsx
app/routes/posts/admin/$slug.tsx

## 09. ENV Variables

###  📝 Notes

🤓 Background

- Because we have a `Node.js` server at runtime, most use cases for environment variables are satisfied by `process.env`. However, sometimes we have a need for an environment variable to be used in the client. For example, on my own site you can connect your KCD user account with your discord account. To do that I need to have my `DISCORD_CLIENT_ID` in both the server (for the server-render) and the client (for generating the discord connection URL).
- Because `process.env` is a `Node.js` runtime thing, it's not accessible in React components which will be hydrated on the client. So this won't work:

```jsx
function DiscordConnectionRoute() {
  return (
    <a href={createDiscordUrl(process.env.DISCORD_CLIENT_ID)}>
      Connect Discord
    </a>
  );
}
```

It'll work on the server, but it'll blow up on the client. And that's actually a good thing. We wouldn't want all our process.env to go to the client. Only specific information that's ok for the public to have access to.

So how do we get that data from our server to the client? Well, it turns out that environment variables are no different from any other data our server can send to our client. We can send it via the loader!

```jsx
import type { LoaderFunctionArgs } from "@remix-run/node";
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader({ request }: LoaderFunctionArgs) {
  return json({
    DISCORD_CLIENT_ID: process.env.DISCORD_CLIENT_ID,
  });
}

function DiscordConnectionRoute() {
  const { DISCORD_CLIENT_ID } = useLoaderData<typeof loader>();
  return <a href={createDiscordUrl(DISCORD_CLIENT_ID)}>Connect Discord</a>;
}
```

This works fine for simple cases, but I want to show you a handy trick. This trick allows us to access these environment variables globally. This is useful because environment variables are (or at least should be) immutable and globally accessible. So how do you make something globally accessible in the client? That's right, put it on window! So what you can do is add a script tag that sets a variable on the window. So check this out:

```jsx
import type { LoaderArgs } from "@remix-run/node";
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader({ request }: LoaderArgs) {
  return json({
    ENV: {
      DISCORD_CLIENT_ID: process.env.DISCORD_CLIENT_ID,
    },
  });
}

export default function App() {
  const data = useLoaderData<typeof loader>();

  return (
    <html>
      {/* ... */}
      <body>
        {/* ... */}
        <script
          dangerouslySetInnerHTML={{
            __html: `window.ENV = ${JSON.stringify(data.ENV)};`,
          }}
        />
      </body>
    </html>
  );
}
```

Now remember, our components run on both the client and the server, so if we really want this to be accessible globally, we need to set this on the server as well. So you can pop open your `app/entry.server.tsx` and set it on `global` for `Node.js`:

```jsx
global.ENV = {
  DISCORD_CLIENT_ID: process.env.DISCORD_CLIENT_ID,
};
```

Sweet! And now you can use that variable in any context and it will be globally available. So this should work (remember, this code runs both on the server and client):

```jsx
function DiscordConnectionRoute() {
  return <a href={createDiscordUrl(ENV.DISCORD_CLIENT_ID)}>Connect Discord</a>;
}
```

**Important note:** using globals like this is strongly discouraged for things that can change as the application runs or for things that are unique on a per-user basis. Also keep in mind that doing this makes these values accessible to the client, so make sure you only include values that you're ok the user knowing.

To take things further, we can package this up in a function and use that function instead of hard-coding it in two places. And we can make it typesafe as well so you'll get autocomplete when typing `ENV`. 🔥

💪 Exercise

This example is a tiny bit contrived. In an upcoming exercise, we want to make sure that only the admin user can create blog posts. We're going to configure who the admin user is via an environment variable called `ADMIN_EMAIL`. Normally you'd want this to only be a server-side environment variable (so `process.env.ADMIN_EMAIL` would be sufficient), but we're going to share this environment variable with the client-side.

Start out by creating an `env.server.ts` file that exports a getEnv function which returns all the environment variables we want shared between the frontend and the backend:

```jsx
export function getEnv() {
  return {
    ADMIN_EMAIL: process.env.ADMIN_EMAIL,
  };
}
```

Now you can use that to set the variables to both the server and the client.

### 🗃 Files

- app/env.server.ts <-- you create this
- app/entry.server.tsx
- app/root.tsx

### 💯 Extra Credit

#### 1. Help TypeScript help us

You need to do three things for fancy auto-complete and type checking of this global ENV variable:

1. Use `invariant` inside `getEnv` to assert that the environment variable is set
2. Get the type of ENV from `ReturnType<typeof getEnv>`
3. Stick `ENV` on the global for both server and browser code.

That third one is a bit trickier if you've never done it before so I'm going to give it to you:

```jsx
// App puts these on
declare global {
  var ENV: ENV;
  interface Window {
    ENV: ENV;
  }
}
```

Give that a shot and see if you can get autocomplete for global environment variables.

#### 🗃 Files

- `app/env.server.tsx`

## 10. Admin User

### 📝 Notes

### 🤓 Background

Most apps have some level of protection that certain routes need. For example, the user who tweeted a tweet should be able to edit that tweet, but not anybody else 🙄 Or maybe only an admin user can access and modify data. You may even have more complicated permissions and role-based authorization in place.

Whatever the case may be, you're probably going to need to protect a user's information.

It's easy to forget, but important to remember, that every route that exports a `loader` / `action` in your Remix app is an API endpoint. This means that people can hit those endpoints directly (via `cURL` for example). It's also important to remember that Remix runs all of your `loader`s concurrently.

This is why you can't just add protection to a parent route's loader and assume everything else is safe. Because that parent loader and all its children will be run at the same time. The child won't wait for the parent to do the check before starting to handle the request. Additionally, when the user navigates between sibling routes within a single parent, the parent route's loader won't even be called.

In the future, Remix will have a better answer for this specific use case. But yes, in a Remix app current, you have to add protection to every loader and action that needs it. If you really want, you can get around this today by integrating Remix with another routing server that supports middleware like express and do the check before the request makes it to Remix.

Ok, that said, how do we protect a route? Most of the time, you want to redirect the user to the `/login` route if they're trying to access something that requires a login. Remember how you can throw responses? Well, you can throw a redirect response and Remix will send that along to the user. This is awesome because it means you can perform a check and not worry about whether code will continue to execute. The throw stops the execution.

So, here's a simple example:

```jsx
import type { LoaderFunctionArgs } from "@remix-run/node";
import { json, redirect } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export async function loader({ request }: LoaderArgs) {
  const userIsAllowed = await userHasPermission(request, "emails.write");
  if (!userIsAllowed) {
    throw redirect("/login");
  }
  // otherwise, we're good... keep going...
  return json({});
}
```

Ok, so what makes this especially awesome though is that this means you can abstract that bit into a function:

```jsx
async function requireUserPermission(request: Request, permission: string) {
  const userIsAllowed = await userHasPermission(request, permission);
  if (!userIsAllowed) {
    throw redirect("/login");
  }
}

export async function loader({ request }: LoaderArgs) {
  await requireUserPermission("emails.write");
  return json({});
}
```

How cool is that!? I think it's pretty cool. Let's apply this to our own app.

- We want to protect every route under `/posts/admin`. First we want to make an abstraction for this in `app/server.session.ts`. Then we'll use that abstraction in the other routes. Remember to apply it to both the `loader` and `action` functions.
- NOTE, the `ADMIN_EMAIL` is set in the `.env` file as `kody@kcd.dev`. The `prisma/seed.ts` script sets that user's password to `kodylovesyou`.

NOTE, the `ADMIN_EMAIL` is set in the `.env` file as `kody@kcd.dev`. The `prisma/seed.ts` script sets that user's password to `kodylovesyou`.



- app/session.server.ts
- app/routes/posts/admin.tsx
- app/routes/posts/admin/index.tsx
- app/routes/posts/admin/$slug.tsx

### 💯 Extra Credit

#### 1. Hide admin links

Let's make a `useOptionalAdminUser` function in `app/utils.ts` which returns the user if they're the admin user (otherwise it returns undefined). We can use the existing `useOptionalUser` and compare the user's email to the `ENV.ADMIN_EMAIL`.

With that, we can conditionally render the link to `/posts/admin` in `app/routes/posts/index.tsx` and we can even add an edit link to the bottom of the posts `app/routes/posts/$slug.tsx` if we're signed in.

#### 🗃 Files

- app/utils.ts
- app/routes/posts/index.tsx
- app/routes/posts/$slug.tsx