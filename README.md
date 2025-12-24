# 🌲 Forest Fire Prediction & Real-Time Monitoring System

![Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![Hadoop](https://img.shields.io/badge/Apache%20Hadoop-66CCFF?style=for-the-badge&logo=apachehadoop&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

## 📖 Overview

This project is an end-to-end Big Data solution designed to predict forest fire risks in real-time. It leverages historical weather and fire data from Turkey to train a Machine Learning model using **Apache Spark**. The system then simulates real-time IoT sensor data via **Apache Kafka**, processes streams using **Spark Structured Streaming**, and stores alerts in **MongoDB**. The entire pipeline relies on **Apache Hadoop HDFS** for reliable distributed storage of models and datasets.

## 🏗️ Architecture Pipeline

The project follows a Lambda Architecture approach comprising three main stages:

1.  **Batch Layer (Model Training):**
    * Ingests historical Fire and Weather data (ERA5) stored in **Hadoop HDFS**.
    * Performs data cleaning, spatiotemporal grid aggregation, and feature engineering.
    * Trains a **Random Forest Classifier** using PySpark ML.
    * Saves the trained model back to **HDFS**.

2.  **Speed Layer (Real-time Simulation & Inference):**
    * **Producer:** Simulates IoT sensors sending seasonal weather data (temp, humidity, wind, location) to a Kafka topic.
    * **Consumer:** Reads streams from Kafka, loads the pre-trained Spark model from **HDFS**, and calculates fire probability in real-time.

3.  **Serving Layer:**
    * Classifies risks into levels (OK, HIGH RISK, CRITICAL).
    * Stores results in MongoDB for dashboard visualization.

## 🛠️ Tech Stack

* **Language:** Python (PySpark)
* **Big Data Engine:** Apache Spark (Core, SQL, ML, Streaming)
* **Message Broker:** Apache Kafka
* **Storage:** Apache Hadoop HDFS (Distributed File System) & MongoDB
* **Environment:** Docker / Jupyter Notebooks

## 📂 Project Structure

```bash
├── raw_data/                 # Historical Datasets (Weather & Fire records)
├── notebooks/
│   ├── spark_processing.ipynb # Data Preprocessing & Model Training (Reads/Writes to HDFS)
│   ├── producer.ipynb         # Kafka Producer (IoT Sensor Simulator)
│   └── consumer.ipynb         # Spark Streaming Consumer & Inference (Loads Model from HDFS)
├── models/                   # Saved Spark ML Models (Stored in HDFS)
└── README.md

```

## 📊 Model Performance

The Random Forest model was trained on historical data (2018-2019) and tested on 2020 data. It handles class imbalance by undersampling the negative class.

* **Accuracy:** ~85.5%
* **ROC AUC:** ~0.928
* **Key Features:** `lat_grid`, `lon_grid`, `humidity_min`, `temp_max`, `month`.

## 🚀 How to Run

### Prerequisites

Ensure you have a Big Data environment set up (preferably using Docker Compose) containing:

* Apache Spark (Master & Workers)
* **Apache Hadoop (Namenode & Datanode)**
* Apache Zookeeper & Kafka
* MongoDB

### Step 1: Train the Model

Run `spark_processing.ipynb`. This notebook will:

1. Read CSV files from HDFS (`hdfs://namenode:9000/raw_data/turkey/*.csv`).
2. Clean and balance the dataset.
3. Train the model and save it to HDFS (`hdfs://namenode:9000/models/turkey_fire_model_v1`).

### Step 2: Start the Data Producer

Run `producer.ipynb`.

* This acts as a simulator generating synthetic weather data for the year 2025.
* It sends JSON messages to the Kafka topic `forest-sensors`.
* *Note: It simulates different seasons (including Summer heatwaves) to test alert logic.*

### Step 3: Start the Stream Consumer

Run `consumer.ipynb`.

* It initializes a Spark Streaming session.
* Loads the saved model from HDFS.
* Consumes data from `forest-sensors`.
* Predicts fire probability and saves alerts to MongoDB (`bigdata.all_sensor_readings`).

## 🚨 Alert Logic

The system categorizes risk based on the model's probability output and current weather conditions:

| Alert Status | Condition |
| --- | --- |
| **CRITICAL FIRE (Extreme)** | Temp ≥ 40°C AND Humidity < 25% |
| **CRITICAL FIRE** | Model Probability ≥ 50% |
| **HIGH RISK ALERT** | Model Probability ≥ 30% AND Temp > 30°C |
| **OK** | Otherwise |

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

**Author:** BAIHICH Mohamed


```
