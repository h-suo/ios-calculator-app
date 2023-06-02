# 계산기

> +, -, ×, ÷ 연산이 가능한 계산기 앱

</br>

## 목차

1. [팀원](#1.)
2. [시각화된 프로젝트 구조](#2.)
3. [타임라인](#3.)
4. [실행 화면(기능 설명)](#4.)
5. [트러블 슈팅](#5.)
6. [참고 링크](#6.)

---

</br>

<a id="1."></a>

## 1. 팀원

| [Erick](https://github.com/h-suo) |
| :---: |
| <img src="https://user-images.githubusercontent.com/109963294/235300758-fe15d3c5-e312-41dd-a9dd-d61e0ab354cf.png" height="150"/> | 

---

<a id="2."></a>

</br>

## 2. 시각화된 프로젝트 구조

### UML

<img src="https://github.com/h-suo/ios-calculator-app/assets/109963294/2d65f2ba-ce48-4a0e-9e08-2d32aa6070f4" height="600"/>

</br>

---

<a id="3."></a>

## 3. 타임라인

### **2023.05.29.(월)**
**Model 구현**
- `CalculatorItemQueue` 를 배열을 이용하여 구현
- `CalculateItem` 프로토콜 생성
    - Double 타입을 확장하여 프로토콜 채택
    - `Operator` 타입 생성 및 프로토콜 채택

**TDD**
- `CalculatorItemQueue`를 테스트하기 위한 `CalculatorItemQueueTests` 파일 생성 및 테스터 케이스 추가

### **2023.05.30.(화)**
**리펙토링**
- `CalculatorItemQueue`를 LinkedList로 개선
    - `CalculateItem` 타입을 데이터로 가지는 `CalculatorItemNode` 구현
    - `CalculatorItemNode` 를 관리하기 위한 `CalculatorItemLinkedList` 구현

### **2023.06.01.(목)**
**리펙토링**
- 프로퍼티들의 접근제어 수정
- 큐의 `dequeue()` 반환값을 사용하지 않아도 오류가 나오지 않도록 `@discardableResult` 추가

**학습**
- 프로토콜을 타입으로 사용할 때와 제네릭을 사용할 때의 성능 차이에 대해 학습
- 학습 내용: [TIL-SOLID, Generic 그리고 Protocol](https://github.com/h-suo/TIL/blob/main/2023.06/06.01%20SOLID%2C%20Generic%20그리고%20Protocol.md)

### **2023.06.02.(금)**
- README 작성

---

</br>

<a id="4."></a>

## 4. 실행 화면(기능 설명)

- **추후 추가 예정**

</br>

---

<a id="5."></a>

## 5. 트러블 슈팅

### queue

**🔥문제점**
- 처음 큐를 구현할때는 배열을 이용한 간단한 큐를 구현하였습니다.
- 하지만 `dequeue`에서 호출되는 `removeFirst`의 시간 복잡도가 O(n)이기 때문에 시간 복잡도가 증가하는 단점이 있었습니다.

<details>
<summary>본문 코드 내용</summary>
    
</br>

```swift
struct CalculatorItemQueue {
    var queue: [CalculateItem] = []
    
    func count() -> Int {
        return queue.count
    }
    
    func isEmpty() -> Bool {
        return queue.isEmpty
    }
    
    mutating func enqueue(_ calculateItem: CalculateItem) {
        queue.append(calculateItem)
    }
    
    mutating func dequeue() -> CalculateItem? {
        return queue.isEmpty ? nil : queue.removeFirst()
    }
}
```

</details>

</br>

**🧯해결방안**
- 더블 스택 큐와 포인터 큐 등 많은 큐 구현 방법을 고민했지만 그 중에서 연결리스트 큐를 사용하여 리펙토링을 진행하였습니다.
- 연결리스트 큐는 데이터와 다음 데이터의 주소값을 기억하는 노드들로 이루어진 자료구조인 연결리스트를 이용한 큐입니다.
    - 연결리스트 특징: 배열과 달리 데이터의 삽입과 삭제가 빠른 대신 검색 속도가 느리고 저장 공간 효율이 낮음(주소값도 기억해야하기 때문에)
- 위에 특징과 같이 검색 속도가 느릴 수는 있지만 큐에서는 데이터 검색이 아닌 삽입과 삭제만 이루어지기 때문에 배열보다 적합한 자료구조라고 생각했습니다.

<details>
<summary>본문 코드 내용</summary>
    
</br>

**CalculatorItemNode**
```swift
class CalculatorItemNode {
    var item: CalculateItem
    var next: CalculatorItemNode? = nil
    
    init(item: CalculateItem) {
        self.item = item
    }
}
```

**CalculatorItemLinkedList**
```swift
struct CalculatorItemLinkedList {
    var head: CalculatorItemNode?
    var tail: CalculatorItemNode?
    var count = 0
    
    mutating func append(_ calculatorItemNode: CalculatorItemNode) {
        if head == nil {
            head = calculatorItemNode
            tail = calculatorItemNode
        } else {
            tail?.next = calculatorItemNode
            tail = calculatorItemNode
        }
        
        count += 1
    }
    
    mutating func removeFirst() -> CalculatorItemNode? {
        guard head != nil else {
            clear()
            return nil
        }
        
        let removeNode = head
        head = head?.next
        count -= 1
        
        if head == nil {
            clear()
        }
        
        return removeNode
    }
    
    mutating func clear() {
        head = nil
        tail = nil
        count = 0
    }
}
```
    
**CalculatorItemQueue**
```swift
struct CalculatorItemQueue {
    private var queue = CalculatorItemLinkedList()
    
    var count: Int {
        return queue.count
    }
    
    var isEmpty: Bool {
        return queue.head == nil
    }
    
    mutating func enqueue(_ calculateItem: CalculateItem) {
        queue.append(CalculatorItemNode(item: calculateItem))
    }
    
    mutating func dequeue() -> CalculateItem? {
        return queue.removeFirst()?.item
    }
    
    mutating func clear() {
        queue.clear()
    }
}
```
    
</details>

---

</br>

<a id="6."></a>

## 6. 참고 링크

- [🍎 Apple-Array](https://developer.apple.com/documentation/swift/array)
- [📚 Swift-generics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/)
- [📚 위키백과-시간 복잡도](https://ko.wikipedia.org/wiki/시간_복잡도)
- [😺 GitHub-Queue](https://github.com/jeonyeohun/Data-Structures-In-Swift/tree/main/Queue)
- [🗒️ Swift 로 Queue 구현하기](https://trumanfromkorea.tistory.com/37)
- [🗒️ 마법 같은 Swift 제네릭 이야기](https://techblog.zepeto.me/마법-같은-swift-제네릭-이야기-2c222ae2798)
- [🗒️ Swift 성능 - 2. 프로토콜로 값타입 다형성 지원하기](https://velog.io/@yohanblessyou/Apple-Understanding-Swift-Performance-2.-프로토콜로-value-type-다형성-지원하기#-existential-container)

---
