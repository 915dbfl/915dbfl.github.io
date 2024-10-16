---
title: "[Compose] LazyColumn + Paging3 적용하기(1): 그 전에 RecyclerView에 Paging3 적용하기"
excerpt: "paging3가 뭔데? 어떻게 적용하는 건데?"
categories:
  - Compose
tag:
  - Paging3

last_modifeid_at: 2024-10-16
toc: true
toc_sticky: true
search: true
---

# 🔗 들어가며

이번 게시글에서는 `Paging3`에 대해 자세히 다뤄보고, Paging3를 적용하기 위해 xml에서는 어떤 작업이 필요했는지 살펴볼 것이다. 또한, 나아가 그런 Paging3를 컴포즈 프로젝트에서는 어떻게 사용할 수 있는지 쭉 정리해보려고 한다.

# 🔗 Paging 라이브러리가 뭔데?
> 로컬 저장소에서나 네트워크를 통해 대규모 데이터 세트의 데이터 페이지를 로드하고 표시할 수 있다. 이 방식을 사용하면 앱에서 네트워크 대역폭과 시스템 리소스를 모두 더 효율적으로 사용할 수 있다. paging 라이브러리의 구성요소는 권장 Android 앱 아키텍처에 맞게 설계되었다.

결국 네트워크를 통해 대규모의 데이터를 가져올 때 Paging 라이브러리를 활용할 수 있다. 그리고 그외 Paging 라이브러리가 주는 이점도 함께 살펴보자!

## ✏️ Paing 라이브러리의 이점
1. paging된 데이터 `메모리 내 캐싱`
2. 중복 요청 삭제 기능
3. 로드된 데이터 끝 감지 -> `무한 스크롤 기능 구현 가능`
4. flow, liveData 및 rxJava 지원
5. 새로고침 및 재시도 기능 포함 오류 처리 지원

## ✏️ 구성 요소
1. `PagingData`
  - 페이지로 나눈 데이터의 컨테이너
  - 데이터를 새로고침할 때마다 상응하는 `PagingData`가 별도로 생성된다.

2. `PagingSource`
  - 데이터의 스냅샷을 `PagingData`의 스트림으로 로드하기 위한 클래스
  - 네트워크 소스 및 로컬 데이터베이스를 포함한 단일 소스에서 데이터를 로드할 수 있다.

  - `Pager.flow`
    - PagingSource의 구성 방법을 정의하는 함수와 `PagingConfig`를 기반으로 `Flow<PagingData>`를 빌드한다.

3. `PagingDataAdapter`
- RecyclerView에 PagingData를 표시하는 `RecyclerView.Adapter`이다.
- `DiffUtil`를 활용해 데이터를 구별한다.
- (해당 부분은 compose에서는 필요 없어진다✨)

4. `RemoteMediator`(옵션)
- 계층된 데이터 소스의 페이징을 처리한다.(로컬 데이터베이스 캐시가 존재하는 네트워크 데이터 소스)

![image](/assets/images/paging_with_architecture.png)

(각 구성요소가 Android 아키텍처 어느 부분에 존재해야 하는지 한 번 확인해보면 다음과 같다.)

# 🔗 RecyclerView에 페이징 적용해보기!

크게 순서는 다음과 같다.
1. 의존성 추가
2. PagingSource 정의
3. PagingData 스트림 설정 - 기존 Repository 구현체 수정
4. RecyclerView 어댑터 정의
5. PagingData 사용하기!
6. refresh 적용하기!

## 1.[의존성 추가](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)

```kotlin
  // 앱의 build.gradle에 의존성 추가
  implementation "androidx.paging:paging-runtime:$paging_version"
```

## 2. PagingSource 정의

우선 PagingSource를 정의해야 한다. 해당 클래스는 위에서 설명했듯이 `데이터의 스냅샷을 PagingData 스트림으로 로드하기 위한` 클래스이이다.

다음과 같이 `PagingSource`를 상속해 클래스를 만든다. 이때 `load`, `getRefreshKey` 함수를 각각 구현해줘야 한다.

`load` 내부 구현 같은 경우, 네트워크 통신 값을 가져오는 로직을 추가적으로 작성해야 하며, `getRefreshKey` 값은 공식 문서에서 제공되는 로직에서 주로 크게 수정되지 않고 사용되는 것 같다.

