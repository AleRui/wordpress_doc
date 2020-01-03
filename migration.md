# Migration Wordpress

### ORIGIN

#### Create Zip file in origin of migration host with all wordpress files content.
```bash
sudo zip -r wp_backup_YYYYMMDD.zip ./origin_path_of_wp_files
```
```bash
sudo tar -czvf wp_backup_YYYYMMDD.zip.tar.gz ./origin_path_of_wp_files
```
#### Create Sql file with dump of database in origin of migration host.
```bash
mysqldump -u username -ppassword database_name > ./path/wpdb_backup_YYYYMMDD.sql
```

#### Download files Zip and Sql.

---

### DESTINATION

#### Check Server Page configuration:
- Available modules:
 LoadModule rewrite_module modules/mod_rewrite.s
```bash
sudo a2enmod rewrite
```
```bash
sudo systemctl restart apache2
```
- Configuring host or virtualHost in destination:
In path: /etc/apache/apache.conf
AllowOverwrite None -> **AllowOverwrite All**

#### Locate zip files in detination.
- Unzip files in destination path:
```bash
sudo unzip wp_backup_YYYYMMDD.tar.gz
```
```bash
sudo gzip -d wp_backup_YYYYMMDD.tar.gz
```
- Change permission if it is necessary (rewrite .htaccess):
```bash
sudo chmod -R 777 /destination_path
```
- Change user and group if it is necessary
```bash
sudo chown -R www-data /destination_path
```
```bash
sudo chgrp -R www-data /destination_path
```

#### Import dump of database.
```bash
mysql -u username -ppassword wordpress < wpdb_backup_YYYYMMDD.sql
```

#### Change path database registers:
```sql
UPDATE wp_options SET option_value = replace(
  option_value,
  'https://www.domain.com',
  'http://localhost/domain'
  )
WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET guid = replace(
  guid,
  'https://www.domain.com',
  'http://localhost/domain'
);

UPDATE wp_posts SET post_content = replace(
  post_content,
  'https://www.domain.com',
  'http://localhost/domain'
);

UPDATE wp_postmeta SET meta_value = replace(
  meta_value,
  'https://www.domain.com',
  'http://localhost/domain'
);
```

#### Config wp-config.
- Config MySQL
```php
define('DB_NAME', 'database_name');
define('DB_USER', 'username');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
```
- Add if it is necessary:
```php
/** Active Debugs de WP*/
define ('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
/** Declare init page home path*/
define('WP_HOME', 'http://localhost/domain');
define('WP_SITEURL', 'http://localhost/domain');
```

#### Update permalink (SAVE Plan, SAVE Changes).
This rewrite .htaccess (it must have permissions).

#### Change links in CSS files or Theme.