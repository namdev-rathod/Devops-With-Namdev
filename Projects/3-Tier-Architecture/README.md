## Master In 3-Tier Architecture || Ultimate Guide For DevOps Engineer

Problem Statement:
-------------------
Design a scalable, secure, and high-availability 3-tier architecture for an e-commerce web application on AWS. 
The architecture should support dynamic content, handle peak loads efficiently, 
and ensure data integrity and security. Maintain Backup & Disaster Recovery.

 - Scalability & Availability:
	The application should be able to handle sudden spikes in traffic, especially during sales and promotions.
	Ensure high availability and fault tolerance to provide an uninterrupted user experience.

 - Security:
	Ensure secure communication between different layers and components of the architecture.
	Protect sensitive user data, including personal and payment information.
	Implement security best practices for data at rest and in transit.

 - Cost-effectiveness:
	Optimize the architecture for cost without compromising performance and security.
	
 - Performance:
	Optimize for low latency and high throughput.
	Ensure the application can deliver a responsive and fast user experience.
	For global user best performance experience

- Layers:

1. Web - Serve static and dynamic content to users.
2. App - Handle business logic and process user requests.
3. DB  - Store and manage application data securely and efficiently.    

Tech Stack:
-----------

- Web Layer	( FrontEnd )
	- AWS EC2 ( In Private Subnet )
	- Load Balancer - Public
	- CloudFront
	- AWS S3 Bucket
	- AWS Route53
	- AWS ACM - Certificate Manager
	- Auto Scaling
	
- Application Layer ( Middle handle logical operations )
	- AWS EC2 ( In Private )
	- Load Balancer - Internal
	- Auto Scaling
	
- Database Layer ( Backend )
		- AWS RDS - MySQL


Steps to follow:
----------------

1. Create Custom VPC
	- 2 Public Subnet
	- 2 Private Subnet
	- Nat Gateway
2. Create EC2 Instance For Web Layer ( In Private Subnet )	
3. Create EC2 Instance For App Layer ( In Private Subnet )
4. Create Public Application Load Balancer For Web
5. Create Internal Application Load Balancer For App
6. Create RDS MySQL Parameter Group
7. Create Subnet Group With Private Subnet Only
8. Create RDS Cluster With MySQL & Multi-AZ Enabled In Private Subnet
9. Create Bastion Host on Public subnet

Additionally,

1. We can create Auto scaling to make it scalable
2. Create CloudFront for web server & use origin as a load balancer


Note:
We can also create auto scaling group to make it fully scalable infrastructure.

# How to connect web and app server from Bastion privately.

- use Putyy and connect bastion host
- Copy pem file on bastion server
- give permission # chmod 400 webkey.pem
- ssh -i webkey.pem ubuntu@10.0.130.64


Setup:
------

# Web Server - FrontEnd
-----------------------

sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2

# One page website

echo "<html><body><h1>Welcome to My E-Commerce Site</h1></body></html>" | sudo tee /var/www/html/index.html

# Create Public facing application load balancer & configure dns entry in Route53

# Check website is working or not 
https://web.awsguruji.net/

# Enable Apache Reverse Proxy For Backend Server

sudo a2enmod proxy
sudo a2enmod proxy_http
sudo systemctl restart apache2

# Edit Apache2 configuration for proxy  setup for backend server.

cd /etc/apache2/sites-available/
vim 000-default.conf

# Add below settings 

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ProxyPreserveHost On
    ProxyPass / http://10.0.130.64:8080/
    ProxyPassReverse / http://10.0.130.64:8080/
	
Note: 10.0.130.64 is a backend server private ip.

# Check apache2 configuration syntax before restart service
- apachectl -t

# systemctl restart apache2


# App Server
------------

sudo apt update
sudo apt install nodejs npm -y

# Sample Node.js application

sudo apt update
sudo apt install nodejs npm -y

# Create a sample Node.js application
cat <<EOF > app.js
const express = require('express');
const mysql = require('mysql');
const app = express();
const port = 8080;

const db = mysql.createConnection({
  host: '<db-instance-endpoint>',
  user: 'ecomuser',
  password: 'ecompassword',
  database: 'ecommerce'
});

db.connect(err => {
  if (err) {
    console.error('error connecting: ' + err.stack);
    return;
  }
  console.log('connected as id ' + db.threadId);
});