```kotlin
class IssuePagingSource(
    // PagingSource는 단일 소스와 연결된다!
    private val remoteIssueDataSource: RemoteIssueDataSource,
    // api 요청을 할 때 필요한 파라미터를 추가한다.
    private val owner: String,
    //...
) : PagingSource<Int, IssueResponseVO>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, IssueResponseVO> {
        try {
            // page가 정의되지 않았다면 default 값은 1로 셋팅한다.
            val nextPageNumber = params.key ?: 1

            val response = remoteIssueDataSource.getIssues(
                owner = owner,
                //...
            )

            return LoadResult.Page(
                data = response.map { it.toVO() },
                prevKey = null,
                nextKey = nextPageNumber + 1
            )

        } catch (e: Exception) {
          // Handle errors in this block and return LoadResult.Error for
          // expected errors (such as a network failure).
          return LoadResult.Error(e)
        }
    }


    override fun getRefreshKey(state: PagingState<Int, IssueResponseVO>): Int? {
      // 가장 가까운 page key를 가져온다.
      // 새로고침되거나 무효화되었을 때 키를 반환하여 load 메서드로 전달한다.

      // either the prevKey or the nextKey; you need to handle nullability
      // here.
      //  * prevKey == null -> anchorPage is the first page.
      //  * nextKey == null -> anchorPage is the last page.
      //  * both prevKey and nextKey are null -> anchorPage is the
      //    initial page, so return null.
        return state.anchorPosition?.let { anchorPosition ->
            val anchorPage = state.closestPageToPosition(anchorPosition)
            anchorPage?.prevKey?.plus(1) ?: anchorPage?.nextKey?.minus(1)
        }
    }
}
```
- PagingSource<Key, Value>
  - Key는 데이터를 로드하는 데 사용되는 식별자를 정의
  - Value는 데이터 자체의 유형

  ex) `Int` 페이지 번호를 Retrofit에 전달하여 네트워크에서 Person 객체의 페이지를 로드한다면 `PagingSource<Int, Person>` 형태가 되는 것이다.

- `LoadParams`
  - 실행할 로드 작업에 관한 정보가 포함되어 있다. (로드할 키와 로드할 항목 수 등)
- `LoadResult`
  - 로드 작업의 결과가 포함된다.
  - 로드 성공 : `LoadResult.Page` 객체 반환
  - 로드 실패 : `LoadResult.Error` 객체 반환

그렇다면 PagingSource를 활용해 어떻게 page 데이터가 가져와지는 것일까??

![image](/assets/images/paging_load_data_withy_key.png)


그리고 페이지 로드 중 `HttpException`, `IOException` 등을 구분해서 에러 처리를 진행할 수도 있다.

```kotlin
catch (e: IOException) {
  // IOException for network failures.
  return LoadResult.Error(e)
} catch (e: HttpException) {
  // HttpException for any non-2xx HTTP status codes.
  return LoadResult.Error(e)
}
```

## 3. PagingData 스트림 설정 - 기존 Repository 구현체 수정

필자의 경우, `remoteIssueDataSource`에 접근해 필요한 api를 호출하는 작업을 Repository에서 진행하고 있었다. 따라서 해당 api 호출을 진행한 후, `PagingData` 스트림으로 변환하는 작업을 RepositoryImpl에서 진행하였다.

```kotlin
override suspend fun getRepoIssueList(
    owner: String,
    //..
): Flow<PagingData<IssueResponseVO>> {
    return Pager(
        pagingSourceFactory = {
            IssuePagingSource(
                remoteIssueDataSource = remoteIssueDataSource,
                owner = owner,
                //..
            )
        },
        config = PagingConfig(enablePlaceholders = false, pageSize = 30)
    ).flow
}
```

`Pager` 객체는 pagineSource 객체에서 `load` 메서드를 호출하여 `LoadParams` 객체를 제공하고 반환되는 `LoadResult` 객체를 수신한다.

여기서 `.flow.cacheIn(viewModelScope)`와 같이 캐싱 기능을 적용할 수 있다. 이를 적용하면 데이터 스트림을 공유 가능하도록 할 수 있으며 제공된 `coroutineScope`를 사용해 로드된 데이터가 캐싱될 수 있다.

## 4. RecyclerView 어댑터 정의

필자의 경우, 기존에 ListAdpater와 DiffUtil을 적용하고 있었다. 따라서 기존 DiffUtil를 유지하고 ListAdapter를 상속해 정의한 Adapter를 `PagingDataAdater`로만 수정해주었다.

아래는 공식 문서에 나온 `PagingDataAdapter` 및 `DiffUitl` 코드이다. 이를 동일하게 적용해주면 된다.

```kotlin
class UserAdapter(diffCallback: DiffUtil.ItemCallback<User>) :
  PagingDataAdapter<User, UserViewHolder>(diffCallback) {
  override fun onCreateViewHolder(
    parent: ViewGroup,
    viewType: Int
  ): UserViewHolder {
    return UserViewHolder(parent)
  }

  override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
    val item = getItem(position)
    // Note that item can be null. ViewHolder must support binding a
    // null item as a placeholder.
    holder.bind(item)
  }
}
```

