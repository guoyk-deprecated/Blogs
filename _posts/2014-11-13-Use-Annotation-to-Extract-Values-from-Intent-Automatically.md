---
layout: post
title: 使用 Annotation 自动获取 Intent 传值
---

没错，是的，这是一篇 Android 相关的文章，`Annotation(注解)` 是 Java 最丧病的一个能力，合理使用可以省下一大坨代码。

一个典型场景，从一个 Activity 启动另一个 Activity，一般通过在 Intent 里面的 Extra 放上可序列化的对象来实现传值。

> 谷歌设计这个的初衷是为了实现跨应用启动 Activity，结果导致了同一个应用内的两个 Activity 无法方便地获取到彼此的引用。(谜之瞎扯)

一般情况下，在第二个 Activity 中会用 `this.field = intent.getSerializableExtra("field_name")` 来一个一个拆出传过来的值。

这样繁琐的过程，必定有好用的解决方案，那就是 `Annotation`。

## 声明 Annotation

```java
package io.yanke.Askala;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Created by yanke on 14/11/13.
 */

  @Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)

  public @interface IntentValue {
    String value() default "";
  }
```

这几乎是最简单的 `Annotation` 声明，定义 `value()` 并且赋了一个默认值，这样就可以使用 `@IntentValue("name")`或者`@IntentValue`的写法，而不是`@IntentValue(key="value")`了。

## 创建一个基类

```java
//...

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  Intent intent = getIntent();                      // 获取 intent
  Field[] fields = this.getClass().getFields();     // 反射获取当前Class的所有成员变量声明

  for (Field field : fields) {                      // 遍历所有成员变量声明
    IntentValue value = field.getAnnotation(IntentValue.class);     // 查找是否有`IntentValue`注解
    if (value != null) {
      String key = value.value();                   // 拿到注解的`value()`
      if (key.equalsIgnoreCase("")) {               // 如果'value()`等于空, 就使用成员变量名当做key
        key = field.getName();
      }
      Object obj = intent.getSerializableExtra(key);// 从 intent 中拿对象，注意 `getSerializableExtra` 可以用于 `String`, `Integer`, 等等
      try {
        field.set(this, obj);                       // 试图使用反射赋值
      } catch (IllegalAccessException e) {
        e.printStackTrace();
      }
    }
  }

  //...
}
```

创建一个基类如上，然后让所有的 Activity 继承它。

## 给成员变量加上注解

```java
public class SecondActivity extends BaseActivity {

  @IntentValue                      // 默认使用成员变量的名字作为key
  public String test = "BBBBB";

  @IntentValue("test2")             // 自定义一个key
  public String testt = "AAAA";

  //...
}
```

## 像往常一样地启动一个 Activity

```java
Intent intent = new Intent();
intent.setClass(getBaseContext(), SecondActivity.class);
intent.putExtra("test", "bbbbbbbbb");
intent.putExtra("test2","aaaaaaaaa");
startActivity(intent);
```

这样，通过`Annotation`，`intent` 的 `extras` 中的 `test` 会被自动地赋值给 `SecondActivity.test`，然后 `test2` 会被赋值给 `SecondActivity.testt`。

是不是一大坨代码都给省掉了，看着倍儿爽。
