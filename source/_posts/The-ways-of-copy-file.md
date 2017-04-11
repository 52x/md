---
title: Java文件拷贝的几种实现方案
date: 2016-08-23 21:24:12
tags: [Java]
toc: true
categories: Java平台
---

在jdk1.7之前，java中没有直接的类提供文件复制功能。下面就文件复制，提出几种方案。

## jdk1.7中的文件复制

在jdk1.7版本中可以直接使用Files.copy(File srcFile, File destFile)方法即可。

```
private static void copyFileUsingJava7Files(File source, File dest)
throws IOException {
    Files.copy(source.toPath(), dest.toPath());
}
```

## 使用FileInputStream复制

<!--more-->

```
/**
 *
 * @Title: copyFileUsingStream
 * @Description: 使用Stream拷贝文件
 * @param: @param source
 * @param: @param dest
 * @param: @throws IOException
 * @return: void
 * @throws
 */
public static void copyFileUsingStream(File source, File dest)
throws IOException {
    InputStream is = null;
    OutputStream os = null;
    try {
        is = new FileInputStream(source);
        os = new FileOutputStream(dest);
        byte[] buffer = new byte[1024];
        int length;
        while ((length = is.read(buffer)) > 0) {
            os.write(buffer, 0, length);
        }
    } finally {
        if(is != null) {
            is.close();
        }
        if(os != null) {
            os.close();
        }
    }
}
```

## 使用FileChannel实现复制

```
/**
 * @throws IOException
 *
 * @Title: copyFileUsingChannel
 * @Description: 使用Channel拷贝文件
 * @param: @param source
 * @param: @param dest
 * @return: void
 * @throws
 */
public static void copyFileUsingChannel(File source, File dest) throws IOException {
    FileChannel sourceChannel = null;
    FileChannel destChannel = null;
    try {
        sourceChannel = new FileInputStream(source).getChannel();
        destChannel = new FileOutputStream(dest).getChannel();
        destChannel.transferFrom(sourceChannel, 0, sourceChannel.size());
    } finally {
        if(sourceChannel != null) {
            sourceChannel.close();
        }
        if(destChannel != null) {
            destChannel.close();
        }
    }
}
```

## 使用Apache Commons IO包中的FileUtils.copyFile复制

```
public static void copyFileUsingApacheIO(File source, File dest) throws IOException {
    FileUtils.copyFile(source, dest);
}
```

## 使用IO重定向实现复制

```
/**
 *
 * @Title: copyFileUsingRedirection
 * @Description: 使用Redirection拷贝文件
 * @param: @param source
 * @param: @param dest
 * @param: @throws IOException
 * @return: void
 * @throws
 */
public static void copyFileUsingRedirection(File source, File dest) throws IOException {
    FileInputStream in = null;
    PrintStream out = null;
    try {
        in = new FileInputStream(source);
        out = new PrintStream(dest);
        System.setIn(in);
        System.setOut(out);
        Scanner sc = new Scanner(System.in);
        while(sc.hasNext()) {
            System.out.println(sc.nextLine());
        }
    } finally{
        if(in != null) {
            in.close();
        }
        if(out != null) {
            out.close();
        }
    }
}
```
