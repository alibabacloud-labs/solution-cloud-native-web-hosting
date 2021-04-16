# Cloud Native WordPress on Alibaba Cloud
Quick start with Wordpress on Alibaba Cloud with cloud native features such as high availability, auto-scaling, etc.

### Project URL
[https://github.com/alibabacloud-labs/solution-cloud-native-wordpress](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress)


### Architecture Overview
![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/archi.png)

### Deployment
#### Terraform
Use terraform to provision VPC, SLB, EIP, ESS, ECS, Redis and PolarDB instances that used in this solution against this .tf file:
[https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/blob/main/deployment/terraform/main.tf](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/blob/main/deployment/terraform/main.tf)


For more information about how to use Terraform, please refer to this tutorial: [https://www.youtube.com/watch?v=zDDFQ9C9XP8](https://www.youtube.com/watch?v=zDDFQ9C9XP8)


### Run Demo
#### Step 1: Install Apache HTTP Server and PHP on ECS

- Logon to ECS via SSH
```bash
ssh root@<EIP_ECS>
```
If you met this error when ssh to ECS, please go to **/Users/xxx/.ssh/known_hosts, VI to edit the file and remove the whole line with the EIP of the target ECS at the very beginning. After that, please SSH to log on again.**

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step1_1.png)


- Run the following command to install required utilities on the instance: 
```bash
yum install -y sysbench unzip zip dstat
```

   - Zip: a utility commonly used for compressing files and folders 
   - Unzip: a utility used for decompressing files and folders.  
   - Sysbench: a benchmarking tool used for system performance testing. 
   - Dstat: a monitoring tool that provides statistics about system performance.  



- Install Apache HTTP Server

Run the following command to install the Apache HTTP server:
```bash
yum -y install httpd
```


- Install PHP

List available versions of PHP:
```bash
dnf module list php
```
Most likely php 7.4 is included, so run the following commands to enable PHP 7.4 (Please make sure the PHP version is new to catch up the requirement of the Wordpress, otherwise Wordpress installation would fail possibly):
```bash
dnf module reset php
dnf module enable php:7.4
dnf install -y php php-opcache php-gd php-curl php-mysqlnd
dnf install -y php-bcmath php-mbstring php-xmlwriter php-xmlreader php-cli php-ldap php-zip php-fileinfo
```
Then restart Apache HTTP Server:
```bash
service httpd restart
```
Create a PHP file to verify the PHP is working:
```bash
vim /var/www/html/info.php
```
Then input the following content in this info.php file, then save and exit.
```php
<?php
phpinfo();
?>
```
Then open the following URL in a Web browser (Note: Replace the <ECS_EIP> placeholder with the Elastic IP address of the ECS instance that you obtained previously): 
```php
http://<ECS_EIP>/info.php
```
If the following page appears, PHP is installed successfully.

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step1_2.png)


#### Step 2: Install and configure Wordpress on ECS
Create a folder, download the WordPress package to this folder, and extract the package. To do this, run the following commands in sequence:
```bash
mkdir -p /opt/WP  
cd /opt/WP 
wget https://wordpress.org/latest.tar.gz 
tar -xzvf latest.tar.gz 
```
Run the following commands in sequence to configure WordPress to access ApsaraDB for PolarDB:
```bash
cd /opt/WP/wordpress/ 
cp wp-config-sample.php wp-config.php 
vim wp-config.php 
```
Complete the database configurations as follows: 


| Setting | Value & description |
| --- | --- |
| DB_NAME | The name of the ApsaraDB for PolarDB database that you created.  In this tutorial, we use "wpdb", which is predefined in resource "alicloud_polardb_database" within Terraform script [https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/blob/main/deployment/terraform/main.tf](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/blob/main/deployment/terraform/main.tf). |
| DB_USER | The user name of the database account you created. In this lab, we use "test_polardb" as predefined within Terraform script.  |
| DB_PASSWORD | The password of the database account you created within Terraform script. In this lab, we use "N1cetest" as predefined within Terraform script. |
| DB_HOST | The VPC-facing endpoint of the ApsaraDB for 
PolarDB cluster that you obtained previously. 
Do not include the port number.  Please use the Cluster endpoint of PolarDB. |

Endpoint on PolarDB web console:

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step2_1.png)

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step2_2.png)

Run the following commands in sequence to copy the wordpress folder to the /var/www/html/ path: 
```bash
cd /var/www/html 
cp -rf /opt/WP/wordpress/* /var/www/html/ 
```
Open the following URL in a Web browser to initialize WordPress: 
```bash
http://<ECS_EIP>
```
Note: Replace the <ECS_EIP> placeholder with the Elastic IP address of the ECS instance that you obtained previously. 


Then complete the settings and click "Install WordPress".

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step2_3.png)

Then the following page shows, which means the installation is success.

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step2_4.png)

#### Step 3: Configure Redis caching
Run the following commands in sequence to download the Redis object cache plugin and unzip the plugin package: 
```bash
cd /opt/WP
wget https://downloads.wordpress.org/plugin/redis-cache.2.0.18.zip 
unzip redis-cache.2.0.18.zip 
```


Run the following commands in sequence to copy the redis-cache folder to the /var/www/html/wp-content/plugins/path and configure WordPress to access ApsaraDB for Redis:
```bash
cp -rf redis-cache /var/www/html/wp-content/plugins/ 
vim /var/www/html/wp-config.php
```
Complete the settings as follows: 
| Setting | Value & Description |
| --- | --- |
| WP_REDIS_HOST | The internal endpoint of the ApsaraDB for Redis instance that you obtained previously. Such as r-xxxxx.redis.singapore.rds.aliyuncs.com |
| WP_REDIS_PORT | The port number.  |
| WP_REDIS_PASSWORD | The password for connecting to the instance.  |

```bash
// Redis settings
define( 'WP_REDIS_HOST', '<Redis URL>' );
define( 'WP_REDIS_CLIENT', 'predis' );
define( 'WP_REDIS_PORT', '6379' );
define( 'WP_REDIS_DATABASE', '0');
define( 'WP_REDIS_PASSWORD', 'test_redis:N1cetest' );
```

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step3_1.png)

Please MAKE SURE this Redis setting block is set at the first settings block of the wp-config.php file as shown in the image above.


Run the following command to copy the object-cache configuration file to the /var/www/html/wp-content/ path:
```bash
cp /var/www/html/wp-content/plugins/redis-cache/includes/object-cache.php /var/www/html/wp-content/ 
```
Log on to WordPress to enable Redis object cache. 

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step3_2.png)

In the left-side navigation pane, click Plugins. Find the Redis Object Cache plugin and click Activate.  
After the plugin is activated, click Settings. 

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step3_3.png)

Verify that the plugin status is Connected. Click Flush Cache to synchronize cache data to the ApsaraDB for Redis instance.

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step3_4.png)

Now, your cloud native Wordpress has been setup successfully. You can visit it via SLB EIP:
```php
http://<SLB_EIP>/
```

![image.png](https://github.com/alibabacloud-labs/solution-cloud-native-wordpress/raw/main/images/step3_5.png)

#### Step 4: Make custom ECS image for auto scaling


#### Step 5: Setup ESS for ECS auto scaling


#### Step 6: Setup DNS domain


