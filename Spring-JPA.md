# 依赖
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
  </dependency>
<dependencies>
```

JPA代理生效

```java
@EnableJpaRepositories
class Config { … }

//或者如下
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
class Configuration { … }
```
详细的配置
```java
@Configuration
@EnableJpaRepositories
@EnableTransactionManagement
class ApplicationConfig {

  @Bean
  public DataSource dataSource() {

    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    return builder.setType(EmbeddedDatabaseType.HSQL).build();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setGenerateDdl(true);

    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
    factory.setJpaVendorAdapter(vendorAdapter);
    factory.setPackagesToScan("com.acme.domain");
    factory.setDataSource(dataSource());
    return factory;
  }

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

    JpaTransactionManager txManager = new JpaTransactionManager();
    txManager.setEntityManagerFactory(entityManagerFactory);
    return txManager;
  }
}
```

# 定义仓库接口

有效的接口实现层次
```java
interface MyRepository extends JpaRepository<User, Long> { }

@NoRepositoryBean
interface MyBaseRepository<T, ID> extends JpaRepository<T, ID> { … }

interface UserRepository extends MyBaseRepository<User, Long> { … }
```

无效的接口实现层次
```java
interface AmbiguousRepository extends Repository<User, Long> { … }

@NoRepositoryBean
interface MyBaseRepository<T, ID> extends CrudRepository<T, ID> { … }

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> { … }
```

领域类注解

```java
interface PersonRepository extends Repository<Person, Long> { … }

@Entity
class Person { … }

interface UserRepository extends Repository<User, Long> { … }

@Document
class User { … }

//混合领域
interface JpaPersonRepository extends Repository<Person, Long> { … }

interface MongoDBPersonRepository extends Repository<Person, Long> { … }

@Entity
@Document
class Person { … }
```
# 查询策略

```java

//“_”避免生成查询歧义
List<Person> findByAddress_ZipCode(ZipCode zipCode);

//特殊的参数和返回值处理
Page<User> findByLastname(String lastname, Pageable pageable);
Slice<User> findByLastname(String lastname, Pageable pageable);
List<User> findByLastname(String lastname, Sort sort);
List<User> findByLastname(String lastname, Pageable pageable);

//表达式
Sort sort = Sort.by("firstname").ascending().and(Sort.by("lastname").descending());

//类型安全的表达式
TypedSort<Person> person = Sort.sort(Person.class);
TypedSort<Person> sort = person.by(Person::getFirstname).ascending().and(person.by(Person::getLastname).descending());

//querydsl
QSort sort = QSort.by(QPerson.firstname.asc()).and(QSort.by(QPerson.lastname.desc()));

//限制结果集
User findTopByOrderByAgeDesc();
Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

//结果集包装
Streamable<Person> findByFirstnameContaining(String firstname)

//自定义包装
class Product { 
  MonetaryAmount getPrice() { … }
}

@RequiredArgConstructor(staticName = "of")
class Products implements Streamable<Product> { 
  private Streamable<Product> streamable;
  public MonetaryAmount getTotal() { 
    return streamable.stream() //
      .map(Priced::getPrice)
      .reduce(Money.of(0), MonetaryAmount::add);
  }
}

interface ProductRepository implements Repository<Product, Long> {
  Products findAllByDescriptionContaining(String text); 
}

//对Vavr的支持 参考： Support for Vavr Collections

//空结果和参数支持
 @Nullable
 User findByEmailAddress(@Nullable EmailAddress emailAdress);  
 Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress);
 Stream<User> readAllByFirstnameNotNull();
 @Query("select u from User u")
 Stream<User> streamAllPaged(Pageable pageable);
 
 //异步查询
 @Async
 Future<User> findByFirstname(String firstname);               
 @Async
 CompletableFuture<User> findOneByFirstname(String firstname); 
 @Async
 ListenableFuture<User> findOneByLastname(String lastname)
```

# 自定义实现基类
```java
class MyRepositoryImpl<T, ID>
  extends SimpleJpaRepository<T, ID> {

  private final EntityManager entityManager;

  MyRepositoryImpl(JpaEntityInformation entityInformation,
                          EntityManager entityManager) {
    super(entityInformation, entityManager);

    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }

  @Transactional
  public <S extends T> S save(S entity) {
    // implementation goes here
  }
}

@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```

# 持久化

```java

//嗅探策略
@MappedSuperclass
public abstract class AbstractEntity<ID> implements Persistable<ID> {
  @Transient
  private boolean isNew = true; 
  
  @Override
  public boolean isNew() {
    return isNew; 
  }

