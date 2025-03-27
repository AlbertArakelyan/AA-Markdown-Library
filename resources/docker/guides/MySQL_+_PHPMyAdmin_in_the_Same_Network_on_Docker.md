# MySQL + PHPMyAdmin in the Same Network on Docker

Running **MySQL** and **PHPMyAdmin** containers in the same **Network** on **Docker**.

<aside>
💡

Please note everything is created from scratch, no **network** or **container** existed before.

</aside>

# Creating network and containers

## 1. **Create a Custom Network** (If Not Already Done)

```bash
docker network create my_bridge_network
```

This network allows containers to communicate by name.

## 2. **Run the First Container** (e.g., MySQL)

Let’s start the **MySQL** container on the same network:

```bash
docker run --name ecom_mysql -p 3306:3306 --network my_bridge_network -e MYSQL_ROOT_PASSWORD=11111111 -d mysql
```

**Key Options:**

- `--network my_bridge_network`: Ensures this container is part of the custom network.
- `-e MYSQL_ROOT_PASSWORD=11111111`: Sets the root password for **MySQL**.
- `-p 3306:3306` Exposes **MySQL** on port `3306` of your host.

## 3. **Run the Second Container** (e.g., phpMyAdmin)

Now, start the **phpMyAdmin** container on the same network, linking it to the MySQL container:

```bash
docker run --name my_phpmyadmin --network my_bridge_network -d -p 8081:80 \
    -e PMA_HOST=ecom_mysql \
    -e PMA_PORT=3306 \
    -e PMA_USER=root \
    -e PMA_PASSWORD=yourpassword \
    phpmyadmin/phpmyadmin
```

**Key Options:**

- `--network my_bridge_network`: Ensures this container is part of the same custom network as `ecom_mysql`.
- `-e PMA_HOST=ecom_mysql`: Specifies the MySQL container’s name as the host for **phpMyAdmin**.
- `-p 8081:80`: Exposes phpMyAdmin on port `8081` of your host.

## 4. **Verify the Setup**

- Check both containers are on the same network:
    
    ```bash
    docker network inspect my_bridge_network
    ```
    
    You should see both `ecom_mysql` and `my_phpmyadmin` in the output. More correctly you should see an object like this as an output:
    
    ```bash
    [
        {
            "Name": "my_bridge_network",
            "Id": "09de01b430b68d31c02b29e4bc323b0504a7e9d635ec430aaea56ebf9826b2d2",
            "Created": "2024-12-25T17:41:50.585739158Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.19.0.0/16",
                        "Gateway": "172.19.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "8e98c1ac69765e41720e76a8362961f4e9acc5104c3c4fc2eb47bc2635714d0f": {
                    "Name": "ecom_mysql",
                    "EndpointID": "d174fb2ad445742ae480dc1c8cdc86c2685c5305663ffb6a838cdaeda8196c53",
                    "MacAddress": "02:42:ac:13:00:02",
                    "IPv4Address": "172.19.0.2/16",
                    "IPv6Address": ""
                },
                "b3548436cde55b345962487252d9e29f942474dcd68cef0dc9a413bb39470f6a": {
                    "Name": "my_phpmyadmin",
                    "EndpointID": "a85259e80a4ef0502256aa07f6ca7960a8ce783965a4ca1e8a783634d07db1a2",
                    "MacAddress": "02:42:ac:13:00:03",
                    "IPv4Address": "172.19.0.3/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
    ```
    
- Access phpMyAdmin: Open your browser and go to:
    
    ```
    http://localhost:8081
    ```
    

You should now be able to connect to your **MySQL** container (`ecom_mysql`) from **phpMyAdmin**.

---

# Interacting with Database from Terminal

Here’s how you can enter the terminal of the **MySQL** container, log in to MySQL, and create or delete a table as an example.

## 1. **Enter the Terminal of the MySQL Container**

Run the following command to get into the terminal of the MySQL container:

```bash
docker exec -it ecom_mysql bash
```

This will open a bash terminal inside your `ecom_mysql` container.

<aside>
💡

Note that `docker exec -it <container_name> bash` is a common practice for entering the **terminal of the container**.

</aside>

## 2. **Log In to MySQL**

Once inside the container, log in to the MySQL client:

```bash
mysql -u root -p
```

When prompted, enter the password you set for `MYSQL_ROOT_PASSWORD` during the container setup (in our case `11111111`).

## 3. **Create a Database**

Let’s create a new database named `example_db`:

```sql
CREATE DATABASE example_db;
```

Switch to the new database:

```sql
USE example_db;
```

## 4. **Create a Table**

Here’s an example to create a table named `users`:

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert some sample data into the table:

```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com'), ('Jane Smith', 'jane@example.com');
```

View the data:

```sql
SELECT * FROM users;
```

## 5. **Delete a Table**

To delete the `users` table, run:

```sql
DROP TABLE users;
```

## 6. **Exit MySQL and the Container**

- Exit the MySQL client:
    
    ```sql
    EXIT;
    ```
    
- Exit the container’s terminal:
    
    ```bash
    exit
    ```
    

---

### Summary Commands Cheat Sheet:

```bash
# Enter MySQL container
docker exec -it ecom_mysql bash

# Log in to MySQL
mysql -u root -p

# Create a database
CREATE DATABASE example_db;

# Switch to the database
USE example_db;

# Create a table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Insert sample data
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

# View data
SELECT * FROM users;

# Delete a table
DROP TABLE users;

# Exit MySQL
EXIT;

# Exit container
exit
```