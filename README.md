# Spring-DATA-JPA

롬복 설명
@Setter: 실무에서 가급적 Setter는 사용하지 않기
@NoArgsConstructor AccessLevel.PROTECTED: 기본 생성자 막고 싶은데, JPA 스팩상
PROTECTED로 열어두어야 함
@ToString은 가급적 내부 필드만(연관관계 없는 필드만)

 JpaRepository 를 사용하는 인터페이스*
public interface MemberRepository extends JpaRepository<Member, Long> {}
제네릭은 <엔티티 타입, 식별자 타입> 설정

스프링 데이터 JPA가 제공하는 쿼리 메소드 기능
조회: find…By ,read…By ,query…By get…By,
예:) findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
COUNT: count…By 반환타입 long
EXISTS: exists…By 반환타입 boolean
삭제: delete…By, remove…By 반환타입 long
DISTINCT: findDistinct, findMemberDistinctBy
LIMIT: findFirst3, findFirst, findTop, findTop3
> 참고: 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.
그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
> 이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점이다.

![image](https://user-images.githubusercontent.com/69129562/196655319-67b17490-0b93-45a2-a823-c4b020444846.png)


@Modifying
벌크 연산을 위해 사용된다. @Query로 다수 로우를 update한다면 오류가 발생한다.
이경우 사용하는 어노테이션인데 영속성 컨텍스트를 거치지 않고 database에 바로 질의 하게되어 log를 찍어보면 updqte된 데이터로 나타나지 않는다.
이경우 @Modifying(clearAutomatically = true)로 해결할수있다

@EntityGraph
연관된 엔티티들을 SQL 한번에 조회하는 방법
ex)
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();
사실상 페치 조인(FETCH JOIN)의 간편 버전
LEFT OUTER JOIN 사용

