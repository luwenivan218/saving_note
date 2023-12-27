> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1066055319)

> 引言 Base64 编码是一种用 64 个字符表示二进制数据的方法，它使用一组 64 个可打印字符来表示二进制数据，每 6 个比特位为一个单元，对应某个可打印字符。注意它并不是一种加密算法，所以 Base64 常用于在不

原

 2023-11-25  阅读 (91)  点赞 (0)

![](https://sike.skjava.com/java-features/202311251000001.png)

引言
--

Base64 编码是一种用 64 个字符表示二进制数据的方法，它使用一组 64 个可打印字符来表示二进制数据，每 6 个比特位为一个单元，对应某个可打印字符。**注意它并不是一种加密算法**，所以 Base64 常用于在不支持二进制数据的系统间传输二进制数据。

但是，Java 8 之前并不支持 Base64，我们需要依赖第三方库如`Apache Commons Codec`或者在 JDK 内部类`sun.misc.BASE64Encoder`和`sun.misc.BASE64Decoder`等不推荐使用的方式来实现 Base64 编码解码。

为了能够提供一个更加标准的、更加安全的方法来进行 Base64 的编码和解码操作，使得开发者们不再需要依赖外部库，Java 8 引入全新的 Base64，同时为了提供更好的性能，Java 为 Base64 的实现做了专门的性能优化。

Base64 的核心原理
------------

在了解 Java 8 中 Base64 API 之前，我们先看 Base64 的实现原理。

Base64 的核心思想是将数据流的每三个字节划分为一组，总共 24 位，再将这 24 位分为 4 组，每组 6 位。由于每组现在只有 6 位，因此它可以表示的最大数值是 `2^6 - 1 = 63`，Base64 编码正是利用这 64 个数字（从 0 到 63）对应到可打印字符的映射关系来工作的。

Base64 编码的步骤如下：

1.  **分组**：输入数据被分成每组 3 字节（24 位）。如果最后一组不足 3 字节，则用 0 填充至 3 字节。
2.  **映射**：每组 24 位被进一步划分为 4 个 6 位的小组。每个 6 位小组将被映射为一个 0-63 之间的数字。
3.  **编码表**：这些数字用作 Base64 编码表中的索引，该表由 64 个字符组成，包括大小写英文字母、数字和两个额外符号（通常是`+`和`/`），还有一个用于填充的`=`符号，以确保输出的字符数为 4 的倍数。
4.  **转换**：每个 6 位的分组对应的数字转换成相应的 Base64 字符。
5.  **填充**：如果原始数据字节长度不是 3 的倍数，最终的编码可能会用`=`字符填充至 4 的倍数长度，这样接收方在解码时能够恢复原始数据。

编码表如下：

<table><thead><tr><th>数值</th><th>字符</th><th>数值</th><th>字符</th><th>数值</th><th>字符</th><th>数值</th><th>字符</th></tr></thead><tbody><tr><td>0</td><td>A</td><td>16</td><td>Q</td><td>32</td><td>g</td><td>48</td><td>w</td></tr><tr><td>1</td><td>B</td><td>17</td><td>R</td><td>33</td><td>h</td><td>49</td><td>x</td></tr><tr><td>2</td><td>C</td><td>18</td><td>S</td><td>34</td><td>i</td><td>50</td><td>y</td></tr><tr><td>3</td><td>D</td><td>19</td><td>T</td><td>35</td><td>j</td><td>51</td><td>z</td></tr><tr><td>4</td><td>E</td><td>20</td><td>U</td><td>36</td><td>k</td><td>52</td><td>0</td></tr><tr><td>5</td><td>F</td><td>21</td><td>V</td><td>37</td><td>l</td><td>53</td><td>1</td></tr><tr><td>6</td><td>G</td><td>22</td><td>W</td><td>38</td><td>m</td><td>54</td><td>2</td></tr><tr><td>7</td><td>H</td><td>23</td><td>X</td><td>39</td><td>n</td><td>55</td><td>3</td></tr><tr><td>8</td><td>I</td><td>24</td><td>Y</td><td>40</td><td>o</td><td>56</td><td>4</td></tr><tr><td>9</td><td>J</td><td>25</td><td>Z</td><td>41</td><td>p</td><td>57</td><td>5</td></tr><tr><td>10</td><td>K</td><td>26</td><td>a</td><td>42</td><td>q</td><td>58</td><td>6</td></tr><tr><td>11</td><td>L</td><td>27</td><td>b</td><td>43</td><td>r</td><td>59</td><td>7</td></tr><tr><td>12</td><td>M</td><td>28</td><td>c</td><td>44</td><td>s</td><td>60</td><td>8</td></tr><tr><td>13</td><td>N</td><td>29</td><td>d</td><td>45</td><td>t</td><td>61</td><td>9</td></tr><tr><td>14</td><td>O</td><td>30</td><td>e</td><td>46</td><td>u</td><td>62</td><td>+</td></tr><tr><td>15</td><td>P</td><td>31</td><td>f</td><td>47</td><td>v</td><td>63</td><td>/</td></tr></tbody></table>

有了这个映射表我们就把任意的二进制转换成 Base64 的编码了，下面大明哥举个例子给大家演示下转换过程。我们将 `sikejava` 字符串转换为 Base64 编码

> **步骤 1：将字符转换为 ASCII 值**

将每个字符转换为对应一个 ASCII 值。

*   `s` -> `115`
*   `i` -> `105`
*   `k` -> `107`
*   `e` -> `101`
*   `j` -> `106`
*   `a` -> `97`
*   `v` -> `118`
*   `a` -> `97`

> **步骤 2：将 ASCII 值转换为二进制**

将每个 ASCII 值转换为 8 位二进制数。

*   `115` -> `01110011`
*   `105` -> `01101001`
*   `107` -> `01101011`
*   `101` -> `01100101`
*   `106` -> `01101010`
*   `97` -> `01100001`
*   `118` -> `01110110`
*   `97` -> `01100001`

串连起来就是：`01110011 01101001 01101011 01100101 01101010 01100001 01110110 01100001`

> **步骤 3：将二进制数据划分为 6 位一组**

将连续的二进制位分成 6 位一组的小块。如果最后一组不足 6 位，需要用 0 填充。

上面二进制分为 6 位一组：`011100 110110 100101 101011 011001 010110 101001 100001 011101 100110 000100`

> **步骤 4：将 6 位二进制数转换为十进制**

每组 6 位的二进制数将被转换成十进制数。

*   `011100` -> `28`
*   `110110` -> `54`
*   `100101` -> `37`
*   `101011` -> `43`
*   `011001` -> `25`
*   `010110` -> `22`
*   `101001` -> `41`
*   `100001` -> `33`
*   `011101` -> `29`
*   `100110` -> `38`
*   `000100` -> `4`

> **步骤 5：将十进制数映射到 Base64 字符**

*   `28` -> `c`
*   `54` -> `2`
*   `37` -> `l`
*   `43` -> `r`
*   `25` -> `Z`
*   `22` -> `W`
*   `41` -> `p`
*   `33` -> `h`
*   `29` -> `d`
*   `38` -> `m`
*   `4` -> `E`

所以，"`sikejava`" 对应的 Base64 编码是 `c2lrZWphdmE=`。最后的`=`符号用于填充，因为 Base64 编码的输出应该是 4 的倍数。

我们用代码验证下 ：

```
System.out.println(Base64.getEncoder().encodeToString("sikejava".getBytes()));

c2lrZWphdmE=



```

Java 8 中的 Base64 API
--------------------

Java 8 中的 Base64 API 提供了三种主要类型的 Base64 编码和解码，他们分别适用于不同的场景和需求。

> **基本 Base64 编码和解码**

*   **编码器**: 使用 `Base64.getEncoder()` 获取。
*   **解码器**: 使用 `Base64.getDecoder()` 获取。

它们提供了基本的 Base64 编码和解码功能，适用于所有需要 Base64 编码的场景。其特点是输出编码字符串不会包含用于换行的字符。

> **URL 和文件名安全 Base64 编码和解码**

*   **编码器**: 使用 `Base64.getUrlEncoder()` 获取。
*   **解码器**: 使用 `Base64.getUrlDecoder()` 获取。

该编解码器适用于 URL 和文件名的 Base64 编码。由于 URL 中的某些字符（如 `+` 和 `/`）有特殊含义，所以需要使用这种编码方式来替换这些字符。使用该编码器，编码输出中的 `+` 和 `/` 字符分别被替换为 `-` 和 `_`，使得编码后的字符串可以安全地用在 URL 和文件名中。

> **MIME 类型 Base64 编码和解码**

*   **编码器**: 使用 `Base64.getMimeEncoder()` 获取。
*   **解码器**: 使用 `Base64.getMimeDecoder()` 获取。

该编码器适用于 MIME 类型（如电子邮件）的内容，其中可能需要支持多行输出。特点是持按照 MIME 类型的要求将输出格式化为每行固定长度的多行字符串。默认每行长度不超过 76 个字符，并在每行后插入 `\r\n`。

Base64 提供了多种编解码的方法，[大明哥](https://www.skjava.com/series/article/www.skjava.com)这里列举几个最常用的：

*   `encode(byte[] src)`: 将给定的字节数组编码为 Base64 字符串。
*   `encodeToString(byte[] src)`: 将给定的字节数组编码为一个 Base64 字符串，并将其直接转换为字符串格式。
*   `decode(String src)`: 将给定的 Base64 字符串解码为字节数组。
*   `decode(byte[] src)`: 将给定的 Base64 编码的字节数组解码为原始字节数组。

Base64 示例
---------

### 基本 Base64 编码

基本编码是最常见的类型，适用于大多数需要 Base64 编码的场景。

```
    @Test
    public void base64BasicTest() {
        String skStr = "skjava";

        
        String encodingStr = Base64.getEncoder().encodeToString(skStr.getBytes());
        System.out.println("encodingStr = " + encodingStr);

        
        String decodeStr = new String(Base64.getDecoder().decode(encodingStr));
        System.out.println("decodeStr = " + decodeStr);
    }

encodingStr = c2tqYXZh
decodeStr = skjava



```

### URL 和文件名安全 Base64 编码

URL 和文件名安全编码会替换掉一些在 URL 中可能会引起问题的字符，比如 `+` 和 `/`。

```
    @Test
    public void base64UrlTest() {
        String skStr = "https://skjava.com/?series=skjava";

        
        String encodingStr = Base64.getUrlEncoder().encodeToString(skStr.getBytes());
        System.out.println("encodingStr = " + encodingStr);

        
        String decodeStr = new String(Base64.getUrlDecoder().decode(encodingStr));
        System.out.println("decodeStr = " + decodeStr);
    }

encodingStr = aHR0cHM6Ly9za2phdmEuY29tLz9zZXJpZXM9c2tqYXZh
decodeStr = https:


```

### MIME 类型 Base64 编码

MIME 类型编码适用于电子邮件或其他 MIME 类型的内容，它支持多行输出。

```
    @Test
    public void base64MimeTest() {
        String skStr = "Hello, MIME-Type Example。\r\n" +
                "sike-java,sike-java-feature;" +
                "sike-javanio,sike-netty";

        
        String encodingStr = Base64.getMimeEncoder().encodeToString(skStr.getBytes());
        System.out.println("encodingStr = " + encodingStr);

        
        String decodeStr = new String(Base64.getMimeDecoder().decode(encodingStr));
        System.out.println("decodeStr = " + decodeStr);
    }

encodingStr = SGVsbG8sIE1JTUUtVHlwZSBFeGFtcGxl44CCDQpzaWtlLWphdmEsc2lrZS1qYXZhLWZlYXR1cmU7
c2lrZS1qYXZhbmlvLHNpa2UtbmV0dHk=
decodeStr = Hello, MIME-Type Example。
sike-java,sike-java-feature;sike-javanio,sike-netty


```