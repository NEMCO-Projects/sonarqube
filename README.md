Here’s a **README.md** you can drop into the root of your SonarQube setup directory to document the entire process.

````markdown
# SonarQube on Ubuntu with Docker Compose

This guide walks through installing and testing **SonarQube (Community Edition)** on Ubuntu using Docker Compose and running a sample scan.

---

## Prerequisites
- Ubuntu 20.04+ with Docker and Docker Compose installed  
  ```bash
  sudo apt update
  sudo apt install docker.io docker-compose-plugin -y
  sudo systemctl enable --now docker
````

* At least 2 GB of RAM

---

## 1️⃣ Directory Setup

Create a folder to hold the Compose file and volumes:

```bash
mkdir ~/sonarqube
cd ~/sonarqube
```

---

## 2️⃣ Create `docker-compose.yml`

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15
    container_name: sonarqube-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - postgres_data:/var/lib/postgresql/data

  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    restart: unless-stopped
    depends_on:
      - postgres
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://postgres:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    ports:
      - "9000:9000"
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs

volumes:
  postgres_data:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
```

---

## 3️⃣ Start the Stack

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

View logs until you see `SonarQube is operational`:

```bash
docker logs -f sonarqube
```

---

## 4️⃣ Access the Web UI

Open a browser:

```
http://<server-ip>:9000
```

Login with:

* **Username:** `admin`
* **Password:** `admin` (you’ll be asked to change it)

Generate a token for scans:

* Go to **My Account → Security → Generate Tokens**
* Copy the token (40-character string)

---

## 5️⃣ Health Check via API

```bash
curl -u admin http://localhost:9000/api/system/health
```
here admin user password will be asked provide password which is set into step 4
Should return:

```json
{"health":"GREEN"}
```

---

## 6️⃣ Run a Sample Scan

Install Java (required by Sonar Scanner):

```bash
sudo apt install openjdk-17-jre -y
```

Download and add Sonar Scanner to PATH:

```bash
sudo apt install unzip
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006.zip
unzip sonar-scanner-cli-5.0.1.3006.zip
export PATH=$PATH:$PWD/sonar-scanner-cli-5.0.1.3006.zip/bin
```

Create a sample project:

```bash
mkdir sample-project && cd sample-project
echo "print('Hello SonarQube')" > hello.py
```

Run the scanner (replace `<TOKEN>` with your actual token, **no angle brackets**):

```bash
sonar-scanner \
  -Dsonar.projectKey=sample \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=squ_f95660038a9a946a924f864171e54b5b43830508
```

When the scan completes, refresh the SonarQube UI to see the analysis.

---

## 7️⃣ Maintenance

* **Stop:** `docker compose down`
* **Update images:**

  ```bash
  docker compose pull
  docker compose up -d
  ```

---

## Notes

* Adjust memory if needed; SonarQube typically needs at least 2 GB RAM.
* Use `sonarqube:<version>-community` if you need a fixed version.

````

Save this as `README.md` in your `~/sonarqube` directory:
```bash
nano ~/sonarqube/README.md
# (paste content, save, exit)
````

You now have a complete reference for setup, testing, and maintenance.
