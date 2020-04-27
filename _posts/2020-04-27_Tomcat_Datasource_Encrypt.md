---
layout: post
title: tomcat datasource encrypt
tag: tomcat datasource encrypt
subtitle: tomcat datasource encrypt
---



# Tomcat Datasource 암호화

update : 2020/04/27



- 개요 : 톰캣 데이터소스 암호화 
- 테스트 환경 
  : tomcat 8.5
  : mariadb 10.1
  : openjdk 1.8 , jdk 1.7




**Encryptor.java**

```java
package com.tomcat.ds.secured;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.SecretKeySpec;

public class Encryptor {
	 private static final String ALGORITHM = "AES";

	    private static final String defaultSecretKey = "tCtmSvT!@34";

	    private Key secretKeySpec;

	    public Encryptor() throws InvalidKeyException, NoSuchAlgorithmException, NoSuchPaddingException,
	            UnsupportedEncodingException {
	        this(null);
	    }

	    public Encryptor(String secretKey) throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException,
	            UnsupportedEncodingException {
	        this.secretKeySpec = generateKey(secretKey);
	    }

	    public String encrypt(String plainText) throws InvalidKeyException, NoSuchAlgorithmException,
	            NoSuchPaddingException, IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException {
	        Cipher cipher = Cipher.getInstance(ALGORITHM);
	        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
	        byte[] encrypted = cipher.doFinal(plainText.getBytes("UTF-8"));
	        return asHexString(encrypted);
	    }

	    public String decrypt(String encryptedString) throws InvalidKeyException, IllegalBlockSizeException,
	            BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
	        Cipher cipher = Cipher.getInstance(ALGORITHM);
	        cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
	        byte[] original = cipher.doFinal(toByteArray(encryptedString));
	        return new String(original);
	    }

	    private Key generateKey(String secretKey) throws UnsupportedEncodingException, NoSuchAlgorithmException {
	        if (secretKey == null) {
	            secretKey = defaultSecretKey;
	        }
	        byte[] key = (secretKey).getBytes("UTF-8");
	        MessageDigest sha = MessageDigest.getInstance("SHA-1");
	        key = sha.digest(key);
	        key = Arrays.copyOf(key, 16); // use only the first 128 bit

	        KeyGenerator kgen = KeyGenerator.getInstance("AES");
	        kgen.init(128); // 192 and 256 bits may not be available

	        return new SecretKeySpec(key, ALGORITHM);
	    }

	    private final String asHexString(byte buf[]) {
	        StringBuffer strbuf = new StringBuffer(buf.length * 2);
	        int i;
	        for (i = 0; i < buf.length; i++) {
	            if (((int) buf[i] & 0xff) < 0x10) {
	                strbuf.append("0");
	            }
	            strbuf.append(Long.toString((int) buf[i] & 0xff, 16));
	        }
	        return strbuf.toString();
	    }

	    private final byte[] toByteArray(String hexString) {
	        int arrLength = hexString.length() >> 1;
	        byte buf[] = new byte[arrLength];
	        for (int ii = 0; ii < arrLength; ii++) {
	            int index = ii << 1;
	            String l_digit = hexString.substring(index, index + 2);
	            buf[ii] = (byte) Integer.parseInt(l_digit, 16);
	        }
	        return buf;
	    }

	    public static void main(String[] args) throws Exception {
	    	Encryptor aes =  new Encryptor(defaultSecretKey);

	    	String DBId = "root";
	    	System.out.println("DBId = " + aes.encrypt(dopDbDevId));

	    	String DBPw = "admin";
	    	System.out.println("DBPw = " + aes.encrypt(dopDbDevPw));
	    }
}
```



**EncryptedDataSourceFactory.java**

