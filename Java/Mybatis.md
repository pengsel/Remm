# Mybatis

**@SelectProvider**

 ```java
public interface UserMapper {
@SelectProvider(type = SqlProvider.class, method = "selectUser")
     @ResultMap("userMap")
     public User getUser(long userId);
}
 ```

根据提供的用户id来查询用户信息，并返回一个User实体bean。