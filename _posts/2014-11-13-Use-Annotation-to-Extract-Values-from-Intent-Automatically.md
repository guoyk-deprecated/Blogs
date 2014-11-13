---
layout: post
title: Use Annotation to Extract Values from Intent Automatically
---

`Annotation` is the most fancy part of Java. Using `Annotation` correctlly could save you a lot of time.

Consider the situation when you want to extract values from `intent` during `onCreate` of a Activity.

The normal way, you have to call `this.field = intent.getSerializableExtra("field_name")` one by one, and handle the key passing in carefully.

With Java Annotation, this process can be simplified to one word by adding a bit seasoning.

## Declare Annotation

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

I declared a very simple `Annotation` named `IntentValue`, the `value()` stored the key, default to `""`.

## Create a base class for Activity

```java
//...

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  Intent intent = getIntent();                      // Get the intent
  Field[] fields = this.getClass().getFields();     // Get all fields of runtime class using Reflect

  for (Field field : fields) {                      // Enumerate over all fields
    IntentValue value = field.getAnnotation(IntentValue.class);     // Get the Annotation
    if (value != null) {
      String key = value.value();                   // Get the value
      if (key.equalsIgnoreCase("")) {               // If 'value()` equals '', use the field name as key
        key = field.getName();
      }
      Object obj = intent.getSerializableExtra(key);    // Get the object from intent, be noticed, `getSerializableExtra` is safe for `String`, `Integer`, etc
      try {
        field.set(this, obj);                       // Try set the field by Reflect
      } catch (IllegalAccessException e) {
        e.printStackTrace();
      }
    }
  }

  //...
}
```

Add code above in `onCreate` of `BaseActivity` and make all activities inherit from it.

## Add Annotation to fields

```java
public class SecondActivity extends BaseActivity {

  @IntentValue                      // Use field name as key in intent by default
  public String test = "BBBBB";

  @IntentValue("test2")             // Use custom key
  public String testt = "AAAA";

  //...
}
```

## Start Activity using Intent normally

```java
Intent intent = new Intent();
intent.setClass(getBaseContext(), SecondActivity.class);
intent.putExtra("test", "bbbbbbbbb");
intent.putExtra("test2","aaaaaaaaa");
startActivity(intent);
```

Voila, value for `test` will be automatically mapped to `SecondActivity.test`, and `test2` will be mapped to `SecondActivity.testt` as specified by annotation.
