---
layout: post
title: "List of DTO 에 Bean Validation을 적용하기"
subtitle: "..."
date: 2022-10-18 14:00:00 +0900
categories: backend
tags: spring
comments: true
published: true
---

`단일 DTO`나 `DTO로 감싼 일급컬렉션`의 경우 Bean Validation이 적용이 됩니다.

하지만 List of DTO는 그렇지 않은데요! (정확한 건 아래의 예시)

```java
@PostMapping("...")
public ... addBeacons(@RequestBody @Valid ValidList<BeaconRequest> requestBody) {
  ...
}
```

이런 경우 List 구현와 @Valid를 이용해서 작성해볼 수 있습니다 !

아래 처럼요!

```java
public class ValidList<E> implements List<E> {

    @Valid
    private List<E> list;

    public ValidList() {
        this.list = new ArrayList<>();
    }

    public static <E> ValidList<E> of(final E... es) {
        final ValidList<E> validList = new ValidList<>();
        for (final E e : es) {
            validList.add(e);
        }
        return validList;
    }

    ...
}
```



```java
public class ValidList<E> implements List<E> {

    @Valid
    private List<E> list;

    public ValidList() {
        this.list = new ArrayList<>();
    }

    public static <E> ValidList<E> of(final E... es) {
        final ValidList<E> validList = new ValidList<>();
        for (final E e : es) {
            validList.add(e);
        }
        return validList;
    }

    public List<E> getList() {
        return list;
    }

    public void setList(final List<E> list) {
        this.list = list;
    }

    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    @Override
    public E get(final int index) {
        return list.get(index);
    }

    @Override
    public E set(final int index, final E element) {
        return list.set(index, element);
    }

    @Override
    public boolean add(final E e) {
        return list.add(e);
    }

    @Override
    public void add(final int index, final E element) {
        list.add(index, element);
    }

    @Override
    public boolean addAll(final Collection<? extends E> c) {
        return list.addAll(c);
    }

    @Override
    public boolean addAll(final int index, final Collection<? extends E> c) {
        return list.addAll(index, c);
    }

    @Override
    public boolean contains(final Object o) {
        return list.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return list.iterator();
    }

    @Override
    public boolean containsAll(final Collection<?> c) {
        return list.containsAll(c);
    }

    @Override
    public boolean removeAll(final Collection<?> c) {
        return list.removeAll(c);
    }

    @Override
    public boolean retainAll(final Collection<?> c) {
        return list.removeAll(c);
    }

    @Override
    public void clear() {
        list.clear();
    }

    @Override
    public E remove(final int index) {
        return list.remove(index);
    }

    @Override
    public int indexOf(final Object o) {
        return list.indexOf(o);
    }

    @Override
    public Object[] toArray() {
        return list.toArray();
    }

    @Override
    public <T> T[] toArray(final T[] a) {
        return list.toArray(a);
    }

    @Override
    public boolean remove(final Object o) {
        return list.remove(o);
    }

    @Override
    public int lastIndexOf(final Object o) {
        return list.lastIndexOf(o);
    }

    @Override
    public ListIterator<E> listIterator() {
        return list.listIterator();
    }

    @Override
    public ListIterator<E> listIterator(final int index) {
        return list.listIterator(index);
    }

    @Override
    public List<E> subList(final int fromIndex, final int toIndex) {
        return list.subList(fromIndex, toIndex);
    }
}
```
