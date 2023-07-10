# Airflow
open the VS Code in  your system and powershell . in  powershell create a directory in D drive
cd D:
mkdir AirFlowDocker
--> you can open the folder from D drive in VS CODE - AirFlowDocker.
open it and 
here is VS Code make new file under AirFlowDocker docker-compose.yml
#yml or yaml both acts as a same extension.
--->Here we will write the docker file in docker-compose.yml file
#code in VS in docker-compose.yml file.
version: '3'

x-airflow-common:
  &airflow-common
  image: apache/airflow:2.1.1-python3.8
  environment:
    &airflow-common-env
    AIRFLOW__CORE__SQL_ALCHEMY_CONN: sqlite:////usr/local/airflow/db/airflow.db
    AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
    - ./db:/usr/local/airflow/db
  user: "${AIRFLOW_UID:-50000}:${AIRFLOW_GID:-50000}"

services:
  airflow-init:       #it uses to crrate a user fir it-only job.
    <<: *airflow-common   #act like a pointer it will go to airlfow-common and will have image.
    entrypoint: /bin/bash -c "/bin/bash -c \"$${@}\""
    command: |
      /bin/bash -c "
        airflow db init
        airflow db upgrade
        airflow users create -r Admin -u admin -e airflow@airflow.com -f admin -l user -p airflow
      "
    environment:
      <<: *airflow-common-env

  airflow-scheduler:
    <<: *airflow-common
    command: scheduler
    environment:
      <<: *airflow-common-env
    restart: always
    depends_on:
      airflow-webserver:
        condition: service_healthy

  airflow-webserver:
    <<: *airflow-common
    command: webserver
    ports:
      - 8081:8080
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
      interval: 10s
      timeout: 10s
      retries: 5
    restart: always
    environment:
      <<: *airflow-common-env

