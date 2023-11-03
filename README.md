# theband_w3

1. Cơ sở dữ liệu:


CREATE DATABASE CUSTOME;
USE CUSTOME;

CREATE TABLE MSTUSER(
    PSN_CD NUMERIC(4) PRIMARY KEY NOT NULL,
    USERID VARCHAR(8),
    PASSWORD VARCHAR(8),
    USERNAME VARCHAR(40),
    DELETE_YMD DATETIME DEFAULT NULL,
    INSERT_YMD DATETIME,
    INSERT_PSN_CD NUMERIC(5) DEFAULT 0,
    UPDATE_YMD DATETIME DEFAULT NULL,
    UPDATE_PSN_CD NUMERIC(5) DEFAULT 0
);

CREATE TABLE MSTCUSTOMER(
    CUSTOMER_ID NUMERIC(8) PRIMARY KEY NOT NULL,
    CUSTOMER_NAME VARCHAR(50),
    SEX VARCHAR(1),
    BIRTHDAY VARCHAR(10),
    EMAIL VARCHAR(40),
    ADDRESS VARCHAR(256),
    DELETE_YMD DATETIME DEFAULT NULL,
    INSERT_YMD DATETIME,
    INSERT_PSN_CD NUMERIC(5) DEFAULT 0,
    UPDATE_YMD DATETIME DEFAULT NULL,
    UPDATE_PSN_CD NUMERIC(5) DEFAULT 0
);




package fjs.cs.model;

import java.math.BigDecimal;
import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@Entity
@Table(name = "MSTUSER")
public class MstUser {
    @Id
    @Column(name = "PSN_CD")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private BigDecimal psnCd;

    @Column(name = "USERID", length = 8)
    private String userId;

    @Column(name = "PASSWORD", length = 8)
    private String password;

    @Column(name = "USERNAME", length = 40)
    private String username;

    @Column(name = "DELETE_YMD")
    @Temporal(TemporalType.TIMESTAMP)
    private Date deleteYmd;

