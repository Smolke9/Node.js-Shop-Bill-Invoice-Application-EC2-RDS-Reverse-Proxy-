# 🧾 Node.js Shop Bill / Invoice Application

## Deployment on AWS with MySQL RDS and Reverse Proxy (Nginx)

This project demonstrates a **production-style Node.js Shop Invoice application** deployed on AWS using:
- Two Linux EC2 instances
- Nginx Reverse Proxy
- Node.js + Express backend
- Amazon RDS MySQL database

---

## 📌 Architecture

Internet  
→ Nginx Reverse Proxy (EC2)  
→ Node.js Application (EC2)  
→ Amazon RDS MySQL  

---

## 🧰 Tools & Technologies
- AWS EC2 (Amazon Linux 2)
- Amazon RDS (MySQL 8.x)
- Node.js (Express)
- Nginx (Reverse Proxy)
- PM2
- MySQL

---

## 1️⃣ AWS Infrastructure Setup

### EC2 – Reverse Proxy
- Port 80: Open to Internet
- Port 22: Your IP

### EC2 – Node Application
- Port 3000: Allowed only from Proxy EC2
- Port 22: Your IP

### Amazon RDS
- Engine: MySQL
- DB Name: shopdb
- Port: 3306
- Access: Only from Node EC2

---

## 2️⃣ Database Setup

```sql
CREATE DATABASE shopdb;
USE shopdb;

CREATE TABLE invoices (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100),
    product_name VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2),
    total DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3️⃣ Node.js Application Setup

```bash
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
mkdir shop-bill-app && cd shop-bill-app
npm init -y
npm install express mysql2 body-parser dotenv
```

---

## 4️⃣ Application Code

### index.js

```js
const express = require('express');
const mysql = require('mysql2');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

const db = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME
});

db.connect(err => {
  if (err) throw err;
  console.log('MySQL Connected');
});

app.get('/', (req, res) => {
  res.sendFile(__dirname + '/invoice.html');
});

app.post('/submit', (req, res) => {
  const { customer, product, quantity, price } = req.body;
  const total = quantity * price;

  const sql = `INSERT INTO invoices 
    (customer_name, product_name, quantity, price, total)
    VALUES (?, ?, ?, ?, ?)`;

  db.query(sql, [customer, product, quantity, price, total], err => {
    if (err) throw err;
    res.send('<h2>Invoice Saved Successfully</h2>');
  });
});

app.listen(3000, () => console.log('App running on port 3000'));
```

---

## 5️⃣ Run Application

```bash
node index.js
```

Using PM2:

```bash
sudo npm install -g pm2
pm2 start index.js
pm2 save
pm2 startup
```

---

## 6️⃣ Nginx Reverse Proxy

```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://NODE_PRIVATE_IP:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7️⃣ Final Access URL

```
http://<PROXY_PUBLIC_IP>
```

---

## 🔄 Traffic Flow

Client → Nginx → Node.js → RDS MySQL

---

## ✅ Conclusion

This project showcases a **secure, scalable Node.js deployment** on AWS using best practices suitable for **DevOps labs, interviews, and real-world applications**.

---

Author: Suraj Molke
