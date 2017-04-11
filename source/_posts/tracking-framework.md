---
title: 视觉跟踪框架
date: 2016-05-30 13:05:09
tags:
top: 10
---

最近打算写一个多目标视觉跟踪的框架，一定要极易维护和扩展，方便在科研和项目中使用。

## 第一阶段 —— 单目标跟踪

### Tracker 类 —— 跟踪器的接口

框定第一帧目标位置，tracking 的任务是在接下来所有帧里定位目标。所以不管跟踪算法多么复杂，在调用者看来应该只需要用到两个函数：

```cpp
void init(const Mat frame, const Rect rect);
Rect track(const Mat frame);
```

<!--more-->

那么视觉跟踪的代码看起来就像这样：

```cpp
// init
tracker.init(init_frame, init_rect)

// loop
Mat frame = getNextFrame();
while (!frame.empty()) {
    // track
    Rect rect = tracker.track(frame);
    show(frame, rect);

    // new frame
    frame = getNextFrame();
}
```

所以我们为所有的 tracking 算法建立一个公共的父类（接口）：Tracker 类。为了判断跟踪器是否已初始化，除了 init 和 track 之外，我们为 Tracker 类添加一个 empty() 函数。包含这三个纯虚函数的 Tracker 类如下：

```cpp
class Tracker {
public:
    Tracker(void) {};
    virtual ~Tracker(void) {}
    
    virtual void        init(const cv::Rect& rect, const cv::Mat& frame) = 0;
    virtual cv::Rect    track(const cv::Mat& frame) = 0;
    virtual bool        empty() const = 0;
};
```

任何一个跟踪算法，继承自 Tracker 类，都必须实现这三个函数。如：

```cpp
class KCFTracker : publick Tracker {
    // ...
}
```

### Sequence 类 —— 读取帧图像

跟踪的帧序列可能来自视频、摄像机或序列图像，从路径读取图像、转换图像格式等繁琐的操作，我们希望封装在一个类 Sequence 里，从而主函数能十分方便地提取 frame，就像这样：

```cpp
seq >> frame;
```

而完全不用管背后的繁琐细节。于是跟踪过程可以很方便地写成：

```cpp
while (seq >> frame) {
    rect = tracker.track(frame);
    show(frame, rect);
}
```

于是我们可以想象，Sequence 类会在构造函数里输入配置（媒体类型、路径等），然后重载 operator>> 函数已读取图像，并能返回 bool 值判断图像读取是否成功。例如，简单的情况，Sequence 类大致长这样：

```cpp
class Sequence {
private:
    std::string         path_video;
    cv::VideoCapture    cap;
    
public:
    Sequence(const Config& c);
    ~Sequence();
    
    bool    operator>> (cv::Mat& frame);
    bool    empty() const;
};
```

### Annotator 类 —— 标注目标初始位置

目标的初始位置通常通过手工标注或检测结果得到。一个常见的需求是，在视频播放过程中按下空格键暂停，然后用鼠标标注目标框，再次按下空格键跟踪该目标。我们将这个标注过程封装在一个 Annotator 类中，希望尽可能简单地调用它实现标注，就像这样：

```cpp
Annotator anno;
rect = anno.annotate(frame);
```

### Config 类 —— 取用配置

视频路径在哪里？选用哪个跟踪器？跟踪对象是视频还是摄像机？参数是什么？……在控制台应用程序中，这些配置可能从文本文件中读取得到，在 UI 界面程序中，可能从用户输入得到。不管怎样，我们需要读取、存储并在合适的时候用到这些配置。我们把这个细节封装在 Config 类里。例如，我们可以从配置文件中读取配置：

```cpp
Config::Config(const std::string& conf_path);
```

并可以十分容易地取用配置，就像这样：

```cpp
Config c(conf_path);
video_path = c["video_path"];
tracker_name = c["tracker"];
```

为了实现这种功能，只要在 Config 类中定义一个 hash map，并重载 operator[] 即可。于是 Config 类可以定义为：

```cpp
class Config {
private:
    std::map<std::string, std::string> conf_map;
    
public:
    Config(const std::string& path_conf);
    ~Config();
    
    std::string operator[] (const std::string& name) const;
};
```

### TrackerFactory 类 —— 跟踪器工厂

我们可能需要测试不同的跟踪器性能，或更换更新更好的跟踪器，我们希望有个函数，只要输入跟踪器的名称，就能返回我想要的跟踪器。此外，不同跟踪器使用不同的参数，初始化的过程用到的变量是不一样的。我们把这个过程封装在一个跟踪器工厂类 TrackerFactory 中。

很简单，TrackerFactory 类只有一个 createTracker 函数。由于跟踪器类型和参数读取在 Config 对象里，所以 createTracker 的输入应该是 Config 对象。就像这样：

```cpp
class TrackerFactory {
public:
    TrackerFactory(){}
    ~TrackerFactory(){}
    Tracker* createTracker(const Config& c);
};
```

### 跟踪主程序

有了上述封装类的支持，主函数中的跟踪代码变得很容易撰写：

