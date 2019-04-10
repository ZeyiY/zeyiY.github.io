---
title: Java关于空指针的防范与思考
date: 2019-04-09 08:37:05
tags:
---

# Java关于空指针的防范与思考

转自<https://juejin.im/post/5b9cab616fb9a05d30174491>

### 空指针分析

对于空指针的出现，其实一般可以归纳为以下几个原因:

- 对于方法入参没有严格校验
- 对于方法返回值没有严格的校验

本质原因：

对于调用的其他API没有充分的了解，使用时不知道API方法的入参是否可以接受null，不确定方法的返回值是否为null。

###空指针避免方法

使调用者明确知道方法的入参是否可以接受null，方法的返回值是否可以返回null。

##### 返回值空指针避免

1. 使用Optional

- Java8前使用 Google Guava的Optional
- Java8 引入了Optional

Optional的选择和使用比较简单。但是Optional并不适用于方法入参。

```
// Java8
public static Optional<String> valueOf(Integer number) {
    if (number == null) {
        return Optional.empty();
    }
    String str = String.valueOf(number);
    return Optional.of(number);
}
```



2. 标注注解

仅仅靠Optional是不够的。方法的返回类型不是Optional不能说明方法是否会返回null，因此需要一种更明确的方式。

##### 方法入参控制值避免

1. 标注注解

### 注解的使用

1. 常用的注解：

findbugs 

- edu.umd.cs.findbugs.annotations.NonNull
- edu.umd.cs.findbugs.annotations.Nullable

jsr305 

- javax.annotation.Nonnull
- javax.annotation.Nullable

spring-core 

- org.springframework.lang.NonNull
- org.springframework.lang.Nullable

javax-validator 

- javax.validation.constraints.NotNull
- javax.validation.constraints.Null

android-support 

- android.support.annotation.NonNull
- android.support.annotation.Nullable

eclipse-jdt 

- org.eclipse.jdt.annotation.NonNull
- org.eclipse.jdt.annotation.Nullable

jetbrains-annotations 

- org.jetbrains.annotations.NotNull
- org.jetbrains.annotations.Nullable

lombok 

- lombok.NonNull

rt.jar 

- com.sun.istack.internal.NotNull
- com.sun.istack.internal.Nullable

2. 选择因素

注解完备性 

- 必须同时支持null注解与非null注解，如果只支持其中一个那么使用场景将会受到很大限制

ide代码检查 

- 在ide中运行的时候如果能够对标注非null的参数和返回值进行校验，那么么将在很大程度上避免空指针

- 校验逻辑大致如下:

  ```
  public static void display(@Nonnull String str) {
      if (str == null) {
          throw new IllegalArgumentException();
      }
      // do something
  }
  复制代码
  ```

ide注解生成 

- 继承父类的方法，是否可以直接继承标注在方法参数和返回类型上的注解，这个特性是很重要的，因为在大型软件系统的中都是采用分层架构，层与层之间进行调用都是通过接口，因此不支持这个特性将会导致开发人员手动在子类方法入参和返回类型上标注注解，这无疑会大大增加工作量。

ide智能提示 

- 会对潜在产生空指针的变量进行高亮显示

findbugs支持 

- 一般的公司都会要求开发人员在ide上安装findbugs，用以扫描代码分析潜在的bug

sonar支持 

- 大型公司都会对代码进行静态分析，一般使用SonarCube，SonarCube可以继承fingbugs和pmd的校验规则，因此支持fingdbugs可以在一定程度上说明也支持Sonar

3. 各类注解支持情况

| 注解支持库            | 空注解   | 非空注解  | findbugs支持 | sonar支持 | ide运行时检查 | ide智能提示 | ide代码生成 |
| --------------------- | -------- | --------- | ------------ | --------- | ------------- | ----------- | ----------- |
| findbugs              | @NonNull | @Nullable | 支持         | 支持      | 支持          | 支持        | 支持        |
| jsr305                | @Nonnull | @Nullable | 支持         | 支持      | 支持          | 支持        | 支持        |
| spring-core           | @NonNull | @Nullable | 不支持       | 不支持    | 不支持        | 支持        | 不支持      |
| javax-validator       | @NotNull | @Null     | 不支持       | 不支持    | 不支持        | 不支持      | 支持        |
| android-support       | @NonNull | @Nullable | 不支持       | 不支持    | 支持          | 支持        | 支持        |
| eclipse-jdt           | @NonNull | @Nullable | 不支持       | 不支持    | 不支持        | 支持        | 支持        |
| jetbrains-annotations | @NotNull | @Nullable | 不支持       | 不支持    | 支持          | 支持        | 支持        |
| lombok                | @NonNull | 不支持    | 不支持       | 不支持    | 不支持        | 不支持      | 不支持      |
| rt.jar                | @NotNull | @Nullable | 不支持       | 不支持    | 不支持        | 不支持      | 不支持      |

######注意

测试使用的ide是Idea，Eclipse存在一定的差异

rt.jar @NotNull @Nullable属于sun的内部包，不要使用，如果有代码检查则不会被允许通过

eclipse-jdt和jetbrains-annotations和对应的ide绑定比较紧密不要轻易使用

javax-validator和lombok主要是运行时的参数检查

fingbugs原生的注解已经不再推荐，推荐使用jsr305的注解

#####结论

使用jsr305的注解

基本类型的入参和返回值是不需要标注@Nonnull和@Nullable注解的；

private方法，package方法，protected方法也是不需要标注的；

public方法上的非基本类型参数和返回值需要标注。

### 示例

* maven依赖

```
<dependency>
    <groupId>com.google.code.findbugs</groupId>
    <artifactId>jsr305</artifactId>
    <version>3.0.2</version>
</dependency>
```

* 使用示例

```
// 非空注解
@Nonnull
public Integer add(@Nonnull Integer number1, @Nonnull Integer number2) {
    Assert.notNull(number1, "number1 must not be null");
    Assert.notNull(number2, "number2 must not be null");
    return numnber1 + number2;
}

// 空注解
public static boolean isBlank(@Nullable String str) {
    return str == null || str.trim().length() == 0;
}

// Optional
@Nonnull
public static Optional<Integer> parseInte(@Nullable String str) {
    if (str == null) {
        return Optional.empty();
    }
    return Optional.of(Integer.parseInt(str));
}
```

* Idea jsr305注解配置

![Idea jsr305注解配置](https://user-gold-cdn.xitu.io/2018/9/15/165dbfd6bbbade6c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)