  @PrePersist 
  @PostLoad
  void markNotNew() {
    this.isNew = false;
  }

  // More code…
}
```

# 查询策略
```java
@Entity
@NamedQuery(name = "User.findByEmailAddress",
  query = "select u from User u where u.emailAddress = ?1")
public class User {
}

public interface UserRepository extends JpaRepository<User, Long> {
  List<User> findByLastname(String lastname);
  User findByEmailAddress(String emailAddress);
}

 @Query("select u from User u where u.firstname like %?1")
  List<User> findByFirstnameEndsWith(String firstname);
  
  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
  
  repo.findByAndSort("stark", new Sort("LENGTH(firstname)"));           
  repo.findByAndSort("targaryen", JpaSort.unsafe("LENGTH(firstname)"));
  
  @EntityGraph(value = "GroupInfo.detail", type = EntityGraphType.LOAD)
  GroupInfo getByGroupName(String name);
  
   @EntityGraph(attributePaths = { "members" })
  GroupInfo getByGroupName(String name);
  
  //一个可能的SPEL用法
  @MappedSuperclass
public abstract class AbstractMappedType {
  …
  String attribute
}

@Entity
public class ConcreteType extends AbstractMappedType { … }

@NoRepositoryBean
public interface MappedTypeRepository<T extends AbstractMappedType>
  extends Repository<T, Long> {

  @Query("select t from #{#entityName} t where t.attribute = ?1")
  List<T> findAllByAttribute(String attribute);
}

public interface ConcreteRepository
  extends MappedTypeRepository<ConcreteType> { … }
```

# 修改
```java
@Modifying
@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
int setFixedFirstnameFor(String firstname, String lastname);
```

## 结果集映射
```java
// 结果映射
class Person {
  @Id UUID id;
  String firstname, lastname;
  Address address;
  static class Address {
    String zipCode, city, street;
  }
}
interface NamesOnly {
  String getFirstname();
  String getLastname();
}
interface PersonRepository extends Repository<Person, UUID> {
  Collection<NamesOnly> findByLastname(String lastname);
}

interface PersonSummary {
  String getFirstname();
  String getLastname();
  AddressSummary getAddress();
  interface AddressSummary {
    String getCity();
  }
}


//表达式结果集映射
interface NamesOnly {

  @Value("#{target.firstname + ' ' + target.lastname}")
  String getFullName();
  //等价于
  default String getFullName() {
    return getFirstname.concat(" ").concat(getLastname());
  }
  
  @Value("#{args[0] + ' ' + target.firstname + '!'}")
  String getSalutation(String prefix);
}

//DTO 映射
class NamesOnly {
  private final String firstname, lastname;
  NamesOnly(String firstname, String lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  String getFirstname() {
    return this.firstname;
  }
  String getLastname() {
    return this.lastname;
  }
  // equals(…) and hashCode() implementations
}
//等价于
@Value
class NamesOnly {
	String firstname, lastname;
}
//动态映射
interface PersonRepository extends Repository<Person, UUID> {

  <T> Collection<T> findByLastname(String lastname, Class<T> type);
}
//此时的调用看起来如下
void someMethod(PersonRepository people) {

  Collection<Person> aggregates =
    people.findByLastname("Matthews", Person.class);

  Collection<NamesOnly> aggregates =
    people.findByLastname("Matthews", NamesOnly.class);
}

```
## 编程性查询

```java
public interface CustomerRepository extends CrudRepository<Customer, Long>, JpaSpecificationExecutor {
 List<T> findAll(Specification<T> spec);
 …
}

public class CustomerSpecs {

  public static Specification<Customer> isLongTermCustomer() {
    return new Specification<Customer>() {
      public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query,
            CriteriaBuilder builder) {

         LocalDate date = new LocalDate().minusYears(2);
         return builder.lessThan(root.get(Customer_.createdAt), date);
      }
    };
  }

  public static Specification<Customer> hasSalesOfMoreThan(MonetaryAmount value) {
    return new Specification<Customer>() {
      public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query,
            CriteriaBuilder builder) {

         // build query here
      }
    };
  }
}

//此时客户端的查询如下
MonetaryAmount amount = new MonetaryAmount(200.0, Currencies.DOLLAR);
List<Customer> customers = customerRepository.findAll(
  isLongTermCustomer().or(hasSalesOfMoreThan(amount)));
```

# 其他使用注意
```java
class UserRepositoryImpl implements UserRepositoryCustom {

  private final EntityManager em;

  @Autowired
  public UserRepositoryImpl(JpaContext context) {
    this.em = context.getEntityManagerByManagedType(User.class);
  }

  …
}
```
