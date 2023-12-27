> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2000691792)

> 为了提高了文件处理的便利性和效率，Java12 新增方法 Files.mismatch(Path,Path)，该方法用于比较两个文件的内容，它返回两个文件内容第一次不匹配的位置索引。如果文件完全相同，则返

原

 2023-12-14  阅读 (54)  点赞 (0)

为了提高了文件处理的便利性和效率，Java 12 新增方法 `Files.mismatch(Path, Path)` ，该方法用于比较两个文件的内容，它返回两个文件内容第一次不匹配的位置索引。如果文件完全相同，则返回 `-1`。

我们新建两个文件，文件内容如下：

```
file_01.txt内容：https://skjava.com，一个专注死磕Java一站式 Java 学习平台，站长是大明哥。
file_02.txt内容：https://skjava.com，一个专注死磕 Java 一站式 Java 学习平台，站长是大明哥。



```

示例：

```
    @Test
    public void mismatchTest() throws IOException {
        Path path1 = Paths.get("/Users/chenssy/Downloads/file_01.txt");
        Path path2 = Paths.get("/Users/chenssy/Downloads/file_02.txt");

        long diffIndex = Files.mismatch(path1,path2);
        if (diffIndex == -1) {
            System.out.println("文件内容一致");
        } else {
            System.out.println("文件内容不一致，index：" + diffIndex);
        }
    }

文件内容不一致，index：39



```

该方法非常适用于文件内容比较的场景，尤其是在需要确认两个文件是否完全相同时。