# JPA 기본키 전략에 대해서..

JPA에서는 PK 생성 전략을 총 4개 제공해준다.

프로젝트를 하면서 새롭게 깨달은 것이 있어 정리하고자 한다.


## AUTO 전략

처음 JPA를 배울 때는 AUTO를 쓰면 그냥 자동으로 기본 IDENTITY 전략으로 생성 되는 줄 알았는데, 그게 아니었다.

프로세스는 이렇다. 우리가 @GeneratedValue에서 AUTO를 전략으로 선택해놓으면 하이버네이트는 @GeneratedValue가 사용된 필드의 데이터 타입을 확인한다.

타입이 숫자타입(Integer, Long) 이라면 하이버네이트 persistance.xml 또는 application.property파일의 spring.jpa.hibernate.use-new-id-generator-mappings = 의 값이 True인지 False인지 확인한다. 

(spring boot 2.0 버전 이상 : defalt값은 True /

spring boot 1.5버전 : default 값은 false 그리고 기본키 생성전략이 auto인경우 IDENTITY 전략을 사용.)

그리고 값이 False로 설정되어 있다면, 

persistance.xml 또는 application.property 파일에 설정되어 있는 하이버네이트 방언(hibernate.dialect) 의 값이 어떤 db의 방언으로 설정되어 있는지 확인하고 그 방언의 기본전략을 사용 하는 듯하다.

false 실행 결과 mysql 같은 경우는 pk컬럼에 auto_increment가 설정 되었다. 즉 IDENTITY 전략이 사용된 것이다.

값이 True인 경우에는 기본적으로 SEQUENCE 전략이 사용 되는 듯 하다. (데이터 베이스가 SEQUENCE를 지원하지 않는다면 TABLE 전략으로 바뀌는 듯 하다.) 

mysql에서는 새로운 테이블이 생성되었고, 그 안에 있는 컬럼 값으로 pk값을 할당하는 것을 확인하였다. 

(애초에 SEQUENCE전략과 TABLE전략은 비슷한 전략이라고 생각함. 테이블이 있고 없고의 차이인 듯하다. 시퀀스를 지원하지 않는 데이터베이스는 TABLE 그리고 지원하는 데이터베이스는 시퀀스를 사용할 듯합니다.)

## IDENTITY 전략

기본키 생성을 Hibernate가 아닌 데이터 베이스가 하도록 위임하는 전략이다. 즉 pk값으로 null을 보내면 db에서 알아서 하나씩 증가시켜서 넣어준다.

그러므로 ID 값은 INSERT 쿼리가 들어간 이후에(COMMIT) 알 수 있다. INSERT 이후에 영속성 컨텍스트에 저장. 

IDENTITY 전략에서는 INSERT 쿼리를 날려야 pk를 알 수 있었기 때문에 버퍼링이 불가능.

## SEQUENCE 전략

데이터베이스에서 제공해주는 SEQUENCE OBJECT를 사용한다. 이것도 IDENTITY전략과 마찬가지로 DB가 자동으로 숫자를 증가시켜서 넣어준다.

마찬가지로 ID 값은 INSERT 쿼리가 들어간 이후(COMMIT)에 알 수 있다. INSERT 이후에 영속성 컨텍스트에 저장.

필요한 경우 버퍼링이 가능하다!

allocationSize로 한번에 가져올 사이즈를 정한다. ( 기본 값은 50 ) 하지만 1~50 을 쓰는 도중에 서버가 내려가면 다시 51부터 시작된다. 50 이전에 사용하지 않은 숫자가 있더라도 스킵됨.

@SequenceGenerator 속성
|속성|	설명	|기본값|
|---|---|---|
|name	|GENERATOR의 이름|	필수|
|sequenceName	|데이터 베이스 시퀀스의 이름	|hibernate_sequence|
|initialValue	|DDL 생성 시에만 사용됨, 시퀀스 DDL을 생성할 때 처음 시작할 수를 지정한다.|	1|
|allocationSize	|시퀀스 한 번에 가져올 수 (성능 최적화에 사용)|	50|

예시)
```java
@Entity
@SequenceGenerator(name = “MEMBER_SEQ_GENERATOR",
				   sequenceName = “MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
                   initialValue = 1,
                   allocationSize = 50)
public class Member {
	@Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
    				generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

## TABLE 전략

키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다. 모든 데이터베이스에서 제한 없이 적용 가능하다. 그러나 성능이 좋지 않으므로 실제 운영에서 잘 사용하지 않는다.

@TableGenerator 속성
|속성|	설명	|기본값|
|---|---|---|
|name	|GENERATOR의 이름|	필수|
|table	|키생성 테이블의 이름	| hibernate_sequence |
|pkColumnName	|시퀀스의 컬럼명|	sequence_name|
|valueColumnName	| 시퀀스 값 컬럼명| next_val|
|	initialValue| 초기값, 마지막으로 생성된 값이 기준이다. | 0 |
|	allocationSize| 시퀀스 한번 호출에 증가 하는 수 | 50 |
|	catalog, schema| 데이터베이스 catalog, schema 이름 | |
|uniqueConstraints(DDL)|유니크 제약 조건을 지정 할 수 있다. | |


예시)
```java
create table MY_SEQUENCES (
	sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)

@Entity
@TableGenerator(name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "MEMBER_SEQ",
        allocationSize = 50)
        
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
            generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

