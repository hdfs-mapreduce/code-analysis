ObjectWritable详解
===================

Hadoop在运行过程中需要不停地对各个节点进行通信，而不同的数据类型对通信的影响很大。
因此，在借鉴Java中Object类的基础上，Hadoop定义了一个新的类ObjectWritable。

ObjectWritable类中并不是对某些Hadoop的基本类型进行封装，而是直接对Java中的某些类型进行封装，例如String、Int、enum等

通过源码分析我们可以看到 ObjectWritable 在构造出来的时候自动传递了一个Object类型的对象作为其内部对象
```
public ObjectWritable(Class declaredClass, Object instance) {
    this.declaredClass = declaredClass;  //获取类型
    this.instance = instance;  //创建类型实例
}
```

ObjectWritable 还提供了一个写方法可以将对象实例写到输出流中，代码如下：
```
public void write(DataOutput out) throws IOException {  //创建写入方法
    writeObject(out, instance, declaredClass, conf);    //调用原生写入方法
}
```

ObjectWritable类不过是在一个基本Object上进行了一层包装，观察write(DataOutput out)方法，其在内部调用了writeObject(out, instance, declaredClass, conf)
方法，部分源码如下
```
if (declaredClass == String.class) {                      //如果为String类型
    UTF8.writeString(out, (String)instance);              //使用UTF8默认写方法
} else if (declaredClass == Byte.TYPE) {                  //如果为byte类型
    out.writeByte( ((Byte)instance)instance.byteValue()); //使用byte写方法
}
```

ObjectWritable通过传递进来的declaredClass获取具体的Class类型，通过调用其特定的写方法，将数据输出到相应的流中，完成数据的写工作。而对于 readObject(DataInput in, Configuration conf)
方法，通过判断相应的declaredClass类型，将也顶类型读取到相应的 Objectinstance中，最终将其返回。
```
...
public void readFields(DataInput in) throws IOException {  //创建读取方法
    readObject(in, this, this.conf);                       //进行数据读取 
}
...
public static Object readObject(DataInput in, ObjectWriteable objectWriteable, Configuration conf) throws IOException {
    Object instance;                                       //创建对象用于构造实例
    String ClassName = UTF8.readString(in);                //使用UTF8格式在输入流读取字符
    Class<?> declareClass = PRIMITIVE_NAMES.get(className);//获取输入类型
    ...
    if (declaredClass == String.class) {                   //如果为String类型
        instance = UTF8.readString(in);                    //使用UTF8默认方法
    } else if (declaredClass == Byte.TYPE) {               //如果为byte类型
        instance = Byte.valueOf(in.readByte());            //使用byte默认读方法
    }
    ...
    return instance;
}
```

