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












-------------------------------------------------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginAction.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.LoginForm;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.LoginService;

/**
 * LoginAction
 * 
 * This action handles the login function
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginAction extends ActionSupport {

	/**
	 * Processes the login request from the user.
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the login request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */

	public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {

		LoginForm loginForm = (LoginForm) form;
		ActionMessages errors = loginForm.validateForm(mapping, request);
		String forward = Constants.FORWARD_FAILURE;
		
		if (!errors.isEmpty()) {
			this.saveErrors(request, errors);
		} else {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

			MSTUSER user = new MSTUSER();
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.notExist"));
				this.saveErrors(request, errors);
			}
		}
		return mapping.findForward(forward);
	}
}



Common----------------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)Constants.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/
package fjs.cs.common;

/**
 * Constants
 * 
 * @version 1.00
 * @since  1.00
 * @author Tam-nlt
 * 
 */
public class Constants {
	// Table MSTUSER
	public static final String TABLE_USER = "MSTUSER";
	// Represents the next action to be performed based on the success result
	public static final String FORWARD_SUCCESS = "success";
	// Represents the next action to be performed based on the failure result
	public static final String FORWARD_FAILURE = "failure";
	// Successful login condition
	public static final int LOGIN_SUCCESS = 1;
	// Mode export
	public static final int EXPORT_MODE = -1;

	// List bean name in Spring Context
	public static final String BEAN_LOGIN = "loginService";

	
	

}




Dao-----------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginDao.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.dao;

import org.hibernate.Query;
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

import fjs.cs.common.Constants;
import fjs.cs.model.MSTUSER;

/**
 * LoginDao
 * 
 * This class provides data access methods for the MSTUSER table in the database.
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginDao extends HibernateDaoSupport {
	
	/**
	 * Get information about logged in user
	 * 
	 * @param user The MSTUSER object
	 * @return the user logged in
	 */
	
	public MSTUSER getLoggedInUser(MSTUSER user) {
	    MSTUSER currentUser = null;
	    try {
	        String loginQuery = "SELECT COUNT(*) FROM " + Constants.TABLE_USER +" WHERE USERID = :userID AND PASSWORD = :password AND DELETE_YMD IS NULL";
	        
	        Query countQuery = getSession().createQuery(loginQuery)
	            .setParameter("userID", user.getUserID())
	            .setParameter("password", user.getPassword());
	        
	        int loginResult = ((Number) countQuery.uniqueResult()).intValue();

	        if (loginResult == Constants.LOGIN_SUCCESS) {
	            String getUserQuery = "FROM " + Constants.TABLE_USER +" WHERE USERID = :userID AND DELETE_YMD IS NULL";
	            Query query = getSession().createQuery(getUserQuery).setParameter("userID", user.getUserID());
	            currentUser = (MSTUSER) query.uniqueResult();
	        }
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	    return currentUser;
	}
}



form--------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginForm.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.form;

import javax.servlet.http.HttpServletRequest;

import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

/**
 * LoginForm
 * 
 * @version 1.00
 * @since 1.00
 * @author Tam-nlt
 *
 */

public class LoginForm extends ActionForm {

	private static final long serialVersionUID = 1L;
	private String userID;
	private String password;
	private int psnCd;

	public int getPsnCd() {
		return psnCd;
	}

