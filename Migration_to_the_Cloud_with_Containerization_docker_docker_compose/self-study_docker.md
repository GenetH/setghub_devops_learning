# What is DOCKER?
### **Docker Overview:**
Docker is a platform that automates the deployment of applications inside lightweight containers. Containers encapsulate everything an application needs to run, ensuring consistency across different environments (development, testing, production).

#### **Key Docker Concepts:**
- **Docker Containers**: Isolated environments where applications run. Containers are created from Docker images.
- **Docker Images**: Templates containing the application and its dependencies.
- **Docker Engine**: The runtime that manages containers and images.
- **Docker Hub**: A cloud-based registry to store and share Docker images.

#### **Basic Docker Commands:**
- **Build an Image**:
  ```bash
  docker build -t <image-name> .
  ```
- **Run a Container**:
  ```bash
  docker run -d --name <container-name> <image-name>
  ```
- **List Running Containers**:
  ```bash
  docker ps
  ```
- **Stop a Container**:
  ```bash
  docker stop <container-name>
  ```
- **Remove a Container**:
  ```bash
  docker rm <container-name>
  ```
- **Remove an Image**:
  ```bash
  docker rmi <image-name>
  ```

---

### **Docker Compose Overview:**
Docker Compose is a tool that allows you to define and manage multi-container Docker applications. It uses a `docker-compose.yml` file to configure services, networks, and volumes.

#### **Key Docker Compose Concepts:**
- **Services**: Containers that run your application components (e.g., web servers, databases).
- **Networks**: Virtual networks connecting containers, allowing them to communicate.
- **Volumes**: Persistent storage that allows data to be shared between containers or persist beyond container restarts.

#### **Basic Docker Compose Commands:**
- **Start Containers**:
  ```bash
  docker-compose up
  ```
- **Stop Containers**:
  ```bash
  docker-compose down
  ```
- **Rebuild Containers**:
  ```bash
  docker-compose build
  ```
- **Run Commands in a Container**:
  ```bash
  docker-compose exec <service-name> <command>
  ```

---

### **Example: `docker-compose.yml` for a Web App**
This example defines a simple web app using NGINX and a PostgreSQL database:

```yaml
version: '3'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - webnet

  db:
    image: postgres:alpine
    environment:
      POSTGRES_USER: example
      POSTGRES_PASSWORD: example
    volumes:
      - dbdata:/var/lib/postgresql/data
    networks:
      - webnet

volumes:
  dbdata:

networks:
  webnet:
```

- **`web` service**: Runs NGINX, serving files from a local `html` directory.
- **`db` service**: Runs PostgreSQL and sets necessary environment variables (`POSTGRES_USER` and `POSTGRES_PASSWORD`).
- Both services are connected through a custom network `webnet`.

---

### **Why Use Docker and Docker Compose?**
- **Isolation**: Each container runs in its isolated environment, ensuring that dependencies and configurations don’t clash.
- **Portability**: Containers can run anywhere, whether on a developer’s local machine or a cloud server.
- **Scalability**: Easily scale services up or down using Docker Compose, ensuring your app can grow as needed.

---

### **Final Thoughts:**
Docker and Docker Compose are essential tools for modern application development. Docker simplifies containerizing applications, and Docker Compose makes it easy to manage multi-container environments. By mastering these tools, you can streamline your development workflows and deploy applications more consistently.
