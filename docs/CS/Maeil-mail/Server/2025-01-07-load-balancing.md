---
title: 로드 밸런싱에 대해서 설명해주세요.
parent: 서버
nav_order: -2
---

<div style="text-align: center; display: flex;
    flex-direction: column;
    align-items: center;">
    <h5>오늘의 질문</h5>
    <div style="color: black; background-color: #F0F3F5; border-radius: 5px; width: 50%; padding: 20px;">
    로드 밸런싱에 대해서 설명해주세요.
    </div>
</div>

---

### 로드밸런싱이란 애플리케이션을 지원하는 리소스 풀에 들어오는 네트워크 트래픽(들어오는 요청)을 균등하게 분산하는 것을 말한다.

로드밸런서는 애플리케이션 서버 앞단에 위치하며, 클라이언트 요청을 지시하고 제어한다. 이를 통해 애플리케이션 가용성, 확장성, 보안 및 성능을 확보할 수 있다.

<br>

### 🚀 로드 밸런싱 알고리즘

> #### Round Robin (라운드 로빈) : 모든 요청이 순서대로 처리되는 방식

``` markdown
서버가 3대 (A,B,C) 있는 경우 ABCABC 순서대로 요청이 전달된다.
```

- 모든 서버의 처리 능력이 동등하고, 요청의 고른 분산이 중요한 경우 고려해볼 수 있다.

- 그러나 서버 부하나 응답 시간을 고려하지 않기 때문에 서버의 처리 능력이 다른 경우 비효율적이다.

<br>

> #### Weighted Round Robin (가중치 라운드 로빈) : 라운드 로빈 방식에 '가중치'라는 개념을 추가한 방식

``` markdown
각 서버의 처리 능력과 가용 자원에 따라 '가중치'를 부여한다.
'가중치'가 높은 서버는 가중치에 비례하여 상대적으로 더욱 많은 요청을 받게 된다.
```

- 라운드 로빈 방식보다 구현이 복잡하지만 각 서버의 처리 능력을 고려하는 방식으로, 라운드 로빈 방식의 단점을 개선한다.

- 다만, 여전히 서버의 상태를 고려하지 않는 방식이라는 점에 유의할 것.

<br>

> #### Least Connections (최소 연결) : 각 서버의 활성 연결 수를 모니터링하고 있는 경우 사용할 수 있는 방식

``` markdown
가장 적은 활성 연결이 존재하는 서버에 요청을 전달한다.
```

- 각 서버의 처리 능력이 다른 경우 유의해야 한다. 처리 능력이 큰 서버는 상대적으로 활성 연결을 더욱 많이 수립할 수 있기 때문이다. 처리 능력이 큰 서버의 capa 는 충분한데 반해 처리 능력이 작은 서버에 요청이 자주 들어갈 수 있으니 비효율적인 결과가 발생할 수 있다.

- 처리 능력이 비슷하지만 특정 이유로 한 서버에 동시 연결 수가 많아지는 상황이 발생하면 고려해볼 만 하다.

<br>

> #### Least Response Time (최소 응답 시간) : 각 서버의 응답 시간을 모니터링하고 있는 경우 사용할 수 있는 방식

``` markdown
응답 시간이 가장 빠른 서버에 요청을 전달한다.
```

- 빠른 응답이 가능하므로 사용자 경험을 개선할 수 있다.

- 응답 시간만으로 요청을 보내기 때문에 서버의 부하 상태, 활성 연결 상태를 고려해야 하는 경우에는 부적합할 수 있다.

<br>

> #### IP Hash : 클라이언트 요청의 IP 를 기반으로 요청을 전달하는 방식

``` markdown
IP 를 이용해 구한 해시값을 기반으로 요청할 서버를 결정한다.
```

- 클라이언트와 서버 간의 친화성 유지에 초점을 맞춘 방식으로, 클라이언트의 상태를 관리하는 데 용이하다.

- 상황에 따라서 부하가 균등하게 이루어지지 않는다는 단점이 있다.

<br>

### 🤓 로드밸런서를 도입하여 활용해본 적이 있나요 ?

NCBT 프로젝트를 진행하며 도입한 Apache 서버에 로드밸런싱을 적용해볼 수 있다. Apache 는 스프링 애플리케이션 서버의 앞단에 위치하여, 443 포트로 들어오는 HTTPS 요청을 받아 처리한다. 우리는 서비스하는 애플리케이션이 하나였기 때문에 '리버스 프록시' 로 해당 도메인에 대한 요청을 스프링 애플리케이션에 전달하는 방식으로 구현했지만, 아래와 같은 방식으로 로드밸런서를 구성해볼 수 있다.

> #### Apache mod_proxy_balancer 활성화

``` bash
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
```

> #### 서버 그룹 설정

``` bash
 <Proxy "balancer://myapp">
    BalancerMember http://<Tomcat 서버1 IP>:8080
    BalancerMember http://<Tomcat 서버2 IP>:8080
    # 각 Tomcat 서버의 IP와 포트를 지정
    ProxySet lbmethod=byrequests
</Proxy>
```

> #### Apache 재시작

``` bash
sudo systemctl restart apache2
```

<br>

🔖 참고자료

[매일메일](https://www.maeil-mail.kr/question/115)

[아파치 로드밸런싱 설정](https://velog.io/@jhwan211/%EB%A6%AC%EB%B2%84%EC%8A%A4-%ED%94%84%EB%A1%9D%EC%8B%9C-%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1-%EC%84%A4%EC%A0%95)

