# SLI, SLO, SLA

- SLI, SLO, SLA는 서비스 신뢰성을 측정하고 관리하기 위한 계층적 개념이다.

## SLI (Service level indicator)

- 서비스의 특정 측면을 정량적으로 측정하는 지표이다.
- 예를 들어,
    - API 응답 시간
    - 시스템 가용성
    - 처리량

## SLO (Service level objective)

- SLI가 달성해야 하는 구체적인 목표치이다.
- 예를 들어,
    - API 응답 시간 -> API 응답의 95%는 200ms 이내여야 한다
    - 시스템 가용성 -> 월간 가용성은 99.9% 이상 유지해야 한다 (전체 시간 대비 정상 응답하는 시간)
    - 처리량 -> 1초간 200개의 요청을 처리할 수 있어야 한다

## SLA (Service level agreement)

- 고객과 맺는 약속으로, SLO를 기반으로 하되 더 제한적이고 보수적으로 SLO를 설정해야 한다. SLA를 위반하기 전에 내부적으로 문제를 감지하고 대응할 수 있는 여유를 두는 것이 중요하다. 위반 시 보상 조항을 포함해야 한다.
- 예를 들어,
    - SLI: 가용성
    - SLO: 가용성 99.9% (error budget 0.1%)
    - SLA: 99.5% 보장, 위반 시 환불

# 구글의 4가지 골든 시그널

- 구글SRE 팀에서는 메트릭 수집할 때 4가지 골든 시그널에 집중해야 한다고 강조한다.
- 4가지 시그널을 잘 관측하면 사용자가 경험하는 문제와 시스템 내부 상태 문제에 모두 대응할 수 있다.
    - Latency + Errors -> 사용자 경험 직접 영향
    - Traffic -> 수요 변화 파악 (ex. Scale Up/Out 변화 시점 예측)
    - Saturation -> 문제를 미리 예측하고 대응할 수 있다.

## 1. Latency

요청을 처리하는 데 얼마나 걸리는가?

```promql
# 성공 요청의 95 percentile 응답 시간
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket{status=~"2.."}[5m])) by (le)
)

# 평균 응답 시간 (성공 요청만)
rate(http_request_duration_seconds_sum{status=~"2.."}[5m])
/
rate(http_request_duration_seconds_count{status=~"2.."}[5m])
```

- 성공한 요청과 실패한 요청의 지연시간을 분리해서 측정해야 한다.
- 실패한 요청은 매우 빠를 수 있어서 평균을 왜곡할 수 있다.

## 2. Traffic

시스템에 대한 수요는 얼마나 되는가?

```promql
# 초당 HTTP 요청 수
sum(rate(http_requests_total[5m])) by (method, endpoint)

# endpoint별 트래픽 분포
sum(rate(http_requests_total[5m])) by (endpoint)

# 전체 트래픽 (모든 status code)
sum(rate(http_requests_total[5m]))
```

- 서비스 유형에 따라 다르게 측정된다.
- HTTP: 초당 요청 수, DB: 초당 트랜잭션 수, 스트리밍: 네트워크 I/O 등

## 3. Errors

실패한 요청은 얼마나 되는가?

```promql
# 에러율 (5xx 에러 비율)
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))

# 에러 발생 건수
sum(rate(http_requests_total{status=~"5.."}[5m])) by (status, endpoint)

# 성공률 (반대 개념)
sum(rate(http_requests_total{status=~"2.."}[5m]))
/
sum(rate(http_requests_total[5m]))
* 100
```

- 명시적 실패(HTTP 500)뿐만 아니라 암묵적 실패(잘못된 콘텐츠 반환)도 포함해야 한다.

## 4. Saturation

시스템 리소스의 포화도는 어떻게 되는가?

```promql
# CPU 사용률
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 메모리 사용률
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# 디스크 사용률
(node_filesystem_size_bytes - node_filesystem_avail_bytes) 
/ 
node_filesystem_size_bytes * 100

# 데이터베이스 커넥션 풀 사용률
hikaricp_connections_active / hikaricp_connections_max * 100

# 스레드 풀 사용률
thread_pool_active_threads / thread_pool_max_threads * 100
```

- 서비스가 처리할 수 있는 최대 용량 대비 현재 사용량.
- 성능 저하가 시작되기 전에 감지하는 것이 중요하다.