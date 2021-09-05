---
title: 욕설감지
categories: [java]
comments: true
---

package discord;

import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

import org.bitbucket.eunjeon.seunjeon.Analyzer;
import org.bitbucket.eunjeon.seunjeon.LNode;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * 로그 분석하는 함수 : analyzer
 * 욕설 추가(커스텀) : dictInput
 * 봇 켜질 때 최초 1회 실행 : setUserDict, makeAbuseList
 * 나머지 함수 : get, set
 * 데이터베이스 손상시 전부 비우고 DataFormat 클래스 실행
 *
 */
public class Userdic {
	static String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
	static String DB_URL = "jdbc:mysql://183.99.87.90:3306/sun?characterEncoding=UTF-8&serverTimezone=UTC";

	static String USERNAME = "root";
	static String PASSWORD = "swhacademy!";
	
	Connection connection = null;
	Statement statement = null;
	
	List<String> abuseList = null;
	
	Userdic(){
		abuseList = makeAbuseDict();	//데이터베이스 에서 코드로 리스트 가져옴
		setUserDict(abuseList);			//코드에서 사용자사전에 값 대입
	}
	
	//욕설 감지 후 감지된 욕설 리스트로 반환
	public List<String> analyzer(String str){
		List<String> a = new ArrayList<String>();
		
		// 형태소 단위 분석
		//디스코드에서 받아온 로그 입력
		for (LNode node : Analyzer.parseJava(str)) {
			if(abuseList.contains(node.morpheme().surface())) {
				a.add(node.morpheme().surface());
			}
		}
		
		return a;
	}

	//사용자가 사용자사전에 욕설 추가하는 함수
	public void dictInput(String abuse) {

		//사용자 사전 추가
		Analyzer.setUserDict(Arrays.asList(abuse).iterator());

		//데이터베이스에 단어 추가
		try{
			Class.forName(JDBC_DRIVER);
			connection = DriverManager.getConnection(DB_URL,USERNAME,PASSWORD);
			System.out.println("MariaDB 연결.");
			statement = connection.createStatement();

			ResultSet rs = statement.executeQuery("SELECT id FROM abuse_dict order by DESC LIMIT 1");
			int id = rs.getInt(1);

			statement.execute(String.format("insert into abuse_dict value(%d, %s)",id,abuse));

		}catch(SQLException se1){
			se1.printStackTrace();
		}catch(Exception ex){
			ex.printStackTrace();
		}finally{
			try{
				if(statement!=null) statement.close();
				if(connection!=null) connection.close();
			}catch(SQLException se2){
			}
		}
		
		//abuseList 변수에 값 추가
		abuseList.add(abuse);
	}

	//데이터베이스의 욕설들을 사용자사전에 추가하는 코드(프로그램 동작 시 1회 실행)
	public void setUserDict(List<String> list) {
		for(String str : list) {
			Analyzer.setUserDict(Arrays.asList(String.format("%s ,-100", str)).iterator());
		}
	}

	//데이터베이스의 욕설들을 리스트 형태로 코드에 들고오는 함수(프로그램 동작 시 1회 실행)
	public List<String> makeAbuseDict() {
		List<String> list = new ArrayList<String>();

		try{
			Class.forName(JDBC_DRIVER);
			connection = DriverManager.getConnection(DB_URL,USERNAME,PASSWORD);
			System.out.println("MariaDB 연결.");
			statement = connection.createStatement();

			ResultSet rs = statement.executeQuery("SELECT abuse FROM abuse_dict");

			while(rs.next()) {
				list.add(rs.getString("abuse"));
			}


		}catch(SQLException se1){
			se1.printStackTrace();
		}catch(Exception ex){
			ex.printStackTrace();
		}finally{
			try{
				if(statement!=null) statement.close();
				if(connection!=null) connection.close();
			}catch(SQLException se2){
			}
		}

		return list;
	}

	
	public List<String> getAbuseList() {
		return abuseList;
	}

	public void setAbuseList(List<String> abuseList) {
		this.abuseList = abuseList;
	}

	public static String getJDBC_DRIVER() {
		return JDBC_DRIVER;
	}
	
	public static void setJDBC_DRIVER(String jDBC_DRIVER) {
		JDBC_DRIVER = jDBC_DRIVER;
	}
	
	public static String getDB_URL() {
		return DB_URL;
	}

	public static void setDB_URL(String dB_URL) {
		DB_URL = dB_URL;
	}

	public static String getUSERNAME() {
		return USERNAME;
	}

	public static void setUSERNAME(String uSERNAME) {
		USERNAME = uSERNAME;
	}

	public static String getPASSWORD() {
		return PASSWORD;
	}
	
	public static void setPASSWORD(String pASSWORD) {
		PASSWORD = pASSWORD;
	}
}