	public void setPsnCd(int psnCd) {
		this.psnCd = psnCd;
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	/**
	 * Validates the user credentials.
	 *
	 * @param mapping The ActionMapping object that contains the mapping information
	 *                between URL and Action.
	 * @param request The HttpServletRequest object that contains the HTTP request
	 *                information from the browser.
	 * @return The ActionMessages object that contains any validation errors.
	 */

	public ActionMessages validateForm(ActionMapping mapping, HttpServletRequest request) {
		ActionMessages errors = new ActionMessages();
		System.out.println(userID +"-"+ password);
		if (userID == null || userID.isEmpty()) {
			errors.add("", new ActionMessage("errors.userID"));
		} else if (password == null || password.isEmpty()) {
			errors.add("", new ActionMessage("errors.password"));
		}
		return errors;
	}
}


hibernate ---------------------
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="fjs.cs.model.MSTUSER" table="MSTUSER">
        <id name="psnCd" type="int">
            <column name="PSN_CD" />
            <generator class="assigned" />
        </id>
        <property name="userID" type="string">
            <column name="USERID" length="50" />
        </property>
        <property name="password" type="string">
            <column name="PASSWORD" length="50" />
        </property>
        <property name="userName" type="string">
            <column name="USERNAME" length="50" />
        </property>
        <property name="deleteYMD" type="timestamp">
            <column name="DELETE_YMD" />
        </property>
        <property name="insertYMD" type="timestamp">
            <column name="INSERT_YMD" />
        </property>
        <property name="insertPSNCD" type="int">
            <column name="INSERT_PSN_CD" />
        </property>
        <property name="updateYMD" type="timestamp">
            <column name="UPDATE_YMD" />
        </property>
        <property name="updatePSNCD" type="int">
            <column name="UPDATE_PSN_CD" />
        </property>
    </class>
</hibernate-mapping>




-----------Model

/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)MSTUSER.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.model;

import java.io.Serializable;
import java.sql.Timestamp;

/**
 * MSTUSER
 * 
 * Data transfer object for MSTUSER entity.
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class MSTUSER implements Serializable {

	private static final long serialVersionUID = 1L;
	
	private int psnCd;
	private String userID;
	private String password;
	private String userName;
	private Timestamp deleteYMD;
	private Timestamp insertYMD;
	private int insertPSNCD;
	private Timestamp updateYMD;
	private int updatePSNCD;

	/**
	 * Default constructor for MSTUSER class.
	 */
	public MSTUSER() {
		super();
	}
	
	/**
	 * Constructs a new instance of MSTUSER with the specified parameters.
	 *
	 * @param userID       the user ID
	 * @param password     the password
	 * @param userName     the user name
	 * @param deleteYMD    the deletion date
	 * @param insertYMD    the insertion date
	 * @param insertPSNCD  the insertion process code
	 * @param updateYMD    the update date
	 * @param updatePSNCD  the update process code
	 */
	
	public MSTUSER(String userID, String password, String userName, Timestamp deleteYMD, Timestamp insertYMD,
				   int insertPSNCD, Timestamp updateYMD, int updatePSNCD) {
		super();
		this.userID = userID;
		this.password = password;
		this.userName = userName;
		this.deleteYMD = deleteYMD;
		this.insertYMD = insertYMD;
		this.insertPSNCD = insertPSNCD;
		this.updateYMD = updateYMD;
		this.updatePSNCD = updatePSNCD;
	}

	/*Getter and Setter*/
	
	public int getPsnCd() {
		return psnCd;
	}

	public void setPsnCd(int psnCd) {
		this.psnCd = psnCd;
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public Timestamp getDeleteYMD() {
		return deleteYMD;
	}

	public void setDeleteYMD(Timestamp deleteYMD) {
		this.deleteYMD = deleteYMD;
	}

	public Timestamp getInsertYMD() {
		return insertYMD;
	}

	public void setInsertYMD(Timestamp insertYMD) {
		this.insertYMD = insertYMD;
	}

	public int getInsertPSNCD() {
		return insertPSNCD;
	}

	public void setInsertPSNCD(int insertPSNCD) {
		this.insertPSNCD = insertPSNCD;
	}

	public Timestamp getUpdateYMD() {
		return updateYMD;
	}

	public void setUpdateYMD(Timestamp updateYMD) {
		this.updateYMD = updateYMD;
	}

	public int getUpdatePSNCD() {
		return updatePSNCD;
	}

	public void setUpdatePSNCD(int updatePSNCD) {
		this.updatePSNCD = updatePSNCD;
	}
}



