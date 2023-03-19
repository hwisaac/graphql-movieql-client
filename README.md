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

- import 구문을 사용하여 ApolloClient, gql, InMemoryCache를 불러옵니다.
- ApolloClient 인스턴스를 생성합니다. uri 속성은 GraphQL API의 엔드포인트를 설정하며, cache 속성은 Apollo Client에서 사용되는 캐시를 설정합니다.
- client.query() 메서드를 호출하여 GraphQL API에 쿼리를 전송합니다. 이때 gql 템플릿 리터럴을 사용하여 쿼리를 작성합니다.
- 쿼리의 결과는 then() 메서드를 사용하여 처리합니다. 이 코드에서는 결과를 콘솔에 출력합니다.
- client 객체를 export하여 다른 모듈에서 사용할 수 있도록 합니다.

<hr />

```js
// index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import client from './client';
import { ApolloProvider } from '@apollo/client';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </React.StrictMode>
);
```

- import { ApolloProvider } from '@apollo/client'; 구문을 사용하여 ApolloProvider 컴포넌트를 불러옵니다.
- ApolloProvider 컴포넌트를 root.render() 메서드로 감싸서 App 컴포넌트에서 client 객체를 사용할 수 있도록 합니다.
- client 객체를 ApolloProvider 컴포넌트의 client 속성에 전달하여 Apollo Client를 초기화합니다.

```js
// routes/Movies.jsx
import { gql, useApolloClient } from '@apollo/client';
import { useEffect, useState } from 'react';

export default function Movies() {
  const [movies, setMovies] = useState([]);
  const client = useApolloClient(); // provider 로 전달된 client
  useEffect(() => {
    client
      .query({
        query: gql`
          {
            allMovies {
              title
              id
            }
          }
        `,
      })
      .then((results) => setMovies(results.data.allMovies));
  }, [client]);
  return (
    <ul>
      {movies.map((movie) => (
        <li key={movie.id}>{movie.title}</li>
      ))}
    </ul>
  );
}
```

위 코드는 React 컴포넌트를 사용하여 GraphQL API에서 영화 목록을 가져오는 예제입니다.

- import 구문을 사용하여 gql, useApolloClient, useEffect, useState를 불러옵니다.
- Movies 컴포넌트를 정의합니다. useState를 사용하여 movies 상태와 setMovies 함수를 생성합니다.
- useApolloClient 훅을 사용하여 Apollo Client 객체를 가져옵니다.
- useEffect 훅을 사용하여 컴포넌트가 마운트될 때 Apollo Client를 사용하여 GraphQL API에 쿼리를 전송합니다.
- 쿼리의 결과는 setMovies 함수를 사용하여 movies 상태에 저장됩니다.
- movies 배열을 map() 메서드로 반복하면서 각각의 영화를 <li> 요소로 렌더링합니다.

즉, 위 코드는 Apollo Client를 사용하여 GraphQL API에서 영화 목록을 가져와 React 컴포넌트에서 렌더링하는 예제입니다. useApolloClient 훅을 사용하여 Apollo Client 객체를 가져와서 query() 메서드를 호출하여 쿼리를 전송하고, useState 훅을 사용하여 상태를 관리합니다.

### Movies.js 수정

```js
import { gql, useQuery } from '@apollo/client';

const ALL_MOVIES = gql`
  query getMovies {
    allMovies {
      title
      id
    }
    allTweets {
      id
      text
      author {
        fullName
      }
    }
  }
`;

export default function Movies() {
  const { data, loading, error } = useQuery(ALL_MOVIES);
  if (loading) {
    return <h1>Loading...</h1>;
  }
  if (error) {
    return <h1>Could not fetch :(</h1>;
  }
  return (
    <ul>
      <h1>Movies</h1>
      {data.allMovies.map((movie) => (
        <li key={movie.id}>{movie.title}</li>
      ))}
      <h1>Tweets</h1>
      {data.allTweets.map((tweet) => (
        <li key={tweet.id}>
          {tweet.text}/by: {tweet.author.fullName}
        </li>
      ))}
    </ul>
  );
}
```

- 쿼리가 ALL_MOVIES에서 allMovies와 allTweets를 모두 가져오도록 변경되었습니다. 이는 영화와 트윗이 서로 다른 도메인에 속해 있기 때문에 좋은 구조가 아닙니다. 서로 다른 쿼리를 작성하거나 서로 다른 컴포넌트를 만드는 것이 좋습니다.
- 로딩 상태를 처리하는 방식이 변경되었습니다. 변경 전에는 useEffect와 useState를 사용하여 로딩 상태를 관리하였지만, 변경 후에는 useQuery의 loading 속성을 사용하여 로딩 상태를 처리합니다.
- 데이터가 없는 경우에 대한 처리가 변경되었습니다. 변경 전에는 useState를 사용하여 초기값을 빈 배열로 설정하였지만, 변경 후에는 data의 존재 여부를 확인하여 렌더링하도록 수정하였습니다.
