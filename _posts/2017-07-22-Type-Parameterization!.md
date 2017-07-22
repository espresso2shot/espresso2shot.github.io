---
layout: post
title: 19장 타입 파라미터화
---

-  이장에서는정보 은닉 기법과 타입 파라미터화, 변성 표기에 대한 소개와 타입 파라미터와 변성간의 상호작용을 소개
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
  - head, tail이 원소 개수에 비례한 시간이 걸림..
- 이 둘을 짬뽕시키면 해결될 듯!
- 큐를 각각 leading과 trailing 두 리스트로 구현한다.
- leading 리스트는 앞으로부터 원소를 저장하고, trailing 리스트는 큐의 원소를 뒤로부터 거꾸로 저장
- 리스트 19.1 기본 함수형 큐

### 19.2 정보 은닉

- 비공개 생성자와 팩토리 메소드
  - 스칼라에서는 명시적으로 주 생성자를 정의하지 않음
  - 클래스 파라미터와 몸체에 의해 암시적으로 주 생성자가 만들어짐
  - 클래스의 파라미터 목록 바로 앞에 private 수식자를 붙여서 주 생성자를 감출 수 있음
```scala
class Queue[T] private (
  private val leading: List[T],
  private val trailing: List[T]
)
```
  - 이 생성자는 오직 클래스 자신과 동반 객체에서만 접근 가능
  - 클라이언트는 어떻게 객체 생성? 보조 생성자를 추가하거나 팩토리 메소드를 추가해서 방법을 제공할 수 있음
- 대안: 비공개 클래스
  - 더 급진적으로 클래스 자체를 감추고, 클래스에 대한 공개 인터페이스만을 제공하는 트레이트를 외부로 노출하는 방법이 있음
```scala
  trait Queue[T] {
    def head: T
    def tail: Queue[T]
    def enqueue(x: T): Queue[T]
  }
  object Queue {
    def apply[T](xs: T*): Queue[T] = new QueueImpl[T](xs.toList, Nil)
    private class QueueImpl[T] (
      private val leading: List[T],
      private val trailing: List[T]
    ) extends Queue[T] {
      def mirror = ...
      def head = ...
      def tail = ...
      def enqueue(x: T) = ...
    }
  }
```

### 19.3 변성 표기

- 위 예에서 Queue는 타입이 아니라 트레이트
- Queue가 타입이 아닌 이유는 타입 파라미터를 받기 때문. 그래서 Queue라는 타입의 변수를 만들 수 없음.
- Queue 트레이트는 Queue[String], Queue[Int] 처럼 파라미터화된 타입을 지정하도록 허용
- Queue는 트레이트이고, Queue[String]은 타입이다. Queue는 또한 *타입 생성자*라고도 한다. 타입 파라미터를 지정하면 타입을 만들 수 있기 때문.
- Queue를 제네릭 트레이트라 부를 수도 있음
- 제네릭이라는 말은 여러 타입을 고려해 포괄적으로 작성한 클래스나 트레이트를 가지고 여러 구체적인 타입을 정의할 수 있다는 뜻
- 공변적
  - S가 T의 서브타입이라면 Queue[S]를 Queue[T]의 서브타입으로 간주한다면 공변적
  - Queue[String]을 Queue[AnyRef]를 필요로 하는 곳에 넘길 수 있다는 뜻
- 스칼라에서는 제네릭 타입은 기본적으로 무공변
  - Queue[String]을 Queue[AnyRef] 대신 사용할 수 없다.
- trait Queue[+T]{...} 처럼 타입 앞에 + 를 붙여주면 서브타입 관계가 그 파라미터에 대해 공변적이라는 뜻
- -를 붙이면 반공변
  - S가 T의 서브타입이라면 Queue[T]를 Queue[S]의 서브타입으로 간주
- 어떤 파라미터의 공변, 반공변, 무공변 여부를 파라미터의 *변성*이라고 부른다.
- +, - 기호는 *변성 표기*라 부른다.
- 공변성으로 인해 타입 건전성이 위배되는 경우  
```scala
class Cell[T](init: T) {
  private[this] var current = init
  def get = current
  def set(x: T) { current = x }
}

//실제로는 통과할 수 없 코드
val c1 = new Cell[String]("abc")
val c2: Cell[Any] = c1
c2.set(1)
val s: String = c1.get 
```
  - 마지막 줄에서 타입 건전성을 위배!
- String Cell 


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
