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

![Docker_Flink](https://github.com/user-attachments/assets/61aeb703-c2c9-48c0-b348-7c5e9728c673)

---

## Video

https://github.com/user-attachments/assets/f00146fd-c65c-480a-a088-150ac7b00c50

---

## Key Features of the Dashboard

1. **Average Active Years of Customers**:  
   - The average active years for customers is 33 years, indicating a longstanding customer base.

2. **FICO Score Profile**:  
   - The average FICO score of customers is 581.111. This shows a relatively moderate credit profile for most customers.

3. **Average Churn Propensity**:  
   - The average propensity to churn is 0.575, suggesting that about half of the customers are at risk of churn.

4. **Churn Propensity vs. Activity Type**:  
   - Customers with new accounts have the highest propensity to churn compared to those with web or mobile interactions. This indicates a need for better onboarding and engagement strategies.

5. **Customer Demographics**:  
   - The gender distribution is fairly balanced, with slightly more female customers than male customers.

6. **Activity Type Distribution**:  
   - The most common activity type is mobile interactions, followed by web and new accounts.

7. **Income Levels by Gender**:  
   - Male customers generally have a higher average income compared to female customers.

8. **Top 10 Customers by Income**:  
   - The top customers based on income include Gates, Chico, Winfred, Lily, Gill, Livia, Carmetta, and Em, with Gates leading significantly.

9. **Customer Activity Type Timeline**:  
   - Web interactions tend to span a broader range of active years compared to mobile and new accounts, indicating a more stable usage pattern.

10. **Oldest Clients**:  
    - The oldest clients (in terms of active years) include Lily, Gill, Em, Chico, and others, with Lily having the longest engagement.

11. **Gender Distribution by Income and Activity Type**:  
    - Female customers with new accounts show a higher average income compared to male counterparts, while male customers dominate income levels in web and mobile interactions.

---

## Insights Derived from the Dashboard

1. **Churn Management**:  
   - High churn rates among new account holders highlight the importance of a robust onboarding strategy. Personalized engagement campaigns and incentives may help retain these customers.

2. **Credit Profile Improvement**:  
   - The moderate FICO score of customers suggests an opportunity to offer financial literacy programs or credit-building initiatives.

3. **Gender-Specific Strategies**:  
   - Higher income among male customers provides scope for premium product offerings, while targeted marketing campaigns for female customers can help balance the income disparity.

4. **Income-Based Targeting**:  
   - Gates, Chico, and other high-income customers can be leveraged for premium offerings, loyalty programs, or exclusive benefits.

5. **Activity Trends**:  
   - Mobile interactions dominate activity types, emphasizing the importance of mobile-friendly platforms and personalized push notifications.

6. **Retention of Long-Standing Customers**:  
   - Engaging the oldest clients with loyalty programs or exclusive benefits can ensure continued patronage and advocacy.

7. **Balanced Demographics**:  
   - The almost equal gender distribution allows for balanced marketing strategies tailored to both genders.  

---

## **Managerial Insights**

### 1. **Customer Retention and Churn Reduction**
   - **Insight**: Customers with new accounts have the highest propensity to churn.  
   - **Action**: Implement personalized onboarding processes, customer education programs, and proactive support for new account holders to reduce churn.  

### 2. **Mobile Dominance in Activity**  
   - **Insight**: Mobile interactions are the most dominant activity type.  
   - **Action**: Prioritize investment in mobile app functionality, performance optimization, and personalized push notifications to enhance customer satisfaction.

### 3. **High-Value Customer Engagement**
   - **Insight**: Gates, Chico, and other high-income customers contribute significantly to revenue.  
   - **Action**: Create exclusive loyalty programs and premium services targeted toward these high-value customers to maximize lifetime value.

### 4. **Gender-Based Targeting**
   - **Insight**: Male customers generally have higher incomes but females are equally engaged.  
   - **Action**: Develop gender-specific marketing strategies—focus on premium product upselling for males and personalized campaigns for female customers.

### 5. **Retention of Long-Standing Customers**
   - **Insight**: Older customers like Lily and Gill have remained loyal for extended periods.  
   - **Action**: Strengthen relationships with longstanding customers through loyalty rewards, appreciation events, and testimonials to turn them into brand advocates.

### 6. **Opportunity for FICO Score-Driven Services**
   - **Insight**: The average FICO score of 581.111 shows room for financial improvement among customers.  
   - **Action**: Offer credit-building tools, financial management education, and credit-related products to improve customer financial profiles.

### 7. **Cross-Selling Opportunities**
   - **Insight**: Web interactions span across a broader active period, indicating stable usage patterns.  
   - **Action**: Leverage web users for cross-selling opportunities such as subscriptions or premium memberships.

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
  
