当涉及到像HashMap等与哈希表结构相关的一些类时，会使用到hashCode方法
默认的hashCode实现一般是内存地址对应的数字，所以不同的对象，hashCode（）的返回值是不一样的
equals(object)相同时，hashCode（）的返回值也要尽量相同，当equals(object)不相同时，hashCode（）的返回没有特别的要求，但是也是尽量不相同以获取好的性能

