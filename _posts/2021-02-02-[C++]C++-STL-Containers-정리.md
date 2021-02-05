---
title:  "[C++] 코딩테스트를 위한 STL Containers 정리 (vector, queue, stack, list, set, map)"
excerpt: "C++ STL Containers"
categories: Code
tags: 
- C++ STL Container
- vector, queue, stack, list
- set, map
author_profile: true
---

 코딩테스트에서 자주 쓰이는 `vector`, `queue`, `stack`부터 활용할 수 있으면 도움이 될 `list`, `set`, `map`까지 정리해보겠다. 여러 C++ STL Container들을 사용할 수 있으면 문제의 유형에 따라 로직을 짜는데 도움이 될 것 같다..

<br>

# std::vector

 `vector`는 기본적으로 `push_back()`, `pop_back()`을 이용하여 데이터를 삽입, 삭제하지만 `insert(iterator, value)`, `erase(iterator)`을 사용하여 중간 원소들에 대해서도 삽입, 삭제가 가능하다. **하지만** 시간 소요가 많이 발생하므로 중간 원소들에 대해 삽입, 삭제를 자주 사용한다면 list나 다른 container을 사용하는 편이 좋다.

 index([])로 중간 원소의 값을 참조할 수 있는 점이 queue, stack보다 좋은 점이다. 중간 원소에 대해 삽입, 삭제가 적고 중간값을 자주 참조한다면 vector을 사용하는 편이 좋다.

<br>

## vector의 선언 방법

```cpp
#include <vector>

vector<T> v;
vector<T> v(10);
vector<T> v(10,1);
vector<T> v(vtemp);
```

<br>

vector의 선언 방식은 위와 같다. template을 사용하기 때문에 T에 원하는 자료형을 넣어주면 된다.

<br>

+ `vector<T> v;`
  + 기본적으로 가장 많이 쓰는 방법이다.
  + 비어있는 vector를 생성한다.
+ `vector<T> v(10);`
  + 기본값으로 초기화된 10칸을 가진 vector를 선언한다.
  + int형은 기본값 0
+ `vector<T> v(10,1);`
  + 1로 초기화 된 10칸을 가진 vector을 선언한다.
+ `vector<T> v(vtemp);`
  + vtemp라는 vector를 복사하여 똑같은 값을 가진 v(vector)를 만든다.

<br>

## vector의 멤버 함수

### Iterator 관련

+ `v.begin()`
  + 맨 앞 원소의 iterator을 반환한다.
+ `v.end()`
  + 맨 뒤 원소의 다음칸 iterator을 반환한다.
  + 보통 반복하며 vector의 끝까지 참조했는지 확인할 때 사용한다.
    ex. `if(iter==c.end())`
+ `v.rbegin()`
  + reverse begin으로 뒤에서 맨 첫 번째 원소의 iterator을 반환한다.
+ `v.rend()`
  + reverse end로 뒤에서부터 세었을 때 맨 뒤 원소의 다음 칸 iterator을 반환한다.
  + 첫 번째 원소의 이전칸을 말한다.

<br>

### Capacity 관련

+ `v.size()`
  + vector의 size를 반환한다.
+ `v.resize(n)`
  + vector의 size를 n으로 설정한다.
  + 만약 원래보다 더 크게 설정한다면 기본값으로 채워진다.
+ `v.capacity()`
  + vector의 capacity를 반환한다.
  + size와 capacity는 다른다. 
    + vector의 원소가 하나 추가될 때마다 동적할당을 하면 비효율적이기 때문에 삽입 시 기존 size가 꽉찼다면 capacity를 1,2,4,8,16,32,64 등 이전값에 2배씩 늘려준다.
      ex. size=6 → capacity=8
+ `v.empty()`
  + vector에 원소가 없다면 1, 있다면 0을 반환한다.

<br>

### Element Access 관련

+ `v[idx]`
  + vector의 idx번째 원소값을 참조한다.
+ `v.at(idx)`
  + vector의 idx번째 원소값을 참조한다.
+ `v.front()`
  + vector의 첫 번째 원소값을 참조한다.
+ `v.back()`
  + vector의 마지막 원소값을 참조한다.

<br>

### Modifier 관련

+ `v.push_back()`
  + vector의 맨 뒤에 원소를 삽입한다.
+ `v.pop_back()`
  + vector의 맨 뒤 원소를 삭제한다.
+ `v.insert(iterator, value)`
  + iterator 위치에 value를 삽입한다.
+ `v.erase(iterator)`
  + iterator에 해당하는 원소를 삭제한다.
+ `v.clear()`
  + vector의 모든 원소를 지운다.

<br>

# std::queue

# std::stack

# std::list

# std::set

# std::map

> 다른 container은 다음에 추가하겠다...ㅎ

> 참고 : http://www.cplusplus.com/reference/vector/vector/