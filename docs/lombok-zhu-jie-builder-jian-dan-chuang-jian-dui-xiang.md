# lombok注解@Builder简单创建对象

0x04：`Java基础`

<!-- more -->

**不废话注解开始：**

## **一、 导入依赖**

``` xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.16.18</version>
	<scope>provided</scope>
</dependency>
```

## **二、创建entity**

```java
import lombok.Builder;
import lombok.Data;

/**
 * @Description //TODO .
 * @Author by LiuChunming on 2020/01/20.
 */
@Builder
@Data
public class Tper {
    private String id;
    private String name;
    private String age;
    private String sex;
    private String hobby;
    private double aDouble;
}
```

> **并添加注解 @Builder @Data**

## **三、开始测试**

```java
@Test
public void builderInterfaceTest() {
    //创建对象
    Tper tper = Tper.builder ().
            name ( "张三" ).
            age ( "19" ).
            sex ( "男" ).
            hobby ( "Code" ).
            aDouble ( 8D ).
            build ();
    System.out.println ( tper );
}
```

- **测试结果：**

​![测试结果](post-images\1579484887683.jpg)

> **Share Over !**