# Serverless Data Lake Day Workshop Documentation

This project documents my experience working on the Serverless Data Lake Day workshop, a hands-on introduction to building a cloud-native, serverless data lake using AWS services. As a beginner in data engineering, I focused on ingesting, cataloging, transforming, and analyzing large datasets without managing infrastructure. The workshop uses the GDELT dataset (170GB of global events data) and covers AWS Glue, Kinesis, S3, Athena, and Quicksight.

![Background](images/overview/image2.png)

## Table of Contents

- [1. Workshop Overview](#1-workshop-overview)
  - [1.1. Key AWS Services Used](#11-key-aws-services-used)
  - [1.2. Solution Architecture](#12-solution-architecture)
- [2. Prerequisites](#2-prerequisites)
  - [2.1. AWS Account Setup](#21-aws-account-setup)
  - [2.2. Create an AWS IAM User](#22-create-an-aws-iam-user)
  - [2.3. Deploy Resources via AWS CloudFormation](#23-deploy-resources-via-aws-cloudformation)
- [3. Lab 1: Data Ingestion & Storage](#3-lab-1-data-ingestion--storage)
- [4. Lab 2: Data Cataloging and ETL](#4-lab-2-data-cataloging-and-etl)
  - [4.1. Lab 2.1: Cataloging Your Data](#41-lab-21-cataloging-your-data)
    - [4.1.1. Create Crawler to Auto Discover Schema](#411-create-crawler-to-auto-discover-schema-of-your-data-in-s3)
    - [4.1.2. Edit the Metadata Schema](#412-edit-the-metadata-schema)
  - [4.2. Lab 2.2: Visually Transform Data with Drag-and-Drop](#42-lab-22-visually-transform-data-with-drag-and-drop)
    - [4.2.1. Create Transformation Job with Glue Studio](#421-create-transformation-job-with-glue-studio)
    - [4.2.2. Run Transformation Script](#422-run-transformation-script)
    - [4.2.3. Monitor and Review Job Execution](#423-monitor-and-review-job-execution)
  - [4.3. Lab 2.3: Interactive ETL Code Development](#43-lab-23-interactive-etl-code-development)
    - [4.3.1. Interactive Manipulations with Glue Serverless Spark](#431-interactive-manipulations-with-glue-serverless-spark)
    - [4.3.2. Automating ETL Tasks with Triggers](#432-automating-etl-tasks-with-triggers)
- [5. Lab 3: Data Analytics & Visualization](#5-lab-3-data-analytics--visualization)
  - [5.1. Lab 3.1: SQL Analytics on Large Scale Open Dataset](#51-lab-31-sql-analytics-on-a-large-scale-open-dataset)
    - [5.1.1. Create Glue DB for GDELT Schema using Athena HIVE DDL](#511-create-glue-db-for-gdelt-schema-using-athena-hive-ddl)
    - [5.1.2. Query GDELT EVENTS Data](#512-query-gdelt-events-data)
  - [5.2. Lab 3.2: Data Visualization](#52-lab-32-data-visualization)
  - [5.3. Lab 3.3: (Optional) Connecting to Athena with 3rd Party BI Tools](#53-lab-33-optional-connecting-to-athena-with-3rd-party-bi-tools-via-jdbc)
- [6. Issues During Progress](#6-issues-during-progress)
  - [6.1. Instruction Misunderstandings](#61-instruction-misunderstandings)
  - [6.2. Resource Management Challenges](#62-resource-management-challenges)
- [7. Conclusion](#7-conclusion)

---

## 1. Workshop Overview

The workshop emphasizes modern data architectures to handle explosive data growth, consolidating silos into scalable data lakes for analytics and ML. Key pillars include:

- scalable storage
- purpose-built services
- unified access/governance
- cost optimization.

### 1.1. Key AWS Services Used

- Amazon Kinesis Firehose: For streaming data ingestion.
- AWS Glue: For ETL (Extract, Transform, Load) and catalog management.
- Amazon S3: For data lake storage.
- Amazon Athena: For SQL-based big data analytics.
- Amazon QuickSight: For data visualization.

### 1.2. Solution architecture:

Streaming ingestion to S3, Glue for cataloging/ETL, Athena for SQL queries, and QuickSight for dashboards.

![Solution Architecture](images/overview/image1.png)

**Key Learnings:** Modern data lakes centralize raw data in its original form, enabling flexible processing and broad access. AWS's serverless services reduce management overhead, supporting diverse patterns like batch/streaming ingestion and unified analytics across structured/unstructured data.

## 2. Prerequisites

I prepared my AWS environment to ensure smooth execution, focusing on security and resource setup.

### 2.1. AWS Account Setup

Use my account

### 2.2. Create an AWS IAM User

In IAM Console, created a user with console access.

- Create name, password, user access
  ![IAM User Creation](images/prerequisites/image1.png)
- Attach policies directly: PowerUserAccess and IAMFullAccess
  ![IAM User Creation](images/prerequisites/image2.png)
- Note Console sign-in URL for login:
  ![IAM User Creation](images/prerequisites/image3.png)
- Login the user
  ![IAM User Creation](images/prerequisites/image4.png)

### 2.3. Deploy some resources via AWS CloudFormation

- Downloaded the CloudFormation template from [CloudFormation Template](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/serverlessDataLakeImmersionIAMcf.json)
- Create [CloudFormation Stack](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template?stackName=SDL-Service-Roles)
- Upload template JSON
  ![CloudFormation Stack](images/prerequisites/image5.png)
- Launched stack "SDL-Service-Roles", and leave as default to create.
  ![CloudFormation Stack](images/prerequisites/image6.png)
  ![CloudFormation Stack](images/prerequisites/image7.png)
  ![CloudFormation Stack](images/prerequisites/image8.png)

**What gets created**: The template provisions IAM roles (SDL-GlueRole, SDL-LambdaRole) and S3 bucket with proper permissions for the workshop.

**Key Learnings:** IAM users with least-privilege policies (e.g., PowerUserAccess) enable safe experimentation. CloudFormation automates infrastructure like S3 buckets and roles, enforcing best practices for data lake governance and cost tagging (e.g., project: serverlessdatalake).

## 3. Lab 1: Data Ingestion & Storage

I ingested sample GDELT data into S3 to simulate a data lake foundation. There are 2 options: Batch ingestion with prepared data, or Streaming data ingestion. For quick data import and cost utilization, I choose batch ingestion.

This option uses AWS CloudShell to run commands using the AWS CLI.

- Launch AWS CloudShell from the console for CLI access.

![CloudShell Launch](images\lab1\image1.png)

![CLI Commands](images\lab1\image2.png)

- Run commands to copy partitioned data from public S3 to my bucket:

```
export MY_ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
aws s3 cp s3://kat-tame-bda-immersion/raw/2019/06/18/20/ s3://sdl-immersion-day-$MY_ACCOUNT_ID/raw/year=2019/month=06/day=18/hour=20/ --recursive
aws s3 cp s3://kat-tame-bda-immersion/raw/2019/06/18/21/ s3://sdl-immersion-day-$MY_ACCOUNT_ID/raw/year=2019/month=06/day=18/hour=21/ --recursive
text
```

- Verified data in S3 Console: Navigated to sdl-immersion-day-[account]/raw/ and confirmed folders/partitions.

![S3 Verification](images\lab1\image3.png)

**Key Learnings:** Batch ingestion via AWS CLI/S3 efficiently populates data lakes from sources like Systems of Record (SoR) or external datasets. Partitioning (e.g., year/month/day/hour) optimizes queries; S3's scalability handles petabyte volumes securely.

## 4. Lab 2: Data Cataloging and ETL

This Lab 2 helps cataloge raw data and built ETL pipelines using AWS Glue, exploring visual and interactive approaches.

### 4.1. Lab 2.1: Cataloging Your Data

#### 4.1.1. Create crawler to auto discover schema of your data in S3

- In Glue Console, created Crawler "sdl-demo-crawler"
  ![Crawler Creation](images\lab2\image1.png)
  ![Crawler Creation](images\lab2\image2.png)
- Added S3 data source (sdl-immersion-day-[account]/raw/), used SDL-GlueRole, output to new database "sdl-demo-data" (on-demand schedule).
  ![Crawler Data Source](images\lab2\image3.png)
- Configure security settings: Select the Existing IAM role as SDL-GlueRole
  ![Select IAM](images\lab2\image4.png)
- Set output and scheduling:

  - Add database
    ![Add database](images\lab2\image5.png)
  - Name database and create
    ![Add database](images\lab2\image6.png)
    ![Add database](images\lab2\image7.png)
  - Back to Set output and scheduling, refresh to load the new database
    ![Set output and scheduling](images\lab2\image8.png)
  - Review and Run Crawler

- Ran the crawler (2 minutes) and verified table "raw" under Tables, showing JSON schema, partitions, and metadata.
  ![Crawler Run](images\lab2\image9.png)

#### 4.1.2. Edit the Metadata Schema

- Verify table "raw" under database and its schema (Check if year, month, day, and hour partition columns are in the table or the indexes are correct)
  ![Tables](images\lab2\image11.png)
  ![Tables](images\lab2\image12.png)
  ![Tables](images\lab2\image13.png)
- Edited schema: Actions > Edit schema
  ![Schema Edit](images\lab2\image14.png)
- To rename partition columns, the old indexes should be deleted first.
  ![Schema Edit](images\lab2\image15.png)
- Select the partition column, use the Edit button and modify the name of the column and partition
  ![Schema Edit](images\lab2\image16.png)
- Repeat with all columns, save as new table and check the table again
  ![Schema Edit](images\lab2\image17.png)
  ![Schema Edit](images\lab2\image18.png)

**Key Learnings:** Glue Crawlers automate schema discovery, populating the Data Catalog for queryable metadata without data movement. Tables define schemas/partitions; editing ensures Hive-compatible naming for efficient partitioning.

### 4.2. Lab 2.2: Visually Transform Data with Drag-and-Drop

After adding Amazon S3 data to Glue catalog, it can easily be queried from services like Amazon Athena. However, it must be transformed beforehand. Therefore, this session demonstrates the transformation process via AWS Glue:

- Convert raw JSON to a compressed columnar format.
- Rename two fields
- Drop a field

#### 4.2.1. Create Transformation Job with Glue Studio

- Created S3 folder for scripts:
  - In S3, choose data lake bucket and add /scripts/ to sdl-immersion-day-[account]/.
    ![Scripts Folder](images\lab2\image19.png)
    ![Scripts Folder](images\lab2\image20.png)
- Authoring the job in Glue Studio

  - In Glue Studio, choose ETL Jobs and Create Visual ETL job
    ![Job Properties](images\lab2\image21.png)
    ![Job Properties](images\lab2\image22.png)
  - Job Detail:
    - Name: transform-json-to-parquet
    - IAM Role: Select SDL-GlueRole
    - Glue version: Glue 4.0 - Supports spark 3.3, Scala 2, Python 3
    - Language: Python 3
    - Worker type: G 1X
    - Requested number of workers: 2
    - Expand Advanced properties, and update the following configurations:
      - Script filename: transform-json-to-parquet.py
      - Script path: s3://sdl-immersion-day-{aws-account-number}/scripts/

  ![Job Properties](images\lab2\image23.png)

  ![Job Properties](images\lab2\image24.png)

- Design transformation logic:

  - Switch to the Visual tab
  - In the Nodes pane, Sources tab, select Amazon S3.
    ![Visual Nodes](images\lab2\image25.png)
  - In the Nodes pane, Target tab, select Amazon S3.
    ![Visual Nodes](images\lab2\image26.png)
  - Configure the node properties:
    - Select Data Catalog table as S3 source type
    - Choose sdl-demo-data for Database
    - Choose raw for Table
      ![Visual Nodes](images\lab2\image27.png)
    - For the Format, choose Parquet
    - For the Compression Type, choose Snappy
    - For the S3 Target Location, choose the data lake bucket and prefix it with compressed-parquet/, so it looks like s3://sdl-immersion-day-{aws-account-number}/compressed-parquet/
    - Keep the rest to the defaults
      ![Visual Nodes](images\lab2\image28.png)
  - Finally, add some transformation logic:

    - Choose the Change Schema node from the Nodes pane.
      ![Visual Nodes](images\lab2\image29.png)
    - In the configuration of the newly added transformation node, set the Node parents to the source S3 bucket node.
    - Mark the color column to Drop
    - Rename datesoldsince to date_start
    - Rename datesolduntil to date_until
      ![Visual Nodes](images\lab2\image30.png)
    - Make sure that target node is populated with the changed schema. Choose the target node and change the Node parents to contain only the Change schema node.
      ![Visual Nodes](images\lab2\image31.png)

  - Save and review generated PySpark script in Script tab.
    ![Generated Script](images\lab2\image32.png)

#### 4.2.2. Run Transformation Script

- In the Glue Studio editor, choose Run on the top right of the screen.
  ![Job Run](images\lab2\image33.png)

#### 4.2.3. Monitor and review Job execution

- Navigate to the Runs tab
- Wait for Run status: Starting -> Running -> Succeeded
  ![Job Run](images\lab2\image34.png)
- Open the Amazon S3 Console and verify the transformed files are now in your Amazon S3 bucket under the path s3://sdl-immersion-day-{aws-account-number}/compressed-parquet/
  ![Job Run](images\lab2\image35.png)
- Return to the Monitoring tab in the AWS Glue Studio Console and scroll down to the Run details. Review the Metrics section at the bottom.
  ![Job Run](images\lab2\image36.png)

**Key Learnings:** Glue Studio's no-code interface generates scalable Spark ETL code for transformations. Serverless execution handles infrastructure; metrics aid tuning for performance/cost.

### 4.3. Lab 2.3: Interactive ETL Code Development

This session explores a more recent feature of AWS Glue, Interactive sessions:

- Serverless, no infrastructure to manage.
- Start sessions quickly for code development, debugging, and testing.

#### 4.3.1. Interactive Manipulations with Glue Serverless Spark

**Create a Jupyter Notebook for Interactive Advanced Data Manipulations**

- This section uses a Jupyter Notebook within AWS Glue Studio for some interactive development and testing of ETL scripts.
- Download the following [Jupyter notebook](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/Lab%202.3%20-%20Advanced%20Data%20Preparation.ipynb)
- Navigate to the [Jobs within AWS Glue Studio](https://console.aws.amazon.com/gluestudio/home#/jobs).
  ![Jobs within AWS Glue Studio](images\lab2\image36-2.jpg)
- Uploaded Jupyter notebook downloaded to Glue Studio (Notebook type, SDL-GlueRole)
  ![Jobs within AWS Glue Studio](images\lab2\image37.png)
- Check the content and rename to "Advanced Data Preparation."
  ![Notebook Upload](images\lab2\image38.png)

**Run Notebook**

The notebook contains further instructions and steps:

- Set account ID
- Connect to raw S3 data via Glue DynamicFrame
- Inspect schema
- Apply transformations (e.g., filters, joins, calculated columns)
- Partition output by department/date
- Write the transformed output back to Amazon S3

Notebook contents #1:
![Notebook Cell 1](images\lab2\image39.png)

Notebook contents #2:
![Notebook Cell 2](images\lab2\image40.png)

Notebook contents #3:
![Notebook Cell 3](images\lab2\image41.png)

Notebook contents #4:
![Notebook Cell 4](images\lab2\image42.png)

Notebook contents #5:
![Notebook Cell 5](images\lab2\image43.png)

Notebook contents #6 and #7:
![Notebook Cell 6-7](images\lab2\image44.png)

Output Verification:

- Folder in S3
  ![Output Verification](images\lab2\image45.png)

- Data partitioned by department:
  ![Output Verification](images\lab2\image46.png)

#### 4.3.2. Automating ETL Tasks with Triggers

In this section, a brief example of some the ETL automation capabilities by AWS Glue is provided.

- Navigate to [Trigger in AWS Glue](https://console.aws.amazon.com/glue/home#etl:tab=triggers).
  ![Trigger Setup](images\lab2\image47.png)
- Add a trigger that runs the ETL job daily.
  ![Trigger Setup](images\lab2\image48.png)
- Select target.
  ![Trigger Setup](images\lab2\image49.png)
- Finally review the trigger and Create it.
  ![Trigger Setup](images\lab2\image50.png)

**Key Learnings:** Interactive Sessions enable REPL-style development for complex ETL (e.g., PySpark joins/enrichments) without clusters. Notebooks integrate with IDEs; triggers automate workflows, supporting event-driven or scheduled processing.

## 5. Lab 3: Data Analytics & Visualization

This session shows how to query the GDELT dataset (large and public) with Athena for analysis, focusing on SQL over large-scale data.

- GDELT Open Dataset: The Global Database of Events, Language and Tone (GDELT) Project
  ![GDELT](images\lab3\image1.png)

### 5.1. Lab 3.1: SQL Analytics on a Large Scale Open Dataset

#### 5.1.1. Create Glue DB for GDELT Schema using Athena HIVE DDL

- In Athena Console, choose the Query your data with Trino SQL option , and Launch query editor.
  ![Athena Settings](images\lab3\image2.png)
- Edit settings
  ![Athena Settings](images\lab3\image3.png)
- Set query results to s3://sdl-immersion-day-[account]/athena-results/, launched editor (Trino SQL).
  ![Athena Settings](images\lab3\image4.png)
  ![Athena Settings](images\lab3\image5.png)
- Confirm the configuration and return to the query editor by choosing the Editor tab.

#### 5.1.2. Query GDELT EVENTS Data

- Created database: `CREATE DATABASE gdelt;`
  ![Database Creation](images\lab3\image6.png)

**Create Metadata Table for GDELT EVENTS Data**

- Created events table: `CREATE EXTERNAL TABLE gdelt.events (...) ROW FORMAT SERDE ... LOCATION 's3://gdelt-open-data/events/';`

![Events Table](images\lab3\image7.png)

**Create Metadata Table for GDELT EVENTS Data**

- Download data and upload to S3

  - Download files: [Event codes](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/CAMEO.eventcodes.txt), [Countries](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/CAMEO.countries.txt), [Types](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/CAMEO.types.txt), [Groups](https://static.us-east-1.prod.workshops.aws/public/57c3164a-dfbf-4d43-a246-79b2ceb70231/static/CAMEO.groups.txt)
  - In S3 data lake bucket 's3://sdl-immersion-day-{aws-account-number}', create Folder named 'gdelt', and click Save
    ![Create S3](images\lab3\image8.png)
  - Uploaded dimension files (eventcodes, countries, types, groups) to S3/gdelt/[folder]/.

  ![S3 Upload](images\lab3\image9.png)

- Return to the Amazon Athena console and run the SQL statements below to create dimension tables
  - Created dimension tables for eventcodes, types, groups, countries: e.g., `CREATE EXTERNAL TABLE gdelt.eventcodes (...) LOCATION 's3://sdl-immersion-day-[account]/gdelt/eventcodes/';`
    ![Dimension Tables](images\lab3\image10.png)
    ![Dimension Tables](images\lab3\image11.png)
    ![Dimension Tables](images\lab3\image12.png)
    ![Dimension Tables](images\lab3\image13.png)

**Query data**

- Find the number of events per year:

```
-- Find the number of events per year
SELECT year,
    COUNT(globaleventid) AS nb_events
FROM gdelt.events
GROUP BY year
ORDER BY year ASC;
```

![Events Query](images\lab3\image14.png)

- Notice the data amount scanned: In about 30 seconds scanned Amazon Athena over 190GB of data across thousands of uncompressed TSV files over 100s of millions of records in Amazon S3.

- Top 10 event categories: Join events with eventcodes, group/order by count.

```
-- Show top 10 event categories
SELECT eventcode,
        gdelt.eventcodes.description,
        nb_events
    FROM (SELECT gdelt.events.eventcode,
                COUNT(gdelt.events.globaleventid) AS nb_events
        FROM gdelt.events
        GROUP BY gdelt.events.eventcode
        ORDER BY nb_events DESC LIMIT 10)
    JOIN gdelt.eventcodes ON eventcode = gdelt.eventcodes.code
ORDER BY nb_events DESC;
```

![Top Events Query](images\lab3\image15.png)

- Count the number of events per year with Barack Obama

```
-- Count Obama events per year
SELECT year,
        COUNT(globaleventid) AS nb_events
    FROM gdelt.events
    WHERE actor1name='BARACK OBAMA'
    GROUP BY year
ORDER BY year ASC;
```

![Obama Events Query](images\lab3\image16.png)

- Obama/Merkel events per category: Filter/join with threshold.

```
SELECT eventcode,
    gdelt.eventcodes.description,
    nb_events
FROM (SELECT gdelt.events.eventcode,
            COUNT(gdelt.events.globaleventid) AS nb_events
    FROM gdelt.events
    WHERE actor1name='BARACK OBAMA'and actor2name='ANGELA MERKEL'
    GROUP BY gdelt.events.eventcode
    ORDER BY nb_events DESC)
JOIN gdelt.eventcodes ON eventcode = gdelt.eventcodes.code
WHERE nb_events >= 20
ORDER BY nb_events DESC;
```

![Obama Query](images\lab3\image17.png)

**Key Learnings:** Athena enables serverless SQL on S3 data (e.g., TSV/CSV) via Glue Catalog, parallelizing queries for seconds-level results on petabytes. External tables map schemas without data duplication; joins across buckets demonstrate unified access.

### 5.2. Lab 3.2: Data Visualization

- Attempted QuickSight setup: Signed up for Enterprise edition, selected region, added S3 buckets (sdl-immersion-day-[account], gdelt-open-data), set account name/email.

![QuickSight Signup](images\lab3\image18.png)
![QuickSight Signup](images\lab3\image19.png)
![QuickSight Signup](images\lab3\image20.png)
![QuickSight Signup](images\lab3\image21.png)
![QuickSight Signup](images\lab3\image22.png)

Note: Skipped further steps—dataset import, dashboard creation—as QuickSight requires a paid subscription unavailable in free tier

### 5.3. Lab 3.3: (Optional) Connecting to Athena with 3rd party BI tools via JDBC

This part uses JDBC connection to connect Athena with SQL Workbench, enabling Business Intelligence, analytics, and reporting on the data that Athena returns from Amazon S3 databases.

- Step 1: get IAM credentials
- Step 2: Download [SQL Workbench](https://www.sql-workbench.eu/)
- Step 3: Download the [Athena JDBC 3.X driver](https://downloads.athena.us-east-1.amazonaws.com/drivers/JDBC/3.0.0/athena-jdbc-3.0.0-with-dependencies.jar)
  ![Athena JDBC 3.X driver](images\lab3\image23.jpg)
- Step 4: Open SQL Workbench
  - Dismiss the connection window
    ![SQL Workbench connection](images\lab3\image24.jpg)
  - File > Manage drivers
    ![SQL Workbench connection](images\lab3\image25.png)
  - Create Sheet:
    - Athena for the Name
    - the downloaded JAR file for the Library
    - com.amazon.athena.jdbc.AthenaDriver for Classname
    - jdbc:athena://WorkGroup=primary;Region=us-east-1; for Sample URL
    - confirm with OK
      ![SQL Workbench connection](images\lab3\image26.png)
  - File > Connect Window
    ![SQL Workbench connection](images\lab3\image27.png)
  - Create a new connection
    - Athena for the Name
    - Athena for the Driver
    - jdbc:athena://WorkGroup=primary;Region=<YOUR EVENT REGION>; for the URL (replace the region placeholder)
    - Copy AWS Access Key into the Username input
    - Copy AWS Secret Key into the Password input
      ![SQL Workbench connection](images\lab3\image28.png)
    - Choose Extended Properties: Create a new property, call it S3OutputLocation and enter data lake S3 URI with /Athena as prefix
      ![SQL Workbench connection](images\lab3\image29.png)
    - Confirm OK to load database
      ![SQL Workbench connection](images\lab3\image30.png)
  - Tools > Database Explorer
    ![Database Explorer](images\lab3\image31.png)
    - Check database
      ![Database Explorer](images\lab3\image32.png)
    - Review data in sdl-demo-data
      ![Database Explorer](images\lab3\image33.png)
  - Work on database using local reporting and visualization tools

**Key Learnings:** JDBC connectivity enables integration with existing BI tools (Tableau, Power BI, SQL Workbench), extending Athena's reach beyond AWS-native services. This approach allows organizations to leverage current investments in BI infrastructure while accessing cloud-scale analytics. The connection uses standard SQL interfaces, making it familiar to existing database users.

## 6. Issues during Progress

### 6.1. Instruction Misunderstandings

**Common Issues Encountered:**

- **Architecture comprehension**: Initially struggled to understand service interactions and data flow
- **Jupyter Notebook configuration**: Required updating account ID variables for proper connectivity
- **Lab 3.3 limitations**: Session tokens were only available in workshop accounts, not personal AWS accounts
- **Data loading errors**: S3 bucket permissions and path configurations caused initial failures

### 6.2. Resource Management Challenges

**Session Continuity Issues:**
After each lab session, resource cleanup was performed to avoid costs, but this created challenges when resuming:

- **IAM user inconsistency**: Redoing Lab 1 with different IAM users caused permission conflicts
- **Missing CloudFormation prerequisites**: Skipping the CloudFormation stack deployment led to CloudShell access errors and missing service roles
- **Data persistence**: Cleaned data had to be re-ingested in subsequent sessions

**Lessons Learned**: For learning environments, consider keeping core infrastructure (IAM roles, S3 buckets) while cleaning up compute resources (Glue jobs, Athena queries) to balance cost and continuity.

## 7. Conclusion

This workshop equipped me with a serverless data lake pipeline: Ingested GDELT data to S3, cataloged/transformed via Glue (visual/interactive ETL), and queried with Athena for rapid insights—all without servers. Cleanup: Delete CloudFormation stack, empty S3 buckets, remove IAM roles to avoid costs.

**Key Overall Learnings**: AWS analytics services enable scalable, governed data flows (both batch and streaming) with minimal operational overhead. The serverless architecture handles exabyte-scale data variety and velocity, allowing teams to focus effort on extracting value rather than managing infrastructure. This approach is ideal for data analysts and architects building future-proof, cost-effective data architectures that can scale with organizational needs.
