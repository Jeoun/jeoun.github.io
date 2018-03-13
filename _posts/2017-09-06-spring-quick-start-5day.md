---
layout: post
author: "
jeoun"
---


# DAY 5. MyBatis And JPA

## Mybatis 와 JPA 왜 쓰나?
지금까지 Spring MVC + JDBC연동을 통해서 간단한 예제들을 해보았다.<br>
그런데 코드를 보게 되면 데이터를 데이터베이스에 넣기 위해 작성된 코드가 많은 범위를 차지하는 것 같다.

그리고 자바 소스안에 자바언어가 아닌 SQL이라는 다른 언어를 작성 하기도 하며,<br>
단지 저장, 수정, 조회만을 위해 동적 바인딩 처리 및 결과 매핑 처리를 하게 되면서 복잡도도 올라가게 되었으며,
비즈니스 로직 외 데이터 처리를 위한 코드 작성을 해야만 했다.

단순 JDBC의 여러 불편함을 해소하고자 여러 프레임워크가 나오기 시작하였는데..<br>
이번 책에서는 MyBatis라는 데이터 매핑 프레임워크와 JPA ORM프레임워크에 대해서 간단한 설명 및 실습을 해보도록하겠다.

## MyBatis
### 개념
MyBatis는 객체 지향 프레임워크에서 DBMS를 쉽게 사용하고자 나온 데이터 매핑 프레임워크이다.

MyBatis의 장점
1. SQL의 독립
2. JDBC코드를 비즈니스 로직과 분리
3. 파라미터 바인딩 및 결과 매핑 구문 제거

즉, 기존 JDBC를 사용할 때와 다르게 비즈니스 로직에만 신경을 쓰면 된다는 것과 JAVA와 SQL의 분리이다.

### 설정
#### 디펜던시 추가
{% highlight xml %}
	compile("org.mybatis.spring.boot:mybatis-spring-boot-starter:1.1.1")
{% endhighlight %}

