---
title: 스파크 서버
categories: [java]
comments: true
---

```python
package discord;
//192.168.0.100:4567/users/[로그]

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
