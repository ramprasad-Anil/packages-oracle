
1st create the tables to insert data 
table 1---
create table  exchange_data(
User_ID number,
Stock_ID varchar2(10),
Stock_Name varchar2(10),
Stock_Count number
);

table2---
create table depository_data(
User_ID number,
Stock_ID varchar2(10),
Stock_Name varchar2(10),
Stock_Count number
);

after create the tables we insert the data by using UTL_files \SQL LODER
to load the date into database

write this command in notepad for two tables like same 
LOAD DATA
INFILE 'D:\despositorydata.csv'
INTO TABLE depository_data
FIELDS TERMINATED BY ','
( user_id, Stock_ID, Stock_Name, Stock_Count )

And then open CMD and write SQLLDR userid/passwor@orcl control=notepadpath.ctl


And after this code---

----create one more table to put the package data where it should match or not
CREATE TABLE stock_comparison_log (
    user_id        NUMBER,
    stock_id       VARCHAR2(50),
    exchange_count NUMBER,
    depository_count NUMBER,
    status         VARCHAR2(20),
    comparison_date DATE
);



CREATE OR REPLACE PACKAGE stock_comparison_pkg AS
    PROCEDURE compare_stock_data;
END stock_comparison_pkg;






CREATE OR REPLACE PACKAGE BODY stock_comparison_pkg AS

    
    PROCEDURE compare_stock_data IS
    BEGIN
      
        DELETE FROM stock_comparison_log;

       
        INSERT INTO stock_comparison_log (user_id, stock_id, exchange_count, depository_count, status, comparison_date)
        SELECT 
           
            COALESCE(e.user_id, d.user_id) AS user_id,
           
            COALESCE(e.stock_id, d.stock_id) AS stock_id,
           
            NVL(e.stock_count, 0) AS exchange_count,
          
            NVL(d.stock_count, 0) AS depository_count,
           
            CASE 
                WHEN NVL(e.stock_count, 0) = NVL(d.stock_count, 0) THEN 'MATCH'
                ELSE 'MISMATCH'
            END AS status,
           
            SYSDATE AS comparison_date
        FROM exchange_data e
        FULL OUTER JOIN depository_data d
            ON e.user_id = d.user_id AND e.stock_id = d.stock_id;
    END compare_stock_data;

END stock_comparison_pkg;


and run the ppackage--

BEGIN
    stock_comparison_pkg.compare_stock_data;
END;





