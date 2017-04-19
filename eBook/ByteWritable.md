ByteWritable详解
===================

ByteWritable是用来包装二进制数组的一个序列化类，其构造方法如下所示:
```
public ByteWriteable(byte[] bytes) {  //使用byte类构造方法
    this.bytes = bytes;               //对内部支持byte赋值
    this.size = bytes.length;         //获取长度
}
```

由构造方法来看，ByteWriteable接受一个二进制数组，通过相应的get()和set()方法对其取值和赋值。
但是值得注意的是，对于ByteWriteable来说，其长度的方法getLength()与获得返回数组值后去长度的值
不一样。

究其原因，ByteWriteable的实现中，除了保存了一个byte[]存放内容，还有一个int size用于表示byte数组
里面前多少位是有效的，后面是无效的，它们一起组成了ByteWriteable的长度。通过调用ByteWriteable的getBytes()
方法返回byte数组的全部内容(长度很可能大于size)，所以在Mapper中进行处理的时候应该只操纵size大小的内容，
后面的应该忽视。