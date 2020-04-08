# MySQL

## Create user and database

```sql
CREATE USER 'telopromo'@'localhost' IDENTIFIED WITH mysql_native_password BY 'secretpassword';
GRANT USAGE ON *.* TO 'telopromo'@'localhost';
CREATE DATABASE IF NOT EXISTS `telopromo`;
GRANT ALL PRIVILEGES ON `telopromo`.* TO 'telopromo'@'localhost';
GRANT ALL PRIVILEGES ON `telopromo\_%`.* TO 'telopromo'@'localhost';
```
