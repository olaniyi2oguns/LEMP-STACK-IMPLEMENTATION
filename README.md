# LEMP-STACK-IMPLEMENTATION
This guide will walk you through installing and configuring a LEMP-stack on Ubuntu 2022 running on an AWS EC2 instance. Each step is written to ensure clarity for beginners.

## The implementation involve Nginx, MySQL, and PHP.

## STEP 0: Setting your instance

Login to your AWS Console and navigate your EC2. Click on instance and lunch new instance 

![AWS](/Lemp-Images/AWS-lunch-instance.png)

Select appropriate key pair
Use your instance ssh to connect to your instance to your terminal to your AWS instance
## STEP 1: Install the Nginx Web Server

After you must have connect to your AWS EC2 instance, 
- update it with `sudo apt update`
- install Nginx with `sudo apt install nginx`. When prompted, press y and hit enter to confirm installation

![install](/Lemp-Images/yes_update.png)

- check if nginx is running by running `sudo systemctl status nginx`

![status](/Lemp-Images/nginx-status.png)

- Go back to your instance and check if you have opened port 22 (ssh) and port 80 (http) in your security group. If you have not, add them to your Security Group inbound rule.
- test if nginx is working locally `curl http://localhost`

![testnginx](/Lemp-Images/nginx-localhost-success.png)

- test nginx on your browser; you can retrieve your public IP address with `curl -s http://169.254.169.254/latest/meta-data/public-ipv4`. Then copy the IP address and paste it your browser. You should see nginx defualt welcome page. 

![Nginx](/Lemp-Images/nginx-web.jpg)

## STEP 2: Install MySQL

- install mysql with `sudo apt install mysql-server` and when prompted, press y and hit enter to confirm installation. 

- Secure your database (mysql) by starting the MySQL installlation script `sudo mysql_secure_installations`. Follow the prompt by entering Y (if you want).


- verify that mysql is running `sudo systemctl status mysql`

![mysql](/Lemp-Images/mysql-status.png)


- login to mysql console confirm it is set `sudo mysql` and if it opens, it means it is working fine. You can leave mysql console with `exit`

## STEP 3: Install PHP

- install PHP and PHP-FPM. You can use this command to install the two packages `sudo apt install php-fpm php-mysql`. Press Y and hit enter when prompted.

- Very if your PHP installation is successful by checking the version with `php -v`. You should see the installed PHP version 

![php-version](/Lemp-Images/php-version.png)

## STEP 4: Configure Nginx to Use PHP

- Create a directory for your website and assign an owner to it
`sudo mkdir /var/www/lempstack`
`sudo chown -R $USER:$USER /var/www/lempstack`

- Create an Nginx Configuration File. You can use an text editor of your choice

`sudo vim /etc/nginx/sites-available/lempstack`

Paste the following configuration into the file:

```
server {
    listen 80;
    server_name lempstack www.lempstack;
    root /var/www/lempstack;

    index index.html index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

- Activate the Configuration by linking the configuration file to sites-enabled:
`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

- Test Nginx for syntax errors with `sudo nginx -t`

- Reload Nginx to ensure your configuration is being effected
`sudo systemctl reload nginx`

- Test the Nginx configuration `echo 'Hello LEMP!' > /var/www/lempstack/index.html`

- Open a browser and visit `http://<Public-IP-Address>` and you should see Hello LEMP

![Hello](/Lemp-Images/Hello-Lemp.jpg)

**Note:** If you don't see `Hello LEMP` on your website, you may need to disable the default configuration that serves the Nginx welcome page:
`sudo unlink /etc/nginx/sites-enabled/default`

## 5: Test PHP with Nginx

- Create a PHP Test File using text editor of your choice `sudo vim /var/www/lempstack/info.php` and the code below to the file
```
<?php
phpinfo();

```

- access the test file by visiting `http://<Public-IP-Address>/info.php`

You should see the PHP info page

![php](/Lemp-Images/php-testing.jpg)

- Remove the Test file (for security):
`sudo rm /var/www/lempstack/info.php`

## 6: Retrieve Data from MySQL with PHP

- Create a Database and User:

`sudo mysql`

`CREATE DATABASE example_database;`

`CREATE USER 'example_user'@'%' IDENTIFIED BY 'PassWord.1';`

`GRANT ALL PRIVILEGES ON example_database.* TO 'example_user'@'%';`

`FLUSH PRIVILEGES;`

`EXIT;`

- Create a Test Table:

`sudo mysql -u example_user -p`

```
CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);
INSERT INTO example_database.todo_list (content) VALUES ("Learn Nginx"), ("Learn PHP"), ("Learn MySQL");

```

- Create a PHP script to fetch data `sudo vim /var/www/lempstack/todo_list.php`

```
<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
$table = "todo_list";

try {
    $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
    echo "<h2>TODO List</h2><ol>";
    foreach ($db->query("SELECT content FROM $table") as $row) {
        echo "<li>" . $row['content'] . "</li>";
    }
    echo "</ol>";
} catch (PDOException $e) {
    echo "Error!: " . $e->getMessage();
}
?>

```

- Access the script by visiting `http://<Public-IP-Address>/todo_list.php`
You should see your ToDo list items displayed.

![ToDo](/Lemp-Images/Final-To-do-list.jpg)

# Congratulations!
You have successfully deployed a fully functional LEMP stack using Nginx, MySQL, and PHP on Ubuntu 2022. Let me know if you need help with further customization!