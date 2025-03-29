---
title: "[실습] Spring Boot 와 Redis 로 캐싱 구현"
parent: Week6 - RESTful 아키텍처
nav_order: -3

toc: true
toc_sticky: true

date: 2025-02-19
last_modified_at: 2025-02-19
---

### 들어가며

지난 시간에 만들어 본 웹 브라우저 캐시는 말 그대로 브라우저 단에서 동작하는 캐시인 것인데, 이것만 사용했을 때는 완전한 캐시를 구현했다고 보기 어려웠다. 그래서 서버 측의 캐시 서버도 만들어보고 싶어져서 이 참에 도전해보자는 생각이 들었다. 둘을 함께 사용해서 성능 최적화를 해보자 !

<br>

### Redis

서버 측 캐시는 Redis 를 활용해볼 것이다. Redis 는 인메모리 데이터베이스로 모든 데이터를 RAM 에 저장하기 때문에 일반적인 디스크 기반 DB 보다 조회 속도가 훨씬 빨라서 캐시 서버로 활용되는 경우가 많다.

<br>

### 초기 설정

1. Spring Boot 의존성 추가

    ``` java
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    ```

2. application.yml 설정

    ``` yml
    spring:
        cache:
            type: redis
        data:
            redis:
            host: localhost
            port: 6379
            jedis:
                pool:
                max-active: 10
                max-idle: 5
                min-idle: 1
                max-wait: 100ms
    ```

3. Cache Manager 추가

    ``` java
    @Configuration
    @EnableCaching
    public class CacheConfig {
        @Bean
        public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
            // ✅ 직렬화 설정
            GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer();

            RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofHours(1)) // TTL
                    .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                    .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer));

            return RedisCacheManager.builder(redisConnectionFactory)
                    .cacheDefaults(cacheConfig)
                    .build();
        }

        @Bean
        public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
            RedisTemplate<String, Object> template = new RedisTemplate<>();
            template.setConnectionFactory(redisConnectionFactory);
            template.setKeySerializer(new StringRedisSerializer());
            GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer();
            template.setValueSerializer(serializer);

            return template;
        }
    }
    ```

<br>

### Spring Boot Cache 어노테이션

| Annotation | 설명 |
|---|---|
| @EnableCaching | Spring Boot Cache 를 사용하기 위해 '캐시 활성화' 를 위한 어노테이션이다. |
| @Cacheable | 캐시 정보를 메모리상에 '저장' 하거나 '조회' 하는 기능을 수행한다. |
| @CachePut | 캐시 정보를 메모리상에 '저장' 하며, 이미 존재하는 경우 갱신하는 기능을 수행한다. |
| @CacheEvict | 캐시 정보를 메모리상에서 '삭제' 하는 기능을 수행한다. |

캐싱은 '자주 조회되는 데이터' 에 대해서 데이터베이스 부하를 줄여 서버의 성능을 개선하기 위해 사용된다. 그러므로 '자주 조회되는 데이터' 가 무엇인지 판단하는 것이 먼저다. 이것저것 모든 데이터를 저장해버리면 메모리 부족으로 이어질 수 있고, 불필요한 데이터 때문에 새로운 데이터를 캐싱할 공간이 부족해서 캐시 미스가 증가하는 성능 문제로 이어질 수도 있다. 어떤 데이터를 캐싱하면 좋을지 먼저 생각해보자. 그리고 캐싱한 데이터와 DB 데이터가 불일치할 수도 있으므로 이 점도 유의해서 진행하자.

<br>

### 실전

나는 학습용으로 상품 상세 설명 페이지를 생각하며 만들었고, 모든 상품을 조회하는 API 와 하나의 상품을 조회하는 API 두 개가 있다. 두 개의 API 가 있으니 두 개의 캐시로 관리해보려고 한다. (사실 캐시가 둘 다 필요할까 싶긴 했지만... 어떤 조회 방식이 더 많이 사용될지 모르기 때문에 두 개를 다 구현해봤다.)

- productsAll : 전체 상품을 조회한 데이터를 저장하는 캐시

- products : 개별 데이터를 저장하는 캐시

``` java
@Service
@Transactional
public class ProductsServiceImpl implements ProductsService {

    private final ProductsRepository productsRepository;
    public ProductsServiceImpl(ProductsRepository productsRepository) {
        this.productsRepository = productsRepository;
    }

    @Override
    @Cacheable(value = "productsAll")
    public List<Products> findAll() {
        List<Products> products = productsRepository.findAll();
        return products;
    }

    @Override
    @Cacheable(value = "products", key = "#id")
    public Products findById(Long id) {
        return productsRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Product not found"));
    }

    @Override
    @CacheEvict(value = {"products","productsAll"}, key = "#products.id", allEntries = true)
    public void save(Products products) {
        productsRepository.save(products);
    }

    @Override
    @CacheEvict(value = {"products","productsAll"}, key = "#products.id", allEntries = true)
    public void update(Products products) {
        productsRepository.save(products);
    }

    @Override
    @CacheEvict(value = {"products","productsAll"}, key = "#id", allEntries = true)
    public void delete(Long id) {
        productsRepository.deleteById(id);
    }

}
```

productsAll, products 두 캐시의 데이터 일관성을 지키기 위해 데이터가 추가되거나 변경되거나 삭제될 때 모든 캐시를 날리도록 설정했다. 다소 극단적인 전략인가 싶긴 했으나 데이터 일관성을 지키는 것이 더 중요하다 생각해서 이렇게 설정했다. 데이터가 많을 경우 캐시가 모두 삭제되면 다시 DB에서 조회해야 하니 부하가 증가할 수 있다. 각자에게 맞는 유연한 전략을 선택하는 것이 좋을 듯 !

<br>

🔖 참고자료

[Spring Boot Cache 이해하고 설정하기](https://adjh54.tistory.com/165)
