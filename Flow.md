##  Building End-to-end ETL Pipeline using Apache Airflow
-------------
To be Covered:
- What is Astro?
- What is Airflow?
- ETL Pipeline Implementation
- Deploy ETL Pipeline using Astro and AWS

-------------
Helpful Links:
- https://www.astronomer.io/docs/astro


Introduction:
-------------
##### A. ETL Pipeline: (Extract Transform Load)
- Understanding:
    - Extract: To extract data from multiple sources (DB, IOT, API's etc..)
    - Transform: Performing Data Wrangling and Opertions to Transform data into a certian format (For e.g. Parquet, JSON, pandasDF etc..)
    - Load: Load & Store the data into a Storage (E.g. MongoDB, PostGreSQL, S3....)

- ETL pipelines need to scheduled to run on Periodic Basis
    - Because,
        - Data Source keeps getting updated
        - Data Load frequency requirement - Hourly, Weekly, Monthly (Automated Data Load is required)

- Now, ETL pipelines should be scheduled to run in a Specific **Workflow** -> Use **Apache Airflow**

##### B. Apache Airflow
- Airflow: Open Source Platform
- Use to Author, Schedul, and manage workflows
- To Create Workflows
    - Use **DAG** [Directed Acyclic Graph]

- DAG [Directed Acyclic Graph]
    - In ETL workflow, each step = 1 Task
    - In this case, 
        - Task-1: Pull data from API 
        - Task-2: Perform Data Transformation
        - Task-3: Store Data in PostgreSQL
    - Each Task is **Directed** (One by One)
    - **No Cyclic operations** can be performed, in Airflow.


##### C. Astronomer.IO(Astro)
- Manages entire Airflow
    - If "Airflow" Workflow is need to run as a Docker -> Astronomer plays a crucial role.
- Airflow job is Deployed using Astro cloud 

------------
##### API Working
- Working: We will refer to a Weather API (Open-source)
    - Once we Provide 'lat' 'lon' 
    - We should get some 'Climatic Information'

##### Project Implementation
**[Step-1]: Setup**
1. Install Astro
- > brew install astro
2. Restart VS Code and Initialize Astro
- > astro dev init
- It Initializes, an empty astro project with Airflow and all required file systems

**[Step-2]]: Create a DAG**
- Go to 'Dags' Folder -> Create new file 'etlweather.py'
    1. Make Imports
        - > from airflow import **DAG** // To create Directed Cyclic Graph (Airflow Job)
        - > from airflow.**providers.https.hook.http** import **HttpHook** // for Task-1, to Call API and extract data
            - As API = Http request, "Airflow" has hooks, for **Http**, that can call the API
        - > from airflow.**providers.postgres.hook.postgres** import **PostgresHook** // for Task-3, to store data into PostgreSQL
        - > from airflow.**decorators** import **task** // To Create Task for each ETL step
        - > from airflow.**utils.dates** import **days_ago** // To manages days for workflow
    
    2. Establish **(I/P + Connection)** Parameters + **Default Arguments**

    3. With DAG (params) - Create TASKS (3 for ETL pipeline)
        - use **with DAG(params)** // Give DAG attributes
            - params: 
                - dag_id='weather_etl_pipeline',  // Any Workflow Name  
                - default_args= default_args, // Arguments to Establish DAG
                - schedule_interval = '@daily', // Establish Frequency
                - catchup=False
        - Define 3 TASKS(Methods). Use **task()** decorator for each TASK
            1. extract_weather_data()
                - Use HTTP Hook to get weather Data // Gets connection details from Airflow (Like a Config.)
                - Build the API endpoint
                - Receive Response using **HttpHook**
                - Use 'if-else' for response O/P based on Status
            2. transform_weather_data(weather_data)
                - Take 'weather_data' as I/P & transform it 
            3. load_weather_data(transformed_data)
                - To load transformed data into PQSQL
                - Use **PostgresHook** to establish connection
                - Use 'cursor' to navigate SQL-Python commands.
                - Execute 'CREATE TABLE' & 'INSERT INTO VALUES' command to store data.
    
    4. Establish DAG Flow
        - Call Above Methods/Tasks in ETL sync
    
**[Step-3]]: Create a DOCKER Compose.yml**
- To Orchestrate the DAG Run
- Create a 'docker-compose.yml' file 
    - Add Code block w.r.t. "PostGre"
        - Create **volumes** // So that even if the program stops & restarts -> the data is not lost.
            - works as a Back-Up

**[Step-4]: Run the Stack**
- > astro dev start
- It starts the development of the **Airflow package 1st**, then starts the building of **"Postgres" connection** as well
- Result: Apache Airflow will open in Local env. on Browser. 
    - Will be able to see DAG, and info. related to it

**[Step-5]: Airflow Config Setup**
- Setup Connection details for API & PostGre SQL
- For "PostGreSQL"
    - Go to "Admin" -> "Connection" 
        - Connection Id: postgres_default
        - Connection Type: Postgres
        - Host:etl-pipeline-airflow_c86c7d-postgres-1(Docker-Container-Postgres-name)
        - Database, Login, Password, Port - Get details from "Docker-Compose.yml"

- For "API"
    - Go to "Admin" -> "Connection" 
        - Connection Id: open_meteo_api
        - Connection Type: HTTP
        - Host: https://api.open-meteo.com/
        - Database, Login, Password, Port - Not required

**[Step-6]: Manual Run the Airflow Job**
- Check If Any Errors.
- **Xcom** Shows the O/P for each Task
- Check **Logs**, and **Graphs**
- Check "PostGreSQL" 
    - PQSQL is running on Docker Containers 
    - Download DBeaver -> Establish connection with PQSQL (Give User name, password from docker-compose.yml)
    - Run "select * from weather_data" to check if Data is available or not
    - If Error Occurs with JDBC Driver settings - refer "https://github.com/dbeaver/dbeaver/issues/3477"


    



    [NOTES]
    1. **Airflow** Provides, **Hooks to Call/Connect any Data Source** (API, S3, MongoDB etc..)



