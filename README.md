study_marking_201503　　2015年3月度課題
======================

Lv.1, Lv.2 共通  
■  
- ユーザーテーブル(USERS)は入会日(JOINED_DATE)と退会日(LEFT_DATE)を持っている  
- 退会していないユーザーの場合、退会日にはNULLが入る

(1-1) ユーザー数の増減を確認するために、日付単位で入会したユーザーの人数と退会したユーザーの人数を一覧化したい。  どうすれば取得できるか？
------

### ユーザーテーブル(USERS) ###
	UID(主キー)	JOINED_DATE	　LEFT_DATE
	1　　　　　　2015-03-01　　2015-03-10
	2　　　　　　2015-03-01　　2015-03-05
	3　　　　　　2015-03-03　　NULL
	4　　　　　　2015-03-03　　2015-03-10
	5　　　　　　2015-03-10　　NULL

### 期待する出力結果 ###
	DATE　　　　JOINED_COUNT　LEFT_COUNT
	2015-03-01　2	　	　　　　0
	2015-03-03　2	　	　　　　0
	2015-03-05　0	　	　　　　1
	2015-03-10　1	　	　　　　2

### スキーマ作成用のSQL ###
	CREATE TABLE USERS (
	  UID INTEGER PRIMARY KEY,
	  JOINED_DATE DATE NOT NULL,
	  LEFT_DATE DATE NULL
	);

	INSERT INTO USERS VALUES (1, '2015-03-01', '2015-03-10');
	INSERT INTO USERS VALUES (2, '2015-03-01', '2015-03-05');
	INSERT INTO USERS VALUES (3, '2015-03-03', NULL);
	INSERT INTO USERS VALUES (4, '2015-03-03', '2015-03-10');
	INSERT INTO USERS VALUES (5, '2015-03-10', NULL);

(1-2)その日時点での会員数も取得してください。
------

### 期待する出力結果 ###
	DATE	    JOINED_COUNT LEFT_COUNT USER_COUNT
	2015-03-01	2			　0　　　　　2
	2015-03-03	2			　0　　　　　4
	2015-03-05	0			　1　　　　　3
	2015-03-10	1			　2　　　　　2



___



Lv.2の方は以下も対象  

■
- イベントとスケジュールという二つのテーブルがある。
- イベントは複数のスケジュールを持つことができる。
- スケジュールを一件も持たないイベントもある（スケジュール未定のイベント）。
- スケジュールには締め切り日が設定されている。
- イベント単位で同じ締め切り日は複数存在しないものとする。

(2-1)次のような条件でイベントとスケジュールの一覧を出したい。
------

- システム日付以降に締め切り日がある、またはスケジュール未定のイベントのみを表示する。
- スケジュールが複数あるイベントはシステム日付に最も近い締め切り日のスケジュールのみを表示する。
- 締め切り日順に並び替える（締め切り日が近いほど上）。ただしスケジュール未定であれば一番下に表示する。
- 締め切り日が同じであればイベントID順に並び替える（IDが小さいものほど上）。

### イベントテーブル ###
	UID(主キー)	EVENT_NAME
	1　　　　　　Completed event
	2　　　　　　No schedule event
	3　　　　　　Continuing event
	4　　　　　　Future event 1
	5　　　　　　Future event 2

### スケジュールテーブル ###
	UID(主キー)	EVENT_ID(外部キー)　	DUE_DATE
	1　　　　　　1　　　　　　　　　　　2015/04/23
	2　　　　　　3　　　　　　　　　　　2015/04/01
	3　　　　　　3　　　　　　　　　　　2015/05/01
	4　　　　　　3　　　　　　　　　　　2015/06/01
	5　　　　　　4　　　　　　　　　　　2015/04/24
	6　　　　　　4　　　　　　　　　　　2015/04/25
	7　　　　　　5　　　　　　　　　　　2015/04/24

### 期待する出力結果(システム日付が 2015/04/24 の場合) ###
	EVENT_ID　EVENT_NAME	　　　SCHEDULE_ID　　DUE_DATE
	4　　　　　Future event 1　　　5　　　　　　2015/04/24
	5　　　　　Future event 2　　　7　　　　　　2015/04/24
	3　　　　　Continuing event　　3　　　　　　2015/05/01
	2　　　　　No schedule event		


### システム日付は便宜的に以下のSQLで取得できる値とする。 ###
	SELECT MAX(SYSDATE) FROM DUMMY_SYSDATE

