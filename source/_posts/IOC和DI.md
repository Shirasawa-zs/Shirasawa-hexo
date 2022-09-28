---
title: IOC和DI
date: 2022-09-23 09:25:00
author: scy
img: https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20220925112758.png
top: true
hide: false
cover: true
toc: false
summary: Spring的IOC和DI
categories: 后端
tags:
  - spring
  - java
---



## IOC 和 DI

学了很久的spring全家桶，但总感觉对这东西理解很模糊，没有系统总结过，学习总要输入和输出，感觉之前的学习总在被动输入，没有主动输出，总感觉少了点东西，今天以我理解的角度来看待spring，可能理解很片面，也可能哪天我成了大牛，重新看看这一篇博客，把他完善的更好

首先我们界定所有的软件思想都需要满足OCP原则，也就是我们软件工程需求的开闭原则，这是我们软件设计的一种思想，具体可以理解是

**对扩展开放，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。**

**对修改封闭，意味着类一旦设计完成，就可以独立完成其工作，而不要对类进行任何修改。**

**只有依赖于抽象。实现开放封闭的核心思想就是对[抽象编程](https://baike.baidu.com/item/抽象编程?fromModule=lemma_inlink)，而不对具体编程，因为抽象相对稳定。让类依赖于固定的抽象，所以对修改就是封闭的；而通过[面向对象](https://baike.baidu.com/item/面向对象?fromModule=lemma_inlink)的继承和对多态机制，可以实现对抽象体的继承，通过覆写其方法来改变固有行为，实现新的扩展方法，所以对于扩展就是开放的。**

以上加粗的字体来自百度百科，如果你想进一步了解，欢迎百度或者Google

同时我们需要有一个共识，我们所有的目的都是为了代码的更高稳定性，有了这个共识，我们就可以开始了。

### 为什么需要IOC,

IOC本质上并不是一种技术，它是一种思想，也就是我们常说的控制反转，但我们为什么需要控制反转，使用控制反转有什么好处是我们需要去考虑的。我们举个栗子

我们首先以当前现象级的手游王者荣耀来说，他有很多英雄，假如你选择了一个英雄并选择放大招

#### 第一版

hero类

```java
public class LiBai {
    public void R(){
        System.out.println("李白放大招了");
    }
}



public class DianWei {
    public void R(){
        System.out.println("典韦放大招了");
    }
}
```

Main类

```JAVA
public class Main {
    public static void main(String[] args) {
       String name = Main.getplayer();
       if(name.equals("Libai")){
           LiBai liBai = new LiBai();
           liBai.R();
       }else if(name.equals("DianWei")){
           DianWei dianWei = new DianWei();
           dianWei.R();
       }

    }

    private static String getplayer(){
        Scanner scanner = new Scanner(System.in);
        String hero = scanner.nextLine();
        return hero;
    }
```





#### 第二版

java类

```java
public interface Hero {
    public void R();
}


public class LiBai implements Hero{
    public void R(){
        System.out.println("李白放大招了");
    }
}

public class DianWei implements Hero{
    public void R(){
        System.out.println("典韦放大招了");
    }
}
```

Main类

```JAVA
public class Main {
    public static void main(String[] args) {
       String name = Main.getplayer();
       Hero hero = null;
       if(name.equals("Libai")){
            hero = new LiBai();

       }else if(name.equals("DianWei")){
           hero = new DianWei();
       }
       hero.R();
    }

    private static String getplayer(){
        Scanner scanner = new Scanner(System.in);
        String hero = scanner.nextLine();
        return hero;
    }
}
```



此时，我们可以思考，如果我们想要去加一个英雄，那么，我们就要if else 或者 switch case 里面去 new 一个对象，然后放大招，每一次我们都要去动里面的代码，有什么方法可以解决这个情况，此时有人说了：接口，没错，接口可以解决这个问题。其实这也引出了我们的标题，为什么要引入IOC

### 如何实现IOC的原理

让我们看看第二版的代码，是不是优雅了很多，但是还是有问题，我们解决了方法调用的问题，让一个接口可以统一方法的调用，但是仍然困扰我们的是我们不能统一对象的实例化。

此时我们同样来借用大佬的一段总结

**1.单纯interface可以统一方法的调用，但是它不能统一对象的实例化**

**2.面向对象主要做两件事情:实例化对象调用方法(完成业务逻辑)**

**3．只有一段代码中没有new的出现，才能保持代码的相对稳定，才能逐步实现OCP**

**4.上面的这句话只是表象，实质是一段代码如果要保持稳定，就不应该负责对象的实例化**

**5.对象实例化是不可能消除的**

**6.把对象实例化的过程，转移到其他的代码片段里**

我们通过以上的总结可以思考如何解决对象实例化(new)的问题，可不可以把对象实例化放进以一个工厂里，等我们需要时工厂给我们加载，由此，我们想到了我们可以用工厂模式来尝试的解决。

工厂模式有三种

简单工厂模式

工厂模式

抽象工厂模式

如果有机会，看完设计模式，也会写一点东西记录一下

如何实现IOC的原理

#### 第三版

Factory类

```java
public class FacotryHero {

    public static Hero getHero (String name){
        Hero hero = null;
        if(name.equals("Libai")){
            hero = new LiBai();

        }else if(name.equals("DianWei")){
            hero = new DianWei();
        }
        return hero;
    }
}
```

Main类

```java
public static void main(String[] args) {
        String name = Main.getplayer();
        Hero hero = FacotryHero.getHero(name);
        hero.R();
    }
```

这样我们的代码看起来就变成了稳定的

我们可以了解，Factory是不稳定的，Main类是稳定的，这时候有人说了，这不是骗人吗,把原来项目里的不稳定代码放进了Factory里就变成稳定的了? 实际上确实如此，我们可以这样想，我们代码的不稳定就是来自于我们需要new的对象，当我们需要创建对象就不可避免的去new对象，如果有一个代码可以帮我们把所有的不稳定隔离起来，也就是帮我们去new所有的对象并返回，那我们的主类代码就是稳定的满足ocp，当然还有一个问题FacotryHero.getHero这串代码本质上来说还是不稳定的，因为如果Factory里没有，我们还是要去动Factory里的代码，这就不满足ocp原则了，只有Factory足够的大，足够的完善，可以帮我们处理所有的请求，我们的代码才能实现真正意义上的稳定，是不是有点感觉了。

我们来聊聊不稳定，我们开发的意义实际上还是面向用户也业务编程，没有了用户，我们什么都不是，用户提出了一个需求，这个需求的解不是唯一的，这不为一的解才是我们代码不稳定的本质原因。就好比一个用户选择了亚瑟，另一个用户选择了李白，这两个不同的需求的变化导致了我们代码的不稳定，我们的工厂里有亚瑟，好的，我直接返回给你，没有李白，不好意思，你的工厂里要自己再去new一个，这就造成了代码的不稳定，总的来说表面上是我们代码的不稳定，实际上是用户需求的变化。这也是我们的面向对象编程，用户的每一个需求都是我们可能创建的对象。那有没有办法在用户提出需求的时候，就已经可以自动创建对象，实际上是有的。

#### 第四版

factory

```java
public static Hero GetHeroByReflect(String name) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        Hero hero = null;
        String strname = "com.scy.test.Hero." + name;
        Class<?> cls = Class.forName(strname);
        //cls.getDeclaredConstructor().newInstance();
        Object obj = cls.newInstance();
        return (Hero) obj;
    }
```

Main

```java
public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
       String name = Main.getplayer();
       Hero hero = FacotryHero.GetHeroByReflect(name);
       hero.R();
   }
```

我们通过用户的字符串输入实现用户自己创建对象并返回，**工厂模式 + 反射**在IOC里面有着大量的应用，但和我们有一定的区别，spring通过工厂模式和反射，当用户一个字符串注入时我们现在的代码每次都需要进行反射，但是spring可能将他放进缓存里，下一次再去创建直接从缓存里拿就行，但工厂模式+反射并不是IOC和DI

IOC是控制反转,DI是依赖注入，我们的工厂模式 + 反射仍然需要用户去输入字符串，容器去创建对象，是一个正常的思维模式，**没有IOC精华的控制反转和依赖注入**

配置文件的变化违反OCP吗，配置文件可以理解是外部的输入，就如同我们的用户输入意义，是一种变化，它的改变不违法OCP原则

### IOC和DI

抓住两个字 **要** 和 **给**

原来的代码

```java
public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
       String name = Main.getplayer();
       Hero hero = FacotryHero.GetHeroByReflect(name);
       hero.R();
   }
```

我们还是需要FactoryHero.get方法向容器去要这个对象，只有对象的创建，没有控制的反转

想要的代码

java

```java
public class A{
    private IC ic;
    
    public void print(){
        this.c.print();
    }
}
```

我们想主动声明一个对象，容器就可以主动把对象返回给我们，而不是去申请。

#### IOC DI DIP

**DIP: Dependency Inversion Principle 依赖倒置**

了解三个概念

- 高层模块不应该依赖底层模块，两者应该都依赖抽象 //高层就是抽象
- 抽象不应该依赖细节
- 细节应该依赖抽象

**DI**

依赖注入

1. 属性注入
2. 构造器注入
3. 等等

前提：我们的容器里有所有的对象，演示

```java
public class Container{
    public void get Bean(){
        //1 构造器注入
        IC ic = new C();
        A a = new A(ic);
        
        //2 属性注入
        A a1 = new A();
        a1.setIC(ic);
    }
}
```

容器的本质是装配对象，比如我们有A, B, C三个对象， C对象依赖了A对象，我们在实现A对象的时候完全不用去考虑C对象，因为我们是面向抽象编程，我们未来编程可能是C接口的实现类，可能是c1,c2,c3等等，我们设计的时候只需要考虑A对象就行了。至于装配的工作交给容器就行了。

**IOC**

IOC就是一种思想，很模糊，没有一个具体的界定.DI就是IOC的一个具体的实现。

比如还是上述的代码，如果我们不采用IOC的话

```java
public class A{
    private IC ic;
    
    public void print(){
        this.c.print();
    }
}
```

主控类是A，也可以理解是我们程序员，因为我们可以决定是否创建

当我们引入了IOC思想，主控类变成了容器，由容器来决定类的依赖注入和装配。

或者程序员负责生产类，产品经理或用户来负责使用，这还不是很理解，等对IOC有进一步的理解再说吧