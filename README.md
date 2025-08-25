🚀 Job Scraping Pipeline with Airflow (Docker Edition)
This project sets up a full job data scraping and enrichment pipeline using Apache Airflow and Docker , suitable for Windows development environments.

📁 Project Structure
project/
├── airflow-docker/           # Docker + Airflow setup
├── dags/                     # Airflow DAGs
│   └── job_pipeline_dag.py
├── src/data_gathering/       # Python scraping + processing scripts
│   ├── GlassdoorDataGathering.py
│   ├── JobSpy.py
│   └── ...
🧱 Setup Instructions
1. Clone or Create Docker Folder
If using the official Apache Airflow repo:

cd C:\metricsPipeline\project
git clone https://github.com/apache/airflow.git airflow-docker
Or create it manually:

mkdir airflow-docker
cd airflow-docker
2. Create docker-compose.yml
Inside airflow-docker/, create a file docker-compose.yml:

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
      AIRFLOW__WEBSERVER__SECRET_KEY: my_super_secret_key
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data

  webserver:
    build: .
    image: custom-airflow:latest
    restart: always
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CORE__FERNET_KEY: ''
      AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
      AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
      AIRFLOW__WEBSERVER__SECRET_KEY: qv3p3v9Iw3YT6KvlXzq2KiZEq0PSoA9kQff8Pym4pQM=
      _PIP_ADDITIONAL_REQUIREMENTS: -r /requirements.txt
    volumes:
      - ../dags:/opt/airflow/dags
      - ../src:/opt/airflow/src
      - ../requirements.txt:/requirements.txt
    ports:
      - "8080:8080"
    command:  >
      bash -c "
        pip install -r /requirements.txt &&
        airflow webserver
      "

  scheduler:
    build: .
    image: custom-airflow:latest
    restart: always
    depends_on:
      - webserver
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__WEBSERVER__SECRET_KEY: qv3p3v9Iw3YT6KvlXzq2KiZEq0PSoA9kQff8Pym4pQM=
      _PIP_ADDITIONAL_REQUIREMENTS: -r /requirements.txt
    volumes:
      - ../dags:/opt/airflow/dags
      - ../src:/opt/airflow/src
      - ../requirements.txt:/requirements.txt
    command: >
      bash -c "
        pip install -r /requirements.txt &&
        airflow scheduler
      "

  airflow-init:
    build: .
    image: custom-airflow:latest
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__WEBSERVER__SECRET_KEY: qv3p3v9Iw3YT6KvlXzq2KiZEq0PSoA9kQff8Pym4pQM=
    volumes:
      - ../dags:/opt/airflow/dags
      - ../src:/opt/airflow/src
    entrypoint: >
      bash -c "
        airflow db init &&
        airflow users create --username airflow --password airflow --firstname Admin --lastname User --role Admin --email admin@example.com
      "
volumes:
  postgres-db-volume:
3. Initialize Airflow Database
cd airflow-docker
I want to delete previous volumen
docker-compose down -v
docker-compose up --build -d


docker build --no-cache -t custom-airflow .

docker-compose up airflow-init
✅ This will:

Initialize Airflow DB
Create the admin user (airflow / airflow)
4. Start Airflow Services
docker-compose up -d
Key Differences:
Rebuild Behavior :
docker-compose up --build: Forces a rebuild of the Docker images before starting the containers.
docker-compose up -d: Starts the containers using the already built images, without rebuilding them.
Running Mode :
docker-compose up --build: Runs in the foreground by default, showing the logs in the terminal (unless -d is added).
When to Use Each:
Use docker-compose up --build if you’ve made changes to the Dockerfile or any files that affect the image, and you need to ensure that the new changes are included when the containers are started.
Use docker-compose up -d if the images have already been built and you just want to start the containers in the background without rebuilding them.
Then open your browser:

http://localhost:8080
Login with:

Username: airflow
Password: airflow
in thedocker file
Use your base Airflow image
FROM apache/airflow:2.7.2-python3.10

USER root

