---
title: "[SWM] Server Driven UI - 실습"
excerpt: "서비스 변화에 유연한 백엔드-모바일 구조 설계"
categories:
  - Android
tag:
  - android
  - server driven ui
  - SWM
  - Mobile Camp

last_modified_at: 2023-07-05
toc: true
toc_sticky: true
search: true
---

이번 실습 게시글을 보기 전, <span style="background-color:#fff5b1">왜 server-driven-ui를 사용해야 하는지를 확실히 이해</span>하고 있어야 한다. 그에 대한 내용은 다음 게시글을 참고하길 바란다!
* [Server Driven UI - 개념 정리](https://915dbfl.github.io/android/server_driven_ui/)

<br>

# 👩🏻‍💻 파싱하고자 하는 Json

하나의 View에 대한 파싱을 진행할 때는 <span style="background-color:#fff5b1">**정해진 DTO, VO가 있기 때문에** </span> 큰 어려움이 없다. 하지만 만약 a/b Test로 server-driven-ui를 하게 된다면 어떠한 뷰를 그려야 할지 클라이언트에서는 알 수가 없다.

다음의 Json을 파싱한다고 해보자!

```
{
  "viewItems": [
    {
      "viewType": "AViewType",
      "content": {
        "title": "This is A ViewType",
        "iconUrl": "https://avatars.githubusercontent.com/u/103282546?s=200&v=4\"
      }
    },
    {
      "viewType": "BViewType",
      "content": {
        "title": "This is B ViewType"
      }
    }
  ]
}
```
내려오는 <span style= "background-color:#fff5b1">viewType에 따라 content에 맞게 커스텀 파싱 로직</span>이 필요해진다.

여기서 사용하는 것이 바로  `JsonDeserializer`이다. 이를 통해 Json 속 viewType을 가져오고 이에 맞는 DTO를 선택하면 된다.

이제부터 server-driven-ui를 진행하기 위해 구현해야 하는 코드들을 살펴볼 것이다. 그 과정을 살펴보면서 새로운 viewType이 추가되었을 때 수정해야 하는 부분을 함께 카운트해볼 것이다.<span style="background-color:#fff5b1">(새로운 viewType 추가를 위해 얼마나 많은 코드 변화가 필요한지가 핵심이므로!)</span>

<br>

# 🤨 DTO, VO 생성: Common

우선 json 파싱에 사용될 DTO, VO가 필요하다. 처음에는 단순히 AViewType, BViewType DTO, VO를 각각 생성하면 된다 생각했다. 하지만 바로 해당 코드를 구현하기 전에 json 구조를 자세히 살펴보자!!

```
    {
      "viewType": "BViewType",
      "content": {
        "title": "This is B ViewType"
      }
    }
```

json을 살펴보면 다음의 구조가 리스트 형태로 반복되는 것을 알 수 있다. 그렇다면 우리는 이 틀에 대한 DTO, VO를 생성해야 할 것이다. <span style = "background-color:#fff5b1">(Common DTO, Common VO)</span>

```kotlin
//CommonVO
data class CommonVO(
  val viewType: String,
  val content: ContentVO
)
```

```kotlin
//CommonDTO
data class CommonDTO(
    val viewType: String?,
    val content: ContentDTO?
) {
    fun toVO(): CommonVO {
        return CommonVO(
            viewType = findClassByItsName(viewType),
            content = content?.toVO() ?: throw IllegalStateException("content cannot be null")
        )
    }
}
```

여기서 하나 집고 갈 부분! <span style = "background-color:#fff5b1">보통 DTO, VO를 따로 구현하지만 그 내용은 같은 경우가 많을 것이다!</span> 그럼 왜 굳이 분리해서 구현해야 할까? 그렇다! <sapn style = "background-color:#fff5b1">백엔드에서 null이 내려오는 경우를 처리하기 위해서이다!</span>

<br>

# 🤨 DTO, VO 생성: Content

그렇다면 이제는 <span style = "background-color:#fff5b1">viewType에 따라 달라지는 content에 대한 두가지 DTO, VO를 생성해야 한다.</span> 우리가 하려고 하는 것은 JsonDeserializer를 통해 Json에서 viewType을 가져오고 그에 맞게 content에 해당하는 DTO로 변환해주는 것이다. 그렇다면 결국 <span style = "background-color:#fff5b1">AViewType과 BVieweType은 같은 Type이어야 한다.</span> 여기서 우리는 `interface`, `enum`, `sealed class`를 사용할 수 있다. 

여기서 `interface`와 `sealed class`를 활용하는 같은 효과를 얻을 수 있다. 다만 enum은 그게 어렵다. 우리는 보통 enum을 사용할 때 다음과 같은 형태로 사용한다.

```
enum class Color(val rgb: Int) {
  RED(0xFF0000),
  GREEN(Ox00FF00)
  BLUE(...)
}
```
이렇게 각 요소가 모든 같은 파라미터 변수를 가지게 된다. 하지만 우리가 생성하고자 하는 AViewType, BViewType은 서로 다른 요소로 구성되어 있다. 따라서 enum을 적용하는 것에는 어려움이 있다. 또한, <span style = "background-color:#fff5b1">enum은 프로퍼티로 data class를 가지지 못한다.</span> 따라서 이 게시글에서는 `sealed class`를 적용하고자 한다. 우선 sealed class는 간단히 어떤 클래스의 하위 클래스를 함께 정의한다고 이해하면 될 것 같다! 조금 더 자세한 것은 아래 코드를 통해 이해해보자!

```kotlin
//ContentVO
//✅새로운 viewType이 들어올 때마다 생성해야 함 +1
sealed class ContentVO {
  data class AContent(
    val title: String,
    val imageUrl: String
  ): ContentVO()

  data class BContent(
    val title: String
  ): ContentVO()

  object UnknownType: ContentVO()
}
```

```kotlin
sealed class ContentDTO {
  //✅새로운 viewType이 들어올 때마다 생성해야 함 +2
    data class AContent(
        val title: String,
        val imageUrl: String
    ) : ContentDTO() {
        override fun toVO(): ContentVO.AContent {
            return ContentVO.AContent(
                title = title,
                imageUrl = imageUrl
            )
        }
    }

    data class BContent(
        val title: String
    ) : ContentDTO() {
        override fun toVO(): ContentVO.BContent {
            return ContentVO.BContent(
                title = title
            )
        }
    }

    object UnknownType : ContentDTO() {
        override fun toVO() = ContentVO.UnknownType
    }

    //확장 함수로 정의해둔 toVO 활용
    open fun toVO(): ContentVO {
        return this.toVO()
    }
}
```
* AType, BType 이외에 UnknownViewType을 생성해 <span style = "background-color:#fff5b1">알지 못하는 ViewType이 들어와도 에러가 나지 않도록 한다.</span>
* Type에 맞는 ViewHolder를 반환해주면 되지 않을까?
  * <span style = "background-color:#fff5b1">domain에서 활용할 예정이기 때문에 Android와 관련된 정보는 넣지 못한다.</span>


<br>

# 🤨 JsonDeserializer

이제는 위에서 언급했던 `JsonDeserializer`를 구현할 것이다. 여기서 우리가 해야 할 것은 <span style = "background-color:#fff5b1">type에 맞게 알맞은 ContentType가 들어있는 CommonDTO를 get하는 것!</span>

```kotlin
class ViewTypeDeserializer : JsonDeserializer<CommonDto> {
    override fun deserialize(
        json: JsonElement?,
        typeOfT: Type?,
        context: JsonDeserializationContext?
    ): CommonDto {
        val jsonObject = json?.asJsonObject ?: throw IllegalArgumentException("Json Parsing 실패")

        val viewType = jsonObject["viewType"].asString
        val content = jsonObject["content"].asJsonObject
        //해당하는 타입에 맞는 ContentDTO를 얻게 된다.⭐️
        val contentDTO: ContentDTO = when (findClassByItsName(viewType)) {
            AViewType -> parseAContent(content)
            BViewType -> parseBContent(content)
            UnknownViewType -> parseUnknownContent()
        }

        return CommonDto(viewType, contentDTO)
    }

    private fun parseAContent(contentJson: JsonObject): ContentDTO.AContent {
        val title = contentJson["title"].asString
        val imageUrl = contentJson["iconUrl"].asString
        return ContentDTO.AContent(title, imageUrl)
    }

    private fun parseBContent(contentJson: JsonObject): ContentDTO.BContent {
        val title = contentJson["title"].asString
        return ContentDTO.BContent(title)
    }

    private fun parseUnknownContent(): ContentDTO.UnknownType {
        return ContentDTO.UnknownType
    }

}
```

그렇다면 이렇게 정의한 ViewTypeDeserializer를 활용해야 한다!

```kotlin
    @Provides
    @Singleton
    fun providesLGTMRetrofit(okHttpClient: OkHttpClient): Retrofit =
        Retrofit.Builder()
            .baseUrl(...)
            .client(okHttpClient)
            .addConverterFactory(
                GsonConverterFactory.create(
                    GsonBuilder()
                        .registerTypeAdapter(CommonDto::class.java, ViewTypeDeserializer())
                        .create()
                )
            )
            .addConverterFactory(GsonConverterFactory.create())
            .build()
```
즉, <span style = "background-color:#fff5b1">retrofit 객체를 얻을 때 `registerTypeAdapter`를 통해 JsonDeserializer를 등록할 수 있다.</span> 그렇다면 여기까지 진행했다면 우리가 원하는 viewType에 맞는 CommonDTO를 받게 된다. 이제는 본격적으로 이를 활용해야 한다. 그렇다면 그에 맞는 `ViewHolder`, `RecyclerViewAdapter` 가 필요하다.

<br>

# 🤨 ViewHolder 생성

우선 AType, BType, UnknownType에 필요한 xml을 모두 정의했다고 가정하자! 그럼 각 Type에 활용될 ViewHolder를 선언해야 한다. 이때 `abstract class`를 활용할 수도 있고, `sealed class`를 활용할 수도 있다. 나는 두 클래스 모두 많이 사용해본 경험이 없어 두 방식의 차이를 명확히 알고 싶어 구글링을 해보았고 간단히 정리하면 다음과 같았다.

* abstract class vs sealed class
  * 큰 차이: <span style = "background-color:#fff5b1">구현체가 같은 패키지에만 있어야 하나</span>
  * 멀티모듈에서 여러 패키지에서 구현을 할 수 있도록 할 것이라면 `abstract class`를 사용하는 것이 맞다.

* + 추가 interface vs abstract class
  * 큰 차이: <span style = "background-color:#fff5b1">공통된 구현 코드가 있는가, 아님 행위만을 구정하는가</span>
  * 공통된 구현 코드가 있다면 `abstract class`를 사용하는 것이 맞다.

그렇다면 우리는 하나의 패키지내에서 활용할 viewHolder들을 만들기 때문에 `sealed class`를 활용해보도록 하자!!

```kotlin
sealed class CommonViewHolder(
  binding: ViewDataBinding
): RecyclerView.ViewHolder(binding.root) {

  abstract fun bind(item: ContentVO)

  //✅새로운 viewType이 들어올 때마다 생성해야 함 +3
  Class ATypeViewHolder(
    private val binding: ItemtSduiABinding
  ): CommonViewHolder(binding) {
    override fun bind(data: ContentVO) {
      data as ContentVO.AContent
      binding.data = data
      Glide.with(binding.root)
        .load(data.imageUrl)
        .into(binding.ivImage)
    }
  }

  Class BTypeViewHolder(
    private val bidning: ItemSduiBBinding
  ): CommonViewHolder(binding) {
    override fun bind(data: ContentVO) {
      data as ContentVO.BContent
      binding.data = data
    }
  }

  Class UnknownViewHolder(
    binding: ImtemSduiUnknownBinding
  ): CommonViewHolder(binding) {
    override fun bind(data: ContentVO) {
      override fun bind(data: ContentVO) {
        /*no-op*/
      }
    }
  }

}
```

<br>

# 🤨 CommonAdapter + Enum Class(ViewType)
그러면 이제 넘겨진 데이터를 타입에 따라 원하는 형태의 ViewHolder로 bind해주는 adapter를 구현할 차례이다. 우선 `ListAdapter`를 상속해 Adapter를 구현하게 되면 `onCreateViewHolder`함수를 오버라이드하게 된다. 이때 들어오는 파라미터 중 `viewType`이라는 값이 있는데 여기에는 각 viewType에 해당하는 숫자가 부여된다. 따라서 우리는 ViewType에 대한 하나의 숫자값이 부여될 수 있도록 enum 클래스로 Type을 정의해둬야 한다.

위 코드 중 함수 `findClassByItsName`를 보았을 것이다. 그에 대한 내용도 아래 코드를 참고하길 바란다.

```kotlin
Enum class ViewType {
  //✅새로운 viewType이 들어올 때마다 생성해야 함 +4
  AViewType,
  BViewType,
  UnknownType;

  companion object {
    fun findClassByItsName(viewTypeString: String?): ViewType {
      values().find { it.name == viewTypeString }?.let {
        return it
      } ?: return UnknownType
    }

    fun getViewTypeByOrdinal(ordinalNum: Int): ViewType {
      return values()[ordinalNum]
    }
  }
}
```

그렇다면 위에서 정의한 `getViewTypeByOrdianl`을 이용해 CommonAdapter를 정의해보자! 그 전에! 아마 각 viewType마다 해당하는 adapter를 만들면 되지 않나? 생각하는 분들이 분명 있을 것이다. 이미 이전 단계에서 <span style = "background-color:#fff5b1">해당하는 ViewType이 무엇인지를 알아냈고, adapter에서는 그에 맞는 viewHolder를 골라 bind만 해주면 된다. 따라서 commonAdapter를 하나만 두어도 된다!</span>

```kotlin
class CommonAdapter: ListAdapter<CommonVO, CommonViewHolder>(
  ItemDiffCallback<CommonVO>(
    onContentsTheSame = {old, new -> old == new},
    onItemsTheSame = {old, new -> old == new}
  )
) {
  
  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CommonViewHolder {
    return getViewHolder(parent, ViewType.getViewTypeByOrdinal(viewType))
  }

  override fun onBindVIewHolder(holder: CommonViewHolder, position: Int) {
    holder.bind(getItem(position).content)
  }

  override fun getItemViewType(position: Int): Int {
    return getItme(position).viewType.ordinal
  }

}
```

<br>

# 😌 정리

SDUI의 개념을 학습하고 이를 직접 코드로 적용하는 과정이 생각보다 헷갈렸다. 또한, 팀원들과 협업하며 코드를 함께 작성하다 보니 평소 내가 짜던 코드 스타일과 달라 헷갈리는 부분이 더 많았던 것 같다. 추후 내가 직접 해당 부분을 구현해보며 그 과정을 조금 더 명확히 익혀야 겠다.

마무리 하기 전 짚고 넘어가야 할 부분이 하나 있다!! 바로 위에서 계속 counting했던 새로운 Type이 들어오면 변경해야 할 부분이다! 여기서는 <span style = "background-color:###fb1">총 4부분</span>을 수정해야 했다! <span style = "background-color:#fff5b1">SDUI의 목적은 새로운 변화에서 코드 변화를 최소화하는 것이었던 것을 기억할 것이다!</span> 그렇다면 위 방식보다 코드 변화를 적게 할 수 있는 방식이 있을까? 다음 시간에는 멘토님의 피드백을 바탕으로 어느 부분을 추가적으로 수정할 수 있는지를 다뤄보도록 하겠다!

<br>

## 📃참고
* SWM 모바일 특강
* <https://jgeun97.tistory.com/222>
* <https://www.inflearn.com/questions/545726/abstract-class%EC%99%80-sealed-class%EB%8A%94-%EC%96%B4%EB%8A%90%EA%B2%BD%EC%9A%B0%EC%97%90-%EB%82%98%EB%88%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EB%82%98%EC%9A%94>