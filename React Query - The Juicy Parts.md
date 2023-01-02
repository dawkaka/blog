So I came across react query when I was building my first startup and I checked it out. On the one of the pages it said if you handled `caching`, `pagination` etc, then you're good, so I was like "I've handled all these myself, I'm good" (I was wrong) and I ended up not using react-query, but then I kept seeing people praise react-query so I decided to use it for my second startup and ooh boy, it's so dang good that I'm glad I didn't use it at first because I now appreciate the problems it solves, the time it saves and the overall developer experience is excellent.
In this article, I'll talk about;

- Problems react-query solves
- Using react-query (focusing on the juicy parts)

## Problems React-query solves

In a typical full-stack application there is an `HTTP` request and response cycle going on. There are different request methods these include; `GET`, `POST`, `PUT`, `DELETE`, `PATCH` etc. These methods can be grouped into two groups i.e `fetching` data from server and `updating` server data. `GET` falls under fetching data and the rest fall under updating data, because of this, the foundation of `react-query` is also fetching and updating data. Just fetching and updating data is easy you can do it with `axios` or the browser `fetch` api, but in most applications just fetching and updating data is not enough to give users a good experience and also not optimizing this process will result to excess load on the server and so you want to `cache` data when necessary, when a user makes a `POST` request you `update already fetched data` or `invalidate cache` if necessary, also if data is too large, you fetch the data in chunks (pages) and so you'll need to handle `pagination`
in summary, react-query solves these common problems;

- querying (parallel queries, dependent queries etc)
- caching
- pagination
- mutations (updating data)
- invalidating and updating cached data (managing state)

React-query is a lot and so based on the above common problems I'll be focusing on the juicy parts.

## Using react-query (focusing on the juicy parts)

Based on my experience, I found the juicy parts to be;

- useQuery
- useQueries
- useMutation
- useQueryClient
- useInfiniteQuery

###1. useQuery
`useQuery` is the hook used to make http `GET` requests.
A basic usage example is

```ts
const { data, isLoading, isError } = useQuery(
  ['users'],
  () => axios.get('https://api.example.com').then((res) => res.data),
  {
    staleTime: 1000,
  }
);
```

The `useQuery` hook takes three arguments, you can also use the object syntax which is basically the same thing.

- query key
- query function and
- options

`useQuery(queryKey, queryFn, options)`

####Query key
The query key is what `react-query` uses to know which cached data to return when you're making the same request.`react-query` checks if there is data associated with the cache key and if the data is valid, then it returns that data.
How does react query know if data is stale or invalid?
you tell `react-query` when cached data should not be used by using the `staleTime` option and `invalidateQuery` method of `useQueryClient` we'll get into those later.
They query key can be a string, an array of strings and an array with nested objects. When a single string is used as query Key, it's converted to an array with that string as the only item.
Examples

```ts
//Valid query keys
queryKey = 'followers'; //Converted to ["followers"]
queryKey = ['followers', { user_name: 'john_doe' }];
queryKey = ['followers', 5, { user_name: 'jane_doe', verified: true }];
```

Query Keys are hashed deterministically, which simply means for query keys with nested objects, the order of the keys of those objects don't matter.
These query keys are all equal

```ts
useQuery(['users', { verified, followers_count }], queryFn, options);
useQuery(['users', { followers_count, verified }], queryFn, options);
```

However for array items the order is important

```ts
// These query keys are not equal
useQuery(['followers', user_name], queryFn, options);
useQuery([user_name, 'followers'], queryFn, options);
```

####Query function
The query function is the function that makes the `API` call and whatever data the function returns is what get assigned to `{data}` returned by the `useQuery` hook.
For example:

```ts
const fetchUsers = () => axios.get('https://api.example.com');
//Here "data" will be axios response
const { data } = useQuery(['users'], fetchUsers);
console.log(data); /*
    {
  // `data` is the response that was provided by the server
  data: {users: ["user1", "user2", "user3", "user4"]},
  status: 200,
  statusText: 'OK',
  headers: {},
  config: {},
  request: {}
}
*/
```

With this value of data if we want to access the data returned by the server (users) will have to use `data.data.users`
But when making `API` mostly is the data returned by the server that we want, so it's good to make your query function return that instead
Example

```ts
const fetchUsers = () =>
  axios.get('https://api.example.com').then((res) => res.data);

//Here "data" will be what server returns
const { data } = useQuery(['users'], fetchUsers);
console.log(data); // {users: ["user1", "user2", "user3", "user4"]
```

You don't need to catch errors, react-query will do that for you and return it as `error` in the useQuery return object, more on that later.

####Options
Options is the optional third parameter of the `userQuery` hook.
There are lot of options you can add to the Options object but as usual, we are going to focus on the juicy ones, you can check the react-query documentation for the rest.
The ones we are focusing on include;

