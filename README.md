# Docker Flink Analytics Project

## Objective  
This project showcases the development of a scalable data pipeline capable of processing streaming data by combining Kafka, Apache Flink, and SQL for real-time analytics. The key objectives are to:  
- Generate synthetic datasets using Avro schema definitions.  
- Stream data into Kafka topics for further processing.  
- Leverage Apache Flink and SQL for real-time data analysis.  
- Create enriched data views to derive actionable insights.  
- Set the foundation for future data visualization with Elasticsearch and Kibana.

---

## Dataset Description
Three datasets representing various aspects of an insurance business were generated and transformed into JSON format for streaming. These include:

### 1. **Customer Activities (`customer_activity`)**
Represents customer interactions and activities.  
Schema:
- `activity_id`: Unique identifier for each activity.
- `customer_id`: Unique identifier for the customer.
- `activity_type`: Type of activity performed by the customer.
- `propensity_to_churn`: Likelihood of customer churn.
- `ip_address`: Customer's IP address.  

**Example Data:**
```json
{"activity_id":1001, "customer_id":203, "activity_type":"Login", "propensity_to_churn":0.35, "ip_address":"192.168.0.1"}
{"activity_id":1002, "customer_id":205, "activity_type":"Purchase", "propensity_to_churn":0.10, "ip_address":"192.168.0.2"}
```

### 2. **Insurance Offers (`customer_offers`)**
Details about insurance products or offers.  
Schema:
- `offer_id`: Unique identifier for each offer.
- `offer_name`: Name of the insurance offer.
- `offer_url`: URL to access the offer details.  

**Example Data:**
```json
{"offer_id":501, "offer_name":"Life Cover Premium", "offer_url":"http://insurance.com/offers/501"}
{"offer_id":502, "offer_name":"Health Insurance Basic", "offer_url":"http://insurance.com/offers/502"}
```

### 3. **Customers (`customer_details`)**
Details of registered customers.  
Schema:
- `customer_id`: Unique identifier for the customer.
- `first_name`: Customer's first name.
- `last_name`: Customer's last name.
- `email`: Email address of the customer.
- `gender`: Gender of the customer.
- `income`: Annual income of the customer.
- `fico`: Customer's FICO score.
- `years_active`: Number of years the customer has been active.  

**Example Data:**
```json
{"customer_id":203, "first_name":"John", "last_name":"Doe", "email":"john.doe@example.com", "gender":"Male", "income":75000, "fico":720, "years_active":3}
{"customer_id":205, "first_name":"Jane", "last_name":"Smith", "email":"jane.smith@example.com", "gender":"Female", "income":85000, "fico":740, "years_active":5}
```

---

## Data Pipeline Process

### 1. **Data Generation**
Synthetic datasets were generated from Avro schemas using `gendata.sh`:
```bash
./gendata.sh insurance_customer_activity.avro xyz1.json 10000
./gendata.sh insurance_customers.avro xyz2.json 10000
./gendata.sh insurance_offers.avro xyz3.json 10000
```

### 2. **Data Transformation**
The generated JSON files were transformed into Kafka-compatible formats using a Python script:
```bash
python $HOME/Documents/fake/convert.py
```

### 3. **Kafka Ingestion**
The transformed data was streamed into Kafka topics using `gen_sample.sh`:
```bash
./gen_sample.sh /home/ashok/Documents/gendata/rev_xyz1.json | kafkacat -b localhost:9092 -t test1 -K: -P
./gen_sample.sh /home/ashok/Documents/gendata/rev_xyz2.json | kafkacat -b localhost:9092 -t test2 -K: -P
./gen_sample.sh /home/ashok/Documents/gendata/rev_xyz3.json | kafkacat -b localhost:9092 -t test3 -K: -P
```

### 4. **Real-Time Analysis with Apache Flink**
Flink SQL was used to create real-time analytics tables for each dataset.  

#### Customer Activity Table (`test1_activity`)
```sql
CREATE TABLE test1_activity (
    activity_id BIGINT,
    customer_id BIGINT,
    activity_type STRING,
    propensity_to_churn DOUBLE,
    ip_address STRING
) WITH (
    'connector' = 'kafka',
    'topic' = 'test1',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);
```

#### Insurance Offers Table (`test2_offers`)
```sql
CREATE TABLE test2_offers (
    offer_id BIGINT,
    offer_name STRING,
    offer_url STRING
) WITH (
    'connector' = 'kafka',
    'topic' = 'test2',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);
```

#### Customers Table (`test3_customers`)
```sql
CREATE TABLE test3_customers (
    customer_id BIGINT,
    first_name STRING,
    last_name STRING,
    email STRING,
    gender STRING,
    income BIGINT,
    fico INT,
    years_active INT
) WITH (
    'connector' = 'kafka',
    'topic' = 'test3',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);
```

### 5. **Data Enrichment**
A consolidated view was created to combine all three datasets:
```sql
CREATE VIEW combined_data AS
SELECT 
    t1.activity_id,
    t1.customer_id,
    t1.activity_type,
    t1.propensity_to_churn,
    t1.ip_address,
    t2.offer_name,
    t2.offer_url,
    t3.first_name,
    t3.last_name,
    t3.email,
    t3.gender,
    t3.income,
    t3.fico,
    t3.years_active
FROM test1_activity AS t1
LEFT JOIN test2_offers AS t2
    ON t1.customer_id = t2.offer_id
LEFT JOIN test3_customers AS t3
    ON t1.customer_id = t3.customer_id;
```
### 6. **Dashboard Creation**

