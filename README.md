# Spring-DATA-JPA<br/>

롬복 설명<br/>
@Setter: 실무에서 가급적 Setter는 사용하지 않기<br/>
@NoArgsConstructor AccessLevel.PROTECTED: 기본 생성자 막고 싶은데, JPA 스팩상
PROTECTED로 열어두어야 함<br/>
@ToString은 가급적 내부 필드만(연관관계 없는 필드만)<br/>

 JpaRepository 를 사용하는 인터페이스*<br/>
public interface MemberRepository extends JpaRepository<Member, Long> {}<br/>
제네릭은 <엔티티 타입, 식별자 타입> 설정<br/>

스프링 데이터 JPA가 제공하는 쿼리 메소드 기능<br/>
조회: find…By ,read…By ,query…By get…By,<br/>
예:) findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.<br/>
COUNT: count…By 반환타입 long<br/>
EXISTS: exists…By 반환타입 boolean<br/>
삭제: delete…By, remove…By 반환타입 long<br/>
DISTINCT: findDistinct, findMemberDistinctBy<br/>
LIMIT: findFirst3, findFirst, findTop, findTop3<br/>
> 참고: 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.<br/>
그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.<br/>
> 이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점이다.<br/>

![image](https://user-images.githubusercontent.com/69129562/196655319-67b17490-0b93-45a2-a823-c4b020444846.png)


@Modifying<br/>
벌크 연산을 위해 사용된다. @Query로 다수 로우를 update한다면 오류가 발생한다.<br/>
이경우 사용하는 어노테이션인데 영속성 컨텍스트를 거치지 않고 database에 바로 질의 하게되어 log를 찍어보면 updqte된 데이터로 나타나지 않는다.<br/>
이경우 @Modifying(clearAutomatically = true)로 해결할수있다<br/>

@EntityGraph<br/>
연관된 엔티티들을 SQL 한번에 조회하는 방법<br/>
ex)<br/>
@EntityGraph(attributePaths = {"team"})<br/>
@Query("select m from Member m")<br/>
List<Member> findMemberEntityGraph();<br/>
사실상 페치 조인(FETCH JOIN)의 간편 버전<br/>
LEFT OUTER JOIN 사용<br/>

@QueryHints - 쿼리 실행 로그에 SQL_ID, value(Coment)를 남긴다. <br/>
@Version - 낙관적 검증(Optimistic Lock) 디비에 락을 걸기보다는 충돌 방지(Conflict detection)에 가깝다고 볼 수 있음, 동시성 처리, 트랜잭션이 종료될 때까지 다른 트랜잭션에서 변경하지 않음을 보장합니다. 이를 통해 dirty read와 non-repeatable read를 방지합니다.<br/>
@Lock - 비관적 검증(Pessimistic Lock), 선점 잠금, 다른 트랜잭션과 충동을 가정하고 우선 락, 
 LockModeType.PESSIMISTIC_WRITE
일반적인 옵션. 데이터베이스에 쓰기 락
다른 트랜잭션에서 읽기도 쓰기도 못함. (배타적 잠금)<br/>

LockModeType.PESSIMISTIC_READ<br/>
반복 읽기만하고 수정하지 않는 용도로 락을 걸 때 사용<br/>
다른 트랜잭션에서 읽기는 가능함. (공유 잠금)<br/>

LockModeType.PESSINISTIC_FORCE_INCREMENT<br/>
Version 정보를 사용하는 비관적 락<br/>

스프링 데이터 Auditing 적용 - 현업에서는 엔터티가 변경 될때마다 등록일, 수정일을 명시 해준다. <br/>
@CreatedDate
@LastModifiedDate
@CreatedBy
@LastModifiedBy<br/>
 
@MappedSuperclass - 객체의 입장에서 공통 매핑 정보가 필요할 때 사용한다.<br/>
 
 페이징<br/>
@GetMapping("/members")<br/>
public Page<Member> list(Pageable pageable) {<br/>
  Page<Member> page = memberRepository.findAll(pageable);<br/>
  return page;<br/>
}<br/>
요청 파라미터<br/>
예) /members?page=0&size=3&sort=id,desc&sort=username,desc<br/>
 page: 현재 페이지, 0부터 시작한다.<br/>
 
글로벌 설정<br/>
 spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/<br/>
spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/<br/>

 개별설정 - @PageableDefault<br/>
 
 접두사가 둘이상 - 예제: /members?member_page=0&order_page=1 @Qualifier사용<br/>
 
 엔터티를 API로 노출하면 안되기 때문에 DTO로 변환<br/>
 Page.map() 사용<br/>
@GetMapping("/members")<br/>
public Page<MemberDto> list(Pageable pageable) {<br/>
 return memberRepository.findAll(pageable).map(MemberDto::new);<br/>
}<br/>
 
 페이지 1부터 시작할땐 직접 Pageable을 구현해서 사용해야 한다.<br/>
 
 * save() 메서드*<br/>
새로운 엔티티면 저장( persist )<br/>
새로운 엔티티가 아니면 병합( merge )<br/>
 
  JPA 식별자 생성 전략이 @GenerateValue 면 save() 호출 시점에 식별자가 없으므로 새로운
엔티티로 인식해서 정상 동작한다. 그런데 JPA 식별자 생성 전략이 @Id 만 사용해서 직접 할당이면 이미
식별자 값이 있는 상태로 save() 를 호출한다. 따라서 이 경우 merge() 가 호출된다. merge() 는 우선
DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 따라서
Persistable 를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.<br/>
> 참고로 등록시간( @CreatedDate )을 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할
수 있다. (@CreatedDate에 값이 없으면 새로운 엔티티로 판단)<br/>
 
 public interface Persistable<ID> {<br/>
 ID getId();<br/>
 boolean isNew();<br/>
}
 
