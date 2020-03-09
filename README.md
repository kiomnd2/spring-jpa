#  jpa 

## 목표
* JPA 내부 동작 방식을 이해하지 못하고 사용
* JPA 내부 동작 상식을 그림과 코드로 자세히 설명
* JPA가 어떤 SQL을 만들어 내는지 이해
* JPA가 언제 SQL을 실행하는지 이해


### 1. sql 중심적인 개발의 문제점
- 부한 반복, 지루한 코드..
    - (insert into,,, , update... select ,, )
- 객체 CRUD
    - 기능 추가시 객체와 sql 둘 다 수정

### 2. 객체 vs 관계형 DB 
*  관계형 DB
    * 상속, 즉 연관되있는 테이블간 조회시 쿼리와 객체의 맵핑이 복잡해짐
    * 객체는 참조를 사용 : member.getTeam()
    * 객체는 참조 관계가 한방향, 관계형은 양방향 참조 관계가 적용
    * 관계형 DB애서의 엔티티 신뢰문제 .. 
    * 모든 객체를 미리 조회 할 수 는 없다
    * 객체 조회시 같은 ID로 두개의 객체를 조회해서 같은 결과값을 낸다고 해도, 객체 주소값이 다르므로. 연산자 비교시 false 
* JPA
    * JPA는 애플링케이션과 JDBC 사이에서 동작
    * 적절한 엔티티 객체만 생성하여 JPA에 연산을 요청하면 내부적으로 쿼리를 생성하여 DB에 요청하여 결과를 반환받음
        * 패러다임의 불일치 해결
    * JPA는 인터페이스의 모음
        * 저장 : jpa.persist(member)
        * 조회 : Member member = jpa.find(memeberID);
        * 수정 : member.setName("변경할이름") 
        * 삭제 : jpa.remove(member)
    * 유지보수성이 우수
        * 데이터를 추가해야할 상황일 경우, sql에 손대지 않고 엔티티만 수정하여 해결가능
    * 데이터베이스에 구조에 대해 고민할 필요 없이 jpa의 인터페이스만 호출하면 요구사항에 맞는 쿼리를 수행해줌
    * 신뢰할 수 있는 엔티티, 계층
    * 동일한 트랜젝션에서 조회한 엔티티는 같음을 보장
    * JPA의 성능 최적화 기능
        1. 1차 캐시와 동일성 보장
        2. 트랜잭션을 지원하는 쓰기 지연 ( 버퍼링 기능 ..)
            - JDBC BATCH SQL 기능을 사용해 한번에 SQL 전송
        3. 지연 로딩
            - 객체가 실제 사용될 때 로딩

### 3. JPA

### 4. JPQL
* JPA를 사용하면 엔티티 객체를 중심으로 개발
* 문제는 검색 쿼리
* 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
* 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
* 애플리케이션이 필요한 데이터만  DB에서 불러오려면 결국 검색 조건이 포함된 SQL 이 필요
* JPA는 SQL을 추상화한 JPQL 이라는 객체지향 쿼리 언어 제공
* SQL과 문법 유사
* JPQL은 엔티티 객체를 대상으로 쿼리
* 객체지향 SQL..


### 5. 영속성 관리
* 영속성 컨텍스트
    * 엔티티를 영구 저장하는 환경 이라는 뜻
    * 엔티티 매니저를 통해서 영속성 컨텍스트에 접근 (1:1 혹은 1:N)
* 엔티티의 생명주기
    * 비영속 : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
        * 엔티티 객체를 생성하고 세팅만 한 상태
    * 영속 : 영속성 컨텍스트에 관리되는 상태
        * 생성된 엔티티 객체를 엔티티 매니저에 persist를 통해 메시징 된 상태
    * 준영속 : 영속성 컨텍스트에 저장되었다가 분리된 상태
        * 생성된 엔티티 객체를 엔티티 매니저를 통해 detach 된 상태
    * 삭제 : 삭제된 상태
        * 엔티티 매니저를 통해 엔티티가 remove 된 상태
* 영속성 컨텍스트의 이점
    * 1차 캐시
        * 엔티티 매니저에 1차 캐시가 존재. 엔티티 매니저를 통해 객체를 조회했을 때 먼저 1차캐시를 먼저 조회 그후 DB 조회
    * 동일성 보장
        * 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 DB가 아닌 애플리케이션 차원에서 제공 
    * 트랜잭션을 지원하는 쓰기 지연
        * 쓰기시 엔티티를 1차캐시 저장소에 저장함과 동시에 , insert Sql을 생성하여 쓰기 지연 SQL 저장소에 저장
        * 트랜젝션을 커밋하는 시점에 쓰기지연 SQL 저장소에 저장된 sql이 한번에 flush 됨 그후 db 커밋됨
    * 변경 감지
        * select로 찾아온 엔티티에 대해, 수정 시 변경을 감지하여 해당 쿼리를 수행해줌
        * 1차캐스에는 ID 와 Entity의 스냅샷을 미리 찍어둠. 그후 변경된 값과 스냅샷을 비교하여 변경점에 대한 쿼리를 입력.
    * 지연 로딩
    
    
