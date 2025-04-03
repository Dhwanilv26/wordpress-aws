# WordPress Deployment on AWS (EC2 + RDS)

This guide will walk you through the process of deploying WordPress on AWS using EC2 (Amazon Linux 2) and RDS (MySQL). We'll cover the necessary setup, database creation, EC2 instance configuration, and WordPress installation.

## Prerequisites

- AWS Account with necessary IAM permissions  
- EC2 Instance (Amazon Linux 2)  
- RDS MySQL Instance  
- SSH access to EC2 instance  

## Step 1: Launch an EC2 Instance

1. **Login to AWS Console**
    - Go to the [EC2 Dashboard](https://console.aws.amazon.com/ec2/).

2. **Launch Amazon Linux 2 Instance**
    - Click on `Launch Instance`.
    - Choose Amazon Linux 2 AMI.
    - Select an instance type (e.g., t2.micro for testing).
    - Create a new key pair or select an existing one.
    - In the `Security Groups` settings, allow HTTP (port 80), SSH (port 22), and MySQL/Aurora (port 3306).
    - Launch the instance.

3. **Connect to EC2 Instance**
    - Once the instance is running, connect to it via SSH:
      ```bash
      ssh -i /path/to/your-key.pem ec2-user@<EC2_PUBLIC_IP>
      ```

## Step 2: Launch an RDS Cluster (MySQL)

1. **Login to AWS Console**
    - Go to the [RDS Dashboard](https://console.aws.amazon.com/rds/).

2. **Create an RDS MySQL Instance**
    - Click on `Create Database`.
    - Choose `Standard Create`.
    - Select `MySQL` for the database engine.
    - Choose the version (ensure it’s compatible with WordPress, ideally 5.6 or later).
    - Select instance size (e.g., db.t2.micro for testing).
    - In the `Connectivity` section, ensure the VPC is the same as your EC2 instance.
    - Configure security groups to allow access from your EC2 instance.
    - Set up a master username and password (e.g., `admin`, `password123`).
    - Finish the setup and launch the RDS instance.

3. **Find RDS Endpoint**
    - In the RDS Dashboard, navigate to your instance and note down the **Endpoint** (e.g., `wordpress.cfpgnjehw330.ap-south-1.rds.amazonaws.com`).

## Step 3: Configure EC2 to Connect to RDS

1. **Install MySQL Client on EC2**
    ```bash
    sudo yum install -y mysql
    ```

2. **Export MySQL Host Environment Variable**
    ```bash
    export MYSQL_HOST=yt-wordpress.cfpgnjehw330.ap-south-1.rds.amazonaws.com
    ```

3. **Connect to RDS MySQL Instance**
    ```bash
    mysql -h $MYSQL_HOST -P 3306 -u admin -p
    ```

4. **Create WordPress Database and User**
    ```sql
    CREATE DATABASE wordpress;
    CREATE USER 'wordpressuser' IDENTIFIED BY 'Devarsh1234';
    GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser;
    FLUSH PRIVILEGES;
    EXIT;
    ```

## Step 4: Install Apache and PHP on EC2

1. **Install Apache**
    ```bash
    sudo yum install -y httpd
    ```

2. **Start Apache Service**
    ```bash
    sudo service httpd start
    sudo chkconfig httpd on
    ```

3. **Install PHP and Additional Dependencies**
    ```bash
    sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2
    ```

## Step 5: Download and Configure WordPress

1. **Download WordPress**
    ```bash
    wget https://wordpress.org/latest.tar.gz
    tar -xzf latest.tar.gz
    cd wordpress
    ```

2. **Configure wp-config.php**
    ```bash
    cp wp-config-sample.php wp-config.php
    vim wp-config.php
    ```

    Replace the following placeholders with your RDS instance details:
    ```php
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wordpressuser');
    define('DB_PASSWORD', 'Devarsh1234');
    define('DB_HOST', 'yt-wordpress.cfpgnjehw330.ap-south-1.rds.amazonaws.com');
    ```

3. **Update Secret Keys**
    - Go to [WordPress Secret Key Service](https://api.wordpress.org/secret-key/1.1/salt/) to generate new secret keys and replace the default ones in `wp-config.php`.

## Step 6: Copy WordPress Files to Web Directory

1. **Copy WordPress Files**
    ```bash
    sudo cp -r * /var/www/html/
    ```

2. **Restart Apache**
    ```bash
    sudo service httpd restart
    ```

## Step 7: Access WordPress and Complete Installation

1. **Get EC2 Public IP**
    - Go to the EC2 Dashboard and find the Public IP of your EC2 instance.

2. **Access WordPress in Browser**
    - Open your browser and go to `http://<EC2_PUBLIC_IP>`.
    - You should see the WordPress installation page.
    - Follow the on-screen instructions to complete the WordPress setup (e.g., choose the site title, create an admin user).

## Troubleshooting: RDS MySQL Version Compatibility

If you encounter issues with the MySQL version on RDS, you may need to upgrade or downgrade the MySQL version to be compatible with WordPress. Ensure that the MySQL version you choose is supported by WordPress (5.6 or later).

1. **Check MySQL Version on RDS**
    ```bash
    mysql -h $MYSQL_HOST -P 3306 -u admin -p
    ```
    Then run:
    ```sql
    SELECT VERSION();
    ```

2. **Upgrade or Downgrade MySQL Version**
    - If you need to upgrade or downgrade the MySQL version, you can modify the RDS instance in the AWS Console under the **Modify** section.
    - **If you upgrade your RDS version, ensure that Apache and MySQL services on EC2 are restarted to apply changes:**
      ```bash
      sudo systemctl restart httpd
      sudo systemctl restart mysqld
      ```

## Conclusion

Congratulations! You have successfully deployed WordPress on AWS using EC2 and RDS. You can now start customizing your WordPress site and adding content.
