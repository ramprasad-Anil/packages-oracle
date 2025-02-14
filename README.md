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
