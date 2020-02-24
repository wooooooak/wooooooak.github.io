---
layout: post
title: Databinding without onCreateView - onCreateView is deprecated.
category: Android
tags: [Fragment]
comments: true
---

## Fragment#onCreatView is deprected

`Fragment#onCreatView`가 API level 28부터 deprecated되었다.

이전까지 Fragment를 사용할 때 주로 사용했던 코드는 아래와 같았다.

```kotlin
class MyFragment : Fragment() {
    ...
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(layoutInflater, layoutId, null, false)
        return binding.root
    }
    ...
}
```

데이터 바인딩을 사용하지 않을 경우는 return 부분이 아래와 같았을 것이다.

```kotlin
return inflater.inflate(layoutId, container, false)
```

## How to work without `onCreateView`

기본적으로 Fragment는 OnCreateView 없이도 동작할 수 있다. 다만 Fragment의 생성자로 layoutId를 넘겨줘야 한다.

```kotlin
class MyFragment : Fragment(R.fragment.my_fragment) {
    ...
}
```

데이터바인딩을 사용한다면, `onViewCreated`를 이용하면 된다.

```kotlin
class MyFragment : Fragment(R.fragment.my_fragment) {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        binding = FragmentMyBinding.bind(view)
    }
    ...
}
```
