今天在读关于图片解析的源码，看到这样的代码：

```
/**
     * Decode an input stream into a bitmap. If the input stream is null, or
     * cannot be used to decode a bitmap, the function returns null.
     * The stream's position will be where ever it was after the encoded data
     * was read.
     *
     * @param is The input stream that holds the raw data to be decoded into a
     *           bitmap.
     * @param outPadding If not null, return the padding rect for the bitmap if
     *                   it exists, otherwise set padding to [-1,-1,-1,-1]. If
     *                   no bitmap is returned (null) then padding is
     *                   unchanged.
     * @param opts null-ok; Options that control downsampling and whether the
     *             image should be completely decoded, or just is size returned.
     * @return The decoded bitmap, or null if the image data could not be
     *         decoded, or, if opts is non-null, if opts requested only the
     *         size be returned (in opts.outWidth and opts.outHeight)
     *
     * <p class="note">Prior to {@link android.os.Build.VERSION_CODES#KITKAT},
     * if {@link InputStream#markSupported is.markSupported()} returns true,
     * <code>is.mark(1024)</code> would be called. As of
     * {@link android.os.Build.VERSION_CODES#KITKAT}, this is no longer the case.</p>
     */
    public static Bitmap decodeStream(InputStream is, Rect outPadding, Options opts) {
        // we don't throw in this case, thus allowing the caller to only check
        // the cache, and not force the image to be decoded.
        if (is == null) {
            return null;
        }

        Bitmap bm = null;

        Trace.traceBegin(Trace.TRACE_TAG_GRAPHICS, "decodeBitmap");
        try {
            if (is instanceof AssetManager.AssetInputStream) {
                final long asset = ((AssetManager.AssetInputStream) is).getNativeAsset();
                bm = nativeDecodeAsset(asset, outPadding, opts);
            } else {
                bm = decodeStreamInternal(is, outPadding, opts);
            }

            if (bm == null && opts != null && opts.inBitmap != null) {
                throw new IllegalArgumentException("Problem decoding into existing bitmap");
            }

            setDensityFromOptions(bm, opts);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_GRAPHICS);
        }

        return bm;
    }
```

其中有两句代码：

```
 Trace.traceBegin(Trace.TRACE_TAG_GRAPHICS, "decodeBitmap");
 
 Trace.traceEnd(Trace.TRACE_TAG_GRAPHICS);
```

很好奇，这个Trace是干嘛的？之前接触到有一个TraceView！TraceView 是 Android 平台配备一个很好的性能分析的工具。它可以通过图形化的方式让我们了解我们要跟踪的程序的性能，并且能具体到 method！

跟进源码瞧个究竟，看到这个代码的文件注释：

```

/**
 * Writes trace events to the system trace buffer.  These trace events can be
 * collected and visualized using the Systrace tool.
 *
 * <p>This tracing mechanism is independent of the method tracing mechanism
 * offered by {@link Debug#startMethodTracing}.  In particular, it enables
 * tracing of events that occur across multiple processes.
 * <p>For information about using the Systrace tool, read <a
 * href="{@docRoot}tools/debugging/systrace.html">Analyzing Display and Performance
 * with Systrace</a>.
 */
```

翻译过来就是：

将跟踪事件写入系统跟踪缓冲区。 可以使用Systrace工具收集和可视化这些跟踪事件。

此跟踪机制独立于startMethodTracing（）提供的方法跟踪机制。 特别地，它使得能够跟踪跨多个进程发生的事件。

有关使用Systrace工具的信息，请参阅使用Systrace分析显示和性能。

哈哈，出来一个东西—— [Systrace](https://developer.android.com/studio/profile/systrace.html)。等我完了研究下这个Systrace.

接下来看着两个方法：

### Trace.traceBegin

```
    /**
     * Writes a trace message to indicate that a given section of code has
     * begun. Must be followed by a call to {@link #traceEnd} using the same
     * tag.
     *
     * @param traceTag The trace tag.
     * @param methodName The method name to appear in the trace.
     *
     * @hide
     */
    public static void traceBegin(long traceTag, String methodName) {
        if (isTagEnabled(traceTag)) {
            nativeTraceBegin(traceTag, methodName);
        }
    }
```

写入跟踪信息这标志着要追踪的代码已经开始运行。必须在后面调用traceEnd，并且使用相同的tag.

参数一：根据的标记tag;
参数二：在追踪路径中显示的方法名。

### Trace.traceEnd

```
 /**
     * Writes a trace message to indicate that the current method has ended.
     * Must be called exactly once for each call to {@link #traceBegin} using the same tag.
     *
     * @param traceTag The trace tag.
     *
     * @hide
     */
    public static void traceEnd(long traceTag) {
        if (isTagEnabled(traceTag)) {
            nativeTraceEnd(traceTag);
        }
    }
```

写入追踪信息标志着当前方法已经结束。必须和traceBegin成对的调用并且使用同一个tag.

参数：追踪tag和traceBegin传入的要一致。

至此为止，明白了图片解析里的这两句代码是干嘛的：

/*  * Writes trace events to the kernel trace buffer.  These trace events can be  * collected using the "atrace" program for offline analysis.  */


> 将这些追踪时间写入内部追踪缓存中。这些追踪事件可以被“atrace”程序收集起来，进行离线分析。

但是，再往里面走的话，发现了这东西竟然也走到了Native层：

```
private static native void nativeTraceBegin(long tag, String name);
private static native void nativeTraceEnd(long tag);
```

这两个方法是在哪儿呢？仔细看看了文件发现了有在各种tag上有两句注释：

```
// These tags must be kept in sync with system/core/include/cutils/trace.h.
// They should also be added to frameworks/native/cmds/atrace/atrace.cpp.
```

好吧，原来这东西到最后也给C++去做了！

那么问题了，我有什么理由不去好好学下C++,NDK 呢？





