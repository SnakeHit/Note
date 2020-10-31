# File缓冲流

## PrintWriter

> PrintWriter:是具有自动行刷新的缓冲字符输出流，这是一个高级流。所谓的自动行刷新，意思就是说：在构造函数中指定autoFlush的值为true时，则 println()、printf() 或 format() 方法将自动刷新输出缓冲区(自动调用flush()方法)，但是,自动行刷新无疑会增加写出次数而降低写出效率。

```java
//使用指定文件创建不具有自动行刷新的新 PrintWriter
public PrintWriter(File file);

//创建具有指定文件和字符集且不带自动刷行新的新 PrintWriter
public PrintWriter(File file,String csn);

//根据现有的 OutputStream 创建不带自动行刷新的新 PrintWriter
public PrintWriter(OutputStream out);

//通过现有的 OutputStream 创建新的 PrintWriter(具有自动行刷新)
public PrintWriter(OutputStream out,boolean autoFlush);

//创建具有指定文件名称且不带自动行刷新的新 PrintWriter
public PrintWriter(String fileName);

//创建具有指定文件名称和字符集且不带自动行刷新的新 PrintWriter
public PrintWriter(String fileName,String csn);

//创建新 PrintWriter(具有自动行刷新)
public PrintWriter(Writer out,boolean autoFlush);

```

## BufferWriter

> BufferedWriter:字符缓冲输出流(高级流),将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。 可以指定缓冲区的大小，或者接受默认的大小。在大多数情况下，默认值就足够大了。