    @Column(name = "INSERT_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date insertYmd;

    @Column(name = "INSERT_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal insertPsnCd;

    @Column(name = "UPDATE_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date updateYmd;

    @Column(name = "UPDATE_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal updatePsnCd;

    // Constructors, getters, and setters
}



package fjs.cs.model;

import java.math.BigDecimal;
import java.sql.Timestamp;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@Entity
@Table(name = "MSTCUSTOMER") // Tên bảng phải khớp với tên bảng trong cơ sở dữ liệu
public class MstCustomer {
    @Id
    @Column(name = "CUSTOMER_ID")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private BigDecimal customerId; // Sử dụng BigDecimal thay vì Long để phù hợp với SQL NUMERIC(8)

    @Column(name = "CUSTOMER_NAME", length = 50)
    private String customerName;

    @Column(name = "SEX", length = 1)
    private String sex;

    @Column(name = "BIRTHDAY", length = 10)
    private String birthDay;

    @Column(name = "EMAIL", length = 40)
    private String email;

    @Column(name = "ADDRESS", length = 256)
    private String address;

    @Column(name = "DELETE_YMD")
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp deleteYmd;

    @Column(name = "INSERT_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp insertYmd;

    @Column(name = "INSERT_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal insertPsnCd;

    @Column(name = "UPDATE_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp updateYmd;

    @Column(name = "UPDATE_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal updatePsnCd;

    // Constructors, getters, and setters
}






import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class AbstractCommon {
    public static Connection getConnection() {
        Connection conn = null;
        try {
            // Thay đổi class driver (ví dụ cho SQL Server)
            Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");

            // Thay đổi chuỗi kết nối (connection string) tương ứng
            String URL = "jdbc:sqlserver://localhost:1433;databaseName=yourDatabaseName";
            String USER = "yourUsername";
            String PASS = "yourPassword";
            conn = DriverManager.getConnection(URL, USER, PASS);
        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return conn;
    }
}





https://mkyong.com/spring/maven-spring-hibernate-mysql-example/



@Repository
public class LoginDao extends BaseDao {
	public List<Integer> checkLogin(MSTUSER userDto) {
		String sql = "SELECT COUNT(*) AS CNT FROM MSTUSER WHERE DELETE_YMD IS NULL AND USERID = ? AND PASSWORD = ?";
		List<Integer> list = _jdbcTemplate.query(sql);
		return list;
	}
}



package fjs.cs.dao;

import java.util.List;

import javax.transaction.Transaction;

import org.hibernate.classic.Session;
import org.springframework.stereotype.Repository;

import fjs.cs.model.MSTUSER;
@Repository
public class LoginDao extends BaseDao {
    public MSTUSER checkLogin(String userId, String password) {
        Session session = sessionFactory.openSession();
        Transaction tx = null;
        MSTUSER user = null;
        try {
            tx = session.beginTransaction();
            Criteria criteria = session.createCriteria(MSTUSER.class);
            criteria.add(Restrictions.eq("USERID", userId));
            criteria.add(Restrictions.eq("PASSWORD", password));
            criteria.add(Restrictions.isNull("DELETE_YMD"));
            user = (MSTUSER) criteria.uniqueResult();
            tx.commit();
        } catch (HibernateException e) {
            if (tx != null) {
                tx.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        return user;
    }
}

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateTestConnection {

    public static void main(String[] args) {
        SessionFactory sessionFactory = null;
        Session session = null;

        try {
            // Tạo SessionFactory từ tệp cấu hình Hibernate (hibernate.cfg.xml)
            sessionFactory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();

            // Mở một phiên làm việc Hibernate
            session = sessionFactory.openSession();

            // Kiểm tra kết nối đến cơ sở dữ liệu bằng cách thử lấy thông tin từ cơ sở dữ liệu
            String testQuery = "SELECT 1";
            int result = (int) session.createSQLQuery(testQuery).uniqueResult();
            System.out.println("Kết nối thành công! Kết quả truy vấn: " + result);
        } catch (Exception e) {
            e.printStackTrace();
            System.err.println("Kết nối thất bại: " + e.getMessage());
        } finally {
            // Đóng phiên làm việc
            if (session != null) {
                session.close();
            }
            // Đóng SessionFactory
            if (sessionFactory != null) {
                sessionFactory.close();
            }
        }
    }
}





------------------------

<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="fjs.cs.model.MSTUSER" table="MSTUSER" >
		<id name="psnCd" type="int">
			<column name="PSN_CD"/>
			<generator class="assigned" />
		</id>
		<property name="userID" type="string">
			<column name="USERID" length="50"/>
		</property>
		<property name="password" type="string">
			<column name="PASSWORD" length="50"/>
		</property>
		<property name="userName" type="String">
			<column name="USERNAME" length="50"/>
		</property>
		<property name="deleteYMD" type="timestamp">
			<column name="DELETE_YMD"/>
		</property>
		<property name="insertYMD" type="timestamp">
			<column name="INSERT_YMD"/>
		</property>
		<property name="insertPSNCD" type="int">
			<column name="INSERT_PSN_CD"/>
		</property>
		<property name="updateYMD" type="timestamp">
			<column name="UPDATE_YMD"/>
		</property>
		<property name="updatePSNCD" type="int">
			<column name="UPDATE_PSN_CD"/>
		</property>
	</class>
</hibernate-mapping>




-------applicationContext----

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:orm="http://www.springframework.org/schema/orm"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/orm
		http://www.springframework.org/schema/orm/spring-orm.xsd">
	<!-- Configure Data Source -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	<property name="locations">
		<value>WEB-INF/src/app.properties</value>
	</property>
	<property name="ignoreResourceNotFound" value="true"/>
</bean>

<bean id="dataSource" 
         class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>

<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource"/>
	<property name="mappingResources">
	<list>
		<value>fjs/cs/hibernate/MSTUSER.hbm.xml</value>
		<value>fjs/cs/hibernate/MSTCUSTOMER.hbm.xml</value>
	</list>
	</property>
	<property name="hibernateProperties">
		<props>
			<prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
			<prop key="hibernate.show_sql">true</prop>
			<prop key="hibernate.format_sql">true</prop>
		</props>
	</property>
</bean>

 <!-- Configure Struts 1  -->
<!-- DAO Beans  -->
	<bean id="loginDao" class="fjs.cs.dao.LoginDao">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>
	
	<bean id="customerDao" class="fjs.cs.dao.customerDao">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>
	
	<!-- Service Beans -->
	<bean id="loginService" class="fjs.cs.service.LoginService">
		<property name="loginDao" ref="loginDao"></property>
	</bean>
	<bean id="searchService" class="fjs.cs.service.SearchService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="deleteService" class="fjs.cs.service.DeleteService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="editService" class="fjs.cs.service.EditService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="importService" class="fjs.cs.service.ImportService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	
</beans>



---------------------- app.properties

# -- set infomation for the database connection --
db.serverName=TOI-TV-VM\SQLEXPRESS
db.dbName=customersystem
db.portNumber=1433
db.userID=sa
db.password=Abc12345
db.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
db.url=jdbc:sqlserver://localhost:1433;databaseName=customersystem



-----------------------web.xml

<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app
  PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.2//EN"
  "http://java.sun.com/j2ee/dtds/web-app_2_2.dtd">

<web-app>
  <display-name>Struts Blank Application</display-name>

  <!-- Standard Action Servlet Configuration (with debugging) -->
  <servlet>
    <servlet-name>action</servlet-name>
    <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
    <init-param>
      <param-name>config</param-name>
      <param-value>/WEB-INF/struts-config.xml</param-value>
    </init-param>
    <init-param>
      <param-name>debug</param-name>
      <param-value>2</param-value>
    </init-param>
    <init-param>
      <param-name>detail</param-name>
      <param-value>2</param-value>
    </init-param>
    <load-on-startup>2</load-on-startup>
  </servlet>


  <!-- Standard Action Servlet Mapping -->
  <servlet-mapping>
    <servlet-name>action</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/applicationContext.xml</param-value>
	</context-param>

  <!-- The Usual Welcome File List -->
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>


  <!-- Struts Tag Library Descriptors -->
  <taglib>
    <taglib-uri>/tags/struts-bean</taglib-uri>
    <taglib-location>/WEB-INF/struts-bean.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-html</taglib-uri>
    <taglib-location>/WEB-INF/struts-html.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-logic</taglib-uri>
    <taglib-location>/WEB-INF/struts-logic.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-nested</taglib-uri>
    <taglib-location>/WEB-INF/struts-nested.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-tiles</taglib-uri>
    <taglib-location>/WEB-INF/struts-tiles.tld</taglib-location>
  </taglib>

</web-app>