#### 환경 설정
{% highlight java %}
  @Configuration
  public class DatabaseConfig {
      @Bean
      @ConfigurationProperties("spring.datasource")
      public DataSource dataSource() {
          return DataSourceBuilder.create().build();
      }
      @Bean
      public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
          SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
          sqlSessionFactoryBean.setDataSource(dataSource);
          sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:/mapper/*.xml"));
          sqlSessionFactoryBean.setTypeAliasesPackage("com.springbook.max.model");
          return sqlSessionFactoryBean.getObject();
      }
      @Bean
      public SqlSessionTemplate sqlSession(SqlSessionFactory sqlSessionFactory){
          return new SqlSessionTemplate(sqlSessionFactory);
      }
  }
{% endhighlight %}
<br>
* SqlSessionTemplate<br>
 마이바티스-스프링의 핵심 모듈로서 SqlSession을 구현되어있다.<br>
 SqlSessionTemplate은 SqlSession이 현재의 스프링 트랜잭션에서 사용될수 있도록 보장한다.<br>
 추가적으로 SqlSessionTemplate은 필요한 시점에 세션을 닫고, 커밋하거나 롤백하는 것을 포함한 세션의 생명주기를 관리한다.

#### Mapper 설정
{% highlight java %}
@Mapper
@Repository
public interface BoardMapper {

    List<BoardVO> selectBoardList();

    void insertBoard(BoardVO boardVO);

    BoardVO getBoard(BoardVO vo);

    void deleteBoard(BoardVO vo);

    void updateBoard(BoardVO vo);
}
{% endhighlight %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.springbook.max.mapper.BoardMapper">
    ........
</mapper>
{% endhighlight %}

### 사용법
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.springbook.max.mapper.BoardMapper">
    <resultMap id="board" type="boardVO">
        <result column="reg_date" property="regDate"/>
    </resultMap>

    <insert id="insertBoard" parameterType="boardVO">
        INSERT INTO board(seq, title, writer, content)
         VALUES
         ((SELECT nvl(max(seq), 0)+1 FROM BOARD), #{title}, #{writer}, #{content})
    </insert>

    <select id="selectBoardList" resultMap="board">
        SELECT * FROM board ORDER BY seq DESC
    </select>

    <update id="updateBoard" parameterType="boardVO">
        UPDATE board
        SET title = #{title}
        , writer = #{writer}
        , content = #{content}
        where seq = #{seq}
    </update>

    <delete id="deleteBoard" parameterType="boardVO">
        DELETE FROM board
        WHERE seq = #{seq}
    </delete>
</mapper>

{% endhighlight %}


---------------------------
## JPA
### 개념
ORM(Object-Relation Mapping)은 자바 객체와 테이블 사이를 매핑해준다.
Mybatis와 다르게 데이터 매퍼의 기준이 아닌 오브젝트와 릴레이션을 실제엔티티와 매핑해준다는 것이다
즉, 자바 객체에 저장된 데이터를 테이블의 ROW정보로 저장하고, 테이블에 저장된 Row정보를 자바 객체로 매핑해준다.<br>

그리고 앞서 Mybatis를 통해서 자바 객체와 데이터를 매핑했지만 SQL를 작성하여 자바 클래스 혹은 외부 xml파일에서 관리해야했다.<br>
하지만 ORM의 가장 큰 특징이자 장점은 DB연동에 필요한 sql를 자동을 생성해준다는 것이다.<br>

JPA(Java Persistence API)는 모든 ORM들의 표준 인터페이스를 제공하고 있다.

### 영속성
엔티티를 영구히 저장하는 환경이라고 하는데
1. 비영속성 : 영속성 컨텍스트와 상관없이 오브젝트만 생성된 상태
2. 영속성 : 영속성 컨텍스트에 저장된 상태 (flush된 상태)
3. 준영속 : 영속성 컨텍스트에서 관리 하지 않는 상태. <br>flush가 되지 않아서 실제 반영되 지 않음. ( detach() 호출 )
4. 삭제 : 실제 삭제된 상태

### 설정
디펜던시 추가
{% highlight xml %}
	compile("org.springframework.boot:spring-boot-starter-data-jpa")
{% endhighlight %}

{% highlight java %}
spring.jpa:
  database: MYSQL
  properties.hibernate.dialect: org.hibernate.dialect.MySQL5InnoDBDialect # jpa가 최적화된 sql를 제공하기 위해..
  properties.hibernate.hbm2ddl.auto: update
  properties.hibernate.format_sql: false
  properties.hibernate.use_sql_comments: false
  properties.hibernate.default_batch_fetch_size: 50
{% endhighlight %}

{% highlight java %}
@Configuration
@EnableJpaAuditing
public class DatabaseConfig {
{% endhighlight %}
@EnableJpaAuditing는 Entity에 있어서 기본적으로 포함되는 createdAt, modifiedAt, createdBy, modifiedBy등을 자동으로 주입시킬 수있도록 해주는 설정이다.

{% highlight xml %}
    <persistence-unit-metadata>
        <persistence-unit-defaults>
            <entity-listeners>
                <entity-listener class="org.springframework.data.jpa.domain.support.AuditingEntityListener"/>
            </entity-listeners>
        </persistence-unit-defaults>
    </persistence-unit-metadata>
{% endhighlight %}
Entity의 Auditing하기 위한 리스너 등록.

### 주요 어노테이션
1. @Entity
  : 특정 클래스를 JPA가 관리하는 엔티티 클래스로 인식하는 가장 중요한 어노테이션.
2. @Id
  : 엔티티 클래스로 등록이 된다면 반드시 PK를 가지고 있어야 한다. 식별자 지정을 위한 어노테이션.
3. @Table
  : 엔티티 클래스를 정의할 때 엔티티 클래스와 매핑되는 테이블 이름을 별도록 지정하지 않으면 같은 이름으로 매핑된다.
   이때 만약 다른 이름의 경우 일때 Table어노테이션응 이용하여 매핑한 이름을 지정하면된다. 또한 테이블의 제약조건 및 스키마 등을 지정할 수 도 있다.
4. @Column (page. 548)
  : 엔티티 클래스의 변수와 테이블의 컬럼을 매핑할 때 사용.<br>
  기본적으로 엔티티 클래스일 경우 동일한 이름으로 매핑되기 때문에 컬럼과 변수간 이름이 다를 경우에 사용된다. 
5. @GeneratedValue
  : @Id로 지정된 식별자에 PK값을 생성하여 저장할 때 사용한다.
6. @Transient
  : 엔티티와 테이블간의 매핑에서 제외하고자 하는 필드에 선언하는 어노테이션.
7. @Temporal
  : java.util.Date 타입의 날짜 데이터를 매핑할 때 사용한다<br>
    (TemporalType.DATE : 날짜, .TIME : 시간 , .TIMESTAMP : 날짜와 시간)
8. @OneToMany, @OneToOne, @ManyToOne
  : 엔티티간의 릴레이션 정의
  
### spring jpa

### 사용법
{% highlight java %}
public interface BoardRepository extends JpaRepository<Board, Integer> {
}
{% endhighlight %}


