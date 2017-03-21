Text类
===================

```
public class Text extends BinaryComparable implements WriteableComparable <BinaryComparable> {
    private static final byte [] EMPTY_BYTES = new byte[0]; //一个空的字节数组
    ...
    private byte[] bytes; //一个字节数组用于存放
    public Text(){ //默认空构造方法
        bytes = EMPTY_BYTES;  //对字节数组赋值
    }
    ...
    public Text(String string) {  //调用字符串构造Text
        set(string);  //调用set方法
    }
    ...
    public void set(String string) {  //字符串set方法
        try {
	    ByteBuffer bb = encode(string, true);  //将字符串转成可变字节
	    bytes = bb.array();  //对字符数组赋值
	    length = bb.limit();  //获得长度
	} catch (CharacterCodingException e) {  //异常处理
	    throw new RuntimeException("should not have happened " + e.toString());
	}
    }
    ...
    public void write(DataOutput out) throws IOException {  //传递一个输出流构建写出方法
        WritableUtils.writeVInt(out, length);  //创建写出长度
	out.write(bytes, 0, length);  //将字符数组整体写出
    }
    public void readFields(DataInput in) throws IOException {  //传递一个输入流构建方法
        int newLength = writeableUtils.readVint(in);  //获得要读取的字符长度
	setCapacity(newLength, false);  //设置容量
	in.readFully(bytes, 0, newLength); //在输入流读取数组
	length = newLength; //获得长度
    }

}
```

以上源码反映了Text类型默认的构造方法，通过构造一个空的字符数组并调用对应的set方法对Text类进行赋值。
被调用的set方法对传递进来的string类型变量进行读取并转换成一个可变字节，之后将其赋值给定义的字节数组。
最后调用了对应的write和readFiles方法通过形参定义了流形式对数据进行输出和输入处理。