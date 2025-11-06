# Detection of Anomalous Navigation Patterns in Wikipedia Clickstream

## Project Overview

This project implements a complete Big Data Analytics (BDA) pipeline using PySpark to detect and analyze anomalous navigation patterns from the large-scale Wikipedia clickstream dataset.
The core problem is that anomaly is not a single, well-defined event. An anomaly could be a massive traffic spike (such as a click bot), a new, never-seen-before navigation path to an article, or a subtle shift in an article's overall traffic pattern (a behavioral shift).
This project tackles this problem by building **three distinct detection models** to capture each type of anomaly. It uses a baseline month (July) to definenormal and a test month (August) to find deviations from the obtained pattern.

- **Data Source:** [Wikipedia Clickstream Data (July & August 2025)](https://dumps.wikimedia.org/other/clickstream/)
- **Tools and Libraries:** Python, Apache Spark (PySpark), Spark ML, Jupyter notebook, Matplotlib

---


##  How to Run

1.  Have a Spark environment (like `pyspark`) installed.
2.  Place the clickstream data (`clickstream-enwiki-2025-07.tsv`, etc.) in the working directory.
3.  Run the `phases.ipynb` notebook first to perform preprocessing, training, and save the models.
4.  Run the `anomaly_eval.ipynb` notebook to load the models, perform detection on the test data, and generate the final analysis.

## Authors

* **Vikasan S Nayak**
* **Dube Prajwal Manoj**
* **Uday Manoj Todi**

Under the guidance of **Dr. Anup Bhat B**.
