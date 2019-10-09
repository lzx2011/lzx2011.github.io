---
title: 使用 Grapicmagick 和 Im4java 处理图片
date: 2016-09-18 21:12:32
categories: Java
tags: Java
---
# 使用 Grapicmagick 和 Im4java 处理图片

ImageMagick是个图片处理工具可以安装在绝大多数的平台上使用，Linux、Mac、Windows都没有问题。GraphicsMagick是在ImageMagick基础上的另一个项目，大大提高了图片处理的性能，在linux平台上，可以使用命令行的形式处理图片。Im4java 和Jmagick 都是开源社区为上面两个工具开发的 Java API，性能和方便度上im4java是更好的选择。

# JMagick vs Im4Java

1. JMagick是一个开源API，利用JNI(Java Native Interface)技术实现了对ImageMagick API的Java访问接口,因此也将比纯Java实现的图片操作函数在速度上要快。JMagick只实现了ImageMagicAPI的一部分功能，它的发行遵循LGPL协议。

2. im4java是ImageMagick的另一个Java开源接口。与JMagick不同之处在于im4java只是生成与ImageMagick相对应的命令行，然后将生成的命令行传至选中的ImageCommand（使用java.lang.ProcessBuilder.start()实现）来执行相应的操作。它支持大部分ImageMagick命令，可以针对不同组的图片多次复用同一个命令行。

im4java只是封装ImageMagick的命令。所以不需要依赖dll，也不存在64位系统调用32位dll的问题.而且im4java支持GraphicsMagick，GraphicsMagick是ImageMagick的分支。相对ImageMagick ,GraphicsMagick更稳定，消耗资源更少。最重要的是不依赖dll环境
所以使用 im4java 是更好的选择。

# Im4java处理图片示例

这篇文章主要是 im4java 的使用，而 im4java 又是对 GraphicsMagick 命令的封装，GraphicsMagick 命令的使用可以看另一篇文章 <a href="http://blog.csdn.net/revitalizing/article/details/52281828" target="_blank">GraphicsMagick 1.3.23 常用命令</a>

先写个包含获取图片信息的简单工具类，如下。

```java
public class Im4JavaUtils {

    private static Logger logger = LoggerFactory.getLogger(Im4JavaUtils.class);

    // 图片质量
    public static final String IMAGE_QUALITY = "quality";
    // 图片高度
    public static final String IMAGE_HEIGHT = "height";
    // 图片宽度
    public static final String IMAGE_WIDTH = "width";
    // 图片格式
    public static final String IMAGE_SUFFIX = "suffix";
    // 图片大小
    public static final String IMAGE_SIZE = "size";
    // 图片路径
    public static final String IMAGE_PATH = "path";

    /**
     * 是否使用 GraphicsMagick
     */
    private static final boolean IS_USE_GRAPHICS_MAGICK = true;

    /**
     * ImageMagick安装路径，windows下使用
     */
    private static final String IMAGE_MAGICK_PATH = "D:\\software\\ImageMagick-6.2.7-Q8";

    /**
     *  gm 命令所在目录
     */
    private static final String GRAPHICS_MAGICK_PATH = "/usr/local/bin";

    /**
     * 水印图片路径
     */
    private static final String watermarkImagePath = "/Users/gary/Documents/Job/ImageProcessTool/Im4java/Linux_logo.jpg";
    /**
     * 水印图片
     */
    private static Image watermarkImage = null;
    static {
        try {
            watermarkImage = ImageIO.read(new File(watermarkImagePath));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 命令类型
     *
     */
    private enum CommandType {
        convert("转换处理"), identify("图片信息"), compositecmd("图片合成");
        private String name;

        CommandType(String name) {
            this.name = name;
        }
    }

    /**
     * 查询图片的基本信息:格式,质量，宽度，高度
     * <p>
     * gm identify -format %w,%h,%d/%f,%Q,%b,%e /Users/gary/Documents/999999999/10005582/1.jpg
     * <p>
     *
     * @param imagePath
     * @return
     */
    public static Map<String, String> getImageInfo(String imagePath) {
        long startTime = System.currentTimeMillis();
        Map<String, String> imageInfo = new HashMap<>();
        try {
            IMOperation op = new IMOperation();
            op.format("%w,%h,%d/%f,%Q,%b,%e");
            op.addImage();
            ImageCommand identifyCmd = getImageCommand(CommandType.identify);
            ArrayListOutputConsumer output = new ArrayListOutputConsumer();
            identifyCmd.setOutputConsumer(output);
            identifyCmd.run(op, imagePath);
            ArrayList<String> cmdOutput = output.getOutput();
            String[] result = cmdOutput.get(0).split(",");
            if (result.length == 6) {
                imageInfo.put(IMAGE_WIDTH, result[0]);
                imageInfo.put(IMAGE_HEIGHT, result[1]);
                imageInfo.put(IMAGE_PATH, result[2]);
                imageInfo.put(IMAGE_QUALITY, result[3]);
                imageInfo.put(IMAGE_SIZE, result[4]);
                imageInfo.put(IMAGE_SUFFIX, result[5]);
            }
        } catch (Exception e) {
            // e.printStackTrace();
            logger.error("图片工具获取图片基本信息异常" + e.getMessage(), e);
        }
        long endTime = System.currentTimeMillis();
        // logger.info("take time: " + (endTime - startTime));
        return imageInfo;
    }

    /**
    * 获取 ImageCommand
    *
    * @param command 命令类型
    * @return
    */
   private static ImageCommand getImageCommand(CommandType command) {
       ImageCommand cmd = null;
       switch (command) {
           case convert:
               cmd = new ConvertCmd(IS_USE_GRAPHICS_MAGICK);
               break;
           case identify:
               cmd = new IdentifyCmd(IS_USE_GRAPHICS_MAGICK);
               break;
           case compositecmd:
               cmd = new CompositeCmd(IS_USE_GRAPHICS_MAGICK);
               break;
       }
       if (cmd != null && System.getProperty("os.name").toLowerCase().indexOf("windows") == -1) {
           cmd.setSearchPath(IS_USE_GRAPHICS_MAGICK ? GRAPHICS_MAGICK_PATH : IMAGE_MAGICK_PATH);
       }
       return cmd;
   }
  }

```

