---
layout: post
title: "[Spring] MultipartFile을 이용한 파일 업로드"
date: 2020-01-02 23:42 +0900
categories: [Spring]
tags: [Spring]
redirect_from: 
- spring/spring-fileupload/
---

## 개요
- 스프링에서 제공하는 `MultipartFile` 이라는 인터페이스를 통해서 파일업로드 하는 방법에 대해서 알아보자.
- 예제는 MVC(thymleafe)를 통한, 간단한 파일 업로드이고, 자신의 홈디렉토리에 저장하는 것을 구현한다.


## 파일 업로드하는 FileService
```java
@Service
public class FileService {

    // application.properties 에 app.upload.dir을 정의하고, 없는 경우에 default 값으로 user.home (System에 종속적인)
    @Value("${app.upload.dir:${user.home}}")
    private String uploadDir;

    public void fileUpload(MultipartFile multipartFile) {
        // File.seperator 는 OS종속적이다.
        // Spring에서 제공하는 cleanPath()를 통해서 ../ 내부 점들에 대해서 사용을 억제한다
        Path copyOfLocation = Paths.get(uploadDir + File.separator + StringUtils.cleanPath(multipartFile.getOriginalFilename()));
        try {
            // inputStream을 가져와서
            // copyOfLocation (저장위치)로 파일을 쓴다.
            // copy의 옵션은 기존에 존재하면 REPLACE(대체한다), 오버라이딩 한다
            Files.copy(multipartFile.getInputStream(), copyOfLocation, StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            e.printStackTrace();
            throw new FileStorageException("Could not store file : " + multipartFile.getOriginalFilename());
        }

    }
}
```
스프링에서 제공하는 `MultipartFile` 이라는 인터페이스를 이용해서, HTTP multipart 요청을 처리한다. 
MultipartFile 요청은 큰 파일을 청크 단위로 쪼개서 효율적으로 파일 업로드 할 수 있게 해준다.
MultipartFile 인터페이스를 열어보면 다음과 같이 나와있다. 
> A representation of an uploaded file received in a multipart request.

이제, 다시 본로으로 돌아와서 실제 위의 작성한 코드를 살펴보면 
Paths.get() 메서드를 통해서 저장할 위치(copyOfLocation)을 지정한다. 여기서 중요한 것은 `File.seperator` 가 OS 종속적으로 mac이나 리눅스 계열에서는 `/` 윈도우에서는 `\` 구분자를 나타낸다. 또한, `StringUtils.cleanPath`를 사용한다. 


그리고, Files.copy()메서드를 통해서 multipartFile의 inputstream을 얻어서 위에서 지정한 copyOfLocation에 파일을 쓴다. 
정확히 Files.copy()메서드의 정의를 살펴보면 다음과 같다. 
>  Copies all bytes from an input stream to a file. On return, the input
>  stream will be at end of stream.
input stream으로 얻은 모든 bytes 들을 file로 copy한다.


## FileController
```java
@Controller
public class FileController {

    @Autowired
    private FileService fileService;

    @GetMapping
    public String index(){
        return "upload";
    }

    @PostMapping("/upload")
    public String fileUpload(@RequestParam("file") MultipartFile file, RedirectAttributes redirectAttributes) {
        fileService.fileUpload(file);

        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded " + file.getOriginalFilename() + "!");

        return "redirect:/";
    }
```


## upload.html 파일

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>File Upload Example</h1>
<form method="post" th:action="@{/upload}" enctype="multipart/form-data">
    <input type="file" name="file"/>
    <input type="submit" value="submit"/>
</form>
```
여기서는 form 태그에 input type을 file로 받아서, form을 submit하면 `FileController`에 정의했던 API를 타게 된다. 
중요한 건 input의 name를 `file` 로 한 부분이 `FileController` 의 `@ReuqestParam("file")` 이 부분과 일치해야 한다. 
