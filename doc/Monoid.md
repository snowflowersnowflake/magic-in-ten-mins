# 十分钟魔法练习：单位半群

### By 「玩火」

> 前置技能：Java基础

## 半群（Semigroup）

半群是一种代数结构，在集合 `A` 上包含一个将两个 `A` 的元素映射到 `A` 上的运算即 `<> : (A, A) -> A​` ，同时该运算满足**结合律**即 `(a <> b) <> c == a <> (b <> c)` ，那么代数结构 `{<>, A}` 就是一个半群。

比如在自然数集上的加法或者减法可以构成一个半群，再比如字符串集上字符串的连接构成一个半群。

## 单位半群（Monoid）

单位半群是一种带单位元的半群，对于集合 `A` 上的半群 `{<>, A}` ，`A`中的元素`a`使`A`中的所有元素`x`满足 `x <> a` 和 `a <> x` 都等于 `x`，则 `a` 就是 `{<>, A}` 上的单位元。

举个例子，`{+, 自然数集}`的单位元就是0，`{*, 自然数集}`的单位元就是1，`{+, 字符串集}`的单位元就是空串`""`。

用Java代码可以表示为：

```java
public interface Monoid<T> {
    public static T empty();
    public static T append(T a, T b);
    public static default T appends(Stream<T> x) {
        return x.reduce(empty(), this::append);
    }
}
```

## 应用：Optional

在Java8中有个新的类`Optional`可以用来表示可能有值的类型，而我们可以对它定义个Monoid：

```java
public class OptionalM<T> implements Monoid<Optional<T>> {
    public static Optional<T> empty() {
        return Optional<T>.empty();
    }
    public static Optional<T> append(Optional<T> a, Optional<T> b) {
        if (a.isPresent()) return a;
        else return b;
    }
}
```

这样对于appends来说我们将获得一串Optional中第一个不为空的值，对于需要进行一连串尝试操作可以这样写：

```java
OptionalM<int>.appends(Stream.of(try1(), try2(), try3(), try4()))
```

## 应用：Ordering

对于`Comparable`接口可以构造出：

```java
public class OrderingM implements Monoid<int> {
    public static int empty() { return 0; }
    public static int append(int a, int b) {
        if (a == 0) return b;
        else return a;
    }
}
```

同样如果有一串带有优先级的比较操作就可以用appends串起来，比如：

```java
public class Student implements Comparable {
    public String name;
    public String sex;
    public Date birthday;
    public String from;
    public compareTo(Object o) {
        Student s = (Student)(o);
        return OrderingM.appends(Stream.of(
            name.compareTo(s.name),
            sex.compareTo(s.sex),
            birthday.compareTo(s.birthday),
            from.compareTo(s.from)
        ));
    }
}
```

这样的写法比一连串`if-else`优雅太多。

## 扩展

在Monoid接口里面加default方法可以支持更多方便的操作：

```java
public interface Monoid<T> {
    //...
    public static default T when(boolean c, T then) {
        if (c) return then;
        else return empty();
    }
    public static default T cond(boolean c, T then, T els) {
        if (c) return then;
        else return els;
    }
}

public class Todo implements Monoid<Runnable> {
    public static Runnable empty() {
        return () -> {};
    }
    public static Runnable append(Runnable a, Runnable b) {
        return () -> { a(); b(); };
    }
}
```

然后就可以像下面这样使用上面的定义:

```java
Todo.appends(Stream.of(
    logic1,
    () -> { logic2(); },
    Todo.when(condition1, logic3)
))
```

> 注：上面的Optional并不是lazy的，实际运用中加上非空短路能提高效率。