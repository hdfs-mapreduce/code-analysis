IntWritable详解
===================

IntWritable可以理解为简单的int类型在Hadoop中的包装，实际上也是如此，部分源代码如下:
```
public class IntWritable implements WritableComparable {
    ...
    public IntWritable(int value) { set(value); }   //设置Interger中取值
    public void set(int value) { this.value = value; }  //设置Interger中取值
    public int get() { return value; }  //获取设定值
    public void readFields(DataInput in) throws IOException {  //创建读取方法
        value = in.readInt();  //通过输入流读取
    }
    public void write(DataOutput out) throws IOException {  //创建写入方法
        out.writeInt(value);  //通过输入流读取
    }
    ...
}
```
可以看出，IntWritable就是在其内部封装了一个int类型的数据，通过set和get方法对其进行值的存取。

为了实现IntWritable类之间的比较操作，IntWritable实现了两种比较方法。第一种是两个IntWritable之间的相互比较，直接对IntWritable中的值进行比较，
源代码如下:
```
public int compareTo(Object o) {
    int thisValue = this.value;   //对比数据准换类型
    int thatValue = ((IntWritable)o).value;  //将被对比的数据进行转换类型
    return (thisValue < thatValue ? -1 : (thisValue==thatValue ? 0: 1));  //进行比较处理
}
```
在compareTo方法中调用IntWritable类中的value方法获得了实际数据，即为原生的int类型，其后通过比较两个原生int类型的大小获得比较结果。

另一种则是通过实现WritableComparator接口中的compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2)方法,通过对流中序列化的数据直接比较
而省去了创建具体对象的资源调度，通过从指定字节中某个位置(s1, s2) 获取指定长度(l1, l2)而直接比较，源码如下：
```
public static class Comparator extends WritableComparator {
    ...
    public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {
        int thisValue = readInt(b1, s1);  //从int数组中读取数值
	int thatValue = readInt(b2, s2);  //从数组中读取比较数值
	return (thisValue < thatValue ? -1 : (thisValue == thatValue ? 0 : 1)); //进行比较处理
    }
}
```

### 提示
* 对IntWritable数据进行处理时，首先应该转化成int类型进行处理。 
* 大小比较方法，第二种快但程序代码编写较为困难，第一种需要消耗资源进行数据转换。