```kotlin
object UserComparator : DiffUtil.ItemCallback<User>() {
  override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
    // Id is unique.
    return oldItem.id == newItem.id
  }

  override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
    return oldItem == newItem
  }
}
```

## 5. PagingData 사용하기!

이제 위에서 증의한 `PagingDataAdapter` 클래스 인스턴스를 RecyclerView와 연결해주고, `Flow<PagingData<T>>`를 collect해 실제 PagingData를 사용해보자!

```kotlin
val viewModel by viewModels<ExampleViewModel>()

val pagingAdapter = UserAdapter(UserComparator)
val recyclerView = findViewById<RecyclerView>(R.id.recycler_view)
recyclerView.adapter = pagingAdapter

// Activities can use lifecycleScope directly; fragments use
// viewLifecycleOwner.lifecycleScope.
lifecycleScope.launch {
  viewModel.flow.collectLatest { pagingData ->
    pagingAdapter.submitData(pagingData)
  }
}
```
`Flow<PagingData<T>>`를 collectLatest를 통해 기존 값의 소비가 마무리되지 않은 상태에서 새로운 값이 들어오면 최신의 값을 소비하도록 한다.

그리고 `PagingDataAdapter`에 `submitData`를 통해 pagingData를 전달해준다.

> 🚨 submitData() 이후 코드들은 늦게 실행될 수 있다!
> 
> submitData는 PagingSource가 비활성화되거나, adapter가 refresh되지 않는 이상 suspend한 후 리턴되지 않는다. 따라서 그 이후 코드를 배치했을 때 실행 시점이 예상보다 늦어질 수 있다.

## 6. Refresh 적용하기!

이제 정말 마지막 단계이다. 바로 `Refresh` 처리이다. 위 `PagingSource`를 정의할 때 `getRefreshKey` 함수를 정의했던 것이 기억날 것이다. 그렇다면 refresh 기능이 제공된다는 것인데,,, 어떻게 사용해야 할까?

```kotlin
pagingAdapter.refresh()
```

정말 간단하다. 필요한 상황에서 `PagingDataAdapter`에서 제공하는 `refresh()` 메소드를 호출하면 된다.

# 🔗 적용을 통해 느낀 점

## 1. 이전: 직접 무한 스크롤을 구현

이전에는 viewModel에서 page 데이터를 관리하고 있고, RecyclerView 끝에 도달했을 때 page 값을 사용해 다음 page의 api를 호출해 데이터를 가져왔었다. 이때 고려해야 할 부분은 크게 다음과 같았다.

1. `page 데이터를 올바르게 증가시기고, 감소하고, 초기화시키는 것 관리`
  - `데이터를 가져오는 작업의 성공/실패 유무에 따라 page 값 처리`
2. `새롭게 가져온 데이터를 이전 데이터에 추가하는 로직 별도 작성`
3. `RecyclerView 마지막 아이템이 보여질 때 api 호출`


이와 같이 직접 무한 스크롤을 구현한다면 고려해야 할 부분이 꽤 많다. 또한, 저렇게 구현했을 때 RecyclerView의 스크롤이 부드럽지 않고 뚝뚝 끊기는 느낌이 들었다.

## 2. paging 적용 후

우선 page 데이터를 별도로 관리하지 않아도 된다는 점이 좋았다. 언제 증가시키고, 감사하고, 초기화해야 하는지 고려하지 않아도 되고, 이를 Paging 라이브러리에서 처리해줘서 너무 편리했다.

또한, 무한 스크롤 동작이 무척 부드러웠다! 그 외에도 `가져온 페이지 데이터를 캐싱`하거나 `중복 요청`을 신경쓰지 않아도 된다는 점 등 paging을 통해서 얻을 수 있는 장점이 꽤 많았다!

## 3. 그렇다면 compose와도 paging 적용이 쉬울까?

이제는 점점 xml보다는 compsoe를 활용해 ui를 구현하고 있는데, 그렇다면 compose와 함께 paging을 활용하는 것도 가능할까?

(오히려 더 간단해진다!!✨✨✨)

다음 게시글에서는 compose + paging3 적용 방식을 다뤄보고자 한다.

## 🔗 참고자료
- [https://developer.android.com/topic/libraries/architecture/paging/v3-paged-data](https://developer.android.com/topic/libraries/architecture/paging/v3-paged-data)
- [https://developer.android.com/codelabs/android-paging?hl=ko#0](https://developer.android.com/codelabs/android-paging?hl=ko#0)