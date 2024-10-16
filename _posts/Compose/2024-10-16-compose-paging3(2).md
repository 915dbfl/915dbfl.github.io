---
title: "[Compose] LazyColumn + Paging3 적용하기(2): 이제 Compose + Paging3 적용해보자!"
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

[이전 게시글](https://915dbfl.github.io/compose/compose-paging3(1)/)에서 RecyclerView에 Paging3을 적용하는 방법에 대해 살펴보았다. 그렇다면,,,
이제는 Compose에 Paging3를 적용해보고자 한다!!

이미 xml 기반의 뷰에 Paging3를 적용해뒀다면 이전하는 작업은 정말 간단할 것이다!! 우선 다시 한 번 Paging 라이브러리란 무엇인지, 어떤 이점을 가지고 있는지 짚고 넘어가자!

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

# 🔗 RecyclerView에 페이징 적용하는 과정

RecyclerView에 Paging을 적용하기 위해서 진행해야 했던 작업들을 살펴보자!

1. 의존성 추가
2. PagingSource 정의
3. PagingData 스트림 설정 - 기존 Repository 구현체 수정
4. RecyclerView 어댑터 정의
5. PagingData 사용하기!
6. refresh 적용하기!

# 🔗 Compose(LazyColumn)에 페이징 적용하는 과정

compose에 페이징을 적용하는 과정은 RecyclerView에 페이징을 적용하기 위해 필요했던 과정과 유사하다.

1. 의존성 추가
2. PagingSource 정의
3. PagingData 스트림 설정 - 기존 Repository 구현체 수정 (여기까지 동일!!)
4. ~~RecyclerView 어댑터 정의~~
5. PagingData 사용하기 With LazyColumn
6. refresh 적용하기!

Compose에서는 `LazyColumn`을 활용하기 때문에 별도의 Adapter가 필요하지 않다! 따라서 과정이 보다 간단해진 것을 확인할 수 있다!

그렇다면 의존성 추가부터 refresh 적용까지 그 과정을 한 번 살펴보자! 2, 3번 과정은 이전 게시글에서 자세히 설명되어 있기 때문에 이번 게시글에서는 간단하게만 다루고 넘어갈 것이다.

[[Compose] LazyColumn + Paging3 적용하기(1): 그 전에 RecyclerView에 Paging3 적용하기](https://915dbfl.github.io/compose/compose-paging3(1)/)


## 1.[의존성 추가](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)

```kotlin
  // 앱의 build.gradle에 의존성 추가
  implementation "androidx.paging:paging-runtime:$paging_version"
  // compose를 적용하기 위해 추가로 필요한 의존성
  implementation "androidx.paging:paging-compose:$paginer-compose-version"
```

> ⭐️ Note: If your app uses Compose for its UI, use the `androidx.paging:paging-compose` artifact to integrate Paging with your UI layer instead. To learn more, see the API documentation for `collectAsLazyPagingItems()`.

즉, 기존에는 Paging과 UI layer를 연동하기 위해 `PagingDataAdapter`를 활용했지만 compose에서는 LazyColumn을 적용할 경우 adpater가 필요하지 않기 때문에 별도의 `collectAsLazyPagingItems()`를 활용하게 된다.

## 2. PagingSource 정의

(RecyclerView를 구현했을 때와 동일한 내용이니 이미 적용되어 있다면 패스하자!)

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

(여기 또한 RecyclerView에서 Paging을 작업할 때와 동일한 내용이다! 필요하다면 패스하자!)

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

## 4. PagingData 사용하기 With LazyColumn

이제 본격적으로 `LazyColumn`에 PagingData를 적용해보자. 필자의 경우, `UiState`를 속에 PagingData를 감싸 사용하였다.

따라서 우선 Screen에서 collect할 UiState 변수를 선언하자!

```kotlin
private val _issueBoardUiState: MutableStateFlow<UiState<PagingData<IssueResponseVO>>> =
    MutableStateFlow(UiState.Init)
val issueBoardUiState = _issueBoardUiState
    .onStart {
        fetchIssues()
    }.stateIn(
        viewModelScope,
        SharingStarted.WhileSubscribed(5000L),
        UiState.Init
    )
```

여기서 `onStart`를 활용해 `issueBoardPagingData`가 처음 collect될 때 data initialization이 진행되도록 하였다. 이때 `stateIn`을 통해 `SharedStarted.WhileSubscribed(5000L)`를 설정함으로써 collecter가 모두 제거되었을 때 5초간 term을 두어 불필요하게 `fetchIssues`가 진행되는 것을 막았다.

(data initialization 관련 부분이다. 이와 관련해서는 다음의 영상을 참고하자! [The ONLY Correct Way to Load Initial Data In Your Android App?:Philipp Lackner](https://www.youtube.com/watch?v=mNKQ9dc1knI))

그렇다면 이제 PagingData를 가져오는 작업을 진행해보자!

```kotlin
fun fetchIssues() {
  //...
  issueRepository.getRepoIssueList(
      DEFAULT_OWNER,
      //...
  ).collectLatest { pagingData ->
    _issueBoardUiState.emit(UiState.Success(pagingData))
  }
}
```

위 RepositoryImpl 에서 Pager를 통해 return한 `Flow<PagingData<T>>`를 받아와 정의한 mutableStateFlow에 emit을 진행해준다.

이제 해당 mutableStaetFlow를 가져와 Composable 함수에서 collect 해주어야 한다.

```kotlin
val uiState by viewModel.issueBoardPagingData.collectAsStateWithLifecycle()
```
우선 flow의 경우, `collectAsStateWithLifecycle()`을 통해 `lifecycle-aware 방식`으로 최신의 데이터를 collect해 state로 변환한다.

그렇다면 단순히 UiState 내부에 있는 `PagingData`를 가져와 사용하면 될까? 아니다! LazyColumn의 마지막에 도달했을 때 다음 page의 데이터를 가져와 무한 스크롤 기능을 구현하기 위해서는 `paging-compose`에서 제공하는 `collectAsLazyPagingItems()`를 통해 `LazyPagingItems`를 가져와 활용해야 한다.

> `For optimized performance`, use the Paging Library with a lazy list for an infinite list of data.
> 
> 공식 문서를 보면 다음의 문구가 존재한다. 즉, LazyPaingItems를 사용할 경우 성능적으로 더 좋다!

```kotlin
val pagingData = remember((uiState as UiState.Success).data) {
    flow {
        emit((uiState as UiState.Success<PagingData<IssueResponseVO>>).data)
    }
}.collectAsLazyPagingItems()
```
따라서 다음의 작업을 추가적으로 적용하였다. `remember`의 키 값으로 UiState의 데이터를 넘겨주고 해당 값이 바뀔 때마다(여기서 해당 값은 PagingData이다.) flow 빌더를 통해 새로운 flow가 만들어지게 된다.

그리고 새롭게 만들어지 flow를 `collectAsLazyPagingItems()`를 통해 LazyPagingItems를 가져온다. 이제 이를 LazyColumn에서 활용하기만 하면 된다!

```kotlin
LazyColumn {
    items(
        count = pagingData.itemCount,
        key = pagingData.itemKey { it.id }
    ) { index ->
        pagingData[index]?.let { issue ->
            IssueItem(issueResponse = issue)
        }
    }
}
```

## 5. Refresh 적용하기!

이제 정말 마지막 단계이다. 바로 `Refresh` 처리이다. 위 `PagingSource`를 정의할 때 `getRefreshKey` 함수를 정의했던 것이 기억날 것이다. 그렇다면 refresh 기능이 제공된다는 것인데,,, 어떻게 사용해야 할까?

이때 `pullRefresh`를 적용할 수 있다.

```kotlin
// 현재 refreshing이 일어나고 있는지를 boolean 값으로 가지고 있는다.
var isRefreshing by remember { mutableStateOf(false) }
val pullRefreshState = rememberPullRefreshState(
    refreshing = isRefreshing,
    onRefresh = {
        // LazyPagingItems의 데이터가 refresh 된다.
        pagingData.refresh()
    })

Box(Modifier.pullRefresh(pullRefreshState).fillMaxSize()) {
    LazyColumn(
        modifier = modifier.pullRefresh(pullRefreshState)
    ) {
        items(
            count = pagingData.itemCount,
            key = pagingData.itemKey { it.id }
        ) { index ->
            pagingData[index]?.let { issue ->
                IssueItem(issueResponse = issue, navigateToIssueDetailScreen)
            }
        }
    }
    PullRefreshIndicator(
        refreshing = isRefreshing,
        state = pullRefreshState,
        modifier = Modifier.align(Alignment.TopCenter)
    )
}
```
- `PullRefreshState`: scroll component에 pull-to-refresh 행동을 추가할 때 사용하는 것이다. 
  - 생성할 때는 `rememberPullRefreshState`를 활용해야 한다.
  - 이때 전달한 `refreshing` 값이 바뀌면 `PullRefreshState`의 업데이트가 일어난다.

- `PullRefreshIndicator`: refreshing 및 pullRefreshState와 연동해 refresh가 진행될 떄 띄워지는 로딩 인티케이터

<br>

# 🔗 적용을 통해 느낀 점

RecyclerView -> LazyColumn을 적용했을 때 ViewHolder / Adapter를 생성할 필요가 없었기 때문에 정말 간편하게 리스트를 만들 수 있었다. Paging 적용도 마찬가지로 RecyclerView에서보다 LazyColumn과 함께 사용할 떄 그 과정이 더 간편한 것 같다.

## 🔗 참고자료
- [https://developer.android.com/topic/libraries/architecture/paging/v3-paged-data](https://developer.android.com/topic/libraries/architecture/paging/v3-paged-data)
- [https://developer.android.com/codelabs/android-paging?hl=ko#0](https://developer.android.com/codelabs/android-paging?hl=ko#0)
- 추가: [https://developer.android.com/quick-guides/content/lazily-load-list](https://developer.android.com/quick-guides/content/lazily-load-list)