## Installation

### Web Server Setup

1. **Install Apache**
    ```sh
    yum install httpd -y
    ```
    Set hostname:
    ```sh
    vi /etc/hostname
    ```
    Change the hostname to `udc.example.com`.

2. **Install PHP and MySQL**
    ```sh
    dnf module list php
    dnf module -y enable php:8.1
    dnf module -y install php:8.1/common
    yum install mysql -y
    yum install php-mysqli -y
    ```

3. **Check PHP version and enable httpd service**
    ```sh
    php -v
    systemctl enable --now httpd
    ```

4. **Create a test PHP page**
    ```sh
    cd /var/www/html
    vi php_test.php
    ```
    Paste the following content:
    ```php
    <!DOCTYPE html>
    <html>
    <body>
    <h1>My first PHP page</h1>
    <?php
    echo "Hello World!";
    ?>
    </body>
    </html>
    ```

5. **Open the test page in a browser**
    ```sh
    http://<Web server IP Address>/php_test.php
    ```
    If it does not work, stop the firewall:
    ```sh
    setenforce 0
    systemctl stop firewalld
    ```

### Database Server Setup

1. **Install MySQL Server**
    ```sh
    dnf -y install mysql-server
    ```

2. **Configure the character set**
    ```sh
    vi /etc/my.cnf.d/charset.cnf
    ```
    Add the following content:
    ```
    [mysqld]
    character-set-server = utf8mb4
    [client]
    default-character-set = utf8mb4
    ```

3. **Enable MySQL Service**
    ```sh
    systemctl enable --now mysqld
    ```

4. **Secure MySQL Installation**
    ```sh
    mysql_secure_installation
    ```
    Follow the prompts (No, No, Yes, Yes, Yes).

5. **Login to MySQL and test**
    ```sh
    mysql -u root -p
    select user,host from mysql.user;
    show databases;
    ```
    Create a test database and table:
    ```sql
    create database test_database;
    create table test_database.test_table (id int,name varchar(50), address varchar(50), primary key (id));
    insert into test_database.test_table(id, name,address) values("031", "CentOS", "India");
    select * from test_database.test_table;
    drop database test_database;
    exit;
    ```

### User Data Collector Database Setup

1. **Create the database and user**
    ```sql
    mysql -u root -p
    CREATE DATABASE udc;
    CREATE USER 'udc'@'%' IDENTIFIED BY 'Welcome@123';
    GRANT ALL PRIVILEGES ON udc.* TO 'udc'@'%';
    exit;
    ```

2. **Verify connection from the web server**
    ```sh
    mysql -h <DB Server IP> -u udc -p
    ```
    Create a table:
    ```sql
    USE udc;
    CREATE TABLE users ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, age INT NOT NULL, country VARCHAR(255) NOT NULL );
    exit;
    ```

3. **Create folders for uploads**
    ```sh
    mkdir /var/udc
    mkdir /var/udc/uploads
    chmod 777 /var/udc
    chmod 777 /var/udc/uploads
    ```

## Application Pages

### Database Connection Test Page

1. **Create `db_check.php`**
    ```sh
    vi /var/www/html/db_check.php
    ```
    Paste the following content:
    ```php
    <?php
    // MySQL database configuration
    $servername = "192.168.30.9";
    $username = "udc";
    $password = "Welcome@123";
    $dbname = "udc";

    // Create a database connection
    $conn = new mysqli($servername, $username, $password, $dbname);

    // Check for connection errors
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    // Check if the connection is alive using ping
    if ($conn->ping()) {
        echo "MySQL server connection is alive.";
    } else {
        echo "MySQL server connection is not alive.";
    }

    // Close the database connection
    $conn->close();
    ?>
    ```

2. **Test the connection**
    Open the browser and go to:
    ```sh
    http://<Web server IP Address>/db_check.php
    ```

### Main Application Page

1. **Create `main.php`**
    ```sh
    vi /var/www/html/main.php
    ```
    Paste the following content:
    ```php
    <!DOCTYPE html>
    <html>
    <head>
    <title>User Data Collection</title>
    </head>
    <body>
    <?php
    // MySQL database configuration
    $servername = "192.168.30.9";
    $username = "udc";
    $password = "Welcome@123";
    $dbname = "udc";

    // Create a database connection
    $conn = new mysqli($servername, $username, $password, $dbname);

    // Check connection
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        // Collect user data
        $name = $_POST["name"];
        $age = $_POST["age"];
        $country = $_POST["country"];

        // Insert data into the MySQL database
        $sql = "INSERT INTO users (name, age, country) VALUES ('$name', $age, '$country')";
        if ($conn->query($sql) === TRUE) {
            echo "User data has been successfully stored in the database.<br>";
        } else {
            echo "Error: " . $sql . "<br>" . $conn->error;
        }

        // Handle file upload
        $uploadDir = '/var/udc/uploads/';
        $uploadFile = $uploadDir . basename($_FILES['userfile']['name']);
        if (move_uploaded_file($_FILES['userfile']['tmp_name'], $uploadFile)) {
            echo "File is valid, and it has been successfully uploaded.<br>";
        } else {
            echo "File upload failed.<br>";
        }
    }
    ?>

    <h2>Enter User Information</h2>
    <form method="post" enctype="multipart/form-data">
        Name: <input type="text" name="name"><br>
        Age: <input type="number" name="age"><br>
        Country: <input type="text" name="country"><br>
        File Upload: <input type="file" name="userfile"><br>
        <input type="submit" value="Submit">
    </form>

    <?php
    // Close the database connection
    $conn->close();
    ?>
    </body>
    </html>
    ```

## Testing

1. **Ensure the application is working**
    Open the browser and go to:
    ```sh
    http://<Web server IP Address>/main.php
    ```

2. **Submit user data and verify**
    Enter user information and upload a file to ensure the data is stored in the database and the file is uploaded correctly.

## Security Considerations

1. **Stop firewalls for testing**
    ```sh
    setenforce 0
    systemctl stop firewalld
    ```

2. **Remove bridged network for security reasons**
    ```sh
    nmtui
    ```

## Author

- Implemented by Swarupa using VirtualBox, CentOS 8, and PuTTY.

## License

This project is licensed under the MIT License - see the LICENSE.md file for details.

---

You can customize this README further to suit your needs.
