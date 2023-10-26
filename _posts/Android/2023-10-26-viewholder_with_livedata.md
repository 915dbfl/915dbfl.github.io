---
title: "[Error] viewHolder의 liveData가 observe 안된다?"
excerpt: "lifecycleOwner를 올바르게 설정하자!"
categories:
  - Android
tag:
  - liveData
  - Error

last_modified_at: 2023-10-26
toc: true
toc_sticky: true
search: true
---

## 👩🏻‍💻 문제 상황
1. AFragment 안에 recyclerView 존재
2. AFragment에 대한 viewModel 속 isModify: LiveData가 존재
3. isModify를 recyclerView 속 viewHolder에서 dataBinding으로 사용

=> 문제
- isModify값이 변경되어도 viewHolder에서 인식하지 못함

<br>

## 🤨 원인

### <span style = "background-color:#fff5b1">viewHolder는 lifecycleOwnwer가 아니기 때문에</span> liveData를 observe할 수 없음. 

- databinding을 진행하려면 binding에 lifecycleOwner가 필요하다.
- 그렇다면 `root.findViewTreeLifecycleOwner()`를 사용할 순 없을까?
  - `root.findViewTreeLifecycleOwner()`가 사용될 수 있는 시점은 <span style = "background-color:#fff5b1"> 화면에 attach</span> 되는 시점이다.
  - 하지만 <span style = "background-color:#fff5b1">bind -> attach<</span>가 이루어지기 때문에 null 값이 반환돼 결국 lifecycleOwner가 설정되지 않는다.

<br>

### 1. fragment -> adapter -> viewholder로 lifecycleOwnwer를 전해주면 되겠다!

이렇게 하면 가능하다! 하지만 [이렇게 했을 때 문제점](https://stackoverflow.com/questions/63461537/is-it-safe-to-pass-lifecycleowner-into-recyclerview-adapter)이 있다.

  ### <span style = "background-color:#fff5b1">viewHolder는 detach되어 더 이상의 view update가 필요하지 않지만 넘겨준 lifecycleOwner로 인해 liveData가 업데이트 되면 observe가 되는 것이 문제점이다!</span>

<br>

### 2. viewHolder에서 값을 observe해 ui를 업데이트하는 대신 recycleView의 데이터를 update해 ui를 업데이트하는 방식

직접 데이터를 업데이트하고 <span style = "background-color:#fff5b1">해당하는 위치만을</span> 적절히 notify해주는 방식이다.

여러 게시글을 살펴봤을 때, 해당 방법이 가장 이상적인 방법인듯 하다!

<br>

### + 추가

위에서 설명했듯이 `root.findViewTreeLifecycleOwner()`를 사용하려면 viewHolder가 화면에 attach된 후에 사용해야 한다.

```kotlin
    private var Owner: LifecycleOwner? = null
    init {
        itemView.doOnAttach {
            Owner = it.findViewTreeLifecycleOwner()
            binding.lifecycleOwner = Owner
        }
    }
```
`itemView.doOnAttach`를 통해서 <sapn style = "background-color:#fff5b1">attach한 후 진행할 부분을 작성할 수 있다.</span>
따라서 attach한 후 root의 lifecycleOwner를 가져와 binding에 lifecycleOwner로 설정해주면 liveData를 observe할 수 있게 된다.

</br>

## 👩🏻‍💻 참고
* <https://stackoverflow.com/questions/63461537/is-it-safe-to-pass-lifecycleowner-into-recyclerview-adapter>