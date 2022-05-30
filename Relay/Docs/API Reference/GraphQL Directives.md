# GraphQL 지시어

릴레이는 지시어를 통해 RelayCompiler에서 사용되어 적절한 런타임 부산물을 만들기 위해 GraphQL 문서에 추가적인 정보를 제공합니다.
이러한 지시어는 어플리케이션 코드에만 존재하고, GraphQL 서버에 전송되는 요청에서는 제거될 것입니다.

메모: 릴레이 컴파일러는 서버가 지원하는 지시어를 유지하므로, GraphQL 서버에 대한 요청의 일부로 유지되고 생성된 런타임 부산물이 변경되지 않습니다.

## @arguments

`@arguments`는 `@arguementDefinitions`를 정의한 프라그먼트에 인자를 넘겨줄 수 있는 지시어입니다.
```gql
query TodoListQuery($userId:ID){
  ...TodoList_list @arguments(count,$count,userID:$userID)## 여기서 인자를 넘겨줍니다.
}
```

> 최상단의 쿼리에서 하위 프라그먼트로 인자를 내려주기 위해 사용!

## @arugmentDefinitions

`@argumentDefinitions`는 프라그먼트에서 받는 인자들을 구체화하는 지시어입니다. 예를 들어
```gql
fragment TodoList_list on TodoList @argumentDefinitions(
  count: {type:"Int", defaultValue:10 }, # Optional argument
  userId: {type: "ID"}    #required argument
){
  title
  todoItems(userID:$userID, first:$count){ # 프라그먼트 인자는 여기서 변수처럼 사용됩니다.
    ...TodoItem_item
  }
}
```

## @connection(key:String!, filters:[String])

`usePaginationFragment` 와 같이 쓰일 때, 릴레이는 커넥션 필드가 `@connection` 지시어로 표시되길 기대합니다.
이는 `connection` 스펙을 준수하는 필드를 지시합니다.

### Connection
 커넥션 스펙을 구현한 필드입니다.
 


## @refectchable(queryName:String!)

`useRefetchableFragment`와 `usePaginationFragment`를 사용할 때에, 릴레이는 `@refetchable` 지시어를 기대합니다.
`@refetchable` 지시어는 
