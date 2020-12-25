# 스프링 부트 Mysql-MyBatis 설정 샘플
# 의존성을 추가
## Gradle
```java
implementation group: 'org.mybatis.spring.boot', name: 'mybatis-spring-boot-starter', version: '2.1.4'
implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.22'
```

# Mysql 설정 및 테스트
## 설정
스키마명은 없을 경우 제외해도 됨.
***main/resources/application.yml에 작성하기***
```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/{스키마명}?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
    username: # 계정명
    password: # 패스워드
```
## 테스트코드 작성
*/src/test/com/sample/app/db/ConnectionTests.java*
```java
@Slf4j
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ConnectionTests {
    @Autowired
    private SqlSessionFactory sqlSessionFactory;

    @Test
    public void 커넥션_테스트(){
        try(Connection con = sqlSessionFactory.openSession().getConnection()){
            log.info("커넥션 성공!");
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
```
## 테스트 결과
![image](https://user-images.githubusercontent.com/45007556/103111732-80cd3100-4693-11eb-8020-271e64545afd.png)

# MyBatis 설정 및 테스트
## 설정
*/src/main/com/sample/app/config/MyBatisConfig.java*
```java
@RequiredArgsConstructor
@Configuration
public class MyBatisConfig {
    private final ApplicationContext appCtx;

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setMapperLocations(appCtx.getResources("classpath:/mapper/*.xml"));
        return sqlSessionFactoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
## 테스트를 위한 XML 생성
***/src/main/resources/mapper/test.xml***
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "mybatis-3-mapper.dtd">

<mapper namespace="test">
    <select id="selectTest" resultType="string">
        SELECT now()
        FROM dual
    </select>
</mapper>
```

## 테스트
***/src/test/com/sample/app/db/ConnectionTests.java***
```java
@Slf4j
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ConnectionTests {
    @Autowired
    private SqlSessionFactory sqlSessionFactory;

    //...
    @Test
    public void 매퍼_테스트(){
        try(SqlSession sqlSession = sqlSessionFactory.openSession()){
            String sysdate = sqlSession.selectOne("test.selectTest");
            log.info(sysdate);
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
```
# DAO 생성
***/src/main/com/sample/app/dao/TestDao.java***
```java
@RequiredArgsConstructor
@Repository
public class TestDao {
    private final static String NAMESPACE = "test";

    private final SqlSession sqlSession;

    public String selectTest(){
        return sqlSession.selectOne(NAMESPACE+".selectTest");
    }
}
```
## 테스트
```java
@Slf4j
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class TestDaoTest {
    @Autowired
    private TestDao dao;

    @Test
    public void dao_테스트(){
        log.info(dao.selectTest());
    }
}
```