- staleTime
- enabled
- retry
- onSuccess
- onError
- onSettledd

#####staleTime
The time in milliseconds after which data is considered stale. If set to `Infinity` the cached data will never be stale. It defaults to `0`.
When `staleTime` is 0, react-query will always re-fetch data whenever the component is mounted or when user focuses on window (when they focus after leaving the browser) but when `staleTime` is set, react-query by default will not re-fetch data until the time set has passed
Example:

```ts
const { data, isLoading } = useQuery(['users'], fetchUsers, {
  staleTime: 60 * 1000, // 1 minute
});
```

`react-query` by default will only re-fetch data if last fetch time was more than 1 minute ago.

Notice I keep using "by default"? this is because this behavior can be changed, you can tell `react-query` to re-fetch data on component mount or on window focus regardless of if data is stale or not.
Here is how

```ts
const options = {
  refetchOnMount: 'always',
  refetchOnWindowFocus: 'always',
};
```

#####enabled
The `enabled` option is used to tell react-query whether to fetch the data or not, react-query fetches data only if `enabled` is `true`. For example if you have a search feature on your app and you only want to make the `API` call when the user enters up to three characters. You can do that by using the `enabled` option.

Example:

```ts
const [query, setQuery] = useState('');

const { data, isLoading } = useQuery(['search'], searchUsers, {
  enabled: query.length > 2,
});
```

#####retry
`retry` option simply tell `react-query` what to do when a query fails.
if set to `true` react-query will retry infinitely until the query is successful.
if set to `false` react-query will not retry
alternatively you can set it to a number
eg: if set to `3` (`{retry: 3}`) react-query will retry 3 times when the query first fails

