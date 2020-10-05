---
layout: post
title: HackerRank - Organizing Containers of Balls
category: Algorithm
tags: [algorithm, kotlin]
comments: true
---

# Organizing Containers of Balls

[Organizing Containers of Balls 링크](https://www.hackerrank.com/challenges/organizing-containers-of-balls/problem)

- 초기에 각 컨테이너에 들어있는 공의 갯수는 절대 바뀔수 없다(서로 맞바꾸는 행위밖에 없으므로)
- 특정 컨테이너에는 특정 타입의 공으로만 채워져야 한다. 즉 특정 컨테이너는 애초에 특정 타입의 공만 수용할 수 있도록, 특정 타입의 공 갯수 만큼 공을 가지고 있어야만 한다.

초록색 공의 갯수를 10개라고 가정하자. 그렇다면 이미 (어떤 색깔들일지는 몰라도) 공 10개를 담고있는 컨테이너가 존재해야만 한다. 초기에 정해진 컨테이너 안의 공 갯수는 절대 바뀔수가 없기 때문에 꼭 10개를 담고 있는 컨테이너가 있어야만 한다(초기에 담긴 공 갯수는 절대 바뀔수 없다는 점을 주의하자).

마찬가지로 빨간색 공의 갯수가 12개라면, 12개를 미리 담고 있는 컨테이너가 꼭 필요하다. 초기에 11개 혹은 13개를 담고 있는 컨테이너에는 절대 빨간색을 넣을 수 없다.

다른 색의 공이 더 있다고 해도 마찬가지 로직이 반복된다.

결국 (특정 컨테이너가 가진 공의 합 == 특정 타입 공의 합)쌍이 모두 매칭되어야 하며, 하나라도 매칭되지 않으면 불가능하다.

```kotlin
fun organizingContainers(container: Array<Array<Int>>): String {
    val ballsCountOfEachContainer = mutableListOf<Int>()
    val ballsCountOfEachColor = mutableListOf<Int>()
    for (x in container.indices) {
        ballsCountOfEachContainer.add(container[x].sum())
        var sum = 0
        for (y in container.indices) {
            sum += container[y][x]
        }
        ballsCountOfEachColor.add(sum)
    }
    ballsCountOfEachContainer.sort()
    ballsCountOfEachColor.sort()

    ballsCountOfEachContainer.forEachIndexed { index, countOfCotainer ->
        if (countOfCotainer != ballsCountOfEachColor[index]) {
            return "Impossible"
        }
    }
    return "Possible"
}
```