server-----------


/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginService.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.service;

import fjs.cs.dao.LoginDao;
import fjs.cs.model.MSTUSER;

/**
 * Class for handling user login operations.
 * 
 * @version 1.00
 * @since 1.00
 */

public class LoginService{
		
	private LoginDao loginDao;

	public void setLoginDao(LoginDao loginDao) {
		this.loginDao = loginDao;
	}
		
	/**
	 * Retrieves the user details based on the user mapping from LoginForm
	 *
	 * @param user The MSTUSER object
	 * @return A MSTUSER object containing the user details, or null if the user is not found.
	 */	
	
	public MSTUSER getLoggedInUser(MSTUSER user) {
		return loginDao.getLoggedInUser(user);
	}
}


-----------app.propertys


# -- Set information for the database connection --
db.serverName=127.0.0.1
db.dbName=customersystem
db.portNumber=1433
db.userID=admin
db.password=Abc12345
db.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
db.url=jdbc:sqlserver://127.0.0.1:1433;instanceName=SQLEXPRESS;databaseName=customersystem


# -- Login --
login.title=Login Page
login.heading=Login Page!

# -- Search --
search.title=Search Page
search.heading=Search Page!

# -- Edit --
edit.title=Edit Page
edit.heading=Edit Page!

# -- Import --
import.title=Import Page
import.heading=Import Page!

# -- Setting Header --
setting.title=Setting Header Page
setting.heading=Setting Header Page!

# -- Validate login form --
errors.userID= \u30E6\u30FC\u30B6\u30FCID\u3092\u5165\u529B\u3057\u3066\u304F\u3060\u3055\u3044
errors.notExist = \u30E6\u30FC\u30B6\u30FCID\u307E\u305F\u306F\u30D1\u30B9\u30EF\u30FC\u30C9\u304C\u4E0D\u6B63\u3067\u3059\u3002
errors.password = \u30D1\u30B9\u30EF\u30FC\u30C9\u3092\u5165\u529B\u3057\u3066\u304F\u3060\u3055\u3044\u3002

-------applicationContext.xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:orm="http://www.springframework.org/schema/orm"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/jdbc
		http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/orm
		http://www.springframework.org/schema/orm/spring-orm.xsd">
	
	<!-- Configure Data Source -->
 	<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>WEB-INF/src/app.properties</value>
            </list>
        </property>
        <property name="ignoreResourceNotFound" value="true"/>
    </bean>
    
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="${db.driverClassName}" />
		<property name="url" value="${db.url}" />
		<property name="username" value="${db.userID}" />
		<property name="password" value="${db.password}" />
	</bean>


	<!--Configure Hibernate SessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		
		<property name="mappingResources">
        <list>
            <value>fjs/cs/hibernate/MSTUSER.hbm.xml</value>
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


	<!-- Configure Transaction Manager -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>


	<!-- Configure Struts 1 -->
	
	<!-- DAO Beans -->
	 <bean id="loginDao" class="fjs.cs.dao.LoginDao" >
		 <property name="sessionFactory" ref="sessionFactory" />
   	</bean>
	
	<!-- Service Beans -->
   	<bean id="loginService" class="fjs.cs.service.LoginService" >
   		<property name="loginDao" ref="loginDao" />
   	</bean>
</beans>                   



-------------------Struts-config.xml
<?xml version="1.0" encoding="ISO-8859-1" ?>

<!DOCTYPE struts-config PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 1.2//EN"
          "http://jakarta.apache.org/struts/dtds/struts-config_1_2.dtd">

