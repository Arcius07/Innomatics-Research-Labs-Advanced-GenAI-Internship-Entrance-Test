# Food Delivery Data Integration & Analysis

This project demonstrates a complete data workflow (ETL and EDA) for a food delivery service. It involves extracting data from multiple formats (CSV, JSON, and SQL), merging them into a unified dataset, and performing analytical queries to derive business insights.

# Project Overview

The objective of this project is to consolidate disparate data sources—orders, user demographics, and restaurant details—into a single "Golden Dataset" and answer key performance questions regarding revenue, user behavior, and restaurant ratings.

# Dataset Description

The analysis is performed on three primary data sources:

## Orders (CSV): Transactional data including order_id, user_id, restaurant_id, order_date, and total_amount.

## Users (JSON): Customer information including user_id, name, city, and membership status (Regular or Gold).

## Restaurants (SQL): Metadata about restaurants including restaurant_id, cuisine, and rating.

# Requirements

To run the analysis, ensure you have Python installed along with the following libraries:
```
# Run this in your terminal to install dependencies
pip install pandas
```

Note: sqlite3 and json are part of the Python Standard Library and do not require separate installation.

# Complete Implementation Script

You can copy and paste the code below into a single .py file or a notebook cell to execute the entire workflow:
```
import pandas as pd
import sqlite3 as sq

# --- Step 1: Load Data ---
# Load CSV data
orders = pd.read_csv("orders.csv")

# Load JSON data
users = pd.read_json("users.json")

# Load SQL data
connection = sq.connect("restaurants.db")
cursor = connection.cursor()

# Execute SQL script to populate the database
with open("restaurants.sql", "r") as f:
    sql_script = f.read()
cursor.executescript(sql_script)
connection.commit()

# Read SQL table into Pandas
restaurants = pd.read_sql("SELECT * FROM restaurants", connection)

# --- Step 2: Merge the Data ---
# Merge orders with users
merge_df = pd.merge(orders, users, on="user_id", how="left")

# Merge the result with restaurants
final_df = pd.merge(merge_df, restaurants, on="restaurant_id", how="left")

# Export the final "Golden Dataset"
final_df.to_csv("final_food_delivery_dataset.csv", index=False)

# --- Step 3: Key Insights ---
# 1. Total revenue from Gold members by city
gold_members = final_df[final_df["membership"] == "Gold"]
city_revenue = gold_members.groupby("city")["total_amount"].sum().sort_values(ascending=False)
print(f"Top City (Gold): {city_revenue.idxmax()} with ₹{city_revenue.max():.2f}")

# 2. Average order value for Gold members
avg_gold = gold_members["total_amount"].mean()
print(f"Avg Order Value (Gold): ₹{round(avg_gold, 2)}")

# 3. Percentage of orders by Gold members
gold_pct = round((len(gold_members) / len(final_df)) * 100)
print(f"Gold Member Order Volume: {gold_pct}%")

connection.close()
```

# Key Insights & Findings

Top Revenue City (Gold Members): Chennai generated the highest total revenue ($₹1,080,909.79$).

Cuisine Performance: Mexican cuisine holds the highest average order value ($\approx ₹808.02$).

User Behavior: 2,544 distinct users placed orders totaling more than $₹1,000$.

Temporal Trends: The 3rd Quarter saw the highest total revenue ($₹2,037,385.10$).

Quality Metrics: 3,374 orders were placed at restaurants rated $\ge 4.5$.

# How to Use

Ensure orders.csv, users.json, and restaurants.sql are in your working directory.

Run the implementation script provided above.

The final dataset will be generated as final_food_delivery_dataset.csv.