#####onSuccess, onError and onSettled
`onSuccess` is a function that gets called when a query is successful
`onError` is a function that gets called when a query fails and `onSettled` is a function that gets called when query is settled (whether it's successful or it failed)

```ts
const [query, setQuery] = useState('');

const { data, isLoading } = useQuery(['search'], searchUsers, {
  enabled: query.length > 2,
  onSuccess: (data) => {
    // Called when data is successfully fetched
  },
  onError: (error) => {
    // Called when query fails
  },
  onSettled: (data, error) => {
    // Always called regardless of the outcome
  },
});
```

####useQuery return Object
Went through how to use `useQuery`, now let's talk about what it returns. the object returned by the `useQuery` hooks has a lot of properties and will focus on the ...
Here there are

- `data` this is the data returned by the query function if query is successful
- `error` this receives the error as a result of a failed query (network error or error return by server)
- `isLoading` a boolean which is set to `true` if there's no cached data and no query attempt was finished yet and `false` otherwise
- `isError` a boolean which is set to `true` if query fails
- `isFetching` a boolean which is set to `true` whenever the queryFn is executing, which includes initial loading as well as background refetches and `false` otherwise.
- `isSuccess` is `true` if a query is successful

The difference between `isLoading` and `isFetching` is that `isLoading` is only set to `true` on first fetch (i.e no cache data, or query has not be executed before) but `isFetching` is set to `true` on first fetch and when fetching is happening in the background (refetching on mount, refetching on window focus etc)

##2. useQueries
`useQueries` is simply making two or more `useQuery` at the same time, and it returns and array of `useQuery` return object for each query
Usage:

```ts
const results = useQueries({
  queries: [
    { queryKey: ['user', user_name], queryFn: fetchUser, staleTime: Infinity },
    {
      queryKey: ['posts', user_name],
      queryFn: fetchUserPosts,
      staleTime: Infinity,
    },
  ],
});
```

In the above example `results` is an array of `useQuery` return value for each query inside the `queries` array

##3. useMutation
This hooks is used to make update `HTTP` requests. these requests make changes to the data on the server as a result it could make already cached data with `useQuery` in correct, we'll learn how to use `useMutation` in combination with `useQueryClient` to make changes to data on the server and invalidate cached data that may be affected by those changes.

Basic Usage:

```ts
const changeName = useMutation(
  (newName) => {
    return axios.put(`/user/name`, { name: newName });
  },
  {
    onSuccess: (data) => {
      const { message, type } = data.data;
    },
    onError: (err) => {
      console.log(err);
    },
  }
);
```

That is how you initialize a react-query mutation, But this doesn't make any request yet. calling `useMutation` returns and object with the following properties

- mutate
- isLoading
- isError
- isSuccess
- data
- error etc.

We'll be focusing mutate because the rest work similar to that of `useQuery`

`mutate` is function that when called makes the actual update http request.
with regards to the above example, to make api request to change a user's name will do something like this

```tsx
function ChangeName() {
  const changeName = useMutation(
    (newName) => {
      return axios.put(`/user/name`, { name: newName });
    },
    {
      onSuccess: (data) => {
        // Success same as that of useQuery
      },
      onError: (err) => {
        // Error same as that of useQuery
      },
    }
  );
  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        // Call mutate to invoke the useMutation function
        changeName.mutate(new FormData(event.currentTarget).get('new_name'));
      }}
    >
      <input name="new_name" />
      <button type="submit">Change</button>
    </form>
  );
}
```

That's mostly all you need to know about useMutation.
In the above code the use just change their name, assuming we've fetched user data with `useQuery` and used `["user"]` as our query key with staleTime of `Inifinity` this will mean that old name will continue to be showed even though the name has changed. We need a way to tell react-query that the cached data is not longer correct and it should be refetched. we'll discuss that in the next chapter

##4. useQueryClient
The `useQueryClient` hooks returns the `QueryClient` instance which is passed as a prop to `QueryClientProvider` which means it gives us access to the heart of react-query and we can do cools things with it. We'll focus on using it to invalidate queries and also update cached data

#####Invalidating queries
So in the above `useMutation` example we needed a way to tell react-query that the user data is no longer accurate and so it should refetch. This is how we do that using `useQueryClient`.

```tsx
function ChangeName() {
  const queryclient = useQueryClient();
  const changeName = useMutation(
    (newName) => {
      return axios.put(`/user/name`, { name: newName });
    },
    {
      onSuccess: (data) => {
        // This will invalidate the cached user data
        // and the user data will be refetched
        queryclient.invalidateQueries(['user']);
      },
      onError: (err) => {
        // Error same as that of useQuery
      },
    }
  );

  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        // Call mutate to invoke the useMutation function
        changeName.mutate(new FormData(event.currentTarget).get('new_name'));
      }}
    >
      <input name="new_name" />
      <button type="submit">Change</button>
    </form>
  );
}
```

What we just did above is good, but should be we refetch the user data just because the name has changed? NO. Since the change happened on the users browser and we know the new name we could just update the user's cached data with the new name by use `setQueryData` method of the `queryClient` you can use that to modify cached data.
Here's how:

```tsx
function ChangeName() {
  const queryclient = useQueryClient();
  const [newName, setNewName] = useState('');
  const changeName = useMutation(
    (newName) => {
      return axios.put(`/user/name`, { name: newName });
    },
    {
      onSuccess: (data) => {
        // This doesn't cause refetch and
        // we also have correct data
        queryclient.setQueryData(['user'], (oldData) => {
          return { ...oldData, name: new_name };
        });
      },
      onError: (err) => {
        // Error same as that of useQuery
      },
    }
  );

  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        // Call mutate to invoke the useMutation function
        changeName.mutate(newName);
      }}
    >
      <input
        name="new_name"
        value={newName}
        onChange={(e) => setNewName(e.target.value)}
      />
      <button type="submit">Change</button>
    </form>
  );
}
```

Another important aspect of invalidating queries is using the right query keys.

```ts
const queryclient = useQueryClient();

// The code below also invalidates queries with the following keys
// ["user", "followers"]
// ["user", {user_name}]
queryclient.invalidateQueries(['user']);
```

Alternative you can use the `exact` property to make sure only queries that exactly match the query key get invalidated

```ts
const queryclient = useQueryClient();

// Does not invalidate queries with the following keys
// ["user", "followers"]
// ["user", {user_name}]
queryclient.invalidateQueries(['user'], { exact: true });
```

##5. useInfiniteQuery
Assuming you're building an application that has comment section and a post has `2000` comments, when a user clicks to see the comments you wouldn't want to send all `2000` comments to the user reason being the user can not read all `2000` comments, you'll be using too much bandwidth and it'll take a long time to load, instead you'll want to fetch the comments in chunks say `1-10` and when user clicks `load more` or has finished reading those (when the last comment is in view) you then fetch `11-20`, then `21-30` and so on. You can do this with `axios` and `useState` but react-query made incredibly easy.
Like the `useQuery` hook, `useInfiniteQuery` takes `queryKey`, `queryFn` and `options` you can also box `queryKey` and `queryFn` in the options object and it's also what the react-query team recommends.
`useInfiniteQuery` also takes another important option which is `getNextPageParam` and it also returns an important function called `fetchNextPage`. These two functions are very important when using `useInfiteQuery`

Usage:

```ts
const fetchComments = ({ pageParam = 0 }) =>
  axios
    .get(`${BASEURL}/comments/${postID}/${pageParam}`)
    .then((res) => res.data);

const { data, fetchNextPage, hasNextPage, isFetching, isFetchingNextPage } =
  useInfiniteQuery(['comments', { postID }], fetchComments, {
    // Function that tells react-query the page to fetch next
    getNextPageParam: (lastPage, pages) => {
      if (lastPage.pagination.isEnd) {
        return undefined;
      }
      return lastPage.pagination.next;
    },
    staleTime: Infinity,
  });
```

Let's break the above code down starting with `pageParam`
`pageParam` is cursor or the page we're fetching, for the initial fetch, it's 0 or 1 depending on you handle pages on the backend. So what about the subsequent queries how does react-query now the page it should fetch? that's what the `getNextPageParam` does, It is a callback function that react-query passes `lastPage` and `pages` to and then calls the function whatever the function returns becomes the `pageParam` for the next query, if it returns`undefined` it means we've reached the last page and there are no more pages to fetch, If the `getNextPageParam` returns undefined, the `hasNextPage` property of the returned value of `useInfiniteQuery` is set to `false` otherwise it's set to `true`. You can use the `hasNextPage` value to know whether to show `load more` button or not.
the `lastPage` parameter is the data returned by the server after the last fetch and `pages` is data of all fetches (pages)
do determine what page to fetch next, the server normally returns something like `{comments: [], pagination:{next:11, isEnd: false}}`
So the get the next `pageParam` we use `lastPage.pagination.next` and if the `isEnd` is true we return `undefined` to tell react-query that we've reached the end of the comments. Once you have your `getNextPageParam` function set up all you need to do to fetch next page is to call the `fetchNextPage` property of the `useInfiniteQuery` return object. So when user clicks on `load more` you call `fetchNextPage`

Example putting it all together

```tsx
function Comments() {
  const fetchComments = ({ pageParam = 0 }) =>
    axios.get('/post/comments/${postID}/${pageParam}').then((res) => res.data);
  const { data, fetchNextPage, hasNextPage, isFetching } = useInfiniteQuery(
    ['comments', { postID }],
    fetchComments,
    {
      getNextPageParam: (lastPage, pages) => {
        if (lastPage.pagination.end) {
          return undefined;
        }
        return lastPage.pagination.next;
      },
      staleTime: Infinity,
    }
  );
  // Since data returns an array of pages
  // you'll need to combine all the pages into one array
  // so you can map all comments and render on user's screen
  let comments: any[] = [];
  if (data?.pages) {
    for (let page of data?.pages) {
      comments = comments.concat(page.comments);
    }
  }

  return (
    <div>
      {comments.map((comment) => {
        return (
          <Comment
            key={comment.id}
            user_name={comment.user_name}
            comment={comment.comment}
            date={new Date(comment.created_at)}
            likes_count={comment.likes_count}
          />
        );
      })}
      {hasNextPage && (
        <div style={{ height: '10px' }}>
          {!isFetching ? (
            <button onClick={() => fetchNextPage()}>Load more</button>
          ) : (
            <Loading />
          )}
        </div>
      )}
    </div>
  );
}
```

Alternatively, you can use `intersectionObserver` to automatically load comments so user doesn't have to click on `load more` making it an infinite scroll.
Example:

```tsx
function Comments() {
  const fetchComments = ({ pageParam = 0 }) =>
    axios.get('/post/comments/${postID}/${pageParam}').then((res) => res.data);
  const { data, fetchNextPage, hasNextPage, isFetching } = useInfiniteQuery(
    ['comments', { postID }],
    fetchComments,
    {
      getNextPageParam: (lastPage, pages) => {
        if (lastPage.pagination.end) {
          return undefined;
        }
        return lastPage.pagination.next;
      },
      staleTime: Infinity,
    }
  );
  // Since data returns an array of pages
  // you'll need to combine all the pages into one array
  // so you can map all comments and render on user's screen
  let comments: any[] = [];
  if (data?.pages) {
    for (let page of data?.pages) {
      comments = comments.concat(page.comments);
    }
  }

  // Load comments automatically use intersectionObserver api
  const loaderRef = useRef(null);
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          fetchNextPage();
        }
      },
      { threshold: 0.5 }
    );

    if (loaderRef.current) {
      observer.observe(loaderRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div>
      {comments.map((comment) => {
        return (
          <Comment
            key={comment.id}
            user_name={comment.user_name}
            comment={comment.comment}
            date={new Date(comment.created_at)}
            likes_count={comment.likes_count}
          />
        );
      })}
      {hasNextPage && (
        <div ref={loaderRef} style={{ height: '10px' }}>
          {isFetching && <Loading />}
        </div>
      )}
    </div>
  );
}
```

If you are not familiar with `intersectionObserver` you can learn more about it [here](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

##Conclusion
The topics we've discussed in this blog are the ones I found most useful, If you find yourself trying to do something not covered in this article, there is a high chance react-query allows you to do that, check the [react-query documentation](https://tanstack.com/query/v4/docs/react/overview) and you'll find it, the react-query team thought of everything.
