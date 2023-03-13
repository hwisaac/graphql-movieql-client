# graphql tutorial

## apollo client

> ReactJS 의 apollo client 가 graphql 서버와 통신하도록 구성하자.

graphql 의 api 의 url 을 써서 client를 구성할 수 있다.

```js
// client.js
import { ApolloClient, gql, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/',
  cache: new InMemoryCache(),
});

client
  .query({
    query: gql`
      {
        allMovies {
          title
        }
      }
    `,
  })
  .then((data) => console.log(data));

export default client;
```

```js
// index.js
```
