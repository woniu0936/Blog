# `Reflect.java` 全面使用指南

`Reflect.java` 是一个精巧的Java反射库，它通过提供链式调用（Fluent API）的方式，极大地简化了Java原生反射的复杂性。本指南将通过一个全面的示例，展示如何使用它来操作类、字段、方法和内部类，包括静态和私有成员。

## 1. 准备用于演示的类

我们首先创建一个复杂的Java类 `ComprehensiveDemo.java`，它包含了我们将要用反射操作的各种元素。

```java
package com.example.reflect;

import java.util.Arrays;

/**
 * 一个全面的演示类，用于展示Reflect.java的所有特性。
 */
public class ComprehensiveDemo {

    // --- 字段 (Fields) ---
    public static String publicStaticField = "Public Static Field";
    private static String privateStaticField = "Private Static Field";
    protected static String protectedStaticField = "Protected Static Field";
    String defaultStaticField = "Default Static Field"; // 包私有

    public String publicField = "Public Field";
    private String privateField = "Private Field";
    protected String protectedField = "Protected Field";
    String defaultField = "Default Field";

    private final String finalField = "I am final and cannot be changed.";

    // --- 构造方法 (Constructors) ---
    public ComprehensiveDemo() {
        this.privateField = "Initialized by public constructor";
    }

    private ComprehensiveDemo(String privateFieldValue, String publicFieldValue) {
        this.privateField = privateFieldValue;
        this.publicField = publicFieldValue;
    }

    // --- 方法 (Methods) ---
    public static String publicStaticMethod() {
        return "Public Static Method says hi!";
    }

    private static String privateStaticMethod(String name) {
        return "Private Static Method says hi to " + name;
    }

    public void publicMethod() {
        System.out.println("Public Method called. Current private field is: '" + privateField + "'");
    }

    private String privateMethod(int... numbers) {
        return "Private Method received numbers: " + Arrays.toString(numbers);
    }

    // 重载方法
    public void overloadedMethod() {
        System.out.println("Overloaded method with no params.");
    }

    public void overloadedMethod(String text) {
        System.out.println("Overloaded method with String param: " + text);
    }


    // --- 静态嵌套类 (Static Nested Class) ---
    public static class StaticNestedClass {
        public static String nestedStaticField = "Nested Static Field";
        private String nestedInstanceField = "Nested Instance Field";

        public static String nestedStaticMethod() {
            return "Nested Static Method says hi!";
        }

        public String nestedInstanceMethod() {
            return "Nested Instance Method says hi! My field is: " + nestedInstanceField;
        }
    }

    // --- 非静态内部类 (Member Class) ---
    public class InnerClass {
        public String innerField = "Inner Field";

        public String innerMethod() {
            // 可以访问外部类的字段
            return "Inner method accessing outer field: " + ComprehensiveDemo.this.privateField;
        }
    }
}
```

## 2. `Reflect.java` 实战演练

接下来，我们编写 `UsageExample.java` 来演示如何使用 `Reflect.java` 与 `ComprehensiveDemo.java` 进行交互。

