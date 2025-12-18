```sql

--assign row numbers to times when logged on

WITH status_on AS (SELECT ROW_NUMBER() OVER (ORDER BY times) AS row_on,
				                  times,
	                        status
	                   FROM login_details
	                   WHERE status = 'on'
				  ),
--use gaps and islands method to group together sessions logged on

login_groups AS (SELECT ROW_NUMBER() OVER (ORDER BY l.times)-s.row_on AS login_group,
                        s.row_on,
                        l.times AS times,
	                      l.status AS status
	                FROM login_details AS l
	                FULL JOIN status_on AS s ON l.times = s.times
			   ),

--group log off times with log on groups

logoff_groups AS (SELECT COALESCE(login_group,LAG(login_group,1,NULL) OVER (ORDER BY times)) AS login_group,
                           row_on,
	                       times,
	                       status
	                    FROM login_groups)
				 
--select min and max times and count minutes logged on excluding the null logged out periods				 
       
	   SELECT MIN(times) AS log_on,
	       MAX(times) AS log_off,
		  COUNT(login_group)-1 AS duration
		FROM logoff_groups
	    WHERE login_group NOTNULL
		GROUP BY login_group
		ORDER BY log_on
	
``` 	
		
					 
| log_on | log_off | duration |
|--------|---------|----------|
| 10:00:00 | 10:03:00 |	3 |
| 10:04:00 | 10:06:00 |	2 | 
| 10:09:00 | 10:13:00 | 4 |
| 10:15:00 | 10:16:00 | 1 |
