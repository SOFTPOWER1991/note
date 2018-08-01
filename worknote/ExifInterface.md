
## 一、 碰到的问题

写这篇文章的动机源自于这波迭代中碰到的一个问题：
> 在IM拍照时，在三星s7 eadge上拍完照片后从sd上拿到的地址设置给Imageview后显示时，图片旋转了90度。But我拍照的时候明明是竖着拍的，相册预览也是竖着的，为什么拿到图片后就成了横着的？
> 对比了另一台手机锤子坚果U1，没这个问题，因此怀疑是跟相机的机型相关。

想到的解决方案：
> 把读取到的图片作为一个bitmap放在一个画布上，然后旋转画布来控制图片展示的方向。

问题来了：
> 这样确实解决了三星上的问题，但是原来没问题的机型上——歪了。
> 
> 至此，问题明确了：**我如何拿到的图片的实际方向？**


## 二、Exif

借助强大的Google，搜到了一个叫做`Exif`的东西，它是什么呢？

维基百科如是说：
> EXIF：可交换图像文件格式（英语：Exchangeable image file format，官方简称Exif）， 是专门为数码相机的照片设定的，可以记录数码照片的属性信息和拍摄数据。
> 
> 包括：**分辨率，旋转方向，感光度、白平衡、拍摄的光圈、焦距、分辨率、相机品牌、型号、GPS等信息。**
> 
> Exif可以附加于JPEG、TIFF、RIFF等文件之中，为其增加有关数码相机拍摄信息的内容和索引图或图像处理软件的版本信息。

下图是维基百科提供的一个exif图片：

