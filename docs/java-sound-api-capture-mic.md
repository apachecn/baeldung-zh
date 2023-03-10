# Java 声音 API–捕捉麦克风

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sound-api-capture-mic>

## 1.概观

在本文中，我们将看到如何用 Java 捕获麦克风并记录输入的音频，然后保存到一个 WAV 文件中。为了捕捉麦克风传入的声音，我们使用 Java Sound API，它是 Java 生态系统的一部分。

**Java Sound API 是一个强大的 API，用于捕获、处理和回放音频，由 4 个包组成。我们将关注`javax.sound.sampled`包，它提供了捕获输入音频**所需的所有接口和类。

## 2.什么是`TargetDataLine`？

**[`TargetDataLine`](https://web.archive.org/web/20220524005437/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/sound/sampled/TargetDataLine.html)是一种 [`DataLine`](https://web.archive.org/web/20220524005437/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/sound/sampled/DataLine.html) 对象，我们用它来捕获和读取音频相关的数据，它从麦克风等音频捕获设备中捕获数据。**该接口提供了读取和捕获数据所需的所有方法，它从目标数据行的缓冲区中读取数据。

我们可以调用 AudioSystem 的`getLine`()方法，并为其提供`DataLine.Info`对象，该对象提供音频的所有传输控制方法。 [Oracle 文档](https://web.archive.org/web/20220524005437/https://docs.oracle.com/javase/8/docs/technotes/guides/sound/programmer_guide/chapter2.html)详细解释了 Java Sound API 是如何工作的。

让我们来看一下用 Java 从麦克风捕获音频所需的步骤。

## 3.捕捉声音的步骤

为了保存捕获的音频，Java 支持:AU、AIFF、AIFC、SND 和 WAVE 文件格式。我们将使用 WAVE(。wav)文件格式来保存我们的文件。

流程的第一步是初始化`AudioFormat`实例。`AudioFormat`通知 Java 如何解释和处理输入声音流中的信息。在我们的例子中，我们使用下面的`AudioFormat`类构造函数:

```java
AudioFormat(AudioFormat.Encoding encoding, float sampleRate, int sampleSizeInBits, int channels, int frameSize, float frameRate, boolean bigEndian)
```

之后，我们打开一个`DataLine.Info`对象。该对象保存与数据线(输入)相关的所有信息。**使用`DataLine.Info`对象，我们可以创建一个`TargetDataLine`的实例，它将所有输入的数据读入一个音频流。**为了生成`TargetDataLine`实例，我们使用了`AudioSystem.getLine()`方法并传递了`DataLine.Info`对象:

```java
line = (TargetDataLine) AudioSystem.getLine(info);
```

`line`是一个`TargetDataLine`实例，`info`是一个`DataLine.Info`实例。

一旦创建完成，我们就可以打开线来读取所有传入的声音。**我们可以使用一个`AudioInputStream`来读取输入的数据。总之，我们可以将这些数据写入一个 WAV 文件并关闭所有的流。**

为了理解这个过程，我们来看一个记录输入声音的小程序。

## 4.示例应用程序

要查看 Java Sound API 的运行情况，让我们创建一个简单的程序。我们将把它分成三个部分，首先构建`AudioFormat`，其次构建`TargetDataLine`，最后将数据保存为文件。

### 4.1.建筑`AudioFormat`

`AudioFormat`类定义了什么样的数据可以被`TargetDataLine`实例捕获。所以，第一步是初始化`AudioFormat`类实例，甚至在我们打开一个新的数据行之前。`App class`是应用程序的主类，负责所有的调用。我们在名为`ApplicationProperties`的常量类中定义了`AudioFormat`的属性。我们绕过所有必要的参数来构建`AudioFormat` 实例:

```java
public static AudioFormat buildAudioFormatInstance() {
    ApplicationProperties aConstants = new ApplicationProperties();
    AudioFormat.Encoding encoding = aConstants.ENCODING;
    float rate = aConstants.RATE;
    int channels = aConstants.CHANNELS;
    int sampleSize = aConstants.SAMPLE_SIZE;
    boolean bigEndian = aConstants.BIG_ENDIAN;

    return new AudioFormat(encoding, rate, sampleSize, channels, (sampleSize / 8) * channels, rate, bigEndian);
}
```

现在我们已经准备好了我们的`AudioFormat`，我们可以继续前进，构建`TargetDataLine`实例。

### 4.2.建筑`TargetDataLine`

我们使用`TargetDataLine`类从麦克风读取音频数据。在我们的例子中，我们获取并运行`TargetDataLine in the SoundRecorder` 类。`getTargetDataLineForRecord()`方法构建了`TargetDataLine instance`。

我们读取并处理音频输入，并将其转储到`AudioInputStream`对象中。我们创建一个`TargetDataLine` 实例的方法是:

```java
private TargetDataLine getTargetDataLineForRecord() {
    TargetDataLine line;
    DataLine.Info info = new DataLine.Info(TargetDataLine.class, format);
    if (!AudioSystem.isLineSupported(info)) {
        return null;
    }
    line = (TargetDataLine) AudioSystem.getLine(info);
    line.open(format, line.getBufferSize());
    return line;
}
```

### 4.3.构建和填充`AudioInputStream`

到目前为止，在我们的示例中，我们已经创建了一个`AudioFormat`实例，并将其应用于`TargetDataLine,`，打开数据线以读取音频数据。我们还创建了一个线程来帮助自动运行< em >录音机< /em >实例。我们首先在线程运行时构建一个字节输出流，然后将其转换成一个`AudioInputStream`实例。构建`AudioInputStream` 实例所需的参数是:

```java
int frameSizeInBytes = format.getFrameSize();
int bufferLengthInFrames = line.getBufferSize() / 8;
final int bufferLengthInBytes = bufferLengthInFrames * frameSizeInBytes;
```

请注意，在上面的代码中，我们将 bufferSize 减少了 8。我们这样做是为了使缓冲区和数组具有相同的长度，以便记录器可以在读取数据后立即将数据传送到行中。

现在我们已经初始化了所有需要的参数，下一步是构建字节输出流。下一步是将生成的输出流(捕获的声音数据)转换成一个`AudioInputStream`实例。

```java
buildByteOutputStream(out, line, frameSizeInBytes, bufferLengthInBytes);
this.audioInputStream = new AudioInputStream(line);

setAudioInputStream(convertToAudioIStream(out, frameSizeInBytes));
audioInputStream.reset();
```

在我们设置`InputStream`之前，我们将构建字节`[OutputStream](/web/20220524005437/https://www.baeldung.com/java-outputstream):`

```java
public void buildByteOutputStream(final ByteArrayOutputStream out, final TargetDataLine line, int frameSizeInBytes, final int bufferLengthInBytes) throws IOException {
    final byte[] data = new byte[bufferLengthInBytes];
    int numBytesRead;

    line.start();
    while (thread != null) {
        if ((numBytesRead = line.read(data, 0, bufferLengthInBytes)) == -1) {
            break;
        }
        out.write(data, 0, numBytesRead);
    }
}
```

然后我们[将字节`Outstream`转换成一个`AudioInputStream`T3，如下所示:](/web/20220524005437/https://www.baeldung.com/convert-byte-array-to-input-stream)

```java
public AudioInputStream convertToAudioIStream(final ByteArrayOutputStream out, int frameSizeInBytes) {
    byte audioBytes[] = out.toByteArray();
    ByteArrayInputStream bais = new ByteArrayInputStream(audioBytes);
    AudioInputStream audioStream = new AudioInputStream(bais, format, audioBytes.length / frameSizeInBytes);
    long milliseconds = (long) ((audioInputStream.getFrameLength() * 1000) / format.getFrameRate());
    duration = milliseconds / 1000.0;
    return audioStream;
}
```

### 4.4.将`AudioInputStream`保存到 Wav 文件

我们已经创建并填充了`AudioInputStream` ，并将其存储为`SoundRecorder`类的成员变量。我们将使用`SoundRecorder`实例 getter 属性在`App`类中检索这个`AudioInputStream`,并将其传递给`WaveDataUtil`类:

```java
wd.saveToFile("/SoundClip", AudioFileFormat.Type.WAVE, soundRecorder.getAudioInputStream());
```

`WaveDataUtil`类有将`AudioInputStream`转换成. wav 文件的代码:

```java
AudioSystem.write(audioInputStream, fileType, myFile);
```

## 5.结论

本文展示了一个使用 Java Sound API 通过麦克风捕获和记录音频的快速示例。本教程的全部代码可在 GitHub 上的[处获得。](https://web.archive.org/web/20220524005437/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)