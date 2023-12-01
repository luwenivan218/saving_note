> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/antgan/article/details/52103966)

> [Protobuf 协议](https://worktile.com/tech/share/prototol-buffers)，全称：Protocol [Buffer](https://so.csdn.net/so/search?q=Buffer&spm=1001.2101.3001.7020)  
> 它跟 JSON，XML 一样，是一个规定好的数据传播格式。不过，它的序列化和[反序列化](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96&spm=1001.2101.3001.7020)的效率太变态了……

来看看几张图你就知道它有多变态。  
![](https://img-blog.csdn.net/20160803143859081)

![](https://img-blog.csdn.net/20160803143926655)

![](https://img-blog.csdn.net/20160803143945754)

### Protobuf 的 Java 实例

#### 一、 安装 Protobuf

[去 Protobuf 的 GitHub 下载](https://github.com/google/protobuf)，解压。

如果你是 Windows 环境，则还要下载多一个东西。[protobuf-2.5.0-windows.zip](http://download.csdn.net/detail/antgan/9593735)。

解压 protobuf-2.5.0-windows.zip，把 protoc.exe 放在 Protobuf 安装目录下的 src 里。（其实放哪都可以）

#### 二、 配置环境变量

编辑系统变量 Path，添加 Protoc.exe 的存放目录。  
![](https://img-blog.csdn.net/20160803145119862)

#### 三、 Eclipse 新建项目

我使用 maven 构建 protobuf 项目，方便引入 **protobuf-java-2.5.0.jar** 依赖。  
在项目根目录创建 proto 文件夹，存放 **proto** 文件。  
![](https://img-blog.csdn.net/20160803150200947)  
maven 依赖 pom.xml

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>2.5.0</version>
</dependency>
```

#### 四、编写. proto 文件

在 proto 文件夹下编写 **person-entity.proto**，如下（[proto 协议的规则点这查看](https://worktile.com/tech/share/prototol-buffers)）

```
option java_outer_classname = "PersonEntity";//生成的数据访问类的类名  
message Person {  
  required int32 id = 1;//同上  
  required string name = 2;//必须字段，在后面的使用中必须为该段设置值  
  optional string email = 3;//可选字段，在后面的使用中可以自由决定是否为该字段设置值
}  
```

#### 四、使用 protoc.exe 编译成 java 类

有两种方法：  
1. 使用 Java Rumtime 执行 cmd 命令  
2. 直接打开 cmd 运行命令也行。

##### 1. 使用 Java Rumtime 执行 cmd 命令

**util 包下新建 GenerareClass 类**  
![](https://img-blog.csdn.net/20160803151005609)

```
/**
 * protoc.exe
 * @author ganhaibin
 *
 */
public class GenerateClass {
    public static void main(String[] args) {
        String protoFile = "person-entity.proto";//  
        String strCmd = "d:/dev/protobuf-master/src/protoc.exe -I=./proto --java_out=./src/main/java ./proto/"+ protoFile;  
        try {
            Runtime.getRuntime().exec(strCmd);
        } catch (IOException e) {
            e.printStackTrace();
        }//通过执行cmd命令调用protoc.exe程序  
    }
}
```

命令格式如下。

```
protoc.exe -I=proto的输入目录 --java_out=java类输出目录 proto的输入目录包括包括proto文件
```

##### 2. 直接打开 cmd 运行命令

![](https://img-blog.csdn.net/20160803145741085)

生成的 PersonEntity.java 类

![](https://img-blog.csdn.net/20160803145845617)

#### 五、测试

编写 Test 类，模拟序列化和反序列化过程。

```
public class Test {
    public static void main(String[] args) throws IOException {
        //模拟将对象转成byte[]，方便传输
        PersonEntity.Person.Builder builder = PersonEntity.Person.newBuilder();
        builder.setId(1);
        builder.setName("ant");
        builder.setEmail("ghb@soecode.com");
        PersonEntity.Person person = builder.build();
        System.out.println("before :"+ person.toString());

        System.out.println("===========Person Byte==========");
        for(byte b : person.toByteArray()){
            System.out.print(b);
        }
        System.out.println();
        System.out.println(person.toByteString());
        System.out.println("================================");

        //模拟接收Byte[]，反序列化成Person类
        byte[] byteArray =person.toByteArray();
        Person p2 = Person.parseFrom(byteArray);
        System.out.println("after :" +p2.toString());
    }
}
```

输出如下  
![](https://img-blog.csdn.net/20160803151628811)

### 后记

我想，拿 protobuf 协议储存数据，或者作为聊天文本的传输协议，那效率肯定让人咋舌。嘿嘿。