![](http://7xkl0t.com1.z0.glb.clouddn.com/18-5-7/1504753.jpg)

问题来了：
> 知道了Exif这个信息，对我有什么用？
> 
> > 可以看到在exif中有一个叫做图像方向的东西，那是否可以借助这个属性来解决我的问题呢？

## 三、ExifInterface 源码解析

求助强大的Google 爸爸：
> exif  android

爸爸给我呈现了如下结果：

1. [ExifInterface 官方文档](https://developer.android.com/reference/android/media/ExifInterface)
2. [ExifInterface 支持库简介](http://developers.googleblog.cn/2017/01/exifinterface.html)

有了这两个文档我的问题迎刃而解。

看看ExifInterface是毛：
> 乍一看这是个接口挺迷的，点进文档一看是个class。
> 
> ExifInterface是Android为我们提供的一个支持库，随着 25.1.0 支持库的发布，支持库大家庭迎来了一名新成员：ExifInterface 支持库。由于 Android 7.1 引入了对框架 ExifInterface 的重大改进，最低可以支持到API 9+。
> 
> 在build.gradle文件中引入下面的代码，便可以使用ExifInterface了：
> 
> `implementation 'com.android.support:exifinterface:27.1.1'`

如何使用ExifInterface解决我的问题，定位到它的源码，可以看到它为我们提供了3个构造方法：

```
	/**
     * 从给定的图片路径中读取图片的exif tag信息.
     */
    public ExifInterface(String filename) throws IOException {
        ......
        try {
            ......
            loadAttributes(in);
        } finally {
            IoUtils.closeQuietly(in);
        }
    }

    /**
     * 从指定的图像文件描述符中读取Exif标签. 属性突变仅支持可写和可搜索的文件描述符. 此构造函数不会倒回给定文件描述符的偏移量。开发人员在使用后应关闭文件描述符。
     */
    public ExifInterface(FileDescriptor fileDescriptor) throws IOException {
        ......
        try {
            in = new FileInputStream(fileDescriptor);
            loadAttributes(in);
        } finally {
            IoUtils.closeQuietly(in);
        }
    }

    /**
     * 从给定的输入流中读取图片的exif 信息. 对文件输入流的属性图片不支持. 开发者在使用完之后应该关闭输入流.
     */
    public ExifInterface(InputStream inputStream) throws IOException {
        ......
        loadAttributes(inputStream);
    }
```

可以看到的是在这三个构造方法里无一例外的都调用了loadAttributes（inputstream）方法。

接下来跟踪到loadAttributes方法中：

```
 /**
     * This function decides which parser to read the image data according to the given input stream
     * type and the content of the input stream. In each case, it reads the first three bytes to
     * determine whether the image data format is JPEG or not.
     */
    private void loadAttributes(@NonNull InputStream in) throws IOException {
        try {
            // Initialize mAttributes.
            for (int i = 0; i < EXIF_TAGS.length; ++i) {
                mAttributes[i] = new HashMap();
            }

            // Process RAW input stream
            if (mAssetInputStream != null) {
                long asset = mAssetInputStream.getNativeAsset();
                if (handleRawResult(nativeGetRawAttributesFromAsset(asset))) {
                    return;
                }
            } else if (mSeekableFileDescriptor != null) {
                if (handleRawResult(nativeGetRawAttributesFromFileDescriptor(
                        mSeekableFileDescriptor))) {
                    return;
                }
            } else {
                in = new BufferedInputStream(in, JPEG_SIGNATURE_SIZE);
                if (!isJpegInputStream((BufferedInputStream) in) && handleRawResult(
                        nativeGetRawAttributesFromInputStream(in))) {
                    return;
                }
            }

            // Process JPEG input stream
            getJpegAttributes(in);
            mIsSupportedFile = true;
        } catch (IOException e) {
            // Ignore exceptions in order to keep the compatibility with the old versions of
            // ExifInterface.
            mIsSupportedFile = false;
            Log.w(TAG, "Invalid image: ExifInterface got an unsupported image format file"
                    + "(ExifInterface supports JPEG and some RAW image formats only) "
                    + "or a corrupted JPEG file to ExifInterface.", e);
        } finally {
            addDefaultValuesForCompatibility();

            if (DEBUG) {
                printAttributes();
            }
        }
    }
```
从注释得到如下信息：


1. 这个方法根据输入的数据流类型和数据流内容来决定使用哪种类型的解析器来解析这个流数据。不论在哪一种类型的中，它都会读取前3个字节的数据来决定这是否是JPEG格式的图片。
 
   换而言之——**只有JPEG格式的图片才会携带exif数据，像PNG，WebP这类的图片就不会有这些数据。**

2. 如果是JPEG类型的数据会将mIsSupportedFile 设置为true，并且调用getJpegAttributes（in）方法类获取JPEG中属性信息
3. 在try-catch的finally方法中调用了addDefaultValuesForCompatibility()方法，这个方法会为每个JPEG格式的图片添加默认的属性。

瞜一眼addDefaultValuesForCompatibility的代码：

```
 private void addDefaultValuesForCompatibility() {
        // The value of DATETIME tag has the same value of DATETIME_ORIGINAL tag.
        String valueOfDateTimeOriginal = getAttribute(TAG_DATETIME_ORIGINAL);
        if (valueOfDateTimeOriginal != null) {
            mAttributes[IFD_TIFF_HINT].put(TAG_DATETIME,
                    ExifAttribute.createString(valueOfDateTimeOriginal));
        }

        // Add the default value.
        if (getAttribute(TAG_IMAGE_WIDTH) == null) {
            mAttributes[IFD_TIFF_HINT].put(TAG_IMAGE_WIDTH,
                    ExifAttribute.createULong(0, mExifByteOrder));
        }
        if (getAttribute(TAG_IMAGE_LENGTH) == null) {
            mAttributes[IFD_TIFF_HINT].put(TAG_IMAGE_LENGTH,
                    ExifAttribute.createULong(0, mExifByteOrder));
        }
        if (getAttribute(TAG_ORIENTATION) == null) {
            mAttributes[IFD_TIFF_HINT].put(TAG_ORIENTATION,
                    ExifAttribute.createULong(0, mExifByteOrder));
        }
        if (getAttribute(TAG_LIGHT_SOURCE) == null) {
            mAttributes[IFD_EXIF_HINT].put(TAG_LIGHT_SOURCE,
                    ExifAttribute.createULong(0, mExifByteOrder));
        }
    }
```

这段代码可以得到如下信息：

> 对于每一张JPEG图片都会添加默认的属性信息，包含：

> * 图片的宽、高：TAG_IMAGE_WIDTH、TAG_IMAGE_LENGTH
> * 图片的方向：TAG_ORIENTATION ，它的值大致有如下几个：

> >  1. ORIENTATION_FLIP_HORIZONTAL
> >  1. ORIENTATION_FLIP_VERTICAL

> >  1. ORIENTATION_NORMAL

> > 1.  ORIENTATION_ROTATE_180

> > 1. ORIENTATION_ROTATE_270

> > 1.  ORIENTATION_ROTATE_90

> > 1. ORIENTATION_TRANSPOSE

> > 1.  ORIENTATION_TRANSVERSE
> > 1.  ORIENTATION_UNDEFINED

> * 图片光源：TAG_LIGHT_SOURCE（我猜的，不一定对）


## 四、解决我的问题

**源码读到这里，已经了然了：我只要拿到当前图片的orientation，如果有旋转那么给它转一下，就可以了。**

接下来的问题：
> 如何拿到图片的方向？
> 从文档里看到，ExifInterface为我们提供了如下方法:
> 
> 1. getAttribute(String tag)
> 2. getAttributeDouble(String tag, double defaultValue)
> 3. getAttributeInt(String tag, int defaultValue)

下面给出这个问题的解决方案，步骤如下：

> 1. 根据选中的图片路径获取ExifInterface;
> 2. 从 ExifInterface中获取到当前图片的旋转方向；
> 3. 把对应路径的图片Bitmap映射到一个画布上
> 4. 通过Matrix旋转画布，解决方向的问题。

```
Matrix mat = new Matrix();
Bitmap bitmap = BitmapFactory.decodeFile(path, options);
ExifInterface ei = new ExifInterface(path);
int orientation = ei.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL);
switch (orientation) {
    case ExifInterface.ORIENTATION_ROTATE_90:
        mat.postRotate(90);
        break;
    case ExifInterface.ORIENTATION_ROTATE_180:
        mat.postRotate(180);
        break;
}
return Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), mat, true);
```

至此我的问题就解决了。

## 五、看我家萌汪的Exif信息：

先上图 ：

![](http://7xkl0t.com1.z0.glb.clouddn.com/18-5-7/75480058.jpg)

你可能会很好奇，为毛这个图片是转了个呢？没错，这就是用三星手机拍的照片。

来读取下它的信息：

```
ExifInterface exifInterface = new ExifInterface(path);

String orientation = exifInterface.getAttribute(ExifInterface.TAG_ORIENTATION);
String dateTime = exifInterface.getAttribute(ExifInterface.TAG_DATETIME);
String make = exifInterface.getAttribute(ExifInterface.TAG_MAKE);
String model = exifInterface.getAttribute(ExifInterface.TAG_MODEL);
String flash = exifInterface.getAttribute(ExifInterface.TAG_FLASH);
String imageLength = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_LENGTH);
String imageWidth = exifInterface.getAttribute(ExifInterface.TAG_IMAGE_WIDTH);
String latitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LATITUDE);
String longitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_LONGITUDE);
String latitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_LATITUDE_REF);
String longitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_LONGITUDE_REF);
String exposureTime = exifInterface.getAttribute(ExifInterface.TAG_EXPOSURE_TIME);
String aperture = exifInterface.getAttribute(ExifInterface.TAG_APERTURE);
String isoSpeedRatings = exifInterface.getAttribute(ExifInterface.TAG_ISO);
String dateTimeDigitized = exifInterface.getAttribute(ExifInterface.TAG_DATETIME_DIGITIZED);
String subSecTime = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME);
String subSecTimeOrig = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME_ORIG);
String subSecTimeDig = exifInterface.getAttribute(ExifInterface.TAG_SUBSEC_TIME_DIG);
String altitude = exifInterface.getAttribute(ExifInterface.TAG_GPS_ALTITUDE);
String altitudeRef = exifInterface.getAttribute(ExifInterface.TAG_GPS_ALTITUDE_REF);
String gpsTimeStamp = exifInterface.getAttribute(ExifInterface.TAG_GPS_TIMESTAMP);
String gpsDateStamp = exifInterface.getAttribute(ExifInterface.TAG_GPS_DATESTAMP);
String whiteBalance = exifInterface.getAttribute(ExifInterface.TAG_WHITE_BALANCE);
String focalLength = exifInterface.getAttribute(ExifInterface.TAG_FOCAL_LENGTH);
String processingMethod = exifInterface.getAttribute(ExifInterface.TAG_GPS_PROCESSING_METHOD);


Log.e("TAG", "## orientation=" + orientation);
Log.e("TAG", "## dateTime=" + dateTime);
Log.e("TAG", "## make=" + make);
Log.e("TAG", "## model=" + model);
Log.e("TAG", "## flash=" + flash);
Log.e("TAG", "## imageLength=" + imageLength);
Log.e("TAG", "## imageWidth=" + imageWidth);
Log.e("TAG", "## latitude=" + latitude);
Log.e("TAG", "## longitude=" + longitude);
Log.e("TAG", "## latitudeRef=" + latitudeRef);
Log.e("TAG", "## longitudeRef=" + longitudeRef);
Log.e("TAG", "## exposureTime=" + exposureTime);
Log.e("TAG", "## aperture=" + aperture);
Log.e("TAG", "## isoSpeedRatings=" + isoSpeedRatings);
Log.e("TAG", "## dateTimeDigitized=" + dateTimeDigitized);
Log.e("TAG", "## subSecTime=" + subSecTime);
Log.e("TAG", "## subSecTimeOrig=" + subSecTimeOrig);
Log.e("TAG", "## subSecTimeDig=" + subSecTimeDig);
Log.e("TAG", "## altitude=" + altitude);
Log.e("TAG", "## altitudeRef=" + altitudeRef);
Log.e("TAG", "## gpsTimeStamp=" + gpsTimeStamp);
Log.e("TAG", "## gpsDateStamp=" + gpsDateStamp);
Log.e("TAG", "## whiteBalance=" + whiteBalance);
Log.e("TAG", "## focalLength=" + focalLength);
Log.e("TAG", "## processingMethod=" + processingMethod);
```

得到的log如下：

```
05-07 18:40:40.813 27181-27181/zhanggeng.www.exifdemo E/TAG: ## orientation=6
    ## dateTime=2018:04:21 14:32:41
    ## make=samsung
    ## model=SM-G9350
    ## flash=0
    ## imageLength=3024
    ## imageWidth=4032
    ## latitude=34/1,0/1,536875/10000
    ## longitude=109/1,0/1,97687/10000
    ## latitudeRef=N
    ## longitudeRef=E
    ## exposureTime=0.002544529262086514
    ## aperture=1.7
    ## isoSpeedRatings=50
    ## dateTimeDigitized=2018:04:21 14:32:41
    ## subSecTime=null
    ## subSecTimeOrig=null
    ## subSecTimeDig=null
    ## altitude=816000/1000
05-07 18:40:40.814 27181-27181/zhanggeng.www.exifdemo E/TAG: ## altitudeRef=0
    ## gpsTimeStamp=06:32:06
    ## gpsDateStamp=2018:04:30
    ## whiteBalance=0
    ## focalLength=420/100
    ## processingMethod=null
```

以上是这张照片的所有Exif信息，至于具体值是什么意思，我也不懂，借助[HandShaker](https://www.smartisan.com/apps/handshaker)，来看一眼：
![](http://7xkl0t.com1.z0.glb.clouddn.com/18-5-7/68784784.jpg)

**上面拿到的Exif属性信息，其实就是上图查看的属性信息。**

竟然可以看到我当时拍照的地点，岂不是暴露了我的行踪，不怕：
> 在Android相机的设置中关闭“位置信息” 就看不到拍照的地点了。



参考链接:

0. [HandShaker](https://www.smartisan.com/apps/handshaker ) 是[锤子科技](https://www.smartisan.com/)出品的一款在Mac上使用的Android文件管理器，很好用——免费的；
1. [ExifInterface 官方文档](https://developer.android.com/reference/android/media/ExifInterface)
2. [ExifInterface 支持库简介](http://developers.googleblog.cn/2017/01/exifinterface.html)