```java
import com.lody.virtual.helper.utils.Reflect;
import java.lang.reflect.Constructor;

public class UsageExample {

    public static void main(String[] args) {
        final String CLASS_NAME = "com.example.reflect.ComprehensiveDemo";

        System.out.println("--- Reflect.java 综合使用教程 ---");

        // === 1. 类加载 ===
        System.out.println("\n=== 1. 加载类 ====");
        // 使用 on() 方法，传入完整的类名
        Reflect demoClass = Reflect.on(CLASS_NAME);
        System.out.println("成功加载类: " + demoClass.get()); // .get() 返回被包装的原始对象

        // === 2. 静态字段操作 ===
        System.out.println("\n=== 2. 静态字段操作 ====");
        // 读取私有静态字段
        String privateStaticFieldValue = demoClass.get("privateStaticField");
        System.out.println("读取私有静态字段: " + privateStaticFieldValue);
        // 修改私有静态字段
        demoClass.set("privateStaticField", "Modified Private Static Field");
        System.out.println("修改后私有静态字段: " + demoClass.get("privateStaticField"));

        // === 3. 静态方法调用 ===
        System.out.println("\n=== 3. 静态方法调用 ====");
        // 调用公共静态方法
        String pubStaticResult = demoClass.call("publicStaticMethod").get();
        System.out.println("调用公共静态方法: " + pubStaticResult);
        // 调用私有静态方法（带参数）
        String privStaticResult = demoClass.call("privateStaticMethod", "Gemini").get();
        System.out.println("调用私有静态方法: " + privStaticResult);

        // === 4. 构造器与实例创建 ===
        System.out.println("\n=== 4. 构造器与实例创建 ====");
        // 4.1 使用公共无参构造器
        Reflect instance1 = demoClass.call("new"); // "new" 是库定义的关键字
        System.out.println("使用公共构造器创建实例: " + instance1.get());
        
        // 4.2 使用私有构造器
        // 对于私有或特定参数的构造器，需要先获取Constructor对象
        try {
            Constructor<?> privateConstructor = demoClass.type().getDeclaredConstructor(String.class, String.class);
            // 使用 on() 包装 Constructor 对象，再调用 call("newInstance", ...)
            Reflect instance2 = Reflect.on(privateConstructor).call("newInstance", "private_val", "public_val");
            System.out.println("使用私有构造器创建实例: " + instance2.get());
            System.out.println("  - 实例的私有字段值为: " + instance2.get("privateField"));
        } catch (Exception e) {
            e.printStackTrace();
        }

        // === 5. 实例字段操作 ===
        System.out.println("\n=== 5. 实例字段操作 ====");
        System.out.println("实例原始私有字段: " + instance1.get("privateField"));
        instance1.set("privateField", "Modified by reflection");
        System.out.println("实例修改后私有字段: " + instance1.get("privateField"));
        System.out.println("读取final字段: " + instance1.get("finalField"));

        // === 6. 实例方法调用 ===
        System.out.println("\n=== 6. 实例方法调用 ====");
        // 调用公共方法
        instance1.call("publicMethod");
        // 调用私有方法 (带可变参数)
        String privateMethodResult = instance1.call("privateMethod", 1, 2, 3).get();
        System.out.println("调用私有方法返回: " + privateMethodResult);
        // 调用重载方法
        System.out.print("调用重载方法(无参): ");
        instance1.call("overloadedMethod");
        System.out.print("调用重载方法(有参): ");
        instance1.call("overloadedMethod", "with parameter");

        // === 7. 静态嵌套类操作 ===
        System.out.println("\n=== 7. 静态嵌套类操作 ====");
        final String NESTED_CLASS_NAME = "com.example.reflect.ComprehensiveDemo$StaticNestedClass";
        Reflect nestedClass = Reflect.on(NESTED_CLASS_NAME);
        System.out.println("加载静态嵌套类: " + nestedClass.get());
        // 访问嵌套类的静态字段
        System.out.println("嵌套类静态字段: " + nestedClass.get("nestedStaticField"));
        // 调用嵌套类的静态方法
        System.out.println("调用嵌套类静态方法: " + nestedClass.call("nestedStaticMethod").get());
        // 创建嵌套类的实例
        Reflect nestedInstance = nestedClass.call("new");
        System.out.println("创建嵌套类实例: " + nestedInstance.get());
        // 调用嵌套类的实例方法
        System.out.println("调用嵌套类实例方法: " + nestedInstance.call("nestedInstanceMethod").get());

        // === 8. 非静态内部类操作 ===
        System.out.println("\n=== 8. 非静态内部类操作 ====");
        final String INNER_CLASS_NAME = "com.example.reflect.ComprehensiveDemo$InnerClass";
        // 非静态内部类比较特殊：它的构造器需要一个外部类的实例。
        try {
            // 1. 获取内部类的Class对象
            Class<?> innerClassType = Class.forName(INNER_CLASS_NAME);
            // 2. 获取它的构造器，第一个参数是外部类的类型
            Constructor<?> innerConstructor = innerClassType.getDeclaredConstructor(demoClass.type());
            // 3. 创建实例，将外部类实例(instance1.get())作为参数传入
            Reflect innerInstance = Reflect.on(innerConstructor).call("newInstance", instance1.get());
            System.out.println("创建内部类实例: " + innerInstance.get());
            // 访问内部类字段
            System.out.println("内部类字段: " + innerInstance.get("innerField"));
            // 调用内部类方法
            System.out.println("调用内部类方法: " + innerInstance.call("innerMethod").get());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 3. 核心API总结

- **`Reflect.on(String className)`**: 加载一个类，返回一个包装了`Class`对象的`Reflect`实例。这是所有静态操作的入口。
- **`Reflect.on(Object object)`**: 包装一个已有的对象，返回一个`Reflect`实例。这是所有实例操作的入口。
- **`.get(String fieldName)`**: 获取指定字段的值。可以是静态或实例字段。
- **`.set(String fieldName, Object value)`**: 设置指定字段的值。
- **`.call(String methodName, Object... args)`**: 调用指定名称的方法。可以是静态或实例方法。
- **`.call("new", Object... args)`**: 调用构造方法创建新实例。
- **`.get()`**: 从`Reflect`包装器中取出原始的`Object`、`Class`或方法返回值。
- **`.type()`**: 获取被包装对象的`Class`类型。

希望这份详尽的指南能帮助您更好地理解和使用`Reflect.java`！
