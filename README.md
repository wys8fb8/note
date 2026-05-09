批量上架失败: Unexpected <class 'sqlalchemy.exc.IntegrityError'>: (pymysql.err.IntegrityError) 
(1048, "Column 'stage' cannot be null") [SQL: INSERT INTO artwork_review_records (artwork_id, stage, action, change_type, reviewer_id, reviewer_note)
 VALUES (%s, %s, %s, %s, %s, %s)] [parameters: (375, None, 'put_on_shelf', 'listing', None, None)] (Background on this error at: https://sqlalche.me/e/20/gkpj)
