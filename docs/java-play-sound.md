# 如何用 Java 播放声音

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-play-sound>

## 1。概述

在本教程中，我们将学习如何用 Java 播放声音。Java 声音 API 被设计成可以流畅连续地播放声音，甚至是很长的声音。

作为本教程的一部分，我们将使用 Java 提供的`Clip`和`SourceDataLine`声音 API 来播放一个音频文件。我们还将播放不同的音频格式文件。

此外，我们将讨论每个 API 的优缺点。此外，我们将看到一些第三方 Java 库也可以播放声音。

## 2。播放声音的 Java API

一般来说， `javax.sound`包中的 Java Sound APIs 提供了两种回放音频的方法。在这两种方法之间，声音文件数据的指定方式有所不同。Java Sound APIs 可以以流式缓冲方式和内存中非缓冲方式处理音频传输。Java 的两个最著名的声音 API 是`Clip`和`SourceDataLine.`

### 2.1。`Clip` API

API 是一个用于 Java 的无缓冲或内存声音 API。`Clip`类是`javax.sound.sampled`包的一部分，**是在读取和播放一个短声音文件**时有用的 **。在播放之前，整个音频文件被加载到内存中，用户可以完全控制播放。**

除了循环播放声音，它还允许用户在任意位置开始播放。

让我们首先创建一个示例类`SoundPlayerWithClip`，它实现了`LineListener`接口，以便接收用于回放的行事件(`OPEN`、`CLOSE`、`START`和`STOP`)。我们将从`LineListener` 实现`update()`方法来检查回放状态:

```java
public class SoundPlayerUsingClip implements LineListener {

    boolean isPlaybackCompleted;

    @Override
    public void update(LineEvent event) {
        if (LineEvent.Type.START == event.getType()) {
            System.out.println("Playback started.");
        } else if (LineEvent.Type.STOP == event.getType()) {
            isPlaybackCompleted = true;
            System.out.println("Playback completed.");
        }
    }
}
```

其次，让我们从项目的资源文件夹中读取音频文件。我们的资源文件夹包含三种不同格式的音频文件，即 WAV、MP3 和 MPEG:

```java
InputStream inputStream = getClass().getClassLoader().getResourceAsStream(audioFilePath);
```

第三，从文件流中，我们将创建一个`AudioInputStream`:

```java
AudioInputStream audioStream = AudioSystem.getAudioInputStream(inputStream);
```

现在，我们将创建一个`DataLine.Info`对象:

```java
AudioFormat audioFormat = audioStream.getFormat();
DataLine.Info info = new DataLine.Info(SourceDataLine.class, audioFormat);
```

让我们从这个`DataLine.Info`创建一个`Clip`对象并打开流，然后调用`start`开始播放音频:

```java
Clip audioClip = (Clip) AudioSystem.getLine(info);
audioClip.addLineListener(this);
audioClip.open(audioStream);
audioClip.start();
```

最后，我们需要关闭任何打开的资源:

```java
audioClip.close();
audioStream.close();
```

一旦代码运行，音频文件将会播放。

由于音频被预加载到内存中，我们有许多其他有用的 API 可以从中受益。

我们可以使用`Clip.loop`方法循环连续播放音频片段。

例如，我们可以将其设置为播放音频五次:

```java
audioClip.loop(4); 
```

或者，我们可以将其设置为无限期播放音频(或直到中断):

```java
audioClip.loop(Clip.LOOP_CONTINUOUSLY);
```

`Clip.setMicrosecondPosition`设定媒体位置。当剪辑下次开始播放时，它将从这个位置开始。例如，从第 30 秒开始，我们可以将其设置为:

```java
audioClip.setMicrosecondPosition(30_000_000);
```

### 2.2。`SourceDataLine` API

API 是一个用于 java 的缓冲或流式声音 API。`SourceDataLine`类是 `javax.sound.sampled`包的一部分，它可以播放无法预载入内存的长声音文件。

**当我们希望优化大音频文件的内存或流式传输实时音频数据时，使用`SourceDataLine`会更有效。**如果我们事先不知道声音有多长，什么时候结束，这也很有用。

让我们首先创建一个示例类，并从项目的 resource 文件夹中读取音频文件。我们的资源文件夹包含三种不同格式的音频文件，即 WAV、MP3 和 MPEG:

```java
InputStream inputStream = getClass().getClassLoader().getResourceAsStream(audioFilePath);
```

其次，从文件输入流中，我们将创建一个`AudioInputStream`:

```java
AudioInputStream audioStream = AudioSystem.getAudioInputStream(inputStream);
```

现在，我们将创建一个`DataLine.Info`对象:

```java
AudioFormat audioFormat = audioStream.getFormat();
DataLine.Info info = new DataLine.Info(Clip.class, audioFormat);
```

让我们从这个`DataLine.Info`创建一个`SourceDataLine`对象，打开流，调用`start`开始播放音频:

```java
SourceDataLine sourceDataLine = (SourceDataLine) AudioSystem.getLine(info);
sourceDataLine.open(audioFormat);
sourceDataLine.start();
```

现在，在`SourceDataLine`、**的情况下，音频数据是分块加载的，我们需要提供缓冲区大小**:

```java
private static final int BUFFER_SIZE = 4096;
```

现在，让我们从`AudioInputStream`读取音频数据，并将其发送到`SourceDataLine’s`播放缓冲区，直到其到达流的末尾:

```java
byte[] bufferBytes = new byte[BUFFER_SIZE];
int readBytes = -1;
while ((readBytes = audioStream.read(bufferBytes)) != -1) {
    sourceDataLine.write(bufferBytes, 0, readBytes);
}
```

最后，让我们关闭任何打开的资源:

```java
sourceDataLine.drain();
sourceDataLine.close();
audioStream.close();
```

一旦代码运行，音频文件将会播放。

在这里，我们不需要实现任何`LineListener` 接口。

### 2.3。`Clip`与`SourceDataLine`和的比较

让我们讨论一下两者的利弊:

| **夹子** | **SourceDataLine** |
| 支持从音频中的任何位置播放。
见`setMicrosecondPosition(long)`或`setFramePosition(int).` | 无法从声音中的任意位置开始播放。 |
| 支持循环播放(全部或部分声音)。
参见`setLoopPoints(int, int)` 和 `loop(int).` | 无法播放(循环播放)全部或部分声音。 |
| 播放前可以知道声音的持续时间。
见`getFrameLength()` 或 `getMicrosecondLength().` | 播放前无法知道声音的时长。 |
| 可以在当前位置停止播放，稍后继续播放。参见`stop() `和` start()` | 中途不能停止和恢复播放。 |
| 不适合播放大的音频文件，因为它在内存中。 | 它适用于播放较长的声音文件或实时播放声音。 |
| `Clip’s` `start`()方法确实会播放声音，但是不会阻塞当前线程(它会立即返回)，所以需要实现`LineListener`接口来知道播放状态。 | 与`Clip`不同，我们不必实现`LineListener`接口来知道回放何时完成。 |
| 无法控制将什么声音数据写入音频线的播放缓冲区。 | 可以控制将什么声音数据写入音频线的播放缓冲区。 |

### 2.4。Java API 支持 MP3 格式

目前，`Clip`和`SourceDataLine`都可以播放 AIFC、AIFF、AU、SND 和 WAV 格式的音频文件。

我们可以使用`AudioSystem`检查支持的音频格式:

```java
Type[] list = AudioSystem.getAudioFileTypes();
StringBuilder supportedFormat = new StringBuilder("Supported formats:");
for (Type type : list) {
    supportedFormat.append(", " + type.toString());
}
System.out.println(supportedFormat.toString());
```

**然而，我们无法使用 Java 声音 API`Clip`和`SourceDataLine.`** 播放流行的音频格式 MP3/MPEG，我们需要寻找一些可以播放 MP3 格式的第三方库。

如果我们将 MP3 格式文件提供给`Clip`或`SourceDataLine` API，我们将得到`UnsupportedAudioFileException`:

```java
javax.sound.sampled.UnsupportedAudioFileException: could not get audio input stream from input file
    at javax.sound.sampled.AudioSystem.getAudioInputStream(AudioSystem.java:1189)
```

## 3。播放声音的第三方 Java API

让我们来看看一对第三方库，它们也可以播放不同的音频格式文件。

### 3.1.JavaFX 库

[JavaFX](/web/20220703180127/https://www.baeldung.com/javafx) 有`Media`和`MediaPlayer`类，会播放 MP3 文件。它还可以播放其他音频格式，如 WAV。

让我们创建一个示例类，并使用`Media`和`MediaPlayer`类来播放我们的 MP3 文件:

```java
String audioFilePath = "AudioFileWithMp3Format.mp3";
SoundPlayerUsingJavaFx soundPlayerWithJavaFx = new SoundPlayerUsingJavaFx();
try {
    com.sun.javafx.application.PlatformImpl.startup(() -> {});
    Media media = new Media(
      soundPlayerWithJavaFx.getClass().getClassLoader().getResource(audioFilePath).toExternalForm());
    MediaPlayer mp3Player = new MediaPlayer(media);
    mp3Player.play();
} catch (Exception ex) {
    System.out.println("Error occured during playback process:" + ex.getMessage());
}
```

这个 API 的一个优点是它可以播放 WAV、MP3 和 MPEG 音频格式。

### 3.2.JLayer 库

[JLayer](https://web.archive.org/web/20220703180127/https://github.com/umjammer/jlayer) 库**可以播放 MPEG 格式的音频格式，包括 MP3。** **但是，它不能播放 WAV 之类的其他格式。**

让我们使用 Javazoom `Player`类创建一个示例类:

```java
String audioFilePath = "AudioFileWithMp3Format.mp3";
SoundPlayerUsingJavaZoom player = new SoundPlayerUsingJavaZoom();
try {
    BufferedInputStream buffer = new BufferedInputStream(
      player.getClass().getClassLoader().getResourceAsStream(audioFilePath));
    Player mp3Player = new Player(buffer);
    mp3Player.play();
} catch (Exception ex) {
    System.out.println("Error occured during playback process:" + ex.getMessage());
}
```

## 4。结论

在本文中，我们学习了如何用 Java 播放声音。我们还学习了两种不同的 Java 声音 API，`Clip`和`SourceDataLine`。后来，我们看到了`Clip`和`SourceDataLine`API 之间的差异，这将帮助我们为任何用例选择合适的 API。

最后，我们看到一些第三方库也可以播放音频并支持其他格式，如 MP3。

和往常一样，本文中使用的示例代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220703180127/https://github.com/eugenp/tutorials/tree/master/javax-sound)