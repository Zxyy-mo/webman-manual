# 스트레스 테스트

## 스트레스 테스트 결과에는 어떤 요소가 영향을 미치나요?
* 스트레스 테스트 머신과 서버 간 네트워크 지연 (내부 네트워크 또는 로컬 테스트를 권장)
* 스트레스 테스트 머신과 서버 간 대역폭 (내부 네트워크 또는 로컬 테스트를 권장)
* HTTP keep-alive 여부 (활성화 권장)
* 충분한 동시 요청 수 (외부에서 최대한 많은 동시 요청을 활성화시키는 것이 좋음)
* 서버 프로세스 수가 적절한지 (helloworld 비즈니스의 경우 CPU 수와 같은 프로세스 수를 권장하며, 데이터베이스 비즈니스의 경우 CPU의 네 배 이상을 권장)
* 비즈니스 자체의 성능 (예: 외부 데이터베이스 사용 여부)

## HTTP keep-alive는 무엇인가요?
HTTP Keep-Alive 메커니즘은 단일 TCP 연결을 통해 여러 HTTP 요청과 응답을 보내는 기술로, 성능 테스트 결과에 큰 영향을 미치며, keep-alive를 비활성화하면 QPS가 크게 감소할 수 있습니다.
현재 대부분의 브라우저는 기본적으로 keep-alive를 활성화하며, 즉 브라우저가 특정 http 주소에 대한 연결을 닫지 않고 유지하여 다음 요청에서 이 연결을 재사용하여 성능을 향상시킵니다.
스트레스 테스트시 keep-alive를 활성화하는 것이 좋습니다.

## HTTP keep-alive를 활성화하는 방법은 무엇인가요?
ab 프로그램을 사용하여 스트레스 테스트하는 경우 -k 매개 변수를 추가해야 합니다. 예를 들면 `ab -n100000 -c200 -k http://127.0.0.1:8787/`와 같이 사용합니다.
apipost는 gzip 헤더를 반환해야 keep-alive를 활성화할 수 있습니다 (apipost의 버그 참조).
다른 스트레스 테스트 프로그램은 일반적으로 기본적으로 활성화되어 있습니다.

## 외부 네트워크로 스트레스 테스트할 때 왜 QPS가 매우 낮을까요?
외부 네트워크 지연으로 인해 QPS가 매우 낮아지는 것은 정상적인 현상입니다. 예를 들어 baidu 페이지를 스트레스 테스트하면 QPS가 몇십 개 정도일 수 있습니다.
내부 네트워크 또는 로컬 테스트를 권장하며 네트워크 지연의 영향을 배제해야 합니다.
외부 스트레스 테스트를 수행해야 하는 경우 대역폭을 충분히 확보하기 위해 동시 요청 수를 증가시키는 것이 좋습니다.

## Nginx 프록시를 거친 후 왜 성능이 저하될까요?
Nginx 실행에는 시스템 리소스가 필요합니다. 동시에 Nginx와 webman 간의 통신도 일정한 리소스를 소비합니다.
그러나 시스템 리소스는 한정되어 있으므로 webman은 모든 시스템 리소스에 접근할 수 없으므로 전체 시스템의 성능이 약간 저하될 수 있습니다.
Nginx 프록시로 인한 성능 영향을 최소화하기 위해서는 Nginx 로그를 비활성화하고(`access_log off;`) Nginx와 webman 간의 keep-alive를 활성화하는 것이 좋습니다(nginx 프록시 참조).
또한 https는 http보다 더 많은 리소스를 사용하며 SSL/TLS 핸드셰이크, 데이터 암호화 및 패킷 크기 증가로 인해 성능이 저하됩니다. 스트레스 테스트 시 단일 연결을 사용하는 경우(keep-alive 비활성화), 매번 요청에 대해 추가적인 SSL/TLS 핸드셰이크 통신이 필요하여 성능이 크게 감소할 수 있습니다. https는 keep-alive를 활성화하는 것이 좋습니다.

## 시스템이 성능 한계에 도달했는지 어떻게 알 수 있나요?
일반적으로 CPU가 100%인 경우 시스템 성능이 한계에 도달했다고 볼 수 있습니다. CPU에 여유가 있다면 아직 한계에 도달하지 않은 것입니다. 이때 동시 요청 수를 증가시켜 QPS를 높일 수 있습니다.
동시 요청 수를 증가시켜도 QPS가 향상되지 않으면 webman 프로세스 수가 충분하지 않은 것일 수 있으며, 이 경우 webman 프로세스 수를 증가시킬 필요가 있습니다. 그래도 성능이 향상되지 않으면 대역폭이 충분한지 확인해야 합니다.