上面的代码中要注意几个问题：

1. 获取 ImageCommand时，在new ConvertCmd(true)，参数要填true，不填的默认使用的是ImageMagick，参数为true时，才会使用grapicmagick的命令。
2. 获取 ImageCommand 后，要设置命令的搜索路径，有时可能会不稳定找不到gm命令，例如 cmd.setSearchPath("/usr/local/bin"); (gm 命令在该路径下)

获取图片信息的命令中有个参数 -format ，该参数可取的值如下。

```java


 %b   file size of image read in
 %c   comment property
 %d   directory component of path
 %e   filename extension or suffix
 %f   filename (including suffix)
 %g   layer canvas page geometry   ( = %Wx%H%X%Y )
 %h   current image height in pixels
 %i   image filename (note: becomes output filename for "info:")
 %k   number of unique colors
 %l   label property
 %m   image file format (file magic)
 %n   exact number of images in current image sequence
 %o   output filename  (used for delegates)
 %p   index of image in current image list
 %q   quantum depth (compile-time constant)
 %r   image class and colorspace
 %s   scene number (from input unless re-assigned)
 %t   filename without directory or extension (suffix)
 %u   unique temporary filename (used for delegates)
 %w   current width in pixels
 %x   x resolution (density)
 %y   y resolution (density)
 %z   image depth (as read in unless modified, image save depth)
 %A   image transparency channel enabled (true/false)
 %C   image compression type
 %D   image dispose method
 %G   image size ( = %wx%h )
 %H   page (canvas) height
 %M   Magick filename (original file exactly as given,  including read mods)
 %O   page (canvas) offset ( = %X%Y )
 %P   page (canvas) size ( = %Wx%H )
 %Q   image compression quality ( 0 = default )
 %S   ?? scenes ??
 %T   image time delay
 %W   page (canvas) width
 %X   page (canvas) x offset (including sign)
 %Y   page (canvas) y offset (including sign)
 %Z   unique filename (used for delegates)
 %@   bounding box
 %#   signature
 %%   a percent sign
 \n   newline
 \r   carriage return


```

# 压缩图片质量

```java
  /**
     * 图片压缩
     * <p>
     * 拼装命令示例: gm convert -quality 80 /apps/watch.jpg /apps/watch_compress.jpg
     *
     * @param srcImagePath
     * @param destImagePath
     * @param quality
     * @throws Exception
     */
    public static void compressImage(String srcImagePath, String destImagePath, double quality) throws Exception {
        IMOperation op = new IMOperation();
        op.quality(quality);
        op.addImage();
        op.addImage();
        ImageCommand cmd = getImageCommand(CommandType.convert);
        cmd.run(op, srcImagePath, destImagePath);
    }

```

# 裁剪图片

```java
 /**
     * 裁剪图片
     *
     * @param imagePath 源图片路径
     * @param newPath   处理后图片路径
     * @param x         起始X坐标
     * @param y         起始Y坐标
     * @param width     裁剪宽度
     * @param height    裁剪高度
     * @return 返回true说明裁剪成功, 否则失败
     */
    public static boolean cutImage(String imagePath, String newPath, int x, int y, int width, int height) {
        boolean flag = false;
        try {
            IMOperation op = new IMOperation();
            op.addImage(imagePath);
            /** width：裁剪的宽度 * height：裁剪的高度 * x：裁剪的横坐标 * y：裁剪纵坐标 */
            op.crop(width, height, x, y);
            op.addImage(newPath);
            ConvertCmd convert = new ConvertCmd(true);
            convert.run(op);
            flag = true;
        } catch (IOException e) {
            System.out.println("文件读取错误!");
            flag = false;
        } catch (InterruptedException e) {
            flag = false;
        } catch (IM4JavaException e) {
            flag = false;
        } finally {

        }
        return flag;
    }

```