```java
package com.tomcat.ds.secured;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.sql.SQLException;
import java.util.Properties;

import javax.crypto.BadPaddingException;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.naming.Context;
import javax.sql.DataSource;

import org.apache.tomcat.jdbc.pool.DataSourceFactory;
import org.apache.tomcat.jdbc.pool.PoolConfiguration;
import org.apache.tomcat.jdbc.pool.XADataSource;


public class EncryptedDataSourceFactory extends DataSourceFactory {

    private Encryptor encryptor = null;

    public EncryptedDataSourceFactory() {
        try {
            encryptor = new Encryptor(); // If you've used your own secret key, pass it in...
        } catch (InvalidKeyException | NoSuchAlgorithmException | NoSuchPaddingException | UnsupportedEncodingException e) {
        	System.out.println("Error instantiating decryption class." + e);
            throw new RuntimeException(e);
        }
    }

    @Override
    public DataSource createDataSource(Properties properties, Context context, boolean XA) throws InvalidKeyException,
            IllegalBlockSizeException, BadPaddingException, SQLException, NoSuchAlgorithmException,
            NoSuchPaddingException {
        // Here we decrypt our password.
        PoolConfiguration poolProperties = EncryptedDataSourceFactory.parsePoolProperties(properties);
        poolProperties.setUsername(encryptor.decrypt(poolProperties.getUsername()));
        poolProperties.setPassword(encryptor.decrypt(poolProperties.getPassword()));

        // The rest of the code is copied from Tomcat's DataSourceFactory.
        if (poolProperties.getDataSourceJNDI() != null && poolProperties.getDataSource() == null) {
            performJNDILookup(context, poolProperties);
        }
        org.apache.tomcat.jdbc.pool.DataSource dataSource = XA ? new XADataSource(poolProperties)
                : new org.apache.tomcat.jdbc.pool.DataSource(poolProperties);
        dataSource.createPool();

        return dataSource;
    }

}
```



**Command**

```bash
# 작업 디렉토리 생성

$ cd ~
$ mkdir encryptor
$ cd encryptor
$ mkdir -p src/com/tomcat/ds/secured
$ mkdir bin

# 위의 2개의 java 파일을 src/com/tomcat/ds/secured 에 만듭니다. (파일명 동일 생성 필요)

# java 컴파일 (classpath에 tomcat-jdbc.jar 가 있어야 하므로 tomcat 먼저 설치 필요)
$ cd src
$ javac -cp /app/tomcat/lib/tomcat-jdbc.jar src/com/tomcat/ds/secured/*.java -d ../bin

# 암호화 결과 확인
$ cd ../bin
$ java com.tomcat.ds.secured.Encryptor

DBId:e305a05db7c311961d1277dc5010b824
DBPw:586b19a6817483b0837bdc79c6fd6e39

# jar 묶음
$ jar -cf dsencryptor17.jar cj/dop/common/secured/*

# tomcat 공용 library 디렉토리로 복사
$ cp dsencryptor17.jar /app/tomcat/lib/
```



**Configure server.xml**

```xml
<Resource name="jdbc/testdb" auth="Container"
		factory="cj.dop.common.secured.EncryptedDataSourceFactory"
                type="javax.sql.DataSource"
                maxActive="100"
                maxIdle="30"
                maxWait="10000"
                validationQuery="SELECT 1"
                validationInterval="30000"
                username="e305a05db7c311961d1277dc5010b824"
                password="586b19a6817483b0837bdc79c6fd6e39"
                driverClassName="org.mariadb.jdbc.Driver"
                url="jdbc:mariadb://172.17.0.2:3306/testdb?autoReconnect=true"
                removeAbandoned="true"
                removeAbandonedTimeout="60"
                logAbandoned="true"/>
```



**Test WebPage** (dbcon.jsp)

```jsp
<%@ page language="java" contentType="text/html; charset=euc-kr"%>
<%@ page import = "java.sql.*, javax.naming.*, javax.sql.*" %>

<%
 InitialContext ic = new InitialContext();
 DataSource ds = (DataSource)ic.lookup("java:comp/env/jdbc/testdb");

try {
 Connection con = ds.getConnection();
 out.println("Use DataSoruce TEST :<b>Connect Success!!<b>");
 con.close();
} catch(Exception e)
{out.println(e.getMessage());}
%>
```