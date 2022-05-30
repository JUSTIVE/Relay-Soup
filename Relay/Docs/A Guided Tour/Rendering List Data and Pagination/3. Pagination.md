# 페이지네이션

커넥션을 따라 실제로 페이지네이션을 수행하려면, 우리는 `usePaginationFragment`에서 가능한 `loadNext` 함수를 사용해서 아이템의 다음 페이지를 가져와야 합니다.

```typescript
import type {FriendsListPaginationQuery} from 'FriendsListPaginationQuery.graphql';
import type {FriendsListComponent_user$key} from 'FriendsList_user.graphql';

const React = require('React');

const {graphql, usePaginationFragment} = require('react-relay');

const {Suspense} = require('React');

type Props = {
  user: FriendsListComponent_user$key,
};

function FriendsListComponent(props: Props) {
  const {data, loadNext} = usePaginationFragment<FriendsListPaginationQuery, _>(
    graphql`
      fragment FriendsListComponent_user on User
      @refetchable(queryName: "FriendsListPaginationQuery") {
        name
        friends(first: $count, after: $cursor)
        @connection(key: "FriendsList_user_friends") {
          edges {
            node {
              name
              age
            }
          }
        }
      }
    `,
    props.user,
  );

  return (
    <>
      <h1>Friends of {data.name}:</h1>
      <div>
        {(data.friends?.edges ?? []).map(edge => {
          const node = edge.node;
          return (
            <Suspense fallback={<Glimmer />}>
              <FriendComponent user={node} />
            </Suspense>
          );
        })}
      </div>

      <Button
        onClick={() => {
          loadNext(10)
        }}>
        Load more friends
      </Button>
    </>
  );
}

module.exports = FriendsListComponent;
```

위에서 일어나는 일들을 하나씩 살펴봅시다.

 - `loadNext` 함수는 서버로부터 커넥션 안의 아이템을 얼마나 더 가져올지 정의합니다.
 이 경우, 만약 `loadNext`가 호출된다면 리스트 내의 다음 10개의 친구를 가져올 것입니다.

- 만약 다음 아이템을 가져오는 요청이 완료되면, 커넥션은 자동으로 업데이트되고, 컴포넌트가 커넥션의 마지막 아이템과 함께 리렌더링 될것입니다.
이 경우, 이는 `friends` 필드가 항상 우리가 페칭해온 아이템으로 렌더링되는 것을 의미합니다.
기본적으로, 릴레이는 새 아이템을 컴포넌트에서 사용할 수 있도록 커넥션에 새 아이템을 자동으로 추가할 것입니다.
만약 다른 동작이 필요하다면, `페이지네이션 고급 사례` 구역을 확인하세요.

- `loadNext` 는 컴포넌트혹은 새 자식 컴포넌트를 서스펜스할 수 있습니다.
이는 위에 사용된 컴포넌트가 `<Suspense>` 바운더리 안에 있어야 함을 의미합니다.

가끔, 더 불러올 수 있는 항목이 있을 지 알아야 할 때가 있습니다.
이것을 수행하기 위해서는, `usePagination`에서 사용 가능한 `hasNext` 값을 사용할 수 있습니다.

```typescript
import type {FriendsListPaginationQuery} from 'FriendsListPaginationQuery.graphql';
import type {FriendsListComponent_user$key} from 'FriendsList_user.graphql';

const React = require('React');
const {Suspense} = require('React');

const {graphql, usePaginationFragment} = require('react-relay');

type Props = {
  user: FriendsListComponent_user$key,
};

function FriendsListComponent(props: Props) {
  // ...
  const {
    data,
    loadNext,
    hasNext,
  } = usePaginationFragment<FriendsListPaginationQuery, _>(
    graphql`...`,
    props.user,
  );

  return (
    <>
      <h1>Friends of {data.name}:</h1>
      {/* ... */}

      {/* Only render button if there are more friends to load in the list */}
      {hasNext ? (
        <Button
          onClick={/* ... */}>
          Load more friends
        </Button>
      ) : null}
    </>
  );
}

module.exports = FriendsListComponent;
```

- hasNext는 커넥션에 더 불러올 값이 존재하는 지 지시하는 boolean 값입니다.
이 정보는 다른 UI 컨트롤을 렌더링 할 때에 유용합니다.
위의 사례에서는 커넥션에서 더 불러올 값이 있을 때만 버튼을 보이도록 하였습니다.