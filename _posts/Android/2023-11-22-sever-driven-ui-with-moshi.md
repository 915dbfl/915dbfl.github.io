---
title: "[SDUI] moshi를 적용한 SDUI"
excerpt: ""
categories:
  - Android
tag:
  - Android
  - SDUI

last_modified_at: 2023-11-16
toc: true
toc_sticky: true
search: true
---

## 🔗 들어가기 전
해당 게시글을 통해 moshi를 활용해 SDUI를 진행하는 과정을 하나하나 짚어보고 넘어가고자 한다.

단계는 크게 다음과 같다.
- SDUI Json 설계하기
- DTO/VO 생성하기
- moshi를 활용한 jsonAdapter 설정하기
- viewType에 따른 ViewHolder 정의
- viewType - viewHolder의 손쉬운 매칭을 위한 enum 정의 
- Multi - ViewType을 나타낼 수 있는 ListAdapter 생성하기
- 이를 활용해 SDUI 구축


## 🔗 1. Json 설계

우선 나의 경우, 총 4가지의 viewType이 필요했다.
- CalanedarViewType
- PieChartViewType
- (line)GraphViewType
- TextViewType

이에 대한 상세 설게는 다음 [노션 페이지](https://separated-stick-863.notion.site/ServerDriven-UI-Json-cf513b967af7429893dc301cf9414ec6?pvs=4)를 참고하길 바란다.

## 🔗 2. DTO / VO 설계

### recyclerView에 실질적으로 필요한 데이터들 정의

우선 RecyclerView 속 각각의 viewHolder를 나타낼 때 필요한 데이터를 구성하는 VO를 정의해보자.

하나의 ListAdater를 활용해 뷰타입에 맞는 viewHolder를 찾아 그릴 것이기 때문에 해당 VO들을 통틀어 말할 수 있는 interface가 필요하다.

```kotlin
interface SduiReportContent

data class ReportGraphVO(
    val graphTitle: String,
    val graphColor: String,
    val graphItems: List<ReportGraphItemVO>
): SduiReportContent

data class ReportTextVO (
    val align: String,
    val textItems: List<ReportTextItemVO>
): SduiReportContent

...
```
이와 같이 SduiReportContent interface를 구현하는 각 viewHolder에 필요한 VO들을 정의해준다.

### 통일된 구조를 같는 SduiReportItemVO 정의

이제는 설계에 보는 다음 부분을 나타내는 VO를 정의할 것이다.

```kotlin
interface SduiReportItemVO {
    val viewType: SduiViewType
    val content: SduiReportContent
}
```

이부분에서 핵심은 <span style = "background-color:#fff5b1">viewType 값을 통해 해당 데이터가 어느 viewType을 그리기 위해 필요한지를 파악</span>하는 것이다. 여기서는 우선 구현부터 살펴보자! 

```kotlin
interface SduiReportItemVO {
    val viewType: SduiViewType
    val content: SduiReportContent
}

data class SduiReportGraphItemVO(
    override val viewType: SduiViewType,
    override val content: ReportGraphVO
): SduiReportItemVO

data class SduiReportPieChartItemVO(
    override val viewType: SduiViewType,
    override val content: ReportPieChartVO
): SduiReportItemVO

data class SduiReportTextItemVO(
    override val viewType: SduiViewType,
    override val content: ReportTextVO
): SduiReportItemVO
...
```

여기서 보면 SduiReportItemVO interface를 정의하고 viewType에 맞게 각각 하나의 dataClass를 만들어준다. 그리고 content에서 viewType에 맞는 content를 직접 넣어준다.

이는 추후 <span style = "background-color:#fff5b1">moshi를 활용해 JsonAdapter를 사용하기 위해 필요하게 된다.</span>

### 나머지 VO 작성

```kotlin
data class SduiReportVO(
    val screenName: String,
    val viewContents: List<SduiReportItemVO>
)

data class SduiBaseResponseVO(
    val responseData: SduiReportVO
)
```

그리고 api의 결과를 받을 `SduiBaseResponseDTO`를 마지막으로 정의해준다.

```kotlin
@JsonClass(generateAdapter = true)
data class SduiBaseResponseDTO(
    @Json(name = "responseData")
    val responseData: SduiReportVO?,
) {
    fun toVO() = SduiBaseResponseVO(
        responseData = responseData ?: SduiReportVO("unknown", listOf())
    )
}
```

## 🔗 3. moshi를 활용한 jsonAdapter 설정하기

이제 json 속 `viewType`을 분석하고 그에 맞는 SduiReportItemVO 객체로 파싱하는 작업을 할 수 있도록 Adapter를 설정할 것이다.

이때 우리는 moshi에서 제공하는 `PolymorphicJsonAdapterFactory`를 사용할 것이다.

```kotlin
object JsonAdapter {
    private val sduiJsonAdapter: PolymorphicJsonAdapterFactory<SduiReportItemVO> = PolymorphicJsonAdapterFactory.of(SduiReportItemVO::class.java, "viewType")
        .withSubtype(SduiReportPieChartItemVO::class.java, "PIECHART")
        .withSubtype(SduiReportGraphItemVO::class.java, "GRAPH")
        .withSubtype(SduiReportTextItemVO::class.java, "TEXT")

    val moshi: Moshi = Moshi.Builder()
        .add(sduiJsonAdapter)
        .add(KotlinJsonAdapterFactory())
        .build()
}
```

위 코드를 보면 `viewType`(labelKey)가 `PIECHART / GRAPH / TEXT`인지에 따라 SduiReportItemVO를 구체화한 클래스를 지정해준다.

> 여기서 필자가 헤맸던 부분은 <span style = "background-color:#fff5b1">labelKey로 설정한 항목(viewType)은 SduiReportItemVO를 구체화한 클래스에 모두 들어있어야 한다.</span> 만약 그렇지 않는다면 `missing label`이라는 에러가 뜰 것이다!

마지막으로 이를 활용해 Retrofit을 생성한다.

```kotlin
    @Provides
    @Singleton
    fun providePingRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .addConverterFactory(MoshiConverterFactory.create(JsonAdapter.moshi))
            .client(okHttpClient)
            .baseUrl(BuildConfig.ATT_BASE_URL)
            .build()
    }
```

## 🔗 4. viewType에 따른 ViewHolder 정의

base 객체를 먼저 선언해주고 이를 상속해 필요한 viewHolder를 정의한다.

```kotlin
sealed class BaseSduiViewHolder(
    binding: ViewDataBinding
): RecyclerView.ViewHolder(binding.root) {
    abstract fun bind(reportContent: SduiReportContent)
}
```

```kotlin
class SduiPieChartViewHolder(
    private val binding: ItemSduiPiechartBinding
): BaseSduiViewHolder(binding) {
    override fun bind(reportContent: SduiReportContent) {
        setPieChart(reportContent as ReportPieChartVO)
    }

    private fun setPieChart(reportPieChartData: ReportPieChartVO) {
        binding.chartTitle = reportPieChartData.pieChartTitle
        val dataList = mutableListOf<PieEntry>()
        val colorList = mutableListOf<Int>()
        for (item in (reportPieChartData).pieChartItems) {
            dataList.add(PieEntry(item.categoryCount.toFloat(), item.categoryName))
            colorList.add(Color.parseColor(item.chartColor))
        }
    ...
    }
}
```

## 🔗 5. viewType - viewHolder의 손쉬운 매칭을 위한 enum 정의

하나의 recyclerView에 MultiViewType을 보여주기 위해서는 상황에 따라 필요한 viewHolder를 만들어내는 작업이 필요하다. 

이를 위해 viewType에 맞는 viewHolder를 반환하는 함수를 정의한다.

```kotlin
// layout을 활용해 viewType에 맞는 viewHolder 생성
fun getSduiViewHolder(parent: ViewGroup, viewType: SduiViewType): BaseSduiViewHolder {
    val layout = getLayoutByViewType(viewType)
    val binding: ViewDataBinding = DataBindingUtil.inflate(LayoutInflater.from(parent.context), layout, parent, false)

    return when(viewType) {
        SduiViewType.PIECHART -> SduiPieChartViewHolder(binding as ItemSduiPiechartBinding)
        SduiViewType.GRAPH -> SduiGraphViewHolder(binding as ItemSduiGraphBinding)
        SduiViewType.TEXT -> SduiTextViewHolder(binding as ItemSduiTextBinding)
        SduiViewType.UNKNOWN -> SduiUnknownViewHolder(binding as ItemSduiUnknownBinding)
    }
}

// viewType에 맞는 layout 반환
fun getLayoutByViewType(viewType: SduiViewType): Int {
    return when(viewType) {
        SduiViewType.PIECHART -> R.layout.item_sdui_piechart
        SduiViewType.GRAPH -> R.layout.item_sdui_graph
        SduiViewType.TEXT -> R.layout.item_sdui_text
        SduiViewType.UNKNOWN -> R.layout.item_sdui_unknown
    }
}

```

## 🔗 6. Multi - ViewType을 나타낼 수 있는 ListAdapter 생성하기

```kotlin
class SduiReportAdapter: ListAdapter<SduiReportItemVO, BaseSduiViewHolder>(
    ItemDiffCallback<SduiReportItemVO>(
        onContentTheSame = { old, new -> old == new },
        onItemsTheSame = { old, new -> old.content == new.content }
    )
) {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): BaseSduiViewHolder {
         return getSduiViewHolder(parent, SduiViewType.getViewTypeByOrdinal(viewType)) // 뒷부분에 나옴
    }

    override fun onBindViewHolder(holder: BaseSduiViewHolder, position: Int) {
        val item = getItem(position)
        holder.bind(item.content)
    }

    override fun getItemViewType(position: Int): Int {
        return getItem(position).viewType.ordinal
    }
}
```

이제 위에서 필요한 getViewTypeByOrdinal을 정의할 것이다. 이를 위해서 Sdui viewType을 모아둔 enum class를 정의한다.

```kotlin
enum class SduiViewType{
    PIECHART,
    GRAPH,
    TEXT,
    UNKNOWN;

    companion object {
        fun getViewTypeByOrdinal(ordinalNum: Int): SduiViewType {
            return values()[ordinalNum]
        }
    }
}
```
## 🔗 7. 이를 활용해 SDUI 구축

이제 이를 활용하기만 하면 된다!!

```kotlin
@AndroidEntryPoint
class SaleFragment : BaseFragment<FragmentSaleBinding>(R.layout.fragment_sale) {
    private lateinit var sduiReportAdapter: SduiReportAdapter
    private val saleViewModel by viewModels<SaleViewModel>()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        initSduiReportRecyclerView()
        setReportDataObserver()
        setReportData()
    }

    private fun initSduiReportRecyclerView() {
        sduiReportAdapter = SduiReportAdapter() // 활용
        binding.rvSduiReport.apply {
            adapter = sduiReportAdapter
            layoutManager = LinearLayoutManager(requireContext())
        }
    }

    ...
}

```

## 🔗 마무리하며

지금까지 moshi를 활용한 SDUI 적용법에 대해서 다뤄봤다. moshi에서 제공하는 `PolymorphicJsonAdapterFactory`를 활용한다면 손쉽게 원하는 여러 type으로 json을 파싱할 수 있다. Gson이 제공하는 JsonDeserializer를 활용하는 것보다 moshi를 활용하는 것이 더욱 간편하다는 것을 느꼈다.

또한, 해당 작업을 진행하면서 클래스 상속 vs 인터페이스 구현에 대해 고민해보는 시간을 가질 수 있었다. 
-  class 상속과 interfac는 모두 확장에 유연한 구조를 가진다.
- class를 상속할 경우, 상위 클래스와 강한 결합이 생성된다. 
  - 하위 클래스에는 필요없는 요소가 상위 클래스에 정의되었을 경우, 하위 클래스는 해당 요소를 가지게 된다. 
  - 하지만 interface의 경우, 주요 요소는 구현을 강조하지만 세부 구현은 각 클래스에게 맡긴다.