## 왜 내 스트레스 테스트 결과에서 webman의 성능이 go의 gin 프레임워크보다 낮게 나올까요?
[techempower](https://www.techempower.com/benchmarks/#section=data-r21&hw=ph&test=db&l=zijnjz-6bj&a=2&f=1ekg-cbcw-2t4w-27wr68-pc0-iv9slc-0-1ekgw-39g-kxs00-o0zk-5jsetl-2x8doc-2)의 스트레스 테스트 결과에 따르면 webman은 플레인 텍스트, 데이터베이스 쿼리 등 모든 기준에서 gin의 대략 두 배의 성능을 보입니다.
만약 여러분의 결과가 다르다면 webman에서 ORM을 사용하여 성능 저하를 초래했을 수 있으며, webman+원시 PDO와 gin+원시 SQL을 비교해볼 수 있습니다.

## ORM을 사용한 webman의 성능이 얼마나 감소할까요?
다음은 한 세트의 성능 테스트 데이터입니다.

**환경**
알리바바 클라우드 4코어 4G, 10만 개의 레코드에서 무작위로 1개의 데이터를 조회하여 JSON으로 반환합니다.

**원시 PDO를 사용하는 경우**
webman의 QPS는 1.78만입니다.

**laravel의 Db::table()을 사용하는 경우**
webman의 QPS는 0.94만으로 감소합니다.

**laravel의 Model을 사용하는 경우**
webman의 QPS는 0.72만으로 감소합니다.

thinkORM의 결과는 유사하며 큰 차이가 없습니다.

> **팁**
> ORM을 사용하면 성능이 감소하지만 대부분의 비즈니스에는 충분합니다. 개발 효율성, 유지보수성, 성능 등 여러 가지 지표를 고려하여 균형을 찾아야 합니다.

## 왜 apipost를 사용하여 스트레스 테스트하면 QPS가 매우 낮을까요?
apipost의 스트레스 테스트 모듈에 버그가 있어 서버가 gzip 헤더를 반환하지 않으면 keep-alive를 유지할 수 없어 성능이 크게 감소합니다.
해결 방법으로 데이터를 압축하여 gzip 헤더를 추가하여 반환합니다. 예를 들면 다음과 같습니다.
```php
<?php
namespace app\controller;
class IndexController
{
    public function index()
    {
        return response(gzencode('hello webman'))->withHeader('Content-Encoding', 'gzip');
    }
}
```
이 외에도 특정 상황에서 apipost는 만족스러운 압력을 발휘하지 못할 수 있는데, 이는 동일한 동시 요청일 때 ab보다 약 50% 낮은 QPS가 발생합니다.
스트레스 테스트에는 apipost가 아닌 ab, wrk 또는 다른 전문적인 스트레스 테스트 소프트웨어를 사용하는 것이 좋습니다.

## 적절한 프로세스 수 설정
webman은 기본적으로 CPU * 4의 프로세스 수를 사용합니다. 실제로 네트워크 I/O가 없는 helloworld 비즈니스의 경우 CPU 코어 수와 동일한 프로세스 수를 사용하면 성능이 최적화되며, 프로세스 전환 비용을 줄일 수 있습니다.
데이터베이스, 레디스 등 차단 I/O 비즈니스의 경우 프로세스 수를 CPU의 3-8배로 설정할 수 있습니다. 이 경우 더 많은 프로세스가 필요하므로 동시에 요청을 처리하여 프로세스 전환 비용은 차단 I/O와 비교하여 무시할 수 있습니다.


## 스트레스 테스트에 대한 참고 범위

**클라우드 서버 4코어 4G 16프로세스 로컬 / 내부 네트워크 스트레스 테스트**

| - | keep-alive 활성화 | keep-alive 비활성화 |
|--|-----|-----|
| hello world | 8-16만 QPS | 1-3만 QPS |
| 데이터베이스 단일 쿼리 | 1-2만 QPS | 1만 QPS |

[**제3자 techempower 스트레스 테스트 데이터**](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)

## 스트레스 테스트 명령어 예시

**ab**
```plaintext
# 100,000요청 200동시요청 keep-alive 활성화
ab -n100000 -c200 -k http://127.0.0.1:8787/

# 100,000요청 200동시요청 keep-alive 비활성화
ab -n100000 -c200 http://127.0.0.1:8787/
```

**wrk**
```plaintext
# 200 동시요청 10초간 스트레스 테스트 keep-alive(기본으로 활성화)
wrk -c 200 -d 10s http://example.com
```