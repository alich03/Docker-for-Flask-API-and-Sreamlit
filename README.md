Absolutely! I’m happy to guide you through learning **Docker Compose** and show you how to run multiple services, like a **Flask API** and a **Streamlit app**, in separate containers using Docker Compose.

### What is Docker Compose?
**Docker Compose** is a tool that helps you define and run multi-container Docker applications. Using a **`docker-compose.yml`** file, you can define all the services (containers) needed for your application, their configurations, and how they communicate with each other.

With Docker Compose, you can:
- Define and run multiple containers with a single command.
- Link containers (e.g., Flask API and Streamlit) so they can communicate.
- Simplify the management of complex multi-container applications.

---

### Basic Docker Compose File Structure

A **`docker-compose.yml`** file defines the configuration of all containers (services) and their interactions. It uses **YAML** syntax, making it human-readable.

A typical structure looks like this:

```yaml
version: '3.8'  # Version of Docker Compose syntax

services:
  <service_name>:  # The name of the service (container)
    build: .  # Path to the Dockerfile (usually '.' if it's in the current directory)
    ports:
      - "<host_port>:<container_port>"  # Port mapping between host and container
    environment:
      - VAR_NAME=value  # Environment variables (if needed)
    networks:
      - mynetwork  # Network for communication between containers

networks:
  mynetwork:  # Custom network to allow containers to communicate
    driver: bridge  # Default network driver
```

---

### Step-by-Step: Running Flask API and Streamlit App in Separate Containers Using Docker Compose

Now, let’s walk through how to containerize the **Flask API** and **Streamlit app** you already created, and run them in separate containers using Docker Compose.

### Step 1: Project Directory Setup

First, let’s structure the project directory to include both your Flask API and Streamlit app, along with the `docker-compose.yml` file.

```
docker-compose-project/
│
├── flask-api/
│   ├── app.py                # Flask API code
│   ├── requirements.txt      # Flask dependencies
│   └── Dockerfile            # Dockerfile for Flask API
│
├── streamlit-app/
│   ├── app.py                # Streamlit app code
│   ├── requirements.txt      # Streamlit dependencies
│   └── Dockerfile            # Dockerfile for Streamlit app
│
└── docker-compose.yml        # Docker Compose configuration file
```

### Step 2: Create Dockerfiles for Both Services

#### Flask API Dockerfile (`flask-api/Dockerfile`):

```Dockerfile
# Use Python slim image
FROM python:3.9-slim

# Set working directory inside the container
WORKDIR /app

# Copy requirements.txt and install dependencies
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copy the Flask app code
COPY . /app/

# Expose Flask's default port
EXPOSE 5000

# Command to run the Flask app
CMD ["python", "app.py"]
```

#### Streamlit App Dockerfile (`streamlit-app/Dockerfile`):

```Dockerfile
# Use Python slim image
FROM python:3.9-slim

# Set working directory inside the container
WORKDIR /app

# Copy requirements.txt and install dependencies
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copy the Streamlit app code
COPY . /app/

# Expose Streamlit's default port
EXPOSE 8501

# Command to run the Streamlit app
CMD ["streamlit", "run", "app.py", "--server.enableCORS=false"]
```

---

### Step 3: Create the `docker-compose.yml` File

Now, create a `docker-compose.yml` file that will run both the Flask API and Streamlit app in separate containers.

```yaml
version: '3.8'

services:
  flask-api:
    build: ./flask-api  # Flask API's Dockerfile path
    ports:
      - "5000:5000"  # Expose Flask API on port 5000
    networks:
      - app-network  # Use a custom network to allow communication between containers

  streamlit-app:
    build: ./streamlit-app  # Streamlit app's Dockerfile path
    ports:
      - "8501:8501"  # Expose Streamlit app on port 8501
    depends_on:
      - flask-api  # Make sure Flask API starts first
    networks:
      - app-network  # Same network to allow communication

networks:
  app-network:
    driver: bridge  # Default network driver, allows containers to communicate
```

### Explanation of the `docker-compose.yml`:

- **`services`**: Defines the two services (`flask-api` and `streamlit-app`).
- **`build`**: Path to the respective Dockerfile for each service.
- **`ports`**: Maps the container port to the host machine port, so we can access the services locally.
- **`depends_on`**: Ensures that the Flask API starts before the Streamlit app.
- **`networks`**: Creates a custom network (`app-network`) to allow communication between containers.

---

### Step 4: Building and Running the Containers

With the `docker-compose.yml` file in place, you can now build and run the services using Docker Compose.

#### 4.1 Build the services
Run this command from the root directory (`docker-compose-project`):

```bash
docker-compose build
```

This will build the Docker images for both the Flask API and the Streamlit app based on the Dockerfiles.

#### 4.2 Run the containers
After the build is successful, you can start the containers with:

```bash
docker-compose up
```

This will start both the Flask API and the Streamlit app containers, and they will be accessible at the following URLs:

- Flask API: `http://localhost:5000`
- Streamlit app: `http://localhost:8501`

### Step 5: Communicating Between Containers (Optional)

You may want to have your **Streamlit app** communicate with the **Flask API**. For example, in `streamlit-app/app.py`, you could use Python's `requests` library to call the Flask API.

Example of `streamlit-app/app.py`:

```python
import streamlit as st
import requests

# Streamlit app title
st.title("Streamlit App Communicating with Flask API")

# Call the Flask API
response = requests.get("http://flask-api:5000")  # Use Flask service name as hostname
if response.status_code == 200:
    st.write("Flask API Response:", response.json())
else:
    st.write("Error connecting to Flask API.")
```

Notice that in Docker Compose, the **Flask API service** is reachable by the service name (`flask-api`) as the hostname.

### Step 6: Stopping and Cleaning Up

After testing, you can stop the running containers with:

```bash
docker-compose down
```

This stops and removes the containers, networks, and volumes created by `docker-compose up`.

---

### Final Directory Structure:

```
docker-compose-project/
│
├── flask-api/
│   ├── app.py                # Flask API code
│   ├── requirements.txt      # Flask dependencies
│   └── Dockerfile            # Dockerfile for Flask API
│
├── streamlit-app/
│   ├── app.py                # Streamlit app code
│   ├── requirements.txt      # Streamlit dependencies
│   └── Dockerfile            # Dockerfile for Streamlit app
│
└── docker-compose.yml        # Docker Compose configuration file
```

---

### Conclusion:
You’ve learned how to:
- Set up a **multi-container application** with **Docker Compose**.
- Containerize both a **Flask API** and a **Streamlit app** in separate containers.
- Allow containers to communicate through a custom network.
- Build and run the multi-container app using Docker Compose.

This is just the start. Docker Compose also allows advanced configurations such as volumes for persistent data storage, scaling services, and more.

Let me know if you have any questions or if you’d like to explore other features!
