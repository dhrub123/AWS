## Exam Name - SAA-C02

#### AWS
Amazon Web Service. It is a cloud provider providing servers and services that scale easily on demand.
An account can be created with 12 months worth free service.
aws.amazon.com/free

#### Different AWS Services
<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/Introduction/Different_Services.png"/>

#### AWS BUDGET

Go to MyBilling Dashboard under Billing Service. 
Billing > Budget > Cost Budget > Set your Budget > Give a name to your budget and set period to monthly.
There we can set a fixed budget of 10$. If we do not want to spend anything, we will set 0.01$. AWS will send an email if any cost is incurred above our budget.

We can also configure alerts. 
Configure Alerts > Actual Costs > Alert Threshold (Percentage of Budgeted Amount) - Provide email id.

To find service charges for a month, we have to look under Billing > Bills > Find Month > Select Month > Service Charge

#### Important ports

+ Important ports:
  + FTP: 21
  + SSH: 22
  + SFTP: 22 (same as SSH)
  + HTTP: 80
  + HTTPS: 443 
  + RDS Database Ports
    + PostgreSQL: 5432
    + MySQL and MariaDB: 3306
    + Oracle RDS: 1521
    + MSSQL Server: 1433
    + Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)
