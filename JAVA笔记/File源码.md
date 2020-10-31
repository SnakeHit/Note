# File源码

下面是File源码的成员变量（类变量）

```java
public class File
    implements Serializable, Comparable<File>
{

    /**
     * The FileSystem object representing the platform's local file system.
     * 这句话的意思就是FileSystem代表平台本地的文件系统，如果你用的操作系统是windows那么就是WinNTFileSystem
     * 如果你用的操作系统是Unix，那么就是UnixNTFileSystem
     */
    private static final FileSystem fs = DefaultFileSystem.getFileSystem();
	//产生一个FileSystem对象
    private final String path;

    private static enum PathStatus { INVALID, CHECKED };

    private transient PathStatus status = null;

    final boolean isInvalid() {
        if (status == null) {
            status = (this.path.indexOf('\u0000') < 0) ? PathStatus.CHECKED
                                                       : PathStatus.INVALID;
        }
        return status == PathStatus.INVALID;
    }

    private final transient int prefixLength;

    int getPrefixLength() {
        return prefixLength;
    }
    //这个代表Path是那种类型
	/**	0: Completely relative 完全相对路径
	 *	1: Driver-relative	相对于驱动磁盘的路径（C：）
	 *	2: UNC
	 *	3: Absolute local	绝对路径
	 */
    public static final char separatorChar = fs.getSeparator();
	//这是目录间名称分隔符，'\\' OR '/'
    public static final String separator = "" + separatorChar;
	//转换成字符串类型，方便以后的操作
    public static final char pathSeparatorChar = fs.getPathSeparator();
	//这个是路径间的分割符 ':' OR ';'
    public static final String pathSeparator = "" + pathSeparatorChar;
}
```

## 类变量**FileSystem fs**

` private static final FileSystem fs = DefaultFileSystem.getFileSystem();`

该类变量是`private`和`final`的，它利用`DefaultFileSystem`中的`getFileSystem`方法。

### `DefaultFileSystem`的源码

```java
class DefaultFileSystem {

    /**
     * Return the FileSystem object for Windows platform.
     */
    public static FileSystem getFileSystem() {
        return new WinNTFileSystem();
    }
}
```

由源码可知，`getFileSystem`返回一个`WinNTFileSystem`的对象，下面我们追根溯源找到`WinNTFileSystem`的源码。

### `WinNTFileSystem`的源码

```java
class WinNTFileSystem extends FileSystem {

    private final char slash;
    private final char altSlash;
    private final char semicolon;

    public WinNTFileSystem() {
        slash = AccessController.doPrivileged(
            new GetPropertyAction("file.separator")).charAt(0);
        semicolon = AccessController.doPrivileged(
            new GetPropertyAction("path.separator")).charAt(0);
        altSlash = (this.slash == '\\') ? '/' : '\\';
    }

    private boolean isSlash(char c) {
        return (c == '\\') || (c == '/');
    }

    private boolean isLetter(char c) {
        return ((c >= 'a') && (c <= 'z')) || ((c >= 'A') && (c <= 'Z'));
    }

    private String slashify(String p) {
        if ((p.length() > 0) && (p.charAt(0) != slash)) return slash + p;
        else return p;
    }
}
```

首先，我们可以看到`WinNTFileSystem`类是`FileSystem`类的子类，暂时可以得出其功能是判定文件的分隔符是什么，`Unix`是'/'，`Windows`是'\\\\'。

而`FileSystem`类是一个抽象类，其中包括了一些抽象方法来对文件路径进行一系列操作,==WinNTFileSystem就是其实现类==。

### **FileSystem**类的源码

```java
abstract class FileSystem {

    /* -- Normalization and construction -- */

    /**
     * Return the local filesystem's name-separator character.
     * 返回名称之间的分隔符
     */
    public abstract char getSeparator();

    /**
     * Return the local filesystem's path-separator character.
     * 返回路径之间的分隔符
     */
    public abstract char getPathSeparator();

    /**
     * Convert the given pathname string to normal form.  If the string is
     * already in normal form then it is simply returned.
     * 这个方法的作用是规格化路径名称，使之变得统一起来，如果已经是规格化的名称，则返回。
     */
    public abstract String normalize(String path);

    /**
     * Compute the length of this pathname string's prefix.  The pathname
     * string must be in normal form.
     * 计算路径的前缀长度（根据路径的类型不同，分为4种）
     */
    public abstract int prefixLength(String path);

    /**
     * Resolve the child pathname string against the parent.
     * Both strings must be in normal form, and the result
     * will be in normal form.
     * 将父类路径与子类路径联系起来
     */
    public abstract String resolve(String parent, String child);

    /**
     * Return the parent pathname string to be used when the parent-directory
     * argument in one of the two-argument File constructors is the empty
     * pathname.
     */
    public abstract String getDefaultParent();

    /**
     * Post-process the given URI path string if necessary.  This is used on
     * win32, e.g., to transform "/c:/foo" into "c:/foo".  The path string
     * still has slash separators; code in the File class will translate them
     * after this method returns.
     */
    public abstract String fromURIPath(String path);


    /* -- Path operations -- */

    /**
     * Tell whether or not the given abstract pathname is absolute.
     */
    public abstract boolean isAbsolute(File f);

    /**
     * Resolve the given abstract pathname into absolute form.  Invoked by the
     * getAbsolutePath and getCanonicalPath methods in the File class.
     */
    public abstract String resolve(File f);

    public abstract String canonicalize(String path) throws IOException;


```

