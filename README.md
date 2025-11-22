# **Local PostgreSQL Setup (Windows CMD + DBeaver)**
---

This guide sets up **PostgreSQL locally** using Docker, connects it to your Node.js/Express app, and optionally uses **DBeaver** as a GUI.

---

## **1. Prerequisites**

* Docker Desktop installed and running
* Node.js installed (for Express apps)
* Optional: [DBeaver Community Edition](https://dbeaver.io/download/) for GUI management

---

## **2. Create a Data Folder**

This folder will persist your PostgreSQL data:

```cmd
mkdir "C:\Users\Alvin Kayi\Desktop\PostgreSQL"
```

---

## **3. Run PostgreSQL Docker Container**

```cmd
docker run -d --name postgresql-local -p 5432:5432 -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin123 -e POSTGRES_DB=mydb -v "C:\Users\Alvin Kayi\Desktop\PostgreSQL:/var/lib/postgresql/data" postgres:15
```

* Container name: `postgresql-local`
* Port: `5432`
* Username: `admin`
* Password: `admin123`
* Database: `mydb`
* Volume mapping ensures data persists

Verify the container is running:

```cmd
docker ps
```

---

## **4. Access PostgreSQL via CMD**

Use Docker CLI to enter the PostgreSQL CLI:

```cmd
docker exec -it postgresql-local psql -U admin -d mydb
```

Example SQL commands:

```sql
CREATE TABLE test_table (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO test_table (name) VALUES ('Alvin');
SELECT * FROM test_table;
\q
```

---

## **5. Connect PostgreSQL to Node.js / Express**

1. Install PostgreSQL driver:

```cmd
npm install pg
```

2. Create `db.js`:

```js
const { Pool } = require("pg");

const pool = new Pool({
  user: "admin",
  host: "localhost",
  database: "mydb",
  password: "admin123",
  port: 5432,
});

module.exports = pool;
```

3. Create `app.js`:

```js
const express = require("express");
const pool = require("./db");

const app = express();
const port = 3000;

app.use(express.json());

app.get("/", async (req, res) => {
  try {
    const result = await pool.query("SELECT NOW()");
    res.json({ server_time: result.rows[0].now });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Database connection failed" });
  }
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

Run your Express server:

```cmd
node app.js
```

Open `http://localhost:3000/` — it should return the current PostgreSQL server time.

---

## **6. Using DBeaver (Optional GUI)**

1. Download DBeaver: [https://dbeaver.io/download/](https://dbeaver.io/download/)
2. Open DBeaver → Click **New Database Connection** → Select **PostgreSQL**
3. Enter connection details:

| Field    | Value     |
| -------- | --------- |
| Host     | localhost |
| Port     | 5432      |
| Database | mydb      |
| Username | admin     |
| Password | admin123  |

4. Click **Test Connection** → then **Finish**
5. You can now browse tables, run queries, and manage your database with GUI.

---

## **7. Stopping and Restarting PostgreSQL**

```cmd
docker stop postgresql-local
docker start postgresql-local
docker rm -f postgresql-local   # only if you want to remove the container
```

---

✅ **Notes**

* PostgreSQL runs locally; data persists in `C:\Users\Alvin Kayi\Desktop\PostgreSQL`.
* You can connect both via **CMD** and **DBeaver GUI**.
* Works for Node.js/Express or any other PostgreSQL client.

---

