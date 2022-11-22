---
title: "[Compose] compose 레이아웃 이모저모"
excerpt: "compose 레이아웃 codelab을 진행하며 기억해둘 내용"
categories:
  - Compose
tag:
  - jetpack compose
  - compose layout


last_modifeid_at: 2022-11-22
toc: true
toc_sticky: true
search: true
---

해당 게시글은 [compose의 기본 레이아웃](https://developer.android.com/codelabs/jetpack-compose-layouts?continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fjetpack-compose-for-android-developers-1%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fjetpack-compose-layouts#12) 코드랩을 진행하며 그 과정과 기억해둘 내용을 코드 기반으로 정리한 게시글입니다.


# 🙋‍♀️ 학습 내용

코드랩을 통해 학습할 내용은 크게 다음과 같다.
* **Modifier**
    * 크기, 레이아웃, 동작, 모양변경
    * 접근성 라벨과 같은 정보 추가
    * 사용자 입력처리
    * 클릭, 스크롤, 드래그 등 높은 수준의 상호작용

* **Layout**
    * 레이아웃 정렬옵션
    * Alignment의 종류, 설정하는 법
    * Arrangement를 사용하여 배치하는 법

* **Slot API**
    * 사용자 정의 레이어를 가져오기 위한 패턴

* **Material 요소**
    * 미리 제공되는 요소들로 인해 컴포즈의 생산성 향상
    * Surface, BottomNavigation, Scaffold

<br>

그러다면 본격적으로 코드랩에 들어가기 앞서 우선 해당 코드랩을 통해 구현할 디자인을 살펴보자.

<img src = "https://drive.google.com/uc?id=1BsFy9G8KTHryJITdNBNbBhLJh74LamuL" width = 400 height = 300>

여기서 디자인을 진행할 부분은 크게 다음과 같이 세가지 부분이다.
* 검색창
* 신체의 조화 섹션
* 즐겨찾는 컬렉션 섹션

또한, 컴포저블을 잘 활용하기 위해서 **재사용**되는 하위 구성요소를 파악해보면 다음과 같이 두가지이다.
* 신체의 조화 섹션에서의 요소
* 즐겨찾기 컬렉션 섹션에서의 요소

그렇다면 이제 각 요소들에 해당하는 composable을 구성해보자!!

<br>

# 🙋‍♀️ 검색창 - [Modifier 학습](https://developer.android.com/jetpack/compose/modifiers)

```kotlin
@Composable
fun SearchBar(
   modifier: Modifier = Modifier
) {
   TextField(
       value = "",
       onValueChange = {},
       leadingIcon = {
           Icon(
               imageVector = Icons.Default.Search,
               contentDescription = null
           )
       },
       colors = TextFieldDefaults.textFieldColors(
           backgroundColor = MaterialTheme.colors.surface
       ),
       placeholder = {
           Text(stringResource(R.string.placeholder_search))
       },
       modifier = modifier
           .fillMaxWidth()
           .heightIn(min = 56.dp)
   )
}
```
* `TextField`를 통해서 검색창을 구현한다.
    * `leadingIcon`을 통해 내부 아이콘을 설정한다. TextField의 의미가 설명되어 있으므로 해당 아이콘의 콘텐츠 설명은 필요하지 않는다.
    * `heightIn`을 통해서 **최소 높이**만 설정한다. 사용자가 설정한 글꼴의 크기에 따라 **텍스트 필드 크기가 변하게 된다.**


* SearchBar 컴포저블의 `Modifier`를 `TextField`에 전달하여 **메서드의 호출자가 컴포저블의 디자인과 분위기를 수정할 수 있도록** 하여 유연성을 높이고 재사용을 가능하게 한다.

<br>

# 🙋‍♀️ 신체의 조화 섹션 요소 - 정렬 학습

<img src = "https://drive.google.com/uc?id=1t_DIeXKMuw6Z0Egv5RgYcW2tzpCLeNOW" width = 500 height = 500>

```kotlin
@Composable
fun AlignYourBodyElement(
   @DrawableRes drawable: Int, // 컴포저블을 동적으로 만들기 위해 매개변수를 사용한다.
   @StringRes text: Int,
   modifier: Modifier = Modifier
) {
   Column(
       modifier = modifier,
       horizontalAlignment = Alignment.CenterHorizontally // 상위 컨테이너 내부에서 컴포저블 정렬, Column이므로 하위 요소 가로 정렬 방법 지정
   ) {
       Image(
           painter = painterResource(drawable),
           contentDescription = null,
           contentScale = ContentScale.Crop, // 이미지를 조정(fit, fillbounds, crop)
           modifier = Modifier
               .size(88.dp)
               .clip(CircleShape) // clip을 통해 컨테이너의 모양을 지정
       )
       Text(
           text = stringResource(text),
           style = MaterialTheme.typography.h3,
           modifier = Modifier.paddingFromBaseline(
               top = 24.dp, bottom = 8.dp
           ) // 문자가 놓여 있는 기준선을 기준으로 상단, 하단 간격 설정
       )
   }
}

```
* 각 요소의 이미지는 장식용으로 `contentDescriptino`을 **null**로 설정한다.
    * 이미지 아래 텍스트만으로 의미가 충분히 전달되기 때문이다.
* `clip` 수정자를 통해 컴포저블의 모양을 조정한다.
* `contentScale` 매개변수를 사용해 이미의 크기를 조정한다.
    * **Crop**을 통해 **컨테이너 내부에 맞게 이미지를 자르게 된다.**
* **상위 컨테이너 내부에서 컴포저블을 정렬**하려면 상위 컨테이너의 `alignment`를 설정한다.
    * `Column`의 경우 가로 정렬 방법을 지정, `Row`의 경우 세로 정렬 방법을 지정한다.
    * 상위 컨테이너인 `Column` 내부의 컴포저블을 가로로 정렬하기 위해 `horizontalAlignment`를 사용한다.

<br>

# 🙋‍♀️ 즐겨찾는 컬렉션 카드

<img src = "https://drive.google.com/uc?id=13FdK9ws4XFnLDI4fVzAtJQmpIGeDJ3Pv" width = 500 height = 500>

```kotlin
@Composable
fun FavoriteCollectionCard(
   @DrawableRes drawable: Int, // 동적으로 요소를 생성하기 위해 매개변수를 사용한다.
   @StringRes text: Int,
   modifier: Modifier = Modifier
) {
   Surface(// 모서리가 둥근 컨테이너를 처리하기 위해 Surface를 사용한다.
       shape = MaterialTheme.shapes.small,
       modifier = modifier
   ) {
       Row(
           verticalAlignment = Alignment.CenterVertically, // 상위 컨테이너 내부에서 하위 컴포저블 정렬, Row이므로 하위 세로 정렬 방법 지정
           modifier = Modifier.width(192.dp) // Row의 너비를 설정한다.
       ) {
           Image(
               painter = painterResource(drawable),
               contentDescription = null,
               contentScale = ContentScale.Crop, // 컨테이너 내부에서 이미지를 잘라 이미지를 조정한다.
               modifier = Modifier.size(56.dp) // 이미지 크기를 설정한다.
           )
           Text(
               text = stringResource(text),
               style = MaterialTheme.typography.h3,
               modifier = Modifier.padding(horizontal = 16.dp)
           )
       }
   }
}
```

<br>

# 🙋‍♀️ 신체의 조화 section 배치 - LazyRow 적용

<img src = "https://drive.google.com/uc?id=1XuBZWj4igyBsWPbvr3wZa8LJuOTRueUy" width = 500 height = 500>

```kotlin
@Composable
fun AlignYourBodyRow(
   modifier: Modifier = Modifier
) {
    // 모든 요소를 동시에 렌더링 하는 대신 화면에 표시되는 요소만 렌더링하여 앱의 성능을 유지한다.
   LazyRow(
        //각 하위 컴포저블 사이에 고정된 공간 추가
       horizontalArrangement = Arrangement.spacedBy(8.dp),
       // 단순 패딩으로 잘리는 문제를 해결
       // 동일한 패딩을 유지하되 상위 목록의 경계 내에서 콘텐츠를 자르지 않고 스크롤이 된다.
       contentPadding = PaddingValues(horizontal = 16.dp),
       modifier = modifier
   ) {
       items(alignYourBodyData) { item ->
           AlignYourBodyElement(item.drawable, item.text)
       }
   }
}
```

* **목록 속 하위 컴포저블 배치 방식**
    * Row
    
    <img src = "https://drive.google.com/uc?id=1ni9Y6nqeMLfZRwGOFuJ_DVLs439yLyzA" width = 400 height = 300>

    * Column
    
    <img src = "https://drive.google.com/uc?id=1_w31ZMm-qkCQJv88DjvS63XbbK1zxY5j" width = 300 height = 400>

* 위 배치 방식을 사용하지 않고 `Arrangement.spaceBy()`를 통해 **하위 컴포저블 사이 고정된 공간을 추가**한다.

<br>

# 🙋‍♀️ 즐겨찾는 컬렉션 섹션 - Lazy Grid 적용

<img src = "https://drive.google.com/uc?id=13YigMy9kBuxCf26yLGCM7gGab8hktsH7" width = 300 height = 400>

* `LazyRow` 속 두개의 요소를 `Column`을 통해 배치하며 해당 섹션을 구현할 수 있다.

🙃 여기서는 항목-그리드 요소 매핑을 효과적으로 진행할 수 있는 `LazyHorizontalGrid`를 사용할 것이다.

```kotlin
@Composable
fun FavoriteCollectionsGrid(
   modifier: Modifier = Modifier
) {
   LazyHorizontalGrid( // 항목-그리드 요소 매핑을 효과적으로 진행할 수 있다.
       rows = GridCells.Fixed(2), // 행을 2개로 고정
       contentPadding = PaddingValues(horizontal = 16.dp), // 주어진 경계 안에서 항목을 자르지 않고 패딩을 유지할 수 있다.
       horizontalArrangement = Arrangement.spacedBy(8.dp), // 항목에 좌우 고정된 패딩 적용
       verticalArrangement = Arrangement.spacedBy(8.dp), // 항목에 상하 고정된 패딩 적용
       modifier = modifier.height(120.dp)
   ) {
       items(favoriteCollectionsData) { item ->
           FavoriteCollectionCard(
               drawable = item.drawable,
               text = item.text,
               modifier = Modifier.height(56.dp)
           )
       }
   }
}
```
* 여기서도 `LazyHorizontalGrid` 자체에 좌우 패딩을 추가하면 항목이 잘리는 현상이 일어난다.
    * `contentPadding`을 통해 **패딩을 유지하며 경계 안에서 항목이 잘리지 않도록 한다.**

<br>

# 🙋‍♀️ 홈 화면 구성 - [슬롯 API](https://developer.android.com/jetpack/compose/layouts/basics#slot-based-layouts) 적용

<img src = "https://drive.google.com/uc?id=1TDx_PDJCZnWDsE9SzLatkF4UKikZLxBS" width = 400 height = 500>

위 화면을 구성하기 전에 우리는 하나의 패턴을 확인할 수 있다. `제목-섹션`의 패턴이 반복되며 화면을 구성하고, **각 섹션마다 다른 콘텐츠가 표시된다.**

🙃 여기서 우리는 **슬롯**을 기반으로 레이아웃을 구성할 수 있다.

> 슬롯 기반 레이아웃? => 사용자 정의 레이어를 가져오기 위한 패턴
   개발자가 **원하는 대로 채울 수 있도록** UI에 **빈 공간**을 남겨 둔다. 슬롯 기반 레이아웃을 사용하여 <U>유연한 레이아웃</U>을 만들 수 있고, **코틀린의 고차함수처럼** 컴포저블 함수를 구성할 수 있다.

```kotlin
@Composable
fun HomeSection(
   @StringRes title: Int,
   modifier: Modifier = Modifier,
   content: @Composable () -> Unit // 컴포저블의 슬롯으로 content 매개변수를 통해 원하는 섹션을 받아온다.
) {
   Column(modifier) {
       Text(
           text = stringResource(title).uppercase(Locale.getDefault()),
           style = MaterialTheme.typography.h2,
           modifier = Modifier
               .paddingFromBaseline(top = 40.dp, bottom = 8.dp)
               .padding(horizontal = 16.dp)
       )
       content() // 빈 공간을 원하는 섹션으로 채워넣을 수 있다.
   }
}
```
* 특정 컴포저블을 넘길 때는 일반적으로 `content` 컴포저블 람다`(content: @Composable () -> Unit)`를 사용한다.

<br>

# 🙋‍♀️ 홈 화면 - 목록이 아닌 항목 스크롤시키기

<img src = "https://drive.google.com/uc?id=1EGKTz-ttUyt3Z95YIjiuFyUajXWJOs8-" width = 300 height = 400>

디자인을 바탕으로 화면 구성 방식을 한 번 정리해보자!

* `Row`를 활용해 홈 화면을 구성한다.
    * 일반적으로 **목록에 포함된 요소가 많거나 로드해야 할 데이터 세트가 많을 경우** `Lazy 레이아웃`을 사용한다.
        * 모든 항목을 동시에 내보내면 성능이 저하되고 앱이 느려지기 때문이다.
* `Row`를 통해 화면을 구성한 후, **스크롤 동작을 추가**한다.
    * **Lazy 레이아웃**은 자동으로 **스크롤 동작이 추가된다.**
    * **목록의 개수가 많지 않은 경우**에는 간단하게 `Column` 또는 `Row`를 사용하고 **스크롤 동작을 추가**하면 된다.

```kotlin
@Composable
fun HomeScreen(modifier: Modifier = Modifier) {
   Column(modifier) {
       Spacer(Modifier.height(16.dp))
       SearchBar(Modifier.padding(horizontal = 16.dp))
       HomeSection(title = R.string.align_your_body) {
           AlignYourBodyRow()
       }
       HomeSection(title = R.string.favorite_collections) {
           FavoriteCollectionsGrid()
       }
       Spacer(Modifier.height(16.dp))
   }
}

```
* `Spacer`를 사용하지 않고 **Column에 padding을 설정**하게 되면 요소가 잘리는 현상이 발생한다. 따라서 Spacer를 사용해 Column 내부에 공간을 둔다.
    * `Lazy 레이아웃`에서는 이러한 현상을 해결하기 위해서 `contentPadding`을 사용했다.

🙃 여기서 스크롤 동작을 추가해야 한다. 이를 위해 우리는 `verticalScroll/horizontalScroll` 수정자를 사용한다.
* 스크롤의 현재 상태를 포함하며 외부에서 스크롤 상태를 수정하는 데 사용되는 `ScrollState`가 필요하다.
* 여기서는 **스크롤 상태를 수정할 필요가 없으므로** `rememberScrollState`를 사용하여 영구 ScrollState 인스턴스를 만들면 된다.

```kotlin
@Composable
fun HomeScreen(modifier: Modifier = Modifier) {
   Column(
       modifier
           .verticalScroll(rememberScrollState())
           .padding(vertical = 16.dp)
   ) {
       SearchBar(Modifier.padding(horizontal = 16.dp))
       HomeSection(title = R.string.align_your_body) {
           AlignYourBodyRow()
       }
       HomeSection(title = R.string.favorite_collections) {
           FavoriteCollectionsGrid()
       }
   }
}
```
* 스크롤 동작을 추가하면서 **Column에 padding을 넣어도 요소가 잘리지 않게 된다.**

<br>

# 🙋‍♀️ 하단 탐색바 - ButtonNavigation 컴포저블 사용

<img src = "https://drive.google.com/uc?id=1pFKpML2lAsayuWdxs6n3nc5p7MnYrA7u" width = 400 height = 500>

하단 네비게이션 바는 Compose Material 라이브러리의 `BottomNavigation` 컴포저블을 사용하면 된다. 컴포저블 내에 여러 개의 `ButtonNavigationItem` 요소를 추가할 수 있다.

```kotlin
@Composable
private fun SootheBottomNavigation(modifier: Modifier = Modifier) {
   BottomNavigation( // 하단 탐색바를 쉽게 구현할 수 있는 Material 라이브러리의 BottomNavigation 컴포저블 사용
       backgroundColor = MaterialTheme.colors.background,
       modifier = modifier
   ) {
        // 하나의 BottomNavigationItem이 탐색바의 하나의 요소가 된다.
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.Spa,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_home))
           },
           selected = true,
           onClick = {}
       )
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.AccountCircle,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_profile))
           },
           selected = false,
           onClick = {}
       )
   }
}
```

<br>

# 🙋‍♀️ 최종 앱 화면 구성하기 - [Scaffold](https://developer.android.com/jetpack/compose/layouts/material#scaffold) 적용

위에서 구성한 스크린 화면에 하단 탐색바까지 설정하면 최종 앱 화면이 구성된다. 여기서 사용할 것은 `Scaffold`이다. 이를 사용할 경우, **하단 메뉴에 대한 슬롯을 설정할 수 있다.**

```kotlin
@Composable
fun MySootheApp() {
   MySootheTheme {
       Scaffold(
           bottomBar = { SootheBottomNavigation() }
       ) { padding ->
           HomeScreen(Modifier.padding(padding))
       }
   }
}
```
* **최상위 수준의 컴포저블** 즉, 전체 레이아웃을 나타내는 컴포저블이므로 **Material 테마를 적용한다.**
* `Scaffold` 컴포저블을 사용한다.
    * **하단 메뉴 슬롯**에 우리가 제작한 **SootheBottomNavigation**을 배치한다.
    * **content 슬롯**에 전체 화면을 나타내는 HomeScreen을 배치한다.

<br>

🙂 이렇게 해서 Compose의 기본 레이아웃 코드랩을 마무리했다! 하나부터 열까지 다 새로운 내용이지만 코드랩을 따라하는 과정에서 컴포즈를 어떻게 활용해야 할지 감을 잡을 수 있었던 것 같다! 코드랩 안에서 적용되는 개념들이 추후 많이 사용되는 개념일 것 같아 코드 기반으로 정리했더니 설명보다는 코드가 많은 비중을 차지한 것 같다..! 추후 컴포즈를 사용하게 된다면 해당 코드랩에 대한 내용을 많이 참고할 것 같다!!