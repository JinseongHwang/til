## Spring Data JPA의 Pageable.Sort를 QueryDSL의 OrderSpecifier로 변환하기

생각보다 블로그, 스택오버플로에 동작하지 않거나 확장성이 없는 코드가 많아서..

## 사용 환경

- JDK 17
- Kotlin 1.9
- Spring Data JPA 3.3.2
- QueryDSL 5.1.0

## 유틸 클래스 구현

```kotlin
import com.querydsl.core.types.Order
import com.querydsl.core.types.OrderSpecifier
import com.querydsl.core.types.dsl.EntityPathBase
import com.querydsl.core.types.dsl.PathBuilder
import org.springframework.data.domain.Sort

object OrderSpecifiers {

    fun <T> build(qEntity: EntityPathBase<T>, sort: Sort): Array<OrderSpecifier<*>> {
        val orders = mutableListOf<OrderSpecifier<*>>()
        for (order in sort) {
            val pathBuilder = PathBuilder(qEntity.type, qEntity.metadata)
            orders.add(OrderSpecifier(order.toDirection(), pathBuilder.getString(order.property)))
        }
        return orders.toTypedArray()
    }

    private fun Sort.Order.toDirection(): Order = if (this.isAscending) Order.ASC else Order.DESC
}
```

## 사용법

```kotlin
import com.querydsl.core.BooleanBuilder
import com.querydsl.core.types.Predicate
import com.querydsl.core.types.dsl.DateTimeTemplate
import com.querydsl.core.types.dsl.Expressions
import com.querydsl.core.types.dsl.StringPath
import com.querydsl.jpa.impl.JPAQuery
import com.querydsl.jpa.impl.JPAQueryFactory
import org.springframework.data.domain.Page
import org.springframework.data.domain.PageImpl
import org.springframework.data.domain.Pageable
import org.springframework.stereotype.Component
import java.time.LocalDateTime

@Component
class FooSearchRepositoryImpl(
    private val queryFactory: JPAQueryFactory
) : FooSearchRepository {

    override fun search(params: FooSearchParameters, pageable: Pageable): Page<Foo> {
        val pageContent = queryFactory
            .selectFrom(QFoo.foo)
            .where(searchCond(params))
            .orderBy(*OrderSpecifiers.build(QFoo.foo, pageable.sort))
            .offset(pageable.offset)
            .limit(pageable.pageSize.toLong())
            .fetch()
        val count = qCount(params).fetchOne() ?: 0L
        return PageImpl(pageContent, pageable, count)
    }
    
    private fun qCount(params: FooSearchParameters): JPAQuery<Long> = queryFactory
        .select(QFoo.foo.count())
        .from(QFoo.foo)
        .where(searchCond(params))

    private fun searchCond(params: FooSearchParameters): Predicate {
        return BooleanBuilder() // TODO
    }

    private fun StringPath.toLocalDateTime(): DateTimeTemplate<LocalDateTime> {
        return Expressions.dateTimeTemplate(
            LocalDateTime::class.java,
            "STR_TO_DATE({0}, '%Y-%m-%d %H:%i:%s')",
            this
        )
    }
}
```
- 짠 - 잘 동작한다.