---
title: 스파크 서버
categories: [java]
comments: true
---

`클라이언트: 파이썬 디스코드 봇` <-> `연결고리: 자바 스파크 서버` <-> `서버: 자바 형태소 분석`

기능: 디스코드 봇과 형태소 분석 코드 사이에 욕설 데이터를 주고받는 기능

포함된 오픈소스: sparkjava

1~3행: import, 패키지

7, 9행: 변수 선언

10행: sparkjava의 get기능을 사용해 로컬 웹페이지 구축 및 디스코드 로그 받아옴

13행: 반복문으로 로그 속의 단어들을 하나하나 형태소 분석

14~20행: 로그에서 욕설만 골라내서 CSV형태로 저장

24행: 저장된 CSV를 웹페이지의 body에 올림

자바 코드 전체
```python
package discord;

import static spark.Spark.get;

public class ConnectURL {
	static String body = "";
	public static void main(String[] args)  {
		Userdic a = new Userdic();	//욕설 감지 기능
		get("/:name", (request, response) -> {
			body = "";	//욕설 단어 CSV
			System.out.println("로그: " + request.params(":name"));
			for(String b : a.analyzer(request.params(":name"))) {	//로그을 파싱
				if(a.getAbuseList().contains(b)) {		//욕설이 있는지 체크
					if(body.equals("")) {	//이미 욕이 없었다면
						body = b;			
					}
					else {					//욕이 여러개라면
						body += "," + b;
					}
				}
			}
			System.out.println("욕설:" + body);
			return body;
		});
	}
}
```