Install dependencies for Chrome
RUN apt-get update && apt-get install -y wget curl unzip gnupg fonts-liberation libatk-bridge2.0-0 libnspr4 libnss3 libxss1 libasound2 xdg-utils && rm -rf /var/lib/apt/lists/*

Install Google Chrome stable
RUN wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add - && echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list && apt-get update && apt-get install -y google-chrome-stable && rm -rf /var/lib/apt/lists/*

Set Chrome path for Selenium
ENV CHROME_BIN=/usr/bin/google-chrome ENV PATH="$PATH:/usr/bin/google-chrome"

USER airflow

Copy Python dependencies
COPY requirements.txt . RUN pip install --no-cache-dir -r requirements.txt

5. Trigger Your DAG
Locate your DAG in the UI (job_pipeline_dag)
Toggle it ON
Click ▶️ to run it
View task status, logs, and Gantt chart
🧠 What This Does
This pipeline:

Scrapes jobs from Glassdoor and JobSpy
Cleans and deduplicates data
Translates French descriptions
Uses a local LLM to extract skills and contract types
Loads the final data into MongoDB
✅ You're Production Ready!
Your Airflow pipeline is now:

🕓 Schedulable (e.g., daily)
🔁 Retryable on failure
📊 Trackable via Airflow UI
Want to go further?

📈 Add Prometheus + Grafana monitoring
☁️ Deploy to cloud with Render, Railway, or EC2
📬 Add Slack or Email alerts
Let me know if you want to keep building! 💡

📈 Bonus: Add Metrics Later (Optional)
Once you're comfortable with this:

Add Prometheus exporter to Airflow
Or use Airflow's built-in task duration + success metrics
Export logs for each task from the UI or store in a database for monitoring
🐳 Docker Cheat Sheet for Airflow Projects
🔧 Docker Build & Compose
Command	Description
docker build -t <image_name> .	Build a Docker image from a Dockerfile
docker-compose up	Start all services defined in docker-compose.yml
docker-compose up -d	Start services in detached mode
docker-compose down	Stop and remove containers, networks, and volumes
docker-compose up --build	Rebuild images and start services
docker-compose ps	List running services in the compose setup
🧱 Container Management
Command	Description
docker ps	List running containers
docker ps -a	List all containers (running and stopped)
docker stop <container_name>	Stop a container
docker start <container_name>	Start a stopped container
docker restart <container_name>	Restart a container
docker rm <container_name>	Remove a container
docker rmi <image_name>	Remove an image
🖥️ Accessing Containers
Command	Description
docker exec -it <container_name> bash	Start an interactive shell inside a container
docker exec <container_name> <command>	Run a command inside a running container
exit	Exit the container shell
docker logs <container_name>	View logs from a container
🧼 Volumes & Cleanups
Command	Description
docker volume ls	List Docker volumes
docker volume rm <volume_name>	Remove a specific volume
docker system prune -a	Remove all stopped containers, unused networks/images (dangerous)
docker network ls	List Docker networks
📁 Useful Paths in Airflow Containers
Path	Description
/opt/airflow/dags	Location for DAG files
/opt/airflow/src	Custom scripts (e.g. job scrapers)
/requirements.txt	Path to the Python requirements file inside container
✅ Commands You've Used So Far
Command	Purpose
docker exec -it airflow-docker-webserver-1 bash	Open shell inside webserver container
pip install -r /requirements.txt	Install Python packages inside the container
pip list	See installed Python packages
exit	Exit container terminal
docker-compose up --build	Rebuild and start services
docker-compose down	Stop and clean up all containers
✅ STEP 1: Create prometheus.yml
Create a file named prometheus.yml in the same directory as your docker-compose.yml.

Paste this config inside:

yaml
CopyEdit
global:
  scrape_interval:15s

scrape_configs:
  -job_name:'statsd-exporter'
    static_configs:
      -targets: ['statsd:9102']
This tells Prometheus to scrape metrics from the statsd container.

✅ STEP 2: Start Everything
Run from your project root:

bash
CopyEdit
docker-compose up --build
✅ STEP 3: Verify Services Are Running
Airflow UI: http://localhost:8080
Prometheus UI: http://localhost:9090
Grafana UI: http://localhost:3000
✅ STEP 4: Set Up Grafana
Go to http://localhost:3000
Login:
Username : admin
Password : admin
Add a new Data Source :
Type: Prometheus
URL: http://prometheus:9090
Click Save & Test
✅ STEP 5: Import Airflow Dashboard
Grafana dashboards are available in the community. Use a popular one:

Click “+” → Import
Paste Dashboard ID 14674
Click Load
Choose Prometheus as the data source
Click Import
Once this is done, you'll have beautiful metrics like:

DAG Success / Failure rates
Task durations
Scheduler health
And more!
