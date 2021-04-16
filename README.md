# solution-cloud-native-wordpress
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

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618452637798-418cca31-1866-439e-8a55-aca4dfdc73f7.png#from=paste&height=236&id=udd9cdbe7&margin=%5Bobject%20Object%5D&originHeight=236&originWidth=604&originalType=binary&size=105127&status=done&style=none&taskId=u5ac34d3c-158e-4947-bdf6-645ca494313&width=604)


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

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618477829476-63fd619b-a3a6-4b02-ab53-2c3fc32cef86.png#clientId=u3793cc38-94f2-4&from=paste&height=822&id=uff3092f4&margin=%5Bobject%20Object%5D&originHeight=822&originWidth=948&originalType=binary&size=152454&status=done&style=none&taskId=u5f9c5aad-1ae7-4ded-bd74-2a60853b3e1&width=948)


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

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618476537703-e240c78b-c728-4bf8-bc5a-acd78f25f3e2.png#clientId=u3793cc38-94f2-4&from=paste&height=217&id=u90d13e09&margin=%5Bobject%20Object%5D&originHeight=217&originWidth=725&originalType=binary&size=77706&status=done&style=none&taskId=u8bc2e506-95f5-480b-904b-405a5bde7f8&width=725)

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618476435764-0a0efd58-0116-4dd7-9462-b13f55dc3edd.png#clientId=u3793cc38-94f2-4&from=paste&height=322&id=u148486bc&margin=%5Bobject%20Object%5D&originHeight=322&originWidth=559&originalType=binary&size=187947&status=done&style=none&taskId=u9f55fbd0-d7a0-44de-9d6a-6e65e535033&width=559)

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

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618478161980-13cb7d20-ed28-4ff7-81d1-a8f61f9f016b.png#clientId=u3793cc38-94f2-4&from=paste&height=911&id=uf704a6c9&margin=%5Bobject%20Object%5D&originHeight=911&originWidth=767&originalType=binary&size=229961&status=done&style=none&taskId=uafd81d19-0bf7-4f41-abe7-2d73610949d&width=767)

Then the following page shows, which means the installation is success.

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618478290162-8f1c46f4-faf1-4129-96fb-d512a71f79c4.png#clientId=u3793cc38-94f2-4&from=paste&height=418&id=uacb62d4f&margin=%5Bobject%20Object%5D&originHeight=418&originWidth=753&originalType=binary&size=60829&status=done&style=none&taskId=u6923e8a5-3de0-4b02-9e46-668ebf9081f&width=753)

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

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618543286206-d342cc5d-cca9-44d6-8e1f-1db17d11bad4.png#clientId=u8718978c-8794-4&from=paste&height=450&id=u51bbdbbf&margin=%5Bobject%20Object%5D&originHeight=450&originWidth=612&originalType=binary&size=253648&status=done&style=none&taskId=ue0aa425e-ca8f-42b2-8954-49bc427dc0b&width=612)

Please MAKE SURE this Redis setting block is set at the first settings block of the wp-config.php file as shown in the image above.


Run the following command to copy the object-cache configuration file to the /var/www/html/wp-content/ path:
```bash
cp /var/www/html/wp-content/plugins/redis-cache/includes/object-cache.php /var/www/html/wp-content/ 
```
Log on to WordPress to enable Redis object cache. 

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618479467272-056a1cc5-2e99-480f-a5e2-8b54cd83b939.png#clientId=u3793cc38-94f2-4&from=paste&height=496&id=u697cad79&margin=%5Bobject%20Object%5D&originHeight=496&originWidth=378&originalType=binary&size=74133&status=done&style=none&taskId=u0fd01053-fa4c-49f9-a144-fa18c6d735d&width=378)

In the left-side navigation pane, click Plugins. Find the Redis Object Cache plugin and click Activate.  
After the plugin is activated, click Settings. 

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618543153189-e4dc41f7-d799-433b-9cc0-b54c404b7daa.png#clientId=u8718978c-8794-4&from=paste&height=541&id=u332ee57e&margin=%5Bobject%20Object%5D&originHeight=541&originWidth=1478&originalType=binary&size=363499&status=done&style=none&taskId=ue9660c95-7e86-4a01-a809-7c819a0f088&width=1478)

Verify that the plugin status is Connected. Click Flush Cache to synchronize cache data to the ApsaraDB for Redis instance.

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618543434894-234e5efe-8a36-404c-9eae-f41d99627a49.png#clientId=u8718978c-8794-4&from=paste&height=742&id=ua4e92053&margin=%5Bobject%20Object%5D&originHeight=742&originWidth=922&originalType=binary&size=219319&status=done&style=none&taskId=uddff0d38-6297-4e94-8484-0a6a02bfeb5&width=922)

Now, your cloud native Wordpress has been setup successfully. You can visit it via SLB EIP:
```php
http://<SLB_EIP>/
```

![](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/32590/1618545083746-458b7811-82e2-4521-973f-8e883f51da2f.png#clientId=u8718978c-8794-4&from=paste&height=591&id=ue3af758d&margin=%5Bobject%20Object%5D&originHeight=591&originWidth=1110&originalType=binary&size=144207&status=done&style=none&taskId=u28f742d0-5384-417f-a289-451bd7a8ebb&width=1110)

#### Step 4: Make custom ECS image for auto scaling


#### Step 5: Setup ESS for ECS auto scaling


#### Step 6: Setup DNS domain


