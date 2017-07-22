---
layout: post
title: 19장 타입 파라미터화
---

-  이장에서는정보 은닉 기법과 타입 파라미터화, 변성 표기에 대한 소개와 타입 파라미터와 변성간의 상호작용을 소개한다.
  - 타입 파라미터화를 사용하면 제네릭 클래스와 트레이트를 쓸 수 있음
  - 예를들면, Set은 제네릭이고, 타입 파라미터를 받기 때문에 타입이 Set[T]
- 자바에서는 타입의 파라미터를 쓰지 않아도 되지만, 스칼라에서는 반드시 타입 파라미터를 명시해야만 됨
- 타입 변성은 파라미터 타입 간의 상속 관계를 지정
  - Set[String]이 Set[AnyRef]의 하위 집합인지와 같은 것이 변성에 의해 결정됨
  
### 19.1 함수형 큐

- 함수형 큐는 원소를 추가해도 내용을 바꾸지 않고 새로운 원소를 추가한 새 큐를 반환
- 순수 함수형 큐는 리스트와 유사
- 둘 다 확장이나 변경을 한 다음에도 원래의 객체는 그대로 남아 있는 완전히 변하지 않는 데이터 구조
- head,tail, enqueue 세가지 기본 연산이 상수 시간이 걸려야 한다.
```scala
class SlowAppendQueue[T](elems: List[T]) {
  def head = elems.head
  def tail = new SlowAppendQueue(elems.tail)
  def enqueue(x: T) = new SlowAppendQueue(elems ::: List(x))
}
```
  - enqueue가 문제. 원소 개수에 비례한 시간이 걸림
  
```scala
class SlowHeadQueue[T](smele: List[T]) {
  def head = smele.last
  def tail = new SlowHeadQueue(smele.init)
  def enqueue(x: T) = new SlowHeadQueue(x :: smele)
}
```
  - head, tail이 원소 개수에 비례한 시간이 걸림
  
- 이 둘을 짬뽕시키면 해결될 듯!
- 큐를 각각 leading과 trailing 두 리스트로 구현한다.
- leading 리스트는 앞으로부터 원소를 저장하고, trailing 리스트는 큐의 원소를 뒤로부터 거꾸로 저장
- 리스트 19.1 기본 함수형 큐

### 19.2 정보 은닉

- 비공개 생성자와 팩토리 메소드
  - 스칼라에서는 명시적으로 주 생성자를 정의하지 않음
  - 클래스 파라미터와 몸체에 의해 암시적으로 주 생성자가 만들어짐
  - 클래스의 파라미터 목록 바로 앞에 private 수식자를 붙여서 주 생성자를 감출 수 있음
  - 
```scala
class Queue[T] private (
  private val leading: List[T],
  private val trailing: List[T]
)
```
  - 이 생성자는 오직 클래스 자신과 동반 객체에서만 접근 가능
  - 클라이언트는 어떻게 객체 생성? 보조 생성자를 추가하거나 팩토리 메소드를 추가해서 방법을 제공할 수 있음

```scala
def orderedMergeSort[T <: Ordered[T]] (xs: List[T]): List[T] = {
  def merge(xs: List[T], ys: List[T]): List[T] =
    (xs, ys) match {
      case (Nil, _) => ys
      case (_, Nil) => xs
      case (x :: xs1, y :: ys1) =>
        if (x < y) x :: merge(xs1, ys)
        else y :: merge(xs, ys1)
    }
  val n = xs.length / 2
  if (n == 0) xs
  else {
    val (ys, zs) = xs splitAt n
    merge(orderedMergeSort(ys), orderedMergeSort(zs))
  }
}


```
