# 02. Data Loading

## 📝 Notes

## 🤓 Background

Data loading is built into Remix.

If your web dev background is primarily in the last few years, you're probably used to creating two things here: an API route to provide data and a frontend component that consumes it. In Remix your frontend component is also its own API route and it already knows how to talk to itself on the server from the browser. That is, you don't have to fetch it.

If your background is a bit farther back than that with MVC web frameworks like Rails, then you can think of your Remix routes as backend views using React for templating, but then they know how to seamlessly hydrate in the browser to add some flair instead of writing detached jQuery code to dress up the user interactions. It's progressive enhancement realized in its fullest. Additionally, your routes are their own controller.

The way this works is through a `loader` function that you export from your route module. That goes hand-in-hand with a `useLoaderData` hook in your component. For example:

```tsx
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

export const loader = async ({ request }) => {
  const userId = await requireUserId(request);
  const flights = await getUserFlights(userId);
  return json({ flights }); // <-- send the data from your backend
};

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

One other tip is that the responsibility of the `loader` is to return a `Response` object and the `json` function is simply a helper for creating a `Response` object. It's effectively this:

```tsx
return new Response(JSON.stringify(data), {
  status: 200,
  headers: {
    "Content-Type": "application/json",
  },
});
```

## 💪 Exercise

We're going to start simple by putting our blog posts directly in the loader to start. Here are the blog posts we need to get from our server to the UI:

```tsx
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

Put that in your loader and then get that from the loader to the component using `json` and `useLoaderData`. You can render the posts using this JSX:

```tsx
<main>
  <h1>Posts</h1>
  <ul>
    {posts.map((post) => (
      <li key={post.slug}>
        <Link to={post.slug} className="text-blue-600 underline">
          {post.title}
        </Link>
      </li>
    ))}
  </ul>
</main>
```

## 🗃 Files

- `app/routs/posts/index.tsx`

## 💯 Extra Credit

### 1. Help TypeScript help us

The `json` utility is a generic function and accepts a type which you can use to make sure the types you're using are consistent. We also have a `LoaderFunction` type which we can use to get nice auto-complete on the `loader` function. Here's the same example as above but with types.

```tsx
import type { LoaderFunction } from "@remix-run/node";
import { json } from "@remix-run/node";
import { useLoaderData } from "@remix-run/react";

type LoaderData = {
  flights: Array<{ id: string; number: string }>;
};

export const loader: LoaderFunction = async ({ request }) => {
  const userId = await requireUserId(request);
  const flights = await getUserFlights(userId);
  return json<LoaderData>({ flights }); // <-- send the data from your backend
};

export default function FlightsRoute() {
  const { flights } = useLoaderData() as LoaderData; // <-- get the data into your UI
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

You might be concerned by the `as LoaderData`, but the fact is that the only way to be type-safe over an I/O boundary (like the network in a server-to-client communication) is by adding runtime type checks. We're not going to bother with that today, so we'll stick with `as LoaderData`. Just keep in mind that you never want your `LoaderData` to be anything that can't be serialized by a regular `JSON.stringify` (like dates and regex). This will be improved in the future.

For this extra credit, add `LoaderFunction` and create a `LoaderData` type for a more helpful coding companion in TypeScript.

## 🦉 Elaboration and Feedback

After the instruction, if you want to remember what you've just learned, then
fill out the elaboration and feedback form:

https://ws.kcd.im/?ws=Remix%20Fundamentals&e=2%3A%2002.%20Data%20Loading&em=
