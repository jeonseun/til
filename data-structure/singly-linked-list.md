# 단일 연결 리스트 구현

[전체 소스코드](https://github.com/jeonseun/til-code/blob/main/data-structure/src/main/java/me/seun/list/SinglyLinkedList.java)

단일 연결 리스트는 리스트를 구성하는 노드의 링크가 하나인 연결 리스트를 말한다. 형태는 다음과 같다.

![단일 연결 리스트](./img/singly-linked-list.svg)

모든 구현은 [연결 리스트의 개념](../data-structure/linked-list-concept.md)에서 정의한 `MyList` 인터페이스를 구현하는 것을 기준으로 한다.

## 노드 정의

단일 연결 리스트는 자신의 다음 노드만 가리키기 때문에 하나의 링크와 저장할 값을 가지면 된다. java 코드로 작성하면 다음과 같다.

```java
public class SinglyLinkedList<E> implements MyList<E> {
    private Node<E> head; // 시작 노드를 가리킬 참조

    private Node<E> tail; // 끝 노드를 가리킬 참조

    private static class Node<E> {
        E item; // 저장할 값
        Node next; // 다음 노드를 가리킬 참조

        Node(E item, Node next) {
            this.item = item;
            this.next = next;
        }
    }
}
```

## 삽입 연산

리스트에 요소를 추가하는 연산은 추가하는 위치에 따라 3가지로 구분할 수 있다.

- 리스트 처음에 요소 추가
- 리스트 중간에 요소 추가
- 리스트 끝에 요소 추가

3가지 연산 모두 삽입 연산을 시작하는 위치만 다를 뿐 맥락은 거의 같다. 단 노드가 이미 존재하는 상태에서 추가하는 경우 노드의 링크가 끊어지지 않도록 주의해야 한다.

### 리스트 처음에 요소 추가

리스트에 처음에 요소를 추가할 때는 head가 가리키는 노드가 있는지 확인해야 한다. head가 가리키는 노드가 있는 경우 추가할 노드의 링크를 기존 head가 가리키는 노드로 연결하고 head를 새 노드로 변경해야 한다.

![단일 연결 리스트 처음 노드 추가](img/singly-linked-list-add-first.svg)

만약 head가 가리키는 노드가 없는 경우 head가 새 노드를 가리키도록 하고 마지막 노드를 가리키는 tail을 별도로 유지하는 경우 tail이 새 노드를 가리키도록 하면된다.

```java
@Override
public void addFirst(E e) {
    Node<E> newNode = new Node<>(e, null);

    // head가 가리키는 노드가 없는 경우
    if (head == null) {
        head = newNode;
        tail = newNode;
        return;
    }

    // head가 가리키는 노드가 있는 경우
    Node<E> temp = head;
    head = newNode;
    newNode.next = temp;
}
```

head가 기리키는 노드가 있을 때 해당 노드를 따로 저장해 두지 않고 head를 새 노드로 변경하면 해당 노드에 접근할 방법이 없어지므로 주의해야 한다.

### 리스트 중간에 요소 추가

리스트의 중간(특정 노드의 뒤)에 요소를 추가하기 위해서는 우선 대상 노드를 찾은 뒤 새 노드가 대상 노드의 다음 노드를 가리키도록 링크를 연결해야한다. 이후 대상 노드가 새 노드를 가리키도록 해주면된다.

![단일 연결 리스트 중간 노드 추가](./img/singly-linked-list-add-after.svg)

리스트가 비어있을 경우 연산을 수행할 필요가 없으므로 다른 작업에 앞서서 리스트가 비어있는지 확인한다. 이후 head 부터 시작해서 대상 노드를 찾을 때까지 링크를 타고가며 대상 노드를 찾으면 앞서말한대로 노드간 링크를 다시 연결해주면 된다.

```java
@Override
public void addAfter(E target, E e) {
    validateEmptyList(); // 비어있는 리스트인지 확인

    Node<E> cur = head; // 리스트 순회를 위한 변수

    // 리스트 끝에 도달하거나 대상 노드를 찾을 때까지 진행
    while (cur != null && !cur.item.equals(target)) {
        cur = cur.next;
    }

    // 대상 노드를 찾지 못하고 리스트 끝에 도달한 경우
    if (cur == null) {
        throw new NoSuchElementException();
    }

    // 대상 노드를 찾은 경우
    cur.next = new Node<>(e, cur.next);
}

```

### 리스트 끝에 요소 추가

리스트 끝에 요소를 추가할 때는 끝 노드를 가리키는 tail노드에 다음 노드로 새 노드를 추가하고 tail이 새 노드를 가리키도록 변경해주면 된다. 만약 별도의 tail노드를 유지하지 않는다면 head부터 시작해서 끝 노드까지 순회해야 한다. 만약 빈 리스트인 경우 head가 가리키는 노드로 새 노드를 추가하면 된다.

```java
@Override
public void addLast(E e) {
    Node<E> newNode = new Node<>(e, null);

    // 빈 리스트인 경우
    if (head == null) {
        head = newNode;
        tail = newNode;
        return;
    }

    // 다른 노드가 존재하는 경우
    tail.next = newNode;
    tail = newNode;
}
```

## 삭제 연산

리스트의 요소 삭제는 삭제할 대상 노드를 찾고 해당 노드의 앞과 뒤 노드를 연결해주면 된다.

![단일 연결 리스트 노드 삭제](./img/singly-linked-list-remove.svg)

만약 head가 가리키는 노드가 대상이 될 경우 head의 링크를 대상 노드의 다음 노드로 연결하면 되고 이외의 경우에는 대상 노드를 링크를 타고가며 찾아야 한다. 이 때 대상 노드를 없애려면 대상 노드의 이전 노드에서 링크를 변경해주어야 하므로 현재 노드를 가리킬 cur과 직전 노드를 가리킬 prev로 총 두개의 임시 변수를 써서 리스트를 순회해야 한다. 비어있는 리스트일 경우 별도의 작업이 필요 없으므로 다른 작업에 앞서서 확인한다.

```java
@Override
public void remove(E target) {
    validateEmptyList(); // 비어있는 리스트인지 확인
    
    // head가 삭제 대상 노드인 경우
    if (head.item.equals(target)) {
        head = head.next;
        return;
    }

    Node<E> prev = head; // 직전 노드에 대한 참조
    Node<E> cur = head.next; // 현재 노드에 대한 참조

    // 끝 노드에 도달할 때 까지 진행
    while (cur != null) {
        // 삭제 대상 노드인 경우
        if (cur.item.equals(target)) {
            prev.next = cur.next;
            return;
        }
        prev = cur;
        cur = cur.next;
    }
    // 삭제 대상 노드를 찾지 못한 경우
    throw new NoSuchElementException();
}
```

대상 노드를 찾은 경우 직전 노드의 링크가 대상 노드의 다음 노드에 향하도록 연결시키면 된다. 이렇게 하면 대상 노드에 대한 참조가 사라지고 자바 기준에서 unreachable한 객체가 되므로 gc의 대상이 되어 이후 메모리에서 반환된다.

## 빈 리스트 확인 연산

리스트가 비어있는지 확인하는 방법은 head가 null인지 확인하면 된다. 하나라도 요소가 추가된 경우 head는 특정 노드를 가리키므로 null일 수 없다.

```java
@Override
public boolean isEmpty() {
    return head == null;
}
```

## 리스트 내 특정 요소의 존재 여부 확인 연산

리스트 내의 특정 요소의 존재 여부를 확인하려면 순차 접근을 통해 일치하는 요소를 찾아야 한다. 리스트가 비어있는 경우 특정 요소가 존재할 수 없으므로 다른 작업에 앞서서 확인해주면 된다.

head부터 링크를 타고가며 대상을 찾는다. 대상이 있는 경우 순회를 멈추도록 작성하면된다. 만약 순회가 끝났는데 순회에 사용된 임시 변수 cur의 값이 null인 경우 대상을 찾지 못하고 끝에 도달했다는 뜻이므로 이점을 사용해서 존재 여부를 확인하면 된다.

```java
@Override
public boolean contains(E e) {
    validateEmptyList(); // 비어있는 리스트인지 확인

    Node<E> cur = head;

    // 리스트 끝에 도달하거나 대상 노드를 찾을때 까지 진행
    while (cur != null && !cur.item.equals(e)) {
        cur = cur.next;
    }
    // cur이 null이면: 대상 노드를 찾지 못하고 리스트 끝에 도달한 경우
    // cur이 null이 아니면: 대상 노드를 찾은 경우
    return cur != null;
}
```

## 리스트 사이즈 확인 연산

리스트에 들어있는 요소의 개수를 확인하려면 시작 노드부터 끝 노드까지 순회하면서 하나씩 카운트하면 된다.

그러나 별도의 size 변수를 유지하면서 요소의 삽입, 삭제 시 마다 size 변수의 값을 증감시키도록 구현할 경우 단순히 size 변수의 값을 확인함으로써 리스트의 사이즈를 구할 수 있다.

## 리스트 반복자 제공

리스트의 모든 요소를 반복하기 위한 `Iterator` 구현체를 제공한다. 제공할 `Iterator` 구현체는 다음과 같다.

```java
public class SinglyLinkedList<E> implements MyList<E> {
    @Override
    public Iterator<E> iterator() {
        return new SLLIterator<>(head);
    }

    ...
    private static class SLLIterator<E> implements Iterator<E> {
        private Node<E> cur;

        public SLLIterator(Node<E> cur) {
            this.cur = cur;
        }

        @Override
        public boolean hasNext() {
            return cur != null;
        }

        @Override
        public E next() {
            E item = cur.item;
            cur = cur.next;
            return item;
        }
    }
}
```

단일 연결 리스트의 노드를 순회할 수 있도록 시작 노드인 head를 받아 cur 변수에 유지하면서 반복에 사용한다. cur 변수로 요소에 접근하고 하나의 요소에 접근하고 나면 cur을 다음 노드로 이동시키는 방식으로 구현한다. 따라서 cur의 값이 null인지 아닌지로 다음 요소가 존재하는지 확인할 수 있도록 구현한다.