## 应用实例

```java
package file;
 
import java.io.File;
 
public class TestFile {
 
    public static void main(String[] args) {
        // 绝对路径
        File f1 = new File("d:/LOLFolder");
        System.out.println("f1的绝对路径：" + f1.getAbsolutePath());
        
        // 相对路径,相对于工作目录，如果在eclipse中，就是项目目录
        File f2 = new File("LOL.exe");
        System.out.println("f2的绝对路径：" + f2.getAbsolutePath());
 
        // 把f1作为父目录创建文件对象
        File f3 = new File(f1, "LOL.exe");
 
        System.out.println("f3的绝对路径：" + f3.getAbsolutePath());
    }
}
```

根据`File`类的三个方法，我们观察其源码的实现及其功能。

首先是`File(URL)`，源码如下：

```java
 public File(URI uri) {

        // Check our many preconditions
        if (!uri.isAbsolute())
            throw new IllegalArgumentException("URI is not absolute");
        if (uri.isOpaque())
            throw new IllegalArgumentException("URI is not hierarchical");
        String scheme = uri.getScheme();
        if ((scheme == null) || !scheme.equalsIgnoreCase("file"))
            throw new IllegalArgumentException("URI scheme is not \"file\"");
        if (uri.getAuthority() != null)
            throw new IllegalArgumentException("URI has an authority component");
        if (uri.getFragment() != null)
            throw new IllegalArgumentException("URI has a fragment component");
        if (uri.getQuery() != null)
            throw new IllegalArgumentException("URI has a query component");
        String p = uri.getPath();
        if (p.equals(""))
            throw new IllegalArgumentException("URI path component is empty");

        // Okay, now initialize
        p = fs.fromURIPath(p);
        if (File.separatorChar != '/')
            p = p.replace('/', File.separatorChar);
        this.path = fs.normalize(p);
        this.prefixLength = fs.prefixLength(this.path);
    }
```

然后是`File(String)`的源码

```java
public File(String pathname) {
        if (pathname == null) {
            throw new NullPointerException();
        }
        this.path = fs.normalize(pathname);
        this.prefixLength = fs.prefixLength(this.path);
    }

	//下面是normalize(String path)的实现
	/* Check that the given pathname is normal.  If not, invoke the real
       normalizer on the part of the pathname that requires normalization.
       This way we iterate through the whole pathname string only once. */
    @Override
public String normalize(String path) {
        int n = path.length();
        char slash = this.slash;
        char altSlash = this.altSlash;
        char prev = 0;
        for (int i = 0; i < n; i++) {
            char c = path.charAt(i);
            if (c == altSlash)
                return normalize(path, n, (prev == slash) ? i - 1 : i);
            if ((c == slash) && (prev == slash) && (i > 1))
                return normalize(path, n, i - 1);
            if ((c == ':') && (i > 1))
                return normalize(path, n, 0);
            prev = c;
        }
        if (prev == slash) return normalize(path, n, n - 1);
        return path;
    }
//下面是prefixLength(String path)的实现
@Override
public int prefixLength(String path) {
        char slash = this.slash;
        int n = path.length();
        if (n == 0) return 0;
        char c0 = path.charAt(0);
        char c1 = (n > 1) ? path.charAt(1) : 0;
        if (c0 == slash) {
            if (c1 == slash) return 2;  /* Absolute UNC pathname "\\\\foo" */
            return 1;                   /* Drive-relative "\\foo" */
        }
        if (isLetter(c0) && (c1 == ':')) {
            if ((n > 2) && (path.charAt(2) == slash))
                return 3;               /* Absolute local pathname "z:\\foo" */
            return 2;                   /* Directory-relative "z:foo" */
        }
        return 0;                       /* Completely relative */
    }

```

最后是`File(file,String)`的源码

```java
public File(File parent, String child) {
        if (child == null) {
            throw new NullPointerException();
        }
        if (parent != null) {
            if (parent.path.equals("")) {
                this.path = fs.resolve(fs.getDefaultParent(),
                                       fs.normalize(child));
            } else {
                this.path = fs.resolve(parent.path,
                                       fs.normalize(child));
            }
        } else {
            this.path = fs.normalize(child);
        }
        this.prefixLength = fs.prefixLength(this.path);
    }
```









