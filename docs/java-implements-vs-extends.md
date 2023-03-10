# Java 中的实现与扩展

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-implements-vs-extends>

## 1.概观

在本教程中，我们将讨论继承，这是面向对象编程的关键概念之一。在 Java 中，用于继承的两个主要关键字是 **`extends`和`implements`** 。

## 2.`extends`对`implements`

我们来讨论一下这两个关键词的区别。

**我们使用`extends`关键字来继承一个类的属性和方法。**充当父类的类称为基类，从这个基类继承的类称为派生类或子类。主要是，`extends`关键字用于将父类的功能扩展到派生类。同样，一个基类可以有许多派生类，但是一个派生类只能有一个基类，因为 Java 不支持多重继承。

另一方面，**我们使用`implements`关键字来实现一个接口。一个接口只由抽象方法组成。一个类将实现接口，并根据所需的功能定义这些抽象方法。**与`extends`不同，任何类都可以实现多个接口。****

尽管这两个关键字都与继承的概念一致，但关键字`implements`主要与抽象相关，用于定义契约，而`extends`用于扩展类的现有功能。

## 3.履行

让我们跳到实现，并一个接一个地详细了解 e `xtends`、`implements`和多重继承。

### 3.1.`extends`

让我们首先创建一个名为`Media` 的类，它有`id`、`title`和`artist`。这个类将充当基类。`VideoMedia`和`AudioMedia`将扩展这个类的功能:

```java
public class Media {

    private int id;
    private String title;
    private String artist;
    // standard getters and setters
}
```

现在，让我们创建另一个名为`VideoMedia`的类，它继承了`Media`的`extends`类的属性`.`此外，它还有自己的属性，如`resolution`和`aspectRatio`:

```java
public class VideoMedia extends Media {

    private String resolution;
    private String aspectRatio;
    // standard getters and setters
}
```

类似地，职业`AudioMedia` 也是`extends`职业`Media `，并且将拥有自己的附加属性，如`bitrate`和`frequency`:

```java
public class AudioMedia extends Media {

    private int bitrate;
    private String frequency;
    // standard getters and setters

    @Override
    public void printTitle() {
        System.out.println("AudioMedia Title");
    }
}
```

让我们为基类和派生类创建对象来查看继承的属性:

```java
Media media = new Media();
media.setId(001);
media.setTitle("Media1");
media.setArtist("Artist001");

AudioMedia audioMedia = new AudioMedia();
audioMedia.setId(101);
audioMedia.setTitle("Audio1");
audioMedia.setArtist("Artist101");
audioMedia.setBitrate(3500);
audioMedia.setFrequency("256kbps");

VideoMedia videoMedia = new VideoMedia();
videoMedia.setId(201);
videoMedia.setTitle("Video1");
videoMedia.setArtist("Artist201");
videoMedia.setResolution("1024x768");
videoMedia.setAspectRatio("16:9");

System.out.println(media);
System.out.println(audioMedia);
System.out.println(videoMedia);
```

这三个类都打印相关的属性:

```java
Media{id=1, title='Media1', artist='Artist001'}
AudioMedia{id=101, title='Audio1', artist='Artist101', bitrate=3500, frequency='256kbps'} 
VideoMedia{id=201, title='Video1', artist='Artist201'resolution='1024x768', aspectRatio='16:9'} 
```

### 3.2.`implements`

为了理解抽象和接口，我们将创建一个接口`MediaPlayer` ，它有两个名为`play`和`pause.`的方法。如前所述，这个接口中的所有方法都是抽象的。换句话说，接口只包含方法声明。

**在 Java 中，接口不需要显式声明一个方法为`abstract`或`public`。**实现接口`MediaPlayer` 的类将定义这些方法:

```java
public interface MediaPlayer {

    void play();

    void pause();
}
```

`AudioMediaPlayer` 类`implements` `MediaPlayer,` ，它将定义音频媒体的`play`和`pause`方法:

```java
public class AudioMediaPlayer implements MediaPlayer {

    @Override
    public void play() {
        System.out.println("AudioMediaPlayer is Playing");
    }

    @Override
    public void pause() {
        System.out.println("AudioMediaPlayer is Paused");
    }
}
```

同样，`VideoMediaPlayer implements` `MediaPlayer`也给`play`和`pause`视频媒体提供了一个方法定义:

```java
public class VideoMediaPlayer implements MediaPlayer {

    @Override
    public void play() {
        System.out.println("VideoMediaPlayer is Playing");
    }

    @Override
    public void pause() {
        System.out.println("VideoMediaPlayer is Paused");
    }
}
```

此外，让我们创建一个`AudioMediaPlayer `和`VideoMediaPlayer` 的实例，并为它们调用`play`和`pause `方法:

```java
AudioMediaPlayer audioMediaPlayer = new AudioMediaPlayer();
audioMediaPlayer.play();
audioMediaPlayer.pause();

VideoMediaPlayer videoMediaPlayer = new VideoMediaPlayer();
videoMediaPlayer.play();
videoMediaPlayer.pause();
```

`AudioMediaPlayer` 和`VideoMediaPlayer `分别调用各自的`play`和`pause`的实现:

```java
AudioMediaPlayer is Playing
AudioMediaPlayer is Paused

VideoMediaPlayer is Playing
VideoMediaPlayer is Paused
```

### 3.3.多重遗传

**由于歧义问题，Java 不直接支持多重继承。**当一个类从不止一个父类继承，并且两个父类都有相同名称的方法或属性时，就会出现不明确的问题。因此，子类不能解决要继承的方法或属性的冲突。**然而，一个类可以从多个接口继承。**让我们创建一个界面`AdvancedPlayerOptions`:

```java
public interface AdvancedPlayerOptions {

    void seek();

    void fastForward();
}
```

类`MultiMediaPlayer` `implements` `MediaPlayer` 和`AdvancedPlayerOptions`并定义了两个接口中声明的方法:

```java
public class MultiMediaPlayer implements MediaPlayer, AdvancedPlayerOptions {

    @Override
    public void play() {
        System.out.println("MultiMediaPlayer is Playing");
    }

    @Override
    public void pause() {
        System.out.println("MultiMediaPlayer is Paused");
    }

    @Override
    public void seek() {
        System.out.println("MultiMediaPlayer is being seeked");
    }

    @Override
    public void fastForward() {
        System.out.println("MultiMediaPlayer is being fast forwarded");
    }
}
```

现在，我们将创建一个`MultiMediaPlayer `类的实例，并调用所有实现的方法:

```java
MultiMediaPlayer multiMediaPlayer = new MultiMediaPlayer();
multiMediaPlayer.play();
multiMediaPlayer.pause();
multiMediaPlayer.seek();
multiMediaPlayer.fastForward();
```

正如所料，`MultiMediaPlayer` 调用了它的`play`和`pause`的实现:

```java
MultiMediaPlayer is Playing
MultiMediaPlayer is Paused 
MultiMediaPlayer is being seeked 
MultiMediaPlayer is being fast forwarded
```

## 4.结论

在本教程中，我们讨论了`extends`和`implements`之间的显著差异。此外，我们创建了类和接口来演示`extends`和`implements`的概念。**此外，我们讨论了多重继承以及如何使用接口来实现它。**

GitHub 上的[提供了这个实现。](https://web.archive.org/web/20221006214500/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)