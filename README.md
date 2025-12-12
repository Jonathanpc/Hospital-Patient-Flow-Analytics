# Patient Flow & Bed Occupancy Analytics

A real-time data engineering pipeline that processes hospital patient flow data using Azure cloud services, implementing a medallion architecture (Bronze, Silver, Gold) for analytics and business intelligence.

![Architecture Diagram](architecture.png)

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Workflow](#project-workflow)
- [Data Pipeline](#data-pipeline)
- [Setup & Installation](#setup--installation)
- [What I Learned](#what-i-learned)
- [Challenges Faced](#challenges-faced)
- [Future Enhancements](#future-enhancements)

## 🎯 Overview

This project demonstrates an end-to-end real-time data engineering solution for healthcare analytics. The system ingests streaming patient admission and discharge events, processes them through multiple layers of transformation and cleansing, and delivers business-ready data for analytics and visualization.

**Key Features:**
- Real-time streaming data ingestion via Azure Event Hubs (Kafka)
- Medallion architecture implementation (Bronze → Silver → Gold)
- Automated data quality checks and cleansing
- Slowly Changing Dimension (SCD Type 2) implementation
- Star schema modeling for optimized analytics
- Secure credential management with Azure Key Vault

## 🏗️ Architecture

The solution follows a modern data lakehouse architecture with the following components:
<img width="926" height="534" alt="Screenshot 2025-12-12 at 2 17 32 PM" src="https://github.com/user-attachments/assets/8c7d07a9-fcd8-45a2-b14b-fc608c7ae9e5" />



### Data Flow
1. **Data Source** → Patient flow data generator (Python simulator)
2. **Data Streaming** → Azure Event Hubs (Kafka-compatible) ingests real-time events
3. **Data Lake (Bronze Layer)** → Raw data storage in Delta format
4. **Data Lake (Silver Layer)** → Cleaned and validated data
5. **Data Lake (Gold Layer)** → Business-ready dimensional model (star schema)
6. **Data Warehouse** → Azure Synapse SQL Pool for querying
7. **Analytics** → Power BI for visualization and reporting

### Security Layer
- **Azure Key Vault**: Secure storage of connection strings and credentials
- **Azure Active Directory**: Identity and access management

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **Cloud Platform** | Microsoft Azure |
| **Streaming** | Azure Event Hubs (Kafka) |
| **Orchestration** | Azure Data Factory (Databricks) |
| **Processing** | Apache Spark (PySpark) on Databricks |
| **Storage** | Azure Data Lake Storage Gen2 (ADLS) |
| **Data Format** | Delta Lake |
| **Data Warehouse** | Azure Synapse Analytics (SQL Pool) |
| **Visualization** | Power BI |
| **Security** | Azure Key Vault, Azure Active Directory |
| **Programming** | Python, PySpark, SQL |

## 📊 Project Workflow

### 1. Data Generation & Ingestion
- **Patient Flow Simulator** (`patient_flow_generator.py`) generates synthetic patient data
- Events include: patient demographics, admission/discharge times, department, bed ID
- Data intentionally includes quality issues (5% invalid ages, 5% future timestamps) to test cleansing logic
- Streams to Azure Event Hubs in real-time

### 2. Bronze Layer - Raw Data Ingestion
**Notebook:** `01_bronze_raw_data.py`

- Connects to Event Hubs using Kafka protocol
- Reads streaming data using Spark Structured Streaming
- Writes raw JSON data to ADLS in Delta format
- Maintains checkpoint for exactly-once processing
- No transformations applied at this stage

### 3. Silver Layer - Data Cleansing & Validation
**Notebook:** `02_silver_clean_data.py`

**Transformations Applied:**
- Parse JSON into structured schema
- Convert string timestamps to proper timestamp types
- Handle invalid admission times (nulls or future dates) → replace with current timestamp
- Correct invalid ages (>100) → generate random valid age
- Implement schema evolution to handle missing columns
- Write cleaned data in Delta format with merge schema enabled

### 4. Gold Layer - Business Logic & Dimensional Modeling
**Notebook:** `03_gold_transform.py`

**Dimensional Model Created:**

**Fact Table: `fact_patient_flow`**
- Stores patient admission events
- Includes calculated metrics:
  - `length_of_stay_hours`: Duration between admission and discharge
  - `is_currently_admitted`: Flag for active patients
- Foreign keys to dimension tables

**Dimension Tables:**
- `dim_patient`: Patient demographics with SCD Type 2 implementation
  - Tracks historical changes to patient attributes
  - Maintains `effective_from`, `effective_to`, `is_current` flags
  - Uses hash-based change detection
- `dim_department`: Department and hospital information

**Advanced Techniques:**
- Slowly Changing Dimension (SCD) Type 2 for patient dimension
- Hash-based change detection using SHA-256
- Window functions to get latest patient record
- Surrogate key generation

### 5. Data Warehouse - SQL Pool
**Script:** `SQL_pool_queries.sql`

- Creates external tables in Synapse SQL Pool
- Uses Managed Identity for secure authentication
- Defines Parquet file format for optimized querying
- Enables SQL-based analytics on gold layer data

### 6. Analytics & Visualization
- Power BI connects to Synapse SQL Pool
- Dashboards provide insights on:
  - Patient flow trends
  - Bed occupancy rates
  - Department utilization
  - Length of stay analysis
  - Real-time admission status

## 🚀 Setup & Installation

### Prerequisites
- Azure free trial subscription
- Azure Databricks workspace
- Azure Event Hubs namespace
- Azure Data Lake Storage Gen2 account
- Azure Synapse Analytics workspace
- Azure Key Vault
- Power BI Desktop (Not free version) (for visualization)

### Configuration Steps

1. **Create Azure Resources**
   ```bash
   # Create Resource Group, Event Hub, ADLS, Databricks, Synapse, Key Vault
   ```

2. **Store Secrets in Key Vault**
   - Event Hub connection string
   - Storage account access key
   - SQL Pool credentials

3. **Update Configuration in Notebooks**
   - Replace placeholders in all notebooks:
     - `<<Namespace_hostname>>`
     - `<<Eventhub_Name>>`
     - `<<Connection_String>>`
     - `<<Storageaccount_name>>`
     - `<<Storage_Account_access_key>>`
     - `<<container>>`
     - `<<path>>`

4. **Run Data Simulator**
   ```bash
   python simulator/patient_flow_generator.py
   ```

5. **Execute Databricks Notebooks in Order**
   - 01_bronze_raw_data.py
   - 02_silver_clean_data.py
   - 03_gold_transform.py

6. **Create External Tables in Synapse**
   - Execute SQL_pool_queries.sql

7. **Connect Power BI**
   - Connect to Synapse SQL Pool
   - Build visualizations

## 💡 What I Learned

### Azure Cloud Services
- **Azure Event Hubs**: Learned how to use Event Hubs as a Kafka-compatible streaming service for real-time data ingestion, including SASL authentication and configuration
- **Azure Data Lake Storage (ADLS Gen2)**: Understood hierarchical namespace, container organization, and integration with Databricks
- **Azure Databricks**: Gained hands-on experience with managed Spark clusters, notebook development, and Delta Lake capabilities
- **Azure Synapse Analytics**: Learned to create external tables, use Managed Identity authentication, and optimize queries for analytical workloads
- **Azure Key Vault**: Discovered how Key Vault securely stores connection strings and credentials using secret references rather than hardcoded values, requiring explicit access grants to applications and users for enhanced security
- **Azure Active Directory (AAD)**: Understood identity management and role-based access control (RBAC) in Azure

### Data Engineering Concepts
- **Medallion Architecture (Bronze-Silver-Gold)**: Implemented a layered approach to data quality and transformation, separating raw ingestion from cleansing and business logic
- **Delta Lake**: Learned about ACID transactions, time travel, schema evolution, and merge operations in a data lake environment
- **Spark Structured Streaming**: Gained experience with real-time data processing, checkpoint management, and exactly-once semantics
- **Slowly Changing Dimensions (SCD Type 2)**: Implemented historical tracking of dimension changes using effective dates and current flags
- **Star Schema Design**: Created fact and dimension tables optimized for analytical queries
- **Data Quality Patterns**: Implemented validation rules, null handling, and data correction strategies

### PySpark Programming
- **Window Functions**: Used for partitioning and ranking operations to get latest records
- **Hash-based Change Detection**: Applied SHA-256 hashing for efficient change detection in SCD implementation
- **Delta Merge Operations**: Learned update and insert patterns for upsert operations
- **Schema Evolution**: Handled missing columns dynamically to prevent pipeline failures

### DevOps & Version Control
- **Git with SSH**: Learned that GitHub no longer supports password authentication for Git operations; SSH keys must be generated and added to GitHub for secure remote operations
- **Configuration Management**: Understood the importance of parameterizing configurations and using placeholders for environment-specific values

### Best Practices
- **Separation of Concerns**: Each layer (Bronze/Silver/Gold) has a distinct responsibility
- **Idempotency**: Designed pipelines to handle reprocessing without data duplication
- **Error Handling**: Implemented `failOnDataLoss: false` to handle data skew and `mergeSchema: true` for schema evolution
- **Partitioning Strategy**: Used date-based partitioning for optimized queries

## 🚧 Challenges Faced

### 1. Power BI Access Issues
**Problem:** Unable to access Power BI due to organizational account requirements. The free version of Power BI Desktop requires an organizational email, and I only had access to free versions of Tableau, which limited the final visualization component.

**Solution/Workaround:** 
- Focused on creating the complete data pipeline and SQL-based queries
- Documented the connection process and dashboard requirements for future implementation
- Considered alternative visualization tools for the demo

### 2. Git Authentication with GitHub
**Problem:** Couldn't use Git remote operations with just a password. GitHub deprecated password authentication for Git operations.

**Solution:** 
- Generated SSH keys using `ssh-keygen`
- Added the public key to GitHub account settings
- Configured Git to use SSH URLs (`git@github.com:user/repo.git`) instead of HTTPS
- Successfully established secure remote connection

### 3. Schema Evolution in Streaming
**Problem:** Pipeline failures when new fields were added or fields were missing in incoming data.
<img width="992" height="781" alt="Screenshot 2025-12-11 at 9 32 05 PM" src="https://github.com/user-attachments/assets/c9d75f9a-74d1-45f3-bd3e-46c83db519c2" />


**Solution:**
- Enabled `mergeSchema: true` option in Delta writeStream
- Added dynamic column addition logic in Silver layer to handle missing expected columns
- Implemented schema validation checks before processing

### 4. Delta Lake Checkpoint Management
**Problem:** Checkpoint location conflicts and corruption when restarting streaming jobs.

**Solution:**
- Established clear checkpoint location patterns per streaming query
- Used unique checkpoint paths for Bronze and Silver layers
- Documented checkpoint cleanup procedures for job restarts


## 🔮 Future Enhancements

- [ ] Implement Azure Data Factory for pipeline orchestration and scheduling
- [ ] Add data quality monitoring and alerting using Azure Monitor
- [ ] Implement incremental processing for Gold layer instead of full refresh
- [ ] Add unit tests for transformation logic
- [ ] Create CI/CD pipeline for automated deployment
- [ ] Implement data lineage tracking
- [ ] Add real-time dashboard refresh capabilities
- [ ] Optimize partitioning strategy based on query patterns
- [ ] Implement data retention policies
- [ ] Add support for multiple hospital systems

## 📝 License

This project is created for educational and portfolio purposes.

**Author:** Jonathan Perez-Castro  
**Institution:** Rutgers University New Brunswick
**Contact:** yeriel1322@gmail.com
**LinkedIn:** [linkedin.com/in/jonathanpc](https://linkedin.com/in//jonathan-pc15)


**Note:** Remember to replace all placeholder values (`<<...>>`) with actual configuration values before running the pipeline.

**⭐ If you found this project helpful, please consider starring the repository!**
