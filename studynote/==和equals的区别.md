
如标题所述两者间的区别如下：

* ==,对于基本数据类型的比较，用来比较两个基本数据类型的值是否相同，比如 2 == 2;对于引用数据类型的比较，用来比较两个引用数据类型的地址值是否相同；

* equals,用来比较引用数据类型的值是否相同,对于String类型用来比较两个字符串的值是否相等。


然而Java里的equals方法其实是交给开发者去覆写的，让开发者自己去定义满足什么条件的两个Object是equal的。

Java中默认的 equals方法实现如下：

```
public boolean equals(Object obj) {
    return (this == obj);
}
```

而String类则覆写了这个方法,直观的讲就是比较字符是不是都相同。

```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n-- != 0) {
                if (v1[i++] != v2[j++])
                    return false;
            }
            return true;
        }
    }
    return false;
}

```