### スキーマ作成用のSQL ###
	CREATE TABLE EVENTS (
	  UID integer PRIMARY KEY
	  ,EVENT_NAME varchar(50)
	);

	INSERT INTO EVENTS VALUES (1,'Completed event');
	INSERT INTO EVENTS VALUES (2,'No schedule event');
	INSERT INTO EVENTS VALUES (3,'Continuing event');
	INSERT INTO EVENTS VALUES (4,'Future event 1');
	INSERT INTO EVENTS VALUES (5,'Future event 2');

	CREATE TABLE SCHEDULES (
	 UID integer PRIMARY KEY
	  ,EVENT_ID integer
	  ,DUE_DATE date
	);

	INSERT INTO SCHEDULES VALUES (1,1,'2015/04/23');
	INSERT INTO SCHEDULES VALUES (2,3,'2015/04/01');
	INSERT INTO SCHEDULES VALUES (3,3,'2015/05/01');
	INSERT INTO SCHEDULES VALUES (4,3,'2015/06/01');
	INSERT INTO SCHEDULES VALUES (5,4,'2015/04/24');
	INSERT INTO SCHEDULES VALUES (6,4,'2015/04/25');
	INSERT INTO SCHEDULES VALUES (7,5,'2015/04/24');

	CREATE TABLE DUMMY_SYSDATE (
	  UID integer PRIMARY KEY
	  ,SYSDATE date
	);

	INSERT INTO DUMMY_SYSDATE VALUES (1,'2015/04/24');

■
- 社員と所属履歴という二つのテーブルがある。
- 社員は1件以上の所属履歴をを持つ。
- 所属履歴にはその社員がこれまでに所属してきた部署の履歴が保存される。
- 所属履歴には開始日が設定されている。
- 所属履歴が複数ある場合、開始日が最新の履歴が現在の部署を表す。

(2-2)現在IT部門に所属している社員の一覧を取得するにはどうすれば良いか。
------

### 社員テーブル(EMPLOYEES) ###
	UID(主キー)	EMPLOYEE_NAME
	1　　　　　　John
	2　　　　　　Mary
	3　　　　　　Tom

### 所属履歴テーブル(SECTION_HISTORIES) ###
	UID(主キー)	EMPLOYEE_ID(外部キー)	START_DATE	SECTION_NAME
	1　　　　　　1　　　　　　　　　　　　2014/01/01　	Sales
	2　　　　　　2　　　　　　　　　　　　2014/01/01　	IT
	3　　　　　　3　　　　　　　　　　　　2014/01/01　	IT
	4　　　　　　1　　　　　　　　　　　　2015/01/01　	IT
	5　　　　　　2　　　　　　　　　　　　2015/01/01　	Sales

### 補足 ###
	John(EMPLOYEE_ID: 1)の現在の部署はIT  
	Mary(EMPLOYEE_ID: 2)の現在の部署はSales  
	Tom (EMPLOYEE_ID: 3)の現在の部署はIT  

### 期待する出力結果 ###
	EMPLOYEE_ID	EMPLOYEE_NAME	SECTION_NAME	START_DATE
	1　　　　　　John　　　　　　　IT　　　　　　　　2015/01/01
	3　　　　　　Tom　　　　　　　 IT　　　　　　　　2014/01/01

### 補足 ###
ソート順はEMPLOYEE_IDとする  

### スキーマ作成用のSQL ###
	CREATE TABLE EMPLOYEES (
	  UID integer PRIMARY KEY
	  ,EMPLOYEE_NAME varchar(50)
	);

	INSERT INTO EMPLOYEES VALUES (1,'John');
	INSERT INTO EMPLOYEES VALUES (2,'Mary');
	INSERT INTO EMPLOYEES VALUES (3,'Tom');

	CREATE TABLE SECTION_HISTORIES (
	  UID integer PRIMARY KEY
	  ,EMPLOYEE_ID integer
	  ,START_DATE date
	  ,SECTION_NAME varchar(50)
	);

	INSERT INTO SECTION_HISTORIES VALUES (1,1,'2014/01/01', 'Sales');
	INSERT INTO SECTION_HISTORIES VALUES (2,2,'2014/01/01', 'IT');
	INSERT INTO SECTION_HISTORIES VALUES (3,3,'2014/01/01', 'IT');
	INSERT INTO SECTION_HISTORIES VALUES (4,1,'2015/01/01', 'IT');
	INSERT INTO SECTION_HISTORIES VALUES (5,2,'2015/01/01', 'Sales');
