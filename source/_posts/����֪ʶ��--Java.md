title: 面试知识点回顾--Java
date: 2016/06/02
tags: [面试, Java]

------

去摩根面试了暑期实习，本来自信满满，以为没啥问题，结果最后还是被拒绝了。

现在静下来想想自己面试时回答的问题，确实是很多问题回答的不对或者不全面。当初知道的好多知识忘了。现在总结如下，同时也记录平时遇到的一些面试问题。

<!--more-->

###多继承###
在 Java 中不支持多继承，因为 C++ 中的多继承会存在调用**歧义性**，同时会存在致命的**菱形继承**。
但是为了在 Java 中实现多继承的功能，我们可以使用接口和内部类的机制来实现。

一个类可以 implements 多个接口，在自己本类中具体实现它的细节。从而表现不同的特征。

在本类中可以通过定义多个内部类，每个内部类继承一个外部类，进而可以在本类中扩展外部类的属性。例如下面的代码：

```
public class Father {
    public int strong(){
        return 9;
    }
}

public class Mother {
    public int kind(){
        return 8;
    }
}

public class Son {
    
    /**
     * 内部类继承Father类
     */
    class Father_1 extends Father{
        public int strong(){
            return super.strong() + 1;
        }
    }
    
    class Mother_1 extends  Mother{
        public int kind(){
            return super.kind() - 2;
        }
    }
    
    public int getStrong(){
        return new Father_1().strong();
    }
    
    public int getKind(){
        return new Mother_1().kind();
    }
}
```


### 垃圾回收机制###
###内存溢出###