<struts-config>

	<!-- Form Bean Definitions -->
	<form-beans>
		<form-bean name="loginForm" type="fjs.cs.form.LoginForm" />
	</form-beans>
	
	<!-- Global Exception Definitions -->
	<global-exceptions>
		<!-- sample exception handler <exception key="expired.password" type="app.ExpiredPasswordException" 
			path="/changePassword.jsp"/> end sample -->
	</global-exceptions>


	<!-- Global Forward Definitions -->
	<global-forwards>
		<forward name="/Login" path="/Login.do" />
	</global-forwards>


	<!-- Action Mapping Definitions -->
	<action-mappings>
	    <action path="/Login" forward="/Login" />
			<action path="/Login" name="loginForm"  type="fjs.cs.action.LoginAction" validate="false">
				<forward name="failure" path="/Login"/>
				<forward name="success" path="/Search.do" redirect="true"/>
				<forward name="success" path="/pages/Search.jsp" redirect="true"/>
			</action>
	</action-mappings>
	
	<action-mappings>
	    <action path="/Search" forward="/Search" />
			<action path="/Search" name="customerForm"  type="fjs.cs.action.SearchAction" validate="false">
				<forward name="failure" path="/Search"/>
				<forward name="success" path="/Search.do" redirect="true"/>
			</action>
	</action-mappings>


	<!-- Controller Configuration -->
	<controller processorClass="org.apache.struts.tiles.TilesRequestProcessor" />

	<!-- Message Resources Definitions -->
	<message-resources parameter="app" />

	<!-- Plug Ins Configuration -->
	
	<plug-in className="org.springframework.web.struts.ContextLoaderPlugIn">
    	<set-property property="contextConfigLocation" value="/WEB-INF/applicationContext.xml" />
	</plug-in>
	
	<plug-in className="org.apache.struts.tiles.TilesPlugin">
		<set-property property="definitions-config" value="/WEB-INF/tiles-defs.xml" />
		<set-property property="moduleAware" value="true" />
	</plug-in>

	<plug-in className="org.apache.struts.validator.ValidatorPlugIn">
		<set-property property="pathnames" value="/WEB-INF/validator-rules.xml,/WEB-INF/validation.xml" />
	</plug-in>
</struts-config>




---------------web.xml

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

	<servlet>
		<servlet-name>T001Servlet</servlet-name>
		<jsp-file>/pages/Login.jsp</jsp-file>
	</servlet>
	<servlet-mapping>
		<servlet-name>T001Servlet</servlet-name>
		<url-pattern>/Login</url-pattern>
	</servlet-mapping>
	
	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>


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




---index.jsp


<%@ taglib uri="/tags/struts-logic" prefix="logic" %>
<logic:redirect forward="/Login"/>





---login


<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@include file="../WEB-INF/common/Taglib.jsp" %>
<html:html locale="true">
<head>
<title><bean:message key="login.title"/></title>
<style type="text/css">
	<%@include file="../WEB-INF/css/Login.css" %>
	<%@include file="../WEB-INF/css/Base.css" %>
</style>
<script src="./src/js/Login.js"></script>
<html:base/>
</head>
<body>
	<%@include file="../WEB-INF/common/Header.jsp" %>
	<div class="content">
		<div class="content-text">Login</div>
		<div class="content-login">
			   <form action="./Login.do" method="POST">
				<div class="content-login__header">
					<h3>LOGIN</h3>
					<div id="lblErrorMessage">
						 <html:errors/>
					</div>
				</div>
				<div class="content-login__container">
					<div class="form-group">
						<label for="fullname" class="form-label form-label__userID">User
							Id: </label>
						<input name="userID" id="txtUserID" value="" maxlength="8" class="form-control" />
					</div>
					<div class="form-group__password">
						<label for="password" class="form-label">Password: </label>
						<html:password property="password" styleId="txtPassword"  value='' maxlength="8" styleClass="form-control" />
					</div>
					<div class="form-group__btn">
						<html:submit styleId="btnLogin" styleClass="form-submit" value="Login" />
						<html:reset styleId="btnClear" styleClass="form-submit" value="Clear"/>
					</div>
				</div>
			</form>
		</div>
	</div>
	<%@include file="../WEB-INF/common/Footer.jsp" %>	
</body>
</html:html>


