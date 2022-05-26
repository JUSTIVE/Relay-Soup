# 쿼리 만들기

### 읽을 거리

- [GraphQL에서의 뮤테이션과 쿼리](https://graphql.org/learn/queries/)
- [릴레이의 가이드 투어: 쿼리](https://relay.dev/docs/guided-tour/rendering/queries/)
- [리액트 문서: 데이터 페칭의 서스펜스](https://17.reactjs.org/docs/concurrent-mode-suspense.html)

## 쿼리 만들기

첫 쿼리를 만들어 봅시다!

RescriptRelay의 쿼리는 `%relay()` 확장 노드에 의해 정의되어집니다. 첫 쿼리를 설정하고 결과를 보여줄 컴포넌트를 만들어 봅시다.

```re
/* UserProfile.res */
module Query = %relay(`
  query UserProfileQuery($userId:ID!){
    userById(id:$userId){
      firstName
      lastName
    }
  }
`)
```

> 이름에 주목하세요. Relay의 규칙에 따라, 모든 쿼리는 `<모듈 이름><선택적으로_아무거나>Query` 로 지어져야 합니다. 여기서의 모듈 이름은 파일 이름을 의미하며, ReScript 모듈 이름이 아닙니다.
> 따라서 `UserProfile.res` 파일의 경우, 이 파일의 모든 쿼리들의 이름은 중첩된 모듈이건 아니건 반드시 `UserProfile`로 시작해야 합니다. 모든 쿼리들의 이름은 `Query` 로 끝나야 합니다.

> VScode 코드를 사용하고 있나요? 전용 VScode 확장은 `> Add Query` 명령을 통해 새 쿼리를 만들 수 있게 합니다.

이는 RescriptRelay에서의 쿼리를 정의한 모습입니다. 이는 쿼리를 여러가지 방법으로 사용할 수 있게 하는 여러 개의 훅들과 함수들을 노출시킨 모듈로 변환될 것입니다.(보다 자세한 결과물을 원한다면, [여기](#api-reference)에서 확인할 수 있습니다.). 이 쿼리를 사용하는 컴포넌트를 빠르게 봅시다.

```re
@react.component
let make = (~userId) =>{
  let queryData = Query.use(
    ~variables={
      userId:userId
    },
    ()
  )

  switch queryData.userById{
  | Some(user) =>
      <div>
        {`${user.firstName} ${user.lastName}`->React.string}
      </div>
  | None => React.null
  }
}
```

딱히 멋진 것은 없습니다. 우리는 GraphQL 쿼리에 정의된 대로 변수 `userId`를 이용하여 `Query.use`를 호출할 것입니다. `use`는 서버로 쿼리를 보내고, 데이터를 컴포넌트로 전달하는 리액트 훅입니다.
이는 `suspense` 가통합되어 있으며, 이는 곧 데이터가 도착하지 않았다면 컴포넌트를 suspend 할 것입니다. 이 쿼리는 변수를 변경할 때에 다시 요청될것이며, 쿼리를 설정할 수 있는 수많은 방법들이 있습니다.
`Query.use`에 넘겨질 수 있는 모든 것들을 [여기](#use) 에 있는 레퍼런스 전문에서 확인해 보세요.

쿼리와 상호작용하는 것은 타입-안전합니다. 이는 `variables`와 `queryData`의 타입들이 GraphQL 연산에 정의된 것들과 일치할 것임을 의미합니다. 이는 또한 Rescript 컴파일러가 함수에 어떤 것들을 넘겨줄지, 그리고 받은 데이터를 어떻게 사용할 지 가이드 해준다는 것을 의미합니다.

여기 이제 첫 쿼리가 있습니다! 이 페이지를 계속 읽어 [API 레퍼런스](#api-reference)를 비롯한 쿼리에 관한 정보를 더 보거나, 다음 장인 [프라그멘트 사용하기]("../Using Framgents.md")로 넘어가세요.

## 선 로드된 쿼리들

`Query.use()`를 사용하는 것은 **게으릅니다**, 이는 컴포넌트가 실제로 렌더링되기 전까지 Relay는 데이터를 가져오지 않는다는 것을 의미합니다. Relay에는 **선로드된 쿼리** 라는 개념이 있으며, 이는 UI가 렌더링되어 쿼리를 발동시키는 대신 쿼리를 가능한 한 가장 바르게 로딩할 수 있다는 것을 의미합니다.

> [Relay 문서의 이 장](https://relay.dev/docs/api-reference/use-preloaded-query/)을 읽어 선로드된 쿼리에 대한 개요를 더 보세요.

RescriptRelay에서는 쿼리를 포함한 모든 `%relay()` 노드는 `useLoader` 훅을 자동으로 생성합니다. 이 훅은 3개로 구성된 튜플을 반환합니다. `(option(queryRef), loadQuery, disposeQuery)`.

1. `option(queryRef)` - 모든 쿼리 레퍼런스의 옵션입니다.  
   이 쿼리 레퍼런스는 `Query.usePreloaded`에 `let queryData = Query.usePreloaded(~queryRef=queryRef,())`와 같이 넘겨져 쿼리의 데이터를 가능한 한 가장 빨리 받아올 수 있습니다.

2. `loadQuery` - 이 쿼리를 위해 데이터를 가져오기 시작할 함수입니다.  
   `loadQuery(~variables={...},~fetchPolicy=?, ~networkCacheConfig=?,())` 와 같이 사용할 수 있습니다.
   이 함수를 호출한다면, `queryRef`가 채워질 것이고, queryRef를 usePreloaded에 전달할 수 있습니다.
3. `disposeQuery` - 쿼리 레퍼런스를 수동으로 파기할 수 있는 함수입니다.
   이 함수를 호출하면 `option<queryRef>`를 `None`으로 만들 것입니다.

```re
// SomeComponent.res
module Query = %relay(`
  query SomeComponentQuery($userId: ID!){
    user(id:$userId){
      ...SomeUserComponent_user
    }
  }
`)

@react.component
let make = (~queryRef) => {
  let queryData = Query.usePreloaded(~queryRef,())
  //Use the data for the query here
}

@react.component
let make = (~userId)=>{
  let (queryRef,loadQuery,_disposeQuery) = SomeComponent.Query.useLoader()

  switch queryRef {
  | Some(queryRef) => <SomeComponent queryRef/>
  | None =>
    <button onClick={_=>loadQuery(~variables={id:userId},())}>
      {`See full user`->React.string}
    </button>
  }
}
```

무슨 일이 일어나고 있는지 하나씩 분석해 봅시다.

1. 쿼리를 가지는 `<SomeComponent/>` 라는 컴포넌트를 만들었습니다. 그러나, 컴포넌트는 그 쿼리를 직접 만들지 않습니다.
   대신, 컴포넌트가 실제로 렌더링 될 때까지 기다리지 않고 렌더링하는 부모 컴포넌트가 해당 쿼리를 최대한 빨리 로드하기 시작하기를 기다립니다.
2. `<SomeOtherComponent/>`는 눌렸을 때 `<SomeComponent/>`를 위한 쿼리를 로딩하기 시작하는 버튼을 반환합니다. 이는 쿼리가 물리적으로 가장 빠른 시간(사용자가 버튼을 클릭한 순간) 로딩을 시작한다는 것을 의미합니다.
렌더링이나, 상태 업데이트 등을 기다리지 않고도요. 데이터는 가능한 한 빠르게 요청됩니다.

게으른 접근 방식을 사용하는 것보다 권장되는 매우 유용한 패턴입니다. 요컨대, Query.useLoader를 가능한 한 많이 사용하십시오.