The data in the **customer_activity_dashboard** index was visualized using Kibana.  

- An index pattern for **customer_activity_dashboard** was created in Kibana.  
- A comprehensive dashboard was designed to showcase customer activity types, demographic details, income distribution, and churn propensity.  
---
# Dashboard Analysis 

![Docker_flink](https://github.com/user-attachments/assets/b49a9c6a-4386-402a-aa23-91be82bb1fb1)


---

## Video

https://github.com/user-attachments/assets/f00146fd-c65c-480a-a088-150ac7b00c50

---
### Key Features of the Dashboard:
1. **Customer Profile Metrics**:
   - Average active years of customers: **33 years**.
   - Average FICO score: **581.111**.
   - Average churn propensity: **0.575**.

2. **Activity Type Distribution**:
   - Representation of customer activities segmented into **new account, web open**, and **mobile open**.

3. **Demographic Insights**:
   - Income distribution and gender-wise income trends.
   - Customer demographics: Proportion of males and females.

4. **Top Customers by Income**:
   - Highlights the top 10 customers based on income.

5. **Activity and Retention**:
   - Timeline of activity types and their respective average active years.
   - Churn propensity analyzed by activity type.

6. **Gender and Churn**:
   - Gender-wise propensity to churn.
   - Churn propensity linked with FICO scores.

7. **Income and Churn**:
   - Relationship between average income and churn propensity.

---

### Insights Derived from the Dashboard:
1. **Activity Type and Churn**:
   - Customers with **new accounts** exhibit the highest churn propensity compared to other activity types like **web open** and **mobile open**.

2. **Income and Gender Distribution**:
   - **Females** have a higher average income compared to males.
   - Gender distribution in income shows disparity with significant female dominance in higher income brackets.

3. **Top Earners**:
   - The top customers (e.g., Gates, Em) have income levels exceeding **₹300,000**, indicating a skew in income distribution.

4. **Churn Trends**:
   - Lower FICO scores correlate with higher churn propensity.
   - Females are more likely to churn compared to males.

5. **Retention and Activity**:
   - Customers associated with **web open** activity type have a longer active period compared to other activity types.

---
### **Insights on Propensity to Churn**:
1. **High-Risk Profiles**:
   - Customers with lower FICO scores (<500) and those with a **new account** activity type are at the greatest risk of churn.
   - Females have a higher unique count of churn propensity compared to males, indicating a need for gender-specific retention strategies.

2. **Income and Churn Correlation**:
   - Higher income customers show a relatively low churn propensity, suggesting that higher earnings can be a stabilizing factor.

3. **Actionable Insights**:
   - Develop educational campaigns or credit score improvement programs to retain customers with **low FICO scores**.
   - Increase engagement and enhance the customer journey for new account holders within the first few months of onboarding.
  ---
### **Managerial Insights**:
1. **Retention Strategies**:
   - Focus retention efforts on **new account customers**, as they are at the highest risk of churn.
   - Introduce targeted programs or incentives for customers with **low FICO scores** to improve retention.

2. **Segmented Marketing**:
   - Design tailored marketing strategies for **high-income female customers** to leverage their income potential and reduce churn.

3. **Activity Optimization**:
   - Promote the use of **web-based activities**, as customers engaged through this channel tend to have longer active years.

4. **Data-Driven Interventions**:
   - Leverage FICO score data to proactively address customers at risk of churn.
   - Focus on demographic data to fine-tune customer engagement approaches.

5. **Customer Prioritization**:
   - High-income earners like Gates and Em should be prioritized for premium services or loyalty programs.

---

### Key Learnings from the Project  

1. **Understanding Data Streaming with Apache Flink**:  
   - Gained hands-on experience with **Apache Flink**, a powerful framework for real-time data processing.  
   - Learned how to efficiently handle streaming data and implement transformations to derive meaningful insights.  

2. **Using Flink SQL for Data Analysis**:  
   - Explored the capabilities of **Flink SQL** for querying and processing streaming data in real-time.  
   - Understood the importance of Flink SQL's ability to support complex aggregations, filtering, and joins, making it an efficient tool for data manipulation.  

3. **Data Visualization with Kibana**:  
   - Developed skills in creating interactive dashboards using **Kibana** to visualize streaming data.  
   - Learned how to use various visualizations like bar charts, pie charts, and timelines to interpret key metrics such as churn propensity, income distribution, and customer demographics.  

4. **Customer Insights and Churn Analysis**:  
   - Analyzed customer behavior based on activity types and demographic factors to identify patterns and trends.  
   - Understood how churn propensity varies with activity type, enabling data-driven decision-making.  

5. **Correlation Between Metrics**:  
   - Discovered relationships between **customer income**, **gender**, and **activity type** to identify valuable insights for targeting and retention strategies.  

6. **Real-Time Data Processing**:  
   - Developed an appreciation for real-time data analytics and its role in deriving actionable insights for businesses.  
   - Understood the challenges and solutions associated with managing and analyzing streaming data.  
  
