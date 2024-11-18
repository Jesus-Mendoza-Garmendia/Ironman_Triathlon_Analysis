# Ironman_Triathlon_Analysis
Ironman 70.3 Analysis and Performance Prediction - A data-driven approach to analyze historical Ironman 70.3 Augusta data to predict results for my upcoming Ironman 70.3 Rockford race.

## Efficient Data Extraction: Building the Foundation
With limited publicly available datasets, I began my analysis by web scraping performance data for Ironman 70.3 Augusta races from 2017, 2018, 2019, 2021, 2022, and 2023. These races were selected due to their similarity to the Rockford Ironman 70.3 event I plan to compete in for 2025. Using Python, I extracted critical metrics for thousands of athletes, ensuring a comprehensive dataset. The data was exported to a CSV file and underwent preliminary cleaning to prepare for further analysis. (Note: Web scraping code is stored separately in the repository Ironman_70.3_DataExtraction.)

## Transitioning Data into RStudio for Analytical Precision
The cleaned dataset was imported into RStudio using the read.csv function, with additional formatting applied to time columns to ensure accurate analysis. Additionally, I created a reference table detailing the Half Ironman segment distances (in miles, yards, and meters) to facilitate metric calculations and ensure consistency in future analyses.

## Transforming Raw Times into Actionable Metrics
To interpret athlete performance effectively, I calculated pace and speed metrics for each segment. Swimming speed was expressed as 100m pace, biking speed in miles per hour (MPH), and running speed in minutes per mile. These derived metrics provided deeper insights into performance patterns, enabling meaningful comparisons across athletes and years.

## Unveiling Trends with Summary Tables
To understand performance benchmarks, I summarized average segment times by year for the top 3 athletes in the M18-24 division. This exploration provided a clear view of historical performance trends, serving as a baseline to evaluate my own metrics and identify areas for improvement.

## Predicting the Future of Ironman Performance
I used regression analysis to create a forecast for the 2025 Ironman 70.3 race. This analysis focused on predicting times for the top 1,000 overall athletes and the top 23 in the M18-24 division. By identifying trends in performance improvement over the years, the forecast offers insights into the race's increasing competitiveness, helping set realistic expectations for my performance.

## Where Do I Stand? Placing Myself in the Field
Using the forecasted tables, I predicted my division (M18-24) and overall rankings for 2025. Inputting my current metrics for swimming, biking, and running, the analysis placed me relative to the top athletes. This step provided a personalized benchmark to understand how close I am to achieving my competitive goals.

## Striving for the Podium: Scenario Simulation and Optimization
To explore paths to success, I developed a simulator for segment times based on sequences of performance metrics. By combining simulated results with forecasted data for the top 3 athletes in my division, I identified optimal scenarios for achieving 1st, 2nd, or 3rd place. A custom function pinpointed the closest faster times needed to meet each target, offering actionable performance goals for training.

## Prioritizing Training: Segment Sensitivity Analysis
Using simulated results, I conducted a sensitivity analysis to determine which segment improvements would yield the greatest overall time reductions. This analysis revealed which disciplines—swimming, biking, or running—warrant the most focus, helping prioritize training efforts for maximum impact on race performance.

