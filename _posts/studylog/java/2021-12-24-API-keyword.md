---
layout: post
title: 카카오 책 검색 API 사용
slug: 카카오 책 검색 API 사용
categories: studylog
tags: Java, Springboot, URLEncoder, 카카오API
description: >
  Java
---

# [프로젝트] 카카오 책 검색 API 사용

카카오 책 검색 API를 이용하여 도서 정보를 받아오는 Service를 만들다가 문제가 발생했다. 응답코드는 200으로 API 호출은 잘 된 것 같은데, response가 빈 값으로 들어왔다. 


      
- 내가 기대했던 response

    ![](https://images.velog.io/images/helloimlily/post/7a7a950a-632a-473b-a879-9ff22ce6b230/Untitled.png)

    Postman 이용, '김영하' 키워드로 책 검색 API 테스트 한 결과
    



- 내가 받은 response

    ![](https://images.velog.io/images/helloimlily/post/bc61682c-a2d7-4b96-bf6b-f3ca591b3de5/Untitled2.png)

응답 코드가 200인걸 보면 호출 방법이 잘못된 것 같진 않은데.. 하고 고민하다가 내가 넘겨주는 키워드에 문제가 있는게 아닐까하는 생각이 들었다. 

   

> *키워드가 한글이라서 그런가???*
   
   

한글로 파라미터를 넘겨주는 경우 내가 사용하고자 하는 값이 정확하게 표현되지 못하는 경우가 있다. 그래서 `StringBuilder`로 API 호출을 위한 url을 만들어주기 전, 먼저 `URLEncoder`를 이용해 파라미터로 넘겨줄 키워드를 UTF-8 형식으로 인코딩을 해주었다. 


```java
private void getBook(String keyword) throws IOException {

        try {

            String encodedKeyword = URLEncoder.encode(keyword, "UTF-8");
	    // getBook 함수를 호출하며 넘겨준 keyword를 인코딩

            StringBuilder builder = new StringBuilder();
            builder.append("https://dapi.kakao.com/v3/search/book")
                    .append("?query=")
                    .append(encodedKeyword);

            URL url = new URL(builder.toString());

..... 중략

        } catch (Exception e) {
            System.out.println(e);
        }

    }
```

키워드를 인코딩 해주었더니 내가 원하던 결과를 얻을 수 있었다!! 

![](https://images.velog.io/images/helloimlily/post/404ee651-ba33-48e1-9fb3-16b66ff19b12/Untitled3.png)