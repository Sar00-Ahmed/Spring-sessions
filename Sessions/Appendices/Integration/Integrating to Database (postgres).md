# Via docker command line
### PostgreSQL & pgAdmin (via Docker)

We'll run PostgreSQL and pgAdmin (a web-based GUI for PostgreSQL) as Docker containers.

1.  **Create a Docker Network:** This allows your database and pgAdmin containers to communicate securely. Replace `your-network-name` with a descriptive name (e.g., `abc-network`).

    ```bash
    sudo docker network create your-network-name
    ```

2.  **Run PostgreSQL Container:**
    Replace `yourPassword` with a strong password for the PostgreSQL `postgres` user, and `your-network-name` with the network you created.

    ```bash
    sudo docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=yourPassword -d --restart unless-stopped --network your-network-name postgres
    ```

      * `--name postgres`: Assigns the name "postgres" to the container, making it easy to reference.
      * `-p 5432:5432`: Maps port 5432 on your host to port 5432 in the container (PostgreSQL's default port).
      * `-e POSTGRES_PASSWORD=yourPassword`: Sets the password for the default `postgres` user.
      * `-d`: Runs the container in detached mode (in the background).
      * `--restart unless-stopped`: Ensures the container restarts automatically unless explicitly stopped.
      * `--network your-network-name`: Connects the container to your custom Docker network.
      * `postgres`: Specifies the Docker image to use.
3.  **Run pgAdmin Container:**
Replace `yourEmail@ABC.com` with your email, `yourPassword` with a password for pgAdmin login, and `your-network-name` with your custom network name.

```bash
sudo docker run -p 8080:80     -e 'PGADMIN_DEFAULT_EMAIL=yourEmail@ntgclarity.com' --restart unless-stopped    -e 'PGADMIN_DEFAULT_PASSWORD=yourPassword'  --network networkName     -d --name pgadmin dpage/pgadmin4
    ```

* `-p 5050:80`: Maps port 5050 on your host to port 80 in the container (pgAdmin's default web port).
* `-e 'PGADMIN_DEFAULT_EMAIL=yourEmail@ntgclarity.com'`: Sets the default login email for pgAdmin.
* `-e 'PGADMIN_DEFAULT_PASSWORD=yourPassword'`: Sets the default login password for pgAdmin.
 * `dpage/pgadmin4`: Specifies the pgAdmin Docker image.

# Docker Compose (Recommended)
``` yml
services:
  postgres:
    image: postgres
    container_name: postgres-db1
    ports:
      - "8015:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    environment:
      - POSTGRES_USER=rootuser 
      - POSTGRES_PASSWORD=rootpass
      - POSTGRES_DB=Academy
    networks:
      - db-network

  pgadmin:
    image: dpage/pgadmin4:7
    container_name: pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin123
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    ports:
      - "8080:80"
    depends_on:
      - postgres
    networks:
      - db-network
        
volumes:
  postgres_data:
    driver: local
  pgadmin_data:
    driver: local
        
networks:
  default:
    name: db-network
```



4.  **Access pgAdmin:**
    Open your web browser and go to `http://localhost:8080.` 
    Sign in using the email and password you set in the `PGADMIN_DEFAULT_EMAIL` and `PGADMIN_DEFAULT_PASSWORD` environment variables.

5.  **Configure PostgreSQL Server in pgAdmin:**

      * In pgAdmin, under "Servers", right-click and select **Create** -\> **Server...**.
      * In the "General" tab, give it a **Name**, e.g., `PostgresLocal`.
      * Go to the "Connection" tab and fill in the following details:
          * **Hostname/address:** `postgres` (This is the name of your Docker container for PostgreSQL, as defined by `--name postgres`).
          * **Port:** `5432`
          * **Username:** `postgres`
          * **Password:** `yourPassword` (The password you set in the `POSTGRES_PASSWORD` environment variable when running the PostgreSQL container).
      * Click **Save**.

6.  **Create Databases:**
    You'll need to create two databases within your newly connected PostgreSQL server:
# Common pitfalls
when configuring docker ports for postgres:
port: `8015:5432`

**5432 (internal)**: This is the standard PostgreSQL port **inside the container**.
 - This should remain 5432 as PostgreSQL inside the container expects to run on this port
- If changed, it will require additional configuration
**8015 (external)**: This is the port on your **host machine** that maps to the container
- You can use any available port on your host
- Change this if the default 5432 is already used on your host

> [!warning]
> Docker WILL create the container even if the host port is already in use, but with problematic behavior. And you will see conflicts and authentication errors if you try to connect to postgres via this port if this is the case because it is communicating with the old port not the one created by docker.
