SequenceFile 详解
===================

Hadoop天生就是为了处理大型文件而诞生的，但是并不意味着其不能够用于处理大量的小文件。
这里的小文件是指文件的size小于HDFS中每个Block大小的文件。

首先，我们知道Hadoop的文件索引地址是以元数据的形式存在在NameNode内存中的，过多的小文件同事存在会占用大量的内容空间，
影响NameNode性能。

其次，如果NameNode中也存储有大量的小文件，那么Hadoop在执行时，需要频繁的访问各个DataNode，同时还要不停的生成与释放相关资源。
这样频繁调度非常耗费Hadoop框架的处理能力，成为影响Hadoop计算性能的瓶颈。

因此，需要对小文件进行处理。推荐的方式是使用SequenceFile 格式对文件进行处理。通过将若干规模较小的文件序列化后直接存储到
一个文件中，构造出一个伪文件形式。例如处理多条日志记录时，使用小文件的文件名作为key，而内容作为其Value进行整合。

SequenceFile是将文件经过二进制处理后，生成的具有特定键值对的二进制文件流，简单的说是生成一套Hadoop自己的I/O操作，以便对文件进行读写。
这样做的好处主要有以下特点:
* 便于操作。因为其使用Hadoop中自带的工具，可以方便地进行后期更改。
* 支持定制大小。可以将文件输出问既定大小的文件，例如将若干个文件生成Block规格大小的文件，便于后期再每个节点上进行计算。
* SequenceFile中自带压缩格式可以为输出结果指定相应的压缩格式以便于存储。

```
import java.util.Random;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.SequenceFile;
import org.apache.hadoop.io.Text;

public class SequenceWriteSample {
    private static String meg = "hello word";   //设定字符串写入
    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration();  //获取环境变量
        FileSystem fs = FileSystem.get(conf);       //获取文件系统
        Path path = new Path("sequenceFile");     //定义路径
        Random rand = new Random();                 //定义辅助对象
        SequenceFile.Writer writer = SequenceFile.createWriter(fs, conf, path, IntWritable.class, Text.class);
        //创建写入格式
        for (int i=0; i<100; i++) {
            writer.append(new IntWritable(rand.nextInt(100)), new Text(meg));//写操作
        }
        IOUtils.closeStream(writer); //关闭写入流
    }
}
```

TODO