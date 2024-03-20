## AWS Opensearch와 Spring Boot 통합하기

## 1. 의존성 추가

**build.gradle.kts**
```Kotlin
repositories {
    mavenCentral()
    maven(url = "https://aws.oss.sonatype.org/content/repositories/snapshots/")
}

dependencies {
    // opensearch
    implementation("org.opensearch.client:opensearch-java:2.7.0")
    implementation("org.opensearch.client:spring-data-opensearch:1.2.0")
    implementation("org.opensearch.client:spring-data-opensearch-starter:1.2.0") {
        exclude(group = "org.opensearch.client", module = "opensearch-rest-client-sniffer")
    }
    implementation("software.amazon.awssdk:opensearch:2.25.13")
    // ...
}
```

- 우선 Opensearch, AWS Opensearch 관련 라이브러리를 import 해주자.

![error log](https://github.com/JinseongHwang/til/assets/52629158/fefc686c-2344-4e32-8b9f-28e36c5fa454)
- exclude `org.opensearch.client:opensearch-rest-client-sniffer` 를 반드시 해줘야 한다. 
그렇지 않으면 위와 같이 상당히 성가신 에러가 발생한다. 다른 라이브러리와 충돌이 발생하는 것으로 추측된다. 나는 원인을 못찾아서 한참을 삽질했다.

## 2. 프로퍼티 주입

**application.yml**
```yaml
opensearch:
  host: https://xxx-search-xxx.yyy.region.on.aws
  username: admin
  password: password
```

**OpensearchProperties.kt**
```Kotlin
@ConfigurationProperties(prefix = "opensearch")
data class OpensearchProperties @ConstructorBinding constructor(
    val host: String,
    val username: String,
    val password: String
)
```

**PropertyConfig.kt**
```Kotlin
@Configuration
@EnableConfigurationProperties(OpensearchProperties::class)
class PropertyConfig
```

- 필요한 프로퍼티 주입을 해주자. `@Value` 를 써도 되고 이건 코딩하는 사람 마음대로 하자. 

## 3. Opensearch 클라이언트 빈으로 등록

**OpensearchConfig.kt**
```Kotlin
@Configuration
@EnableElasticsearchRepositories
class OpensearchConfig(
    private val property: OpensearchProperties,
) : AbstractOpenSearchConfiguration() {

    override fun opensearchClient(): RestHighLevelClient {
        val credentialsProvider = org.apache.http.impl.client.BasicCredentialsProvider()
        val credentials = org.apache.http.auth.UsernamePasswordCredentials(property.username, property.password)
        credentialsProvider.setCredentials(org.apache.http.auth.AuthScope.ANY, credentials)

        val builder = RestClient.builder(
            HttpHost.create(property.host)
        ).setHttpClientConfigCallback {
            it.setDefaultCredentialsProvider(credentialsProvider)
        }

        return RestHighLevelClient(builder)
    }
}
```

- 클라이언트를 빈으로 등록한다.
- 나는 RestHighLevelClient를 사용했는데, Opensearch 3버전부터 Deprecated 될 예정이라고 한다. 하지만 자료가 가장 많아서 이걸로 선택했다.

## 4. 인덱스 생성 테스트

**OpensearchController.kt**
```Kotlin
@RestController
@RequestMapping("/api/cluster")
class OpensearchController(
    val opensearchClient: RestHighLevelClient
) {

    @PostMapping("/index")
    fun createIndex(
        @RequestParam name: String
    ): ResponseEntity<String> {
        if (name.isBlank()) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Index name must not be blank.")
        }

        val createIndexRequest = CreateIndexRequest(name)
        val indexResponse = opensearchClient.indices().create(createIndexRequest, RequestOptions.DEFAULT)
        println("Index creation response: $indexResponse")


        return ResponseEntity.status(HttpStatus.CREATED).body("Index=`${name}` created successfully.")
    }
}
```
- 인덱스가 잘 생성되는 것을 확인할 수 있었다.