app.get('/', (req, res) => res.send('Hello from the Application Layer'));
app.get('/products', (req, res) => {
  db.query('SELECT * FROM products', (err, results) => {
    if (err) throw err;
    res.send(results);
  });
});

app.listen(port, () => console.log(`App running on port ${port}`));
EOF


# Run command to install required packages
- npm install express mysql

# Start backend application
- node app.js


# We can also use PM2 here
  https://medium.com/we-code-we-write/why-and-how-you-should-use-pm2-for-a-node-js-application-in-production-5fa19dd3a856


What is PM2?

Pm2 is a process manager that helps to keep your Node.js application alive all the time. 
Pm2 runs in the background as a service or a daemon, managing your Node.js application for you.

# Installation of PM2

Use NPM To Install A Package Called PM2
- sudo npm install pm2 -g

# start backend nodejs application by using pm2
- pm2 start app.js


Ref: 
https://www.digitalocean.com/community/tutorials/how-to-use-pm2-to-setup-a-node-js-production-environment-on-an-ubuntu-vps

# DB Server

1. Create RDS MySQL Aurora and store credentials in Secret Manager
2. Get RDS endpoint & credentials from secret manager
3. Login to bastion server and connect rds from there

# Install MySQL Client package on bastion host to connect database command line

- apt-get install mysql-client
- mysql -h rds-endpoint -u dbusername -p
Enter Password: <rds-password>


# Log in to MySQL and create a database and user
sudo mysql -u root -p
CREATE DATABASE ecommerce;
CREATE USER 'ecomuser'@'%' IDENTIFIED BY 'ecompassword';
GRANT ALL PRIVILEGES ON ecommerce.* TO 'ecomuser'@'%';
FLUSH PRIVILEGES;
EXIT;


CREATE TABLE products (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  description TEXT
);

INSERT INTO products (name, price, description) VALUES ('Sample Product', 19.99, 'This is a sample product.');
INSERT INTO products (name, price, description) VALUES ('Demo Product', 22.99, 'This is a demo product.');
INSERT INTO products (name, price, description) VALUES ('Cosmetics Product', 100, 'This is a cosmetics product.');

# Access the web application 
https://web.awsguruji.net/products


Trouebleshootings:
------------------

1. During connect web, app server by using pem key file you might get key related issues like permissions are open

fixes

chmod 400 keyname.pem
========================================================================================


Project-2
---------

# On Web Server 
- cd /var/www/html

vim form.html

<!DOCTYPE html>
<html>
<head>
    <title>Sample Web Form</title>
</head>
<body>
    <h1>Sample Web Form</h1>
    <form action="/submit" method="post">
        <label for="name">Name:</label><br>
        <input type="text" id="name" name="name" required><br><br>
        <label for="email">Email:</label><br>
        <input type="email" id="email" name="email" required><br><br>
        <label for="message">Message:</label><br>
        <textarea id="message" name="message" required></textarea><br><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>


https://web.awsguruji.net/form.html


# On App Server

- mkdir server-app
- npm init -y
- npm install express mysql body-parser

- cd server-app
- vim server.js 

const express = require('express');
const bodyParser = require('body-parser');
const mysql = require('mysql');
const path = require('path');

const app = express();
const port = 3000;

// MySQL connection
const db = mysql.createConnection({
    host: 'db-server-instance-1.cvpc1scbkvq8.ap-south-1.rds.amazonaws.com',
    user: 'admin',
    password: '#(MkBlIL0N8#69RO?%gKkc%3R#yv',
    database: 'ecommerce'
});

db.connect((err) => {
    if (err) {
        throw err;
    }
    console.log('Connected to MySQL Database');
});

// Middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));

// Routes
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.post('/submit', (req, res) => {
    const { name, email, message } = req.body;
    const query = 'INSERT INTO your_table_name (name, email, message) VALUES (?, ?, ?)';
    db.query(query, [name, email, message], (err, result) => {
        if (err) {
            return res.status(500).send('Error saving data to the database');
        }
        res.send('Data saved successfully');
    });
});

// Start server
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});

- MySQL Database Query

```
CREATE DATABASE ecommerce;
USE ecommerce;
CREATE TABLE your_table_name (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```