```cpp
int main() {
    // definitions

    Config c(CONF_PATH);
    Sequence seq(c);
    Annotator anno;
    TrackerFactory factory;
    Tracker* tracker;
    Rect rect;

    // tracking loop

    Mat frame;
    while (seq >> frame) {
        if (NULL == tracker || tracker->empty()) {
            imshow(WIN_NAME, frame);
        } else {
            Rect rect = tracker->track(frame);
            show(frame, rect);
        }

        // annotate
        if (' ' == waitKey(1)) {
            Rect rect = anno.annotate(WIN_NAME, frame);
            // init
            tracker = factory.createTracker(c);
            tracker->init(rect, frame);
        }
    }
}
```

独立封装的好处在于，容易维护和扩展。例如输入媒介从视频变成了图像序列，我仅需少量地修改 Sequence 的构造函数和 operator>> 函数，而其他任何代码都可以保持原样。又如，目标位置不再由鼠标标注得到，而是由检测结果得到，那么我仅需要把调用 Annotator 的部分换成检测结果的输入即可。

此外，还希望这些独立的类有极低的耦合度，从而每个模块都有很高的可复用性。例如，KCFTracker 类虽然继承自 Tracker，但我希望它只要把 ": public Tracker" 删除掉就能立刻独立使用，而无需 Tracker 头文件。

以上就是跟踪代码的第一个版本，可以在 [version 1.0](http://github.com) 下载，已使用 MIT Lisense 证书开源。

## 第二阶段 —— 多目标跟踪

以上代码实现了单目标的标注和跟踪。为了扩展成多目标的情形，需要做些修改。

首先是 Annotator 的 annotate() 函数，应该允许一次标注多个目标，所以返回值应该是一个 Rect 集合，即 vector<Rect>。

```cpp
void annotate(const std::string& win_name, 
              const cv::Mat& frame, std::vector<cv::Rect>* rects);
```

这里采用引用传值来返回，以更高效。于是多目标跟踪可以简单地通过循环来实现：

```cpp
int main() {
    /***** 定义变量 *****/

    Config c(path_conf);            // 初始化配置信息
    Sequence seq(c);                // 初始化视频序列
    Annotator anno;                 // 初始化标注器
    TrackerFactory factory;         // 初始化跟踪器工厂
    vector<Tracker*> trackers;      // 定义跟踪器集合
    
    /***** 跟踪开始 | 播放视频 *****/
    
    Mat frame, frame_copy;
    while (seq >> frame) {
        // 跟踪目标 | 播放视频

        if (trackers.empty()) {
            // 未初始化，直接播放视频
            imshow(win_name, frame);
        } else {
            // 已初始化，逐一跟踪
            frame.copyTo(frame_copy);
            for (auto iter = trackers.begin(); iter < trackers.end(); iter++) {
                Rect rect = (*iter)->track(frame);
                rectangle(frame_copy, rect, Scalar(255, 255, 0));
            }
            imshow(win_name, frame_copy);
        }
        
        // 标注目标 & 初始化跟踪器（空格键）
        
        if (' ' == waitKey(1) || seq.frameIndex() == 1) {
            // 标注目标位置
            vector<Rect> rects;
            anno.annotate(win_name, frame, &rects);
            
            // 初始化跟踪器
            for (auto iter = rects.begin(); iter < rects.end(); iter++) {
                Tracker* tracker = factory.createTracker(c);
                tracker->init(*iter, frame);
                trackers.push_back(tracker);
            }
        }
    }
    
    /***** 跟踪结束，释放资源 *****/
    
    for (auto iter = trackers.begin(); iter < trackers.end(); ++iter) {
        Tracker* tracker = *iter;
        delete tracker;
    }
}
```

尽管这段代码是可以 work 得不错的，但这么实现多目标跟踪不够优雅。不够优雅的后果，便是难以维护和扩展。例如，如果需要在这些跟踪框中，删除一个目标，那么需要在 annotate() 函数中传入 vector<Tracker*>，删除指定位置的 Tracker* 并释放。这样的耦合会在后面修改代码时遇到更大的问题。

目前有两个想法，来优化 tracking framework。一个是引入 MultiTracker 类，使用一帧 frame 和多个 Rect 初始化，在跟踪后返回 vector<Rect>，并维护这些目标。另一个是引入 Target 类，每个 Target 类对应一个 Tracker，一个 Rect，和首帧、末帧位置等。



从接口层面来讲，我们希望有个多目标跟踪器类 MultiTracker，它封装了内部复杂的实现，而仅对用户留出三个接口：init, track 和 render。对于多目标的初始化和跟踪，封装在 init() 和 track() 函数里，而对于不同目标的着色，封装在 render() 函数里。从而多目标跟踪的主代码可以十分简洁地写成：

```cpp
MultiTracker tracker;
// ...
while (seq >> frame) {
    tracker.track(frame);
    imshow(win_name, tracker.render());
}
```

待续！

## 第三阶段 —— 结构优化

### 基本思路

- Config 类在全局应该只有一个实例，为了避免不一致和节约内存，应当使用单例模式；
- 跟踪算法的源代码与 Tracker 父类（接口）往往不一致，为了便于维护和扩展，不应该直接修改原来的跟踪代码，而是采用适配器模式，为每种跟踪源代码配置一个适配器来实现 Tracker 接口；
- 