# 缩放图片

```java
/**
     * 根据尺寸缩放图片[等比例缩放:参数height为null,按宽度缩放比例缩放;参数width为null,按高度缩放比例缩放]
     *
     * @param imagePath 源图片路径
     * @param newPath   处理后图片路径
     * @param width     缩放后的图片宽度
     * @param height    缩放后的图片高度
     * @return 返回true说明缩放成功, 否则失败
     */
    public static boolean zoomImage(String imagePath, String newPath, Integer width, Integer height) {

        boolean flag;
        try {
            IMOperation op = new IMOperation();
            op.addImage(imagePath);
            if (width == null) {// 根据高度缩放图片
                op.resize(null, height);
            } else if (height == null) {// 根据宽度缩放图片
                op.resize(width);
            } else {
                op.resize(width, height);
            }
            op.addImage(newPath);
            ConvertCmd convert = new ConvertCmd(true);
            convert.run(op);
            flag = true;
        } catch (IOException e) {
            System.out.println("文件读取错误!");
            flag = false;
        } catch (InterruptedException e) {
            flag = false;
        } catch (IM4JavaException e) {
            flag = false;
        }
        return flag;
    }

```

# 图片旋转

```java

  /**
     * 图片旋转(顺时针旋转) 
     * 拼装命令示例: gm convert -rotate 90 /apps/watch.jpg /apps/watch_compress.jpg
     *
     * @param imagePath 源图片路径
     * @param newPath   处理后图片路径
     * @param degree    旋转角度
     */
    public static boolean rotate(String imagePath, String newPath, double degree) {
        boolean flag;
        try {
            // 1.将角度转换到0-360度之间
            degree = degree % 360;
            if (degree <= 0) {
                degree = 360 + degree;
            }
            IMOperation op = new IMOperation();
            op.rotate(degree);
            op.addImage(imagePath);
            op.addImage(newPath);
            ConvertCmd cmd = new ConvertCmd(true);
            cmd.run(op);
            flag = true;
        } catch (Exception e) {
            flag = false;
            System.out.println("图片旋转失败!");
        }
        return flag;
    }

```

# 添加文字水印

```java
 /**
     * 文字水印
     *
     * @param srcImagePath  源图片路径
     * @param destImagePath 目标图片路径
     * @param content       文字内容（不支持汉字）
     * @throws Exception
     */
    public static void addTextWatermark(String srcImagePath, String destImagePath, String content) throws Exception {
        IMOperation op = new IMOperation();

        op.font("ArialBold");
        // 文字方位-东南
        op.gravity("southeast");
        // 文字信息
        op.pointsize(60).fill("#F2F2F2").draw("text 10,10 " + content);
        
        // 原图
        op.addImage(srcImagePath);
        // 目标
        op.addImage(destImagePath);
        ImageCommand cmd = getImageCommand(CommandType.convert);
        cmd.run(op);
    }

```
添加文字前要安装文字包，或直接指向 ttf 文字格式文件上。
OSX 安装

```java

brew install ghostscript

```

>添加中文水印会乱码，待解决。

# 添加图片水印

```java
 /**
     * 图片水印
     *
     * @param srcImagePath  源图片路径
     * @param destImagePath 目标图片路径
     * @param dissolve      透明度（0-100）
     * @throws Exception
     */
    public static void addImgWatermark(String srcImagePath, String destImagePath, Integer dissolve) throws Exception {
        // 原始图片信息
        BufferedImage buffimg = ImageIO.read(new File(srcImagePath));
        int w = buffimg.getWidth();
        int h = buffimg.getHeight();
        IMOperation op = new IMOperation();
        // 水印图片位置
        op.geometry(watermarkImage.getWidth(null), watermarkImage.getHeight(null),
                w - watermarkImage.getWidth(null) - 10, h - watermarkImage.getHeight(null) - 10);
        // 水印透明度
        op.dissolve(dissolve);
        // 水印
        op.addImage(watermarkImagePath);
        // 原图
        op.addImage(srcImagePath);
        // 目标
        op.addImage(destImagePath);
        ImageCommand cmd = getImageCommand(CommandType.compositecmd);
        cmd.run(op);
    }

```

# 参考资料
<a href="http://im4java.sourceforge.net/" target="_blank">im4java官网</a>
<a href="https://github.com/hailin0/im4java-util/blob/master/src/main/java/com/hlin/im4java/util/ImageUtil.java" target="_blank">hailin0/im4java-util</a>

