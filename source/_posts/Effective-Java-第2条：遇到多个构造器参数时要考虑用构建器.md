---
title: Effective Java-第2条：遇到多个构造器参数时要考虑用构建器
date: 2016-12-22 19:05:13
categories: Java
tags: [Java, 设计模式]
---
静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。考虑用一个类表示包装食品外面显示的营养成分标签。这些标签有几个是必须的，还有超过20个可选项。

## 重叠构造器(telescoping constructor)模式
对于这样的类，应该用那种构造器或者静态方法来编写呢？程序员一向习惯采用重叠构造器(telescoping constructor)模式，在这种模式下，你提供第一个只有必要参数的构造器，第二个构造器有一个可选参数，第三个有两个可选参数，以此类推，最好一个构造器包含所有可选参数。
<!-- more -->
下面实例，有四个可选域：
```java
public class NutritionFacts {
    private final int servingSize;          // required
    private final int servings;             // required
    private final int calories;             // optional
    private final int fat;                  // optional
    private final int sodium;               // optional
    private final int carbohydrate;         // optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
一句话，*重叠构造器模式可行，但是当有许多参数的时候，客户端代码会很难编写，并且仍然较难以阅读。*

## JavaBeans模式
还有第二种替代办法，即**JavaBeans模式**，在这种模式下，调用一个无参构造器来创建对象，然后调用setter方法来设置每个必要的参数，以及每个相关的可选参数。
```java
NutritionFacts  cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setFat(35);
cocaCola.setSodium(27);
```

这种模式弥补了重叠构造器模式的不足，说的明白一点，就是创建实例很容易，这样产生的代码读起来也容易。但是，因为构造过程被分到了几个调用中,***在构造过程中JavaBean可能处于不一致的状态。***还有一点，JavaBeas模式组织了把类做成不可变的可能。也就是多线程的时候无法保证其安全。

## Builder模式
直接上代码。

```java
public class NutritionFacts {
    private final int servingSize;          // required
    private final int servings;             // required
    private final int calories;             // optional
    private final int fat;                  // optional
    private final int sodium;               // optional
    private final int carbohydrate;         // optional

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
注意NutritionFacts是不可变的，所有的默认参数值都是单独放在一个地方。builder的setter方法返回builder本身，以便可以把调用链接起来，下面是客户端代码：
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

简而言之，*如果类的构造器或者静态工厂中具有多个参数，设计这种类时，Builder模式就是种不错的选择*，特别是大多数参数都是可选的时候。与使用传统的重叠构造器模式相比，使用Builder模式的客户端代码将更易于阅读和编写，构建器也比JavaBeans模式更加安全。
