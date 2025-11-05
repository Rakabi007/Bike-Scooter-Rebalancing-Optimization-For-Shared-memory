# Geospatial Optimization Engine for Shared Mobility Rebalancing

## Overview

This project addresses the "rebalancing problem" inherent in shared mobility networks, such as Citi Bike or Portland Biketown. It is a geospatial optimization engine that ingests real-time data to identify system imbalances and generates actionable shipping plans for operations teams. The goal is to minimize "no bike" (lost revenue) and "no dock" (operational overhead) scenarios by algorithmically determining the most efficient way to reset the system to a neutral state.

## The Business Problem

Bike-share systems are two-sided marketplaces with spatial constraints. Users require bikes to begin trips (supply) and empty docks to end them (demand). Due to the nature of one-way trips (e.g., morning commutes downhill), bikes naturally accumulate in certain areas while disappearing from others. This imbalance leads to significant business costs:

* **Lost Revenue:** Users unable to find a bike nearby may churn.
* **Operational Overhead:** Users arriving at full stations require support or manual intervention.

My objective was to minimize these costs by creating an automated system to guide rebalancing efforts.

## Approach & Methodology

The project follows standard data science stages: Data Engineering, Feature Engineering, and Modeling.

### 1. Data Engineering (ETL Pipeline)

A live ETL pipeline was built using the **GBFS (General Bikeshare Feed Specification)** standard, rather than relying on static datasets.

* **Extract:** Real-time data is pulled from live API endpoints for station information (static capacities) and station status (current inventory) using `requests`.
* **Transform:** `pandas` is used to merge disparate feeds, clean data, handle missing values, and filter out non-operational stations.
* **Load (Spatial):** Raw latitude/longitude coordinates (degrees) are converted into a projected coordinate system (UTM meters) using `geopandas`. This is crucial for calculating realistic operational distances in meters.

### 2.(Feature Engineering)

Defining a "balanced" station goes beyond simple vacancy. A **Target Fill Rate of 45%** was established as a strategic buffer, ensuring sufficient availability for both renters and returns.

Key engineered features include:
* **Surplus:** The number of bikes to remove from a station.
* **Deficit:** The number of bikes needed at a station.

These features transform raw inventory data into clear operational signals.

### 3. The Algorithm (Modeling)

An initial **Greedy K-Nearest Neighbor** approach was chosen for its speed and interpretability, mimicking human decision-making ("Where is the closest place that needs these bikes?").

* **Why Greedy?** It provides a fast, intuitively understandable solution.
* **Why K=3?** Limiting matches to the three nearest stations ensures generated trips are short and operationally feasible for ground crews.

### 4. Visualization (Delivery)

To make the algorithm's output actionable for operations teams, an interactive map was built using `folium`.
* **Green/Red Markers:** Instantly indicate system state (donor vs. receiver stations).
* **Blue Lines:** Visualize the proposed shipping plan, allowing for easy validation of the model's logic.

## Future Improvements

While the current model serves as a strong baseline, several enhancements are planned for future versions:

* **Predictive Rebalancing:** Moving from reactive (current inventory) to predictive rebalancing by incorporating time-series forecasting models (e.g., Prophet, XGBoost) to anticipate future demand.
* **Optimization:** Replacing the greedy loop with a **Min-Cost Max-Flow network algorithm** (using Google OR-Tools) to find mathematically optimal moves, potentially improving global efficiency.
* **Vehicle Routing Problem (VRP):** Expanding the output from a simple list of shipments to actual driving routes for rebalancing trucks (Station A -> B -> C -> D) using a VRP solver.

## Technologies Used

* Python
* pandas
* geopandas
* requests
* folium
* GBFS (General Bikeshare Feed Specification)

