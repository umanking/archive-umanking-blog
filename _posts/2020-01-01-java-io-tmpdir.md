---
layout: post
title: "[Java] java.io.tmpdir 경로가 존재하지 않을 때"
date: 2020-01-01 14:15 +0900
categories: [Java]
tags: [Java]
redirect_from: 
- java/java-io-tmpdir/
---
java에서 `System.getProperties()`를 찍어보면, 현재 사용중인 시스템에 대한 정보들이 나온다. 이 중에서 java.io.tmpdir인 임시디렉토리에 대해서 알아보자.

## java.io.tmpdir 경로 가지고 오기
System의 getProperty("키")를 통해서 해당 디렉토리를 가지고 온다. 
```java
System.out.println(System.getProperty("java.io.tmpdir"));
```
보틍 가지고 오는 경로는 `/var/tmp/` `/tmp` 이렇게 시작한다(unix 기반이라면)

## java.io.tmpdir 경로 셋팅하기
`java -Djava.io.tmpdir=/home/tmp/` 이런 식으로 argument로 넘겨서 임시 디렉토리를 지정할 수 있고
`System.setPropety("java.io.tmp", "/home/tmp");` 이렇게 셋팅할 수도 있다. 


## 실제 소스 코드 상에서
사실은 이것 때문에 포스팅을 작성하게 되었다. 실제 파일 업로드 하는 코드에서 저 임시 디렉토리 경로를 통해서 파일경로를 만들고, 
Multipart 를 File 로 변환하는 transferTo()메서드를 사용했는데, 저 부분에서 IO exception 이 발생했던 것이다. 

```java
    public File multipartToFile(MultipartFile multipart) throws IOException {
        File tmpFile = new File(System.getProperty("java.io.tmpdir") + System.getProperty("file.separator") + multipart.getOriginalFilename());
        multipart.transferTo(tmpFile);
        return tmpFile;
    }
```


기존의 소스 코드를 보면 해당 tmpFile 경로가 반드시 존재할까? 
그렇지 않다. (정확히는 재현은 못해봤지만 java.io.tmpdir 경로가 존재하지 않는 경우가 있는 듯 보였다)
그렇기 때문에 다음과 같이 tmpFile경로의 존재 여부를 체크하고, 없는 경우에 directory를 만들어 주었다.
```java
    public File multipartToFile(MultipartFile multipart) throws IOException {
        File tmpFile = new File(System.getProperty("java.io.tmpdir") + System.getProperty("file.separator") + multipart.getOriginalFilename());
        if (!tmpFile.exists()) {
            tmpFile.mkdir();
        }
        multipart.transferTo(tmpFile);
        return tmpFile;
    }
```