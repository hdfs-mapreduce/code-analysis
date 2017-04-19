NullWritable详解
===================

NullWritable并不是一个null的封装，而是一个单独的特殊的Hadoop基本类型。NullWriteable的构造方法为空，没有任何数据的读写操作，
其值为空，没有长度计算。源码如下:
```
private NullWriteable () {}

public String toString() {
    return "(null)";        //确认返回值为null
}
public int hashCode() { return 0;}   //hashCode值为0
public int compareTo(Object other) {
    if ((other instanceof NullWriteable)) {
        return 0;
    }
}
```
NullWriteable类存在的目的就是用作占位符。如果在Hadoop中某个位置不需要使用一个具体的值，可以将之声明为NullWriteable,
结果是存储某个值为空的常量。

使用NullWriteable时，建议使用NullWriteable.get()获得一个真正值为null的数据值。