* 플러시(flush)
    * 영속성 컨텍스트의 변경내용을 데이터베이스에 반영
    * 플러시 발생시..!
        * 변경 감지
        * 수정된 엔티티 쓰기 지연 sql 저장소에 등록
        * 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송
        * 엔티티매니저의 flush 메서드 사용시, 쿼리를 미리 볼 수 있다
    * JPQL 쿼리 실행 시 플러시가 자동으로 호출
    * 플러시 모드 옵션 :
        * AUTO : 커밋이나 쿼리를 실행 할 때 플러시 (default)
        * COMMIT : 커밋할 때만 플러시
     * 영속성 컨텍스트를 비우는것이 아님
        * 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
        
* 준영속 상태
    * 영속 상태의 엔티티가 영속성 컨텍스트에서 분리
    * 영속성 컨텍스트가 제공하는 기능을 사용 못함

### 6. 엔티티 매핑
* 엔티티 매핑 소개
    * 객체와 테이블 매핑 : @Entity, @Table
        * @Entity 가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
        * 기본생성자 필수 (파라미터 없는 생성자)
        * final 클래스 enum, interface, inner 클래스 사용X 
        * 저장할 필드에 final 사용 X 
        * Table은 엔티티와 매핑할 테이블 지정
    * 필드와 컬럼 매핑 : @Column
    * 기본키 매핑 : @Id
    * 연관관계 매핑 : @ManyToOne, @JoinColumn
* 데이터베이스 스키마 자동 생성 ()
    * DDL을 애플리케이션 실행 시점에 자동 생성
    * 테이블중싱- > 객체 중심
    * 데이터베이스 방언을 활용, 대이터베이스에 맞는 적설한 DDL 생성
    * 이렇게 생성된 DDL 은 개발에서만 사용
    * <property name="hibernate.hbm2ddl.auto" value="create"/>
        * 애플리케이션 로딩 시점에 엔티티가 매핑되 있는 테이블에 대해 지우고 다시 테이블을 생성, 없으면 생성
        * value = create-drop 시 .. 애플리케이션 종료 시점에 테이블을 드랍
        * value = update 시 alter table
        * value = validate 기존 테이블과 엔티티 테이블이 일치하는지만 확인
        * value = none 
    * 제약조건 추가가능
* 필드와 칼럼 매핑
    * Column : 칼럼매핑
        * name : 필드와 맾핑할 테이블의 칼럼 이름
        * insertable, updatable : 등록,변경 가능 여부
        * nullable(DDL) : null 값의 허용 여부를 설정
        * unique : 유니크 제약조건. 잘 안씀, 다만 유니크 값을 랜덤값이라 실질적으로 잘 안쓰임
        * columnDefinition : 칼럼 정의를 직접 (varcahr(100) default 'EMPTY' ) 해당 내용이 그대로 ddl 문에 들어감
        * length : 길이 제약 조건
    * Transient : 해당 필드를 DB와 매핑 하지 않음
    * temporal : 날짜매핑
        * DB의 날짜정보 매핑 일반적으로 안씀
        * LocalDate나 LocalDateTime 쓸경우 불필요..
    * LOB: 대량 데이터 매핑
        * 문자 매핑시 CLOB, 나머지는 BLOB으로 매핑됨
    * Enumerated : enum 매핑
        * ORDINAL : enum 의 순서를 데이터베이스에 저장 (기본값) // 위험 사용 금지
        * STRING : enum 이름을 데이터베이스에 저장 
* 기본키 매핑
    * @ID : 직접 할당
    * @GeneratedValue (strategy)  //키생성
        * IDENTITY : 데이터베이스에 위임
            * 기본키 생성을 데이터베이스에 위임
            * em.persist()시점에 즉시 sql을 수행하고 db에서 식별자를 조회
        * SEQUENCE : 데이터베이스 시퀀스 
            * em.persist()시점에 SEQ를 조회
            * 시퀀스 한번 호출에 증가하는 수 ( 성능 최적화에 사용 )
        * TABLE : 키 생성요 테이블 사용 @TableGenerator 필요
            * 키생성 전용 테이블을 생성함
            * 장점 : 모든 데이터베이스에 적용가능
            * 단점 : 성능
        * AUTO: 방언에 따라 자동지정
    * 권장하는 식별자 전략
        * 기본키 제약조건 : null 아님 유일 변하면 안됨
        * 권장 : long + 대체키 + 키 생성전략 사용

### 7. 연관관계 매핑 기초
* 단방향 연관관계
* 양방향 연관관계
    * 객체의 양방향 관계는 단방향 관계가 2개 있는것
    * 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.
* 연관관계의 주인과 mappedBy
    * 객체와 테이블간 연관관계를 맺는 차이를 알아야함
    * 객체의 두 관계중 하나를 연관관계의 주인으로 지정
    * 연관관계의 주인만이 외래 키를 관리
    * 주인이 아닌쪽은 읽기만 가능
    * 주인은 mappedBy 속성 사용 X
    * 주인이 아니면 mappedBy 속성으로 지정
* 누구를 주인으로 ?
    * 외래키가 잇는곳을 주인으로 정해라
* 양방향 매핑일 때는 참조자와 주인 둘다 값을 세팅 해주는게 좋다.
    * 순수 객체 상태를 고려해서 항상 양쪽에 값을 세팅
    * 연관관계 편의 메소드를 생성하자
    * 양방향 매핑시 무한 루프를 조심하자
* 양방향 매핑 정리
    * 단방향 매핑만으로도 이미 연관관계 매핑은 완료
    * 양방향 매핑은 반대 방향으로도 조회 기능이 추가된것뿐
    * 되도록 양방향 매핑을 지양. 단방향 매핑만으로도 충분히 요구조건 수행가능
    