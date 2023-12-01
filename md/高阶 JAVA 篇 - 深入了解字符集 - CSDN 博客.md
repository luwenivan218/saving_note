> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Tingfeng__/article/details/134141073?spm=1001.2100.3001.7377&utm_medium=distribute.pc_feed_blog_category.none-task-blog-classify_tag-2-134141073-null-null.nonecase&depth_1-utm_source=distribute.pc_feed_blog_category.none-task-blog-classify_tag-2-134141073-null-null.nonecase)

> 文章浏览阅读1.5k次，点赞66次，收藏58次。常见的编码和解码算法包括ASCII、UTF-8、UTF-16、GBK等。不同的编码和解码算法适用于不同的字符集和应用场景，选择合适的编码和解码算法可以确保字符的正确传输和处理字符集的编码和解码是将字符转换为对应的编码值，或将编码值转换为对应的字符的过程。编码是将字符转换为对应的编码值的过程。在编码过程中，需要使用字符集中定义的编码规则，将每个字符映射到对应的编码值。不同的字符集使用不同的编码规则，例如ASCII字符集使用7位二进制数表示每个字符，UTF-8字符集使用1到4个字节表示不同的字符。

> **🔥博客主页： **[小扳_-CSDN 博客](https://blog.csdn.net/Tingfeng__?spm=1000.2115.3001.5343 "小扳_-CSDN博客")**  
> ❤感谢大家点赞👍收藏⭐评论✍** 

![](https://img-blog.csdnimg.cn/e6e3863ecfc7449c99c5e40bb87a84cf.jpeg) 

![](https://img-blog.csdnimg.cn/876ba8313fe244f89ef10faeebf20ed8.gif)

**文章目录**

 **[1.0 字符集的说明](#t0)**

 **[1.1 ASCII  字符集](#t1)**

 **[1.2 GBK 字符集](#t2)**

 **[1.3 UTF-8 字符集](#t3)**

 **[2.0 字符集的编码与解码](#t4)**

 **[2.1 编码提供了常见的方法](#t5)**

 **[2.2 解码提供了常见的方法](#t6)**

       1.0 [字符集](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E9%9B%86&spm=1001.2101.3001.7020)的说明
--------------------------------------------------------------------------------------------------------

        **字符集是一组字符的集合，字符集包括所有可用的字符，包括字母、数字、标点符号、特殊字符和控制字符等。常见的字符集有 ASCII 字符集、UTF-8 字符集、 GBK 字符集等。**

###         1.1 ASCII  字符集

        **它由 128 个字符组成，包括大写和小写字母、数字、标点符号、特殊字符和控制字符等。ASCII 字符集使用 7 位二进制数表示每个字符，范围从 0 到 127。**

 **只用一个字节大小的容量来 “装” 下这些每一个英文、数字、符号，需要注意的是首位必须为 0。**

###         1.2 GBK 字符集

        **GBK 字符集广泛用于中文环境下的计算机系统和软件，包括操作系统、文字处理软件、网页等。GBK 字符集是一种中文字符集，是在  ASCII  字符集基础上或者兼容的前提下，进行扩展的。**

 **需要重点了解的是，每一个英文和数字、字符都是可以用一个字节大小的容量来 “装” 下，首位必须为 0，对于中文汉字来说，需要每一个字符需要两个字节大小的容量来 “装” 下这超过 21000 个汉字和符号，包括繁体字、生僻字和部分其他语种的字符。需要重点注意的是，首位必须为 1。**

###         1.3 UTF-8 字符集

        **UTF-8 字符集是一种全球通用的字符编码标准，它包含了几乎所有已知的字符，涵盖了世界上所有的语言和符号。UTF-8 字符集的目标是为每个字符提供一个唯一的编码，以便在不同的计算机系统和软件中进行字符的交换和处理。**

 **UTF-8 是一种变长编码方案，使用 1 到 4 个字节来表示不同的字符，适用于在互联网上传输和存储文本数据。每个中文的汉字、字符等占三个字节，每个英文、数字、符号等占一个字节，在编码或者解码为了区分这些字符不混淆，就会有一定的规则。占一个字节的，首位必须为 0；占两个字节的，第一个字节首位三个必须为 110，第二个字节的首位两个必须为 10；占三个字节的，首位四个必须为 1110，第二个字节首位两个为 10，第三个字节首位两个也为 10；占四个字节的，首位五个必须为 11110，第二个字节首位两个为 10，第三个字节首位两个也为 10，第四个字节首位两个也为 10。**

 **小结一下：**

> ![](https://img-blog.csdnimg.cn/d5eb935070c04f0f8847650cd77962fe.png)

        2.0 字符集的编码与解码
---------------------

        **字符集的编码和解码是将字符转换为对应的编码值，或将编码值转换为对应的字符的过程。简单地来说，编码就是将字符转变为编号，这里的编号就是字符集中对应的编码值，而解码就是逆过程，将编号转变为字符。**

###         2.1 编码提供了常见的方法

        **使用 getBytes() ：默认系统提供的编码集。**

 **使用 getBytes(String  charsetName) ：选择自己想要的编码集。**

**代码如下：**

> ```
> import java.util.Arrays;
>  
> public class characterSet {
>     public static void main(String[] args) throws Exception{
>         //编码
>         String name = new String("a我b");
>         //默认系统的提供的字符集进行编码
>         byte[] num = name.getBytes();
>         System.out.println(Arrays.toString(num));
>  
>         //自选的字符集进行编码
>         byte[] num1 = name.getBytes("GBK");
>         System.out.println(Arrays.toString(num1));
>     }
> }
> ```
> 
> **运行结果如下：**
> 
> ![](https://img-blog.csdnimg.cn/e40cd78d41094d1cae7d9b943f347927.png)

###         2.2 解码提供了常见的方法

        **就是用字符串类的构造器，将字节数组放到有参构造器中，就可以得到了相应的字符串了。**

 **使用：String pass = new String(byte bytes[])，默认系统提供的编码集。**

 **使用：String pass = new String(byte bytes[]， String charsetName)，自选编码集。**

**代码如下：**

> ```
> import java.util.Arrays;
>  
> public class characterSet {
>     public static void main(String[] args) throws Exception{
>         //编码
>         String name = new String("a我b");
>         //默认系统的提供的字符集进行编码
>         byte[] num = name.getBytes();
>         System.out.println(Arrays.toString(num));
>  
>         //自选的字符集进行编码
>         byte[] num1 = name.getBytes("GBK");
>         System.out.println(Arrays.toString(num1));
>  
>         //解码
>         //用系统提供的默认字符集
>         String pass = new String(num);
>         System.out.println(pass);
>         //自己选用想要的字符集
>         String pass1 = new String(num1,"GBk");
>         System.out.println(pass1);
>     }
> }
> ```
> 
> **运行结果如下：**
> 
> ![](https://img-blog.csdnimg.cn/6f576ebc3f094b799b49edb0dd78e4f4.png)

        **需要重点注意的是，使用了某一套字符集进行编码，那么必须要使用跟编码使用的相同的一套字符集进行解码。**

**代码如下：**

> ```
> import java.io.UnsupportedEncodingException;
>  
> public class characterSet {
>     public static void main(String[] args) throws UnsupportedEncodingException {
>         String name = "李四";
>         //这里使用了 UTF-8 这一套字符集进行编码
>         byte[] passName = name.getBytes();
>         //如果使用 GBK 这一套字符集进行解码的时候会很很大问题
>         String newName = new String(passName,"GBK");
>         System.out.println(newName);
>     }
> }
> ```
> 
> **运行结果：**
> 
> ![](https://img-blog.csdnimg.cn/720a145c6e894ca1b47019c811917008.png)

        **这里就出现了我不认识的字了，总之，编码与解码都要使用同一套字符集，不然会出现问题。** 

![](https://img-blog.csdnimg.cn/d4340c78d11245b2acf88bacdaba00f4.gif)