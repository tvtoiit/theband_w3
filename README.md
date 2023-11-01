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









