# File 实例

## 拆分

> 找到一个大于100k的文件，按照100k为单位，拆分成多个子文件，并且以编号作为文件名结束。
> 比如文件 eclipse.exe，大小是309k。
>
> 拆分之后，成为四个文件！

```java
package stream;
  
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Arrays;
  
public class TestStream {
  
    public static void main(String[] args) {
        int eachSize = 100 * 1024; // 100k
        File srcFile = new File("d:/eclipse.exe");
//      splitFile(srcFile, eachSize);
    }
  
    /**
     * 拆分的思路，先把源文件的所有内容读取到内存中，然后从内存中挨个分到子文件里
     *
     * @param srcFile
     *            要拆分的源文件
     * @param eachSize
     *            按照这个大小，拆分
     */
    private static void splitFile(File srcFile, int eachSize) {
  
        if (0 == srcFile.length())
            throw new RuntimeException("文件长度为0，不可拆分");
  
        byte[] fileContent = new byte[(int) srcFile.length()];
        // 为了在finally中关闭，需要声明在try外面
        FileInputStream fis = null;
        try {
            fis = new FileInputStream(srcFile);
            fis.read(fileContent);
  
        } catch (IOException e) {
  
            e.printStackTrace();
        } finally {
            // 在finally中关闭
            try {
                if(null!=fis)
                    fis.close();
            } catch (IOException e) {
  
                e.printStackTrace();
            }
        }
  
        int fileNumber;
        if (0 == fileContent.length % eachSize)
            fileNumber = (int) (fileContent.length / eachSize);
        else
            fileNumber = (int) (fileContent.length / eachSize) + 1;
  
        for (int i = 0; i < fileNumber; i++) {
            String eachFileName = srcFile.getName() + "-" + i;
            File eachFile = new File(srcFile.getParent(), eachFileName);
            byte[] eachContent;
  
            if (i != fileNumber - 1)
                eachContent = Arrays.copyOfRange(fileContent, eachSize * i, eachSize * (i + 1));
            else
                eachContent = Arrays.copyOfRange(fileContent, eachSize * i, fileContent.length);
  
            // 为了在finally中关闭，声明在try外面
            FileOutputStream fos = null;
            try {
                fos = new FileOutputStream(eachFile);
                fos.write(eachContent);
  
                System.out.printf("输出子文件%s，其大小是%,d字节%n", eachFile.getAbsoluteFile(), eachFile.length());
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                // finally中关闭
                try {
                    if(null!=fos)
                        fos.close();
                } catch (IOException e) {
  
                    e.printStackTrace();
                }
            }
        }
  
    }

```

## 合并

> 把上述拆分出来的文件，合并成一个原文件。
>
> 以是否能正常运行，验证合并是否正确

```java
  
    /**
     * 合并的思路，就是从eclipse.exe-0开始，读取到一个文件，就开始写出到 eclipse.exe中，直到没有文件可以读
     *
     * @param folder
     *            需要合并的文件所处于的目录
     * @param fileName
     *            需要合并的文件的名称
     * @throws FileNotFoundException
     */
    private static void murgeFile(String folder, String fileName) {
  
        File destFile = new File(folder, fileName);
        // 使用try-with-resource的方式自动关闭流
        try (FileOutputStream fos = new FileOutputStream(destFile);) {
            int index = 0;
            while (true) {
                File eachFile = new File(folder, fileName + "-" + index++);
                if (!eachFile.exists())
                    break;
  
                // 使用try-with-resource的方式自动关闭流
                try (FileInputStream fis = new FileInputStream(eachFile);) {
                    byte[] eachContent = new byte[(int) eachFile.length()];
                    fis.read(eachContent);
                    fos.write(eachContent);
                    fos.flush();
                }
                System.out.printf("把子文件 %s写出到目标文件中%n", eachFile);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
  
        System.out.printf("最后目标文件的大小：%,d字节", destFile.length());
  
    }
  
}
```



