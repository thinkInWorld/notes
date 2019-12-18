# Hibernate的架构
Java 的JPA和Hibernate的本地API都会委托给Hibernate, 然后Hibernate调用JDBC实现。<br>
Session：JDBC中Connect的封装
