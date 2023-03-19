# graphql tutorial

DEMO : https://shimmering-blancmange-345510.netlify.app/

> GraphQL , Apollo client 로 영화 웹앱 만들기

ReactJS 의 apollo client 가 GraphQL 서버와 통신하도록 구성하자.

## apollo client

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

## local only fields

Apollo Client에서 Local Only Fields는 Apollo Client의 캐시에 저장되는 로컬 상태 값을 의미합니다. 이는 GraphQL 서버에서 반환되는 데이터 외에도, 클라이언트 측에서만 사용되는 데이터를 추가할 수 있는 방법입니다.

Local Only Fields를 사용하면, 서버에서 반환되는 데이터에 포함되어 있지 않은 정보를 캐시에 저장하고, 클라이언트에서 이 정보를 사용할 수 있습니다. 이를 통해, Apollo Client를 사용하여 간단한 상태 관리를 수행할 수 있습니다.

예를 들어, 애플리케이션에서 현재 사용자의 인증 상태를 관리하고자 할 때, Local Only Fields를 사용하여 인증 상태를 캐시에 저장하고, 이를 기반으로 UI를 렌더링할 수 있습니다. 이렇게 하면 서버와의 불필요한 통신을 줄이고, 성능을 향상시킬 수 있습니다.

Local Only Fields를 정의하려면, Apollo Client의 cache.writeQuery() 메서드를 사용하여 캐시에 데이터를 저장하면 됩니다. 이를 통해, 로컬 상태 값을 쉽게 관리할 수 있습니다.

```js
// isLiked 필드는 로컬 필드이다.
const GET_MOVIE = gql`
  query getMovie($movieId: ID!) {
    movie(id: $movieId) {
      id
      title
      medium_cover_image
      rating
      isLiked @client
    }
  }
`;
```

> 로컬 필드는 클라이언트에서만 사용되는 필드를 의미하며, 서버에서 직접 제공하지 않는 데이터를 처리할 때 유용합니다. isLiked 필드는 서버에서 제공하는 데이터가 아니라, 사용자가 영화를 좋아요로 표시했는지를 나타내는 로컬 데이터이기 때문에 Local Field로 분류됩니다.

### cache.writeFrament() 메서드

> `cache.writeFragment()` 함수는 Apollo Client에서 제공하는 함수 중 하나로, 로컬 캐시에 있는 데이터를 업데이트할 때 사용됩니다.

```js
// movie.jsx

export default function Movie() {
  const { id } = useParams();
  const {
    data,
    loading,
    client: { cache },
  } = useQuery(GET_MOVIE, {
    variables: {
      movieId: id,
    },
  });

  const onClick = () => {
    cache.writeFragment({
      id: `Movie:${id}`,
      // 어떤 값이 바뀔지 미리 알려주기
      fragment: gql`
        fragment MovieFragment on Movie {
          isLiked
        }
      `,
      // 무슨 값으로 바뀔 것인지 알려주기
      data: {
        isLiked: !data.movie.isLiked,
      },
    });
  };
  return (
    <Container>
      <Column>
        <Title>{loading ? 'Loading...' : `${data.movie?.title}`}</Title>
        <Subtitle>⭐️ {data?.movie?.rating}</Subtitle>
        <button onClick={onClick}>
          {data?.movie?.isLiked ? 'Unlike' : 'Like'}
        </button>
      </Column>
      <Image bg={data?.movie?.medium_cover_image} />
    </Container>
  );
}
```

- `useParams` 훅을 사용하여 현재 페이지 URL의 파라미터를 가져옵니다. 그리고 `useQuery` 훅을 사용하여 `GET_MOVIE` 쿼리를 호출합니다. `GET_MOVIE` 쿼리는 `id` 변수를 받아서 해당 `ID`의 영화 정보를 반환합니다.
- `cache.writeFragment()` 메서드를 사용하여 클라이언트 캐시의 로컬 상태 값을 업데이트합니다. `writeFragment()` 메서드는 `Apollo Client`의 캐시에 새로운 정보를 추가하거나 기존 정보를 업데이트할 때 사용됩니다. 이 메서드를 사용하여 `isLiked` 필드의 값을 토글(toggle)합니다.
  - `id`: `Movie:${id}` 형태로 지정되어 있으며, 캐시에서 업데이트할 데이터를 찾는 데 사용됩니다. 여기서 `id는` GraphQL 서버에 있는 `ID`와는 다르며, 로컬 캐시에 저장된 데이터의 `ID`입니다.
  - `fragment`: `MovieFragment`이라는 이름을 가진 isLiked 필드만을 가지고 있는 프래그먼트입니다. 이 프래그먼트는 `Movie` 타입의 필드 중 `isLiked` 필드만을 가져옵니다.
  - `data`: `isLiked` 필드를 새로운 값으로 업데이트합니다. `!data.movie.isLiked` 구문을 통해 `data` 객체에 있는 `movie` 필드의 `isLiked` 값을 반전시켜서 업데이트합니다.
- 마지막으로, 반환되는 JSX에서 영화 정보와 좋아요 버튼을 렌더링합니다. 버튼 클릭 시 `onClick` 함수가 호출되어 `cache.writeFragment()` 메서드가 실행됩니다. `data.movie`는 서버에서 가져온 영화 정보를 나타내는 객체이며, `data.movie.isLiked`는 현재 영화의 좋아요 상태를 나타냅니다. 따라서 버튼 텍스트는 `data.movie.isLiked` 값에 따라 "`Like`" 또는 "`Unlike`"로 변경됩니다.
