
# RFC: Dynamic Report Builder for NED University of Engineering and Technology
 
**Status**: Proposed 
**Date**: 28.Sep.2024

---

## 1. **Introduction**

This RFC proposes a solution to enable on demand dynamic report generation for NED University of Engineering and Technology (NEDUET). Currently, different departments within NEDUET store their data in various databases (MySQL, MongoDB, SQL Server, Postgres, etc.), making it challenging to aggregate and connect data for university-wide reporting. The absence of a centralized solution for querying across these data stores leads to inefficiencies, slow reporting processes, and potential errors.
Given NEDUET’s compliance, regulatory, and cost constraints, a cloud or proprietary solution is not feasible. This RFC outlines a plan to implement an open-source, on-premise solution using [**Apache Presto/Trino**](https://trino.io/) as the query engine to query from diverse datasources and [**Apache Superset**](https://superset.apache.org/) as a frontend tool for building dynamic dashboards and reports.

---

## 2. **Motivation**

The need for a university-wide reporting solution stems from:

- The **decentralized data** stored across various departments in multiple database systems. These department are the owners of the data. They are the source of generation and this data is being already maintained by the respective department.This data source is already managed through defined protocols.

- Current methods for aggregating and reporting data are slow and error-prone, requiring **manual intervention**. Imagine if we have to make report which aggregate and joins data from Finance department, Humanities department, Mechanical Engineering department then first these departments will be requested to provide the specific data and after that someone has to make sense of this data to produce such reports.

- **Regulatory and compliance constraints** restrict NEDUET from using public cloud-based or proprietary solutions.

- There is a demand for **cost-efficient**, easily maintainable, and open-source solutions. NEDUET is a public university so cost is a constraint. This will be maintained by students where the groups will change year after year. The faculty is occupied with research and teaching. The department specific IT staff will be managing their own IT assets. Specialised skills of data engineering is hard to build. The solution should be simple and easy to maintain consdiering the context of NEDUET.

- **Dynamic report generation** requires aggregating data from various sources in near real-time, but current methods lacks flexibility and automation.

### Goals:

1. **Centralized Query Engine**: Allow seamless querying across different database systems using a unified query engine.

2. **Dynamic Report Building**: Enable easy creation of reports and dashboards with periodic refreshes.

3. **Open-Source Solution**: Ensure the solution is built on open-source technologies to avoid vendor lock-in.

4. **Maintainability**: Create a solution that can be maintained by different groups of students annually with minimal faculty intervention.

---

## 3. **Proposed Solution**

### 3.1. **Overview**
The proposed system consists of two primary components:

- [**Apache Presto/Trino**](https://trino.io/) as the distributed SQL query engine to connect and query data from various database systems.

- [**Apache Superset**](https://superset.apache.org/) as the visualization and report-building platform.

Departments within the university will provide a **read-only user account** to securely access their data stores. Apache Presto/Trino will use these credentials to aggregate data from disparate sources, and Apache Superset will be used to build and refresh dashboards.

### 3.2. **Component Breakdown**

#### 3.2.1 **Apache Presto/Trino**

- **Function**: [**Apache Presto/Trino**](https://trino.io/) will act as a federated query engine, capable of executing SQL queries across multiple data sources, including relational databases (MySQL, Postgres, SQL Server) and NoSQL databases (MongoDB) etc.
- **Key Features**:
  
  - Supports querying across **heterogeneous data sources**.
  - **Open-source** and extensible.
  - Scalable to accommodate future data growth.
  - Designed to run **on-premise**, meeting the compliance requirements.
- **Integration with Databases**:
  
  - Each department will be responsible for creating a **read-only user** with permissions to access the required tables and data. This user will be isolated from the operational database processes to avoid conflicts.This will also help the department for security reasons to understand who accessed the databases.

  
  - Presto/Trino will connect to each department’s database using this read-only account, allowing queries without affecting the operational performance.

#### 3.2.2 **Apache Superset**

- **Function**: [**Apache Superset**](https://superset.apache.org/) will serve as the frontend reporting tool, allowing users to create, schedule, and view dynamic reports and dashboards.
- **Key Features**:

  - **Open-source** and user-friendly interface for non-technical users to build visualizations.
  - **Seamless integration with Presto/Trino** as the query engine.
  - Dashboards can be configured to refresh **periodically** (e.g., every X minutes), ensuring data is up-to-date.
  - Supports a variety of visualization types, enabling flexible report creation tailored to departmental needs.

---

## 4. **Technical Considerations**

### 4.1 **Database Connectivity**
- Presto/Trino will support connections to relational databases (MySQL, Postgres, SQL Server) as well as NoSQL databases (MongoDB).
- Each department will be responsible for ensuring their database is properly configured for external read-only access.

### 4.2 **Security**

- Access control will be implemented at the database level, ensuring that the read-only user cannot modify or delete data.
- Presto/Trino and Superset will be hosted on internal university servers with strict network security rules to prevent unauthorized access.

### 4.3 **Performance**

- [**Apache Presto/Trino**](https://trino.io/) is designed to handle large-scale queries efficiently, but the performance will depend on the size of the datasets and the complexity of queries.
- Dashboards will be designed to optimize performance by aggregating data efficiently and limiting the frequency of refreshes to avoid excessive load.

### 4.4 **Alternative Approach: Centralized Data Warehouse or Data Lake**

Another approach is to create a **centralized persistent store** such as a **data warehouse** or **data lake**. This would involve ingesting data from all departments into a central repository. While this approach has its merits, it comes with additional challenges:

#### Advantages:

- **Centralized Data Access**: All data is located in one place, simplifying access and queries.
- **Faster Query Performance**: Querying a single repository would likely be faster.
- **Unified Schema**: A central repository enforces consistent data structure across departments.

#### Disadvantages:

- **Continuous Maintenance**: Maintaining the data warehouse/data lake would require ongoing maintenance, including ETL (Extract, Transform, Load) processes, and monitoring.

- **Additional IT Infrastructure**: NEDUET would need to allocate IT resources to manage this central repository, adding to the university’s infrastructure burden.

- **Data Freshness**: Since data would be periodically ingested, real-time reporting may be delayed.

- **Complexity in Governance**: Centralized data stores require stricter data governance policies to manage access and compliance.

#### Example:

Imagine a library where each department maintains their own books in separate rooms. A centralized data warehouse is like moving the copy of all the books into one big room, making it easier to find any book but requiring someone to constantly move, organize, and update the books from all rooms. This requires a lot of work and resources. This means that someone has to do the work of moving the books from one room to another. All the rooms with the books has to be managed.

In contrast, the proposed solution is like letting each department keep their books in their own rooms, but having a special person (Presto/Trino) who knows how to find books in all the rooms whenever someone needs them. It’s quicker and easier to maintain, but still allows access to all the books without moving them.

#### Why approach to move data to centralise locations is not ideal for NEDUET:

1. **High Maintenance Overhead**: Faculty and students have limited time and resources, making it impractical to maintain a data warehouse or data lake.

2. **Additional IT Assets**: This approach requires more infrastructure and adds long-term IT costs.More IT assets means more resources to manage. 

3. **Student Turnover**: Each year, a new group of students would need to manage complex ETL processes, making it harder to maintain consistency and quality.
Inorder to make any changes first they have to learn the existing system and then they can make changes.

4. **Regulatory Compliance**: Managing all data in one place increases the risk of compliance violations, as data privacy policies vary across departments.

---

## 5. **Justification for the Proposed Solution**

The proposed approach of using **Apache Presto/Trino** and **Apache Superset** offers a more lightweight, scalable, and maintainable solution:

- **Minimal Maintenance**: Departments keep their own data stores, and the query engine simply accesses them when needed, eliminating the need for continuous data ingestion and transformation.

- **On-Premise and Open Source**: The system is compliant with NEDUET’s regulatory requirements, as all data processing occurs on-site, and the software is open-source.

- **Cost Efficiency**: The solution does not require additional infrastructure or expensive cloud services.
- **Decentralized Data, Centralized Access**: The federated query engine allows access to all departmental data without physically moving or duplicating the data.

---

## 6. **Conclusion**

This RFC proposes a solution to enable dynamic reporting for NED University using open-source technologies. The combination of **Apache Presto/Trino** and **Apache Superset** allows NEDUET to efficiently query data from disparate sources, generate dynamic reports, and comply with regulatory requirements, all while minimizing costs and maintenance efforts.

By adopting this approach, NEDUET can streamline its reporting process without the burden of maintaining a centralized data repository, ensuring a scalable, maintainable solution that can evolve year by year.

---
