# Ironman_Triathlon_Analysis
![image](https://github.com/user-attachments/assets/7d6bc481-1e3e-4338-9865-6d5eee666149)

Welcome to my Ironman 70.3 analysis project, where I combine data analytics with athletic performance to gain actionable insights into one of the most demanding endurance challenges. This project focuses on a 6-year historical analysis of the Ironman 70.3 Augusta race, leveraging data to forecast potential outcomes for my participation in the upcoming Ironman 70.3 Rockford race on June 22, 2025. Through web scraping, advanced data analysis, and predictive modeling, this repository not only helps me understand what it takes to succeed but also provides other athletes with a roadmap for setting and achieving their own goals. The ultimate aim is to uncover trends, identify key areas for improvement, and devise a data-driven training strategy to strive for a podium finish.


## So, What Is an Ironman 70.3?
An Ironman 70.3, also known as a "Half Ironman," is one of the hardest endurance events in the world. It challenges athletes to complete a 70.3-mile race, combining three segments:

__1. Swimming: 1.2 miles (1.9 km) in open water.__

**2. Cycling: 56 miles (90 km) on the bike.**

**3. Running: 13.1 miles (21.1 km), which is a half-marathon.**

Athletes must complete all three segments consecutively within a strict time limit (8 hours and 30 minutes), testing their physical and mental endurance. These races attract participants from all over the world, ranging from seasoned professionals to first-time competitors, each striving to push their limits.

For me, the 2025 Ironman 70.3 Rockford is not just a personal goal but also an opportunity to explore how data analytics can optimize performance. This project captures that journey—breaking down every aspect of the race through data to understand where I currently stand and how I can improve for the future.

## Efficient Data Extraction: Building the Foundation
Data is the foundation of any good analysis, but for this project, I faced a unique challenge: there were no publicly available datasets tailored to Ironman 70.3 races. To overcome this, I turned to web scraping, a method of extracting data directly from websites. Using Python, I carefully collected performance data for the Ironman 70.3 Augusta races from the years 2017, 2018, 2019, 2021, 2022, and 2023. Why these races? Augusta is similar in terrain and conditions to the upcoming Rockford race, making it an ideal comparison.

The web scraping process allowed me to extract critical metrics from thousands of athletes, such as swim, bike, and run times, as well as transition times and overall placements. These metrics were saved into a CSV file to create a raw, unstructured dataset ready for cleaning and transformation. This step ensured I started my analysis with a comprehensive, reliable dataset.

To keep the project streamlined, I stored the web scraping code separately in the Ironman_70.3_DataExtraction repository. While the code isn’t included here, it’s available for review if you’d like to see how I tailored the Python scripts specifically for this analysis.

## Transitioning Data into RStudio for Analytical Precision
To ensure the dataset was ready for a robust analysis, I imported the cleaned data into RStudio using the read.csv function. From there, categorical columns were converted to factors for efficient data handling, while time columns (originally formatted as h:mm:ss) were converted into seconds using the lubridate package for easier numerical operations. These time values were later reformatted back into h:mm:ss for readability in visualizations. Additionally, I created a reference table detailing the official Half Ironman segment distances in miles, yards, and meters. This reference ensures consistency when performing metric calculations or cross-comparing results across analyses. Below is a streamlined code snippet detailing these steps.

### Code Walkthrough
#### 1. Loading and Preparing the Dataset
The dataset is loaded from a CSV file, and categorical columns are converted to factors for better processing and summarization. This ensures seamless handling of group-based operations.

```R
# Load the necessary libraries
library(dplyr)
library(lubridate)
library(timevis)
library(forecast)
library(tidyr)
library(ggplot2)
library(scales)
# Creating Initial Datasets ----
# Loading the CSV file
ironman_data <- read.csv("C:/Users/17737/Downloads/Ironman Analysis folder/Ironman_703_augusta_results.csv", stringsAsFactors = FALSE)

# Convert categorical columns to factors
ironman_data$Year_of_Race <- as.factor(ironman_data$Year_of_Race)
ironman_data$Country <- as.factor(ironman_data$Country)
ironman_data$Gender <- as.factor(ironman_data$Gender)
ironman_data$Division <- as.factor(ironman_data$Division)
ironman_data$Finish_Status <- as.factor(ironman_data$Finish_Status)
```
#### 2. Formatting Time Columns for Analysis
Time columns are converted into seconds for precision in calculations, such as ranking or time-based comparisons, using the lubridate::period_to_seconds function. After performing numerical operations, the times are reformatted back to h:mm:ss for readability.
```R 
# Convert the time columns from h:mm:ss format using lubridate's period function
ironman_data$Overall_Time <- period_to_seconds(hms(ironman_data$Overall_Time))
ironman_data$Swim_Time <- period_to_seconds(hms(ironman_data$Swim_Time))
ironman_data$Bike_Time <- period_to_seconds(hms(ironman_data$Bike_Time))
ironman_data$Run_Time <- period_to_seconds(hms(ironman_data$Run_Time))
ironman_data$Total_Transition_Time <- period_to_seconds(hms(ironman_data$Total_Transition_Time))

# Reconvert the time columns back to the format h:mm:ss for readability
ironman_data <- ironman_data |> 
  mutate(
    Overall_Time = seconds_to_period(Overall_Time),
    Swim_Time = seconds_to_period(Swim_Time),
    Bike_Time = seconds_to_period(Bike_Time),
    Run_Time = seconds_to_period(Run_Time),
    Total_Transition_Time = seconds_to_period(Total_Transition_Time)
  )

# Confirm structure to ensure the columns imported correctly
str(ironman_data)
head(ironman_data)
```
#### 4. Creating a Reference Table for Metric Calculations
A reference table is built to define Half Ironman segment distances in various units. This table serves as a reliable source for metric conversions and ensures consistent analysis across datasets.
```R
# Create table for ironman distance
half_ironman_distances <- data.frame(
  Distance = c("Overall_Distance", "Swim_Distance", "Bike_Distance", "Run_Distance"),
  Miles = c(70.3, 1.2, 56, 13.1),
  Yards = c(123728, 2112, 98560, 23056),
  Meters = c(113136.9, 1931.21, 90123.3, 21082.41)
)

# Print the data frame
print(half_ironman_distances)
```
![image](https://github.com/user-attachments/assets/2f06d28f-ca2e-4b12-bba6-f93544b271f1)
<img width="500" src="https://github-production-user-asset-6210df.s3.amazonaws.com/181696165/387429850-c94e0bc9-f6b4-409f-b772-4d5564bd4956.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20241202%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241202T082422Z&X-Amz-Expires=300&X-Amz-Signature=393ac30128689bbc8b3a5ee62ba3594b6452ce8a9d854dd756029322a4ef7853&X-Amz-SignedHeaders=host" />


## Transforming Raw Times into Actionable Metrics
To better analyze athlete performance, I derived segment-specific pace and speed metrics, converting raw times into meaningful, interpretable values. Swimming speed was calculated as the average pace per 100 meters, biking speed as miles per hour (MPH), and running speed as minutes per mile. These metrics offer valuable insights into performance trends and facilitate detailed comparisons across athletes and events. Below is a detailed explanation of the code used to calculate and integrate these metrics.

### Code Walkthrough
#### 1. Extracting Distances for Calculations
To ensure accurate calculations, the segment distances were retrieved from the half_ironman_distances reference table. This approach eliminates hardcoding values, maintaining flexibility and consistency.
```R
# Extract the segment distances in the correct units from `half_ironman_distances`
swim_distance_m <- half_ironman_distances |> filter(Distance == "Swim_Distance") |> pull(Meters)
bike_distance_miles <- half_ironman_distances |> filter(Distance == "Bike_Distance") |> pull(Miles)
run_distance_miles <- half_ironman_distances |> filter(Distance == "Run_Distance") |> pull(Miles)
```
#### 2. Calculating Pace and Speed Metrics
Using the extracted distances, new columns were added to the dataset to calculate:

- Swim Pace: Time per 100 meters, formatted as h:mm:ss for easy interpretation.
- Bike Speed: Average speed in MPH, rounded to two decimal places.
- Run Pace: Average time per mile, formatted as h:mm:ss.
```R
# Adding calculated pace and speed columns to `ironman_data`
ironman_data <- ironman_data |> 
  mutate(
    # Swim pace: average time per 100 meters in HMS format
    Swim_Pace_100m = seconds_to_period(round((period_to_seconds(Swim_Time) / swim_distance_m) * 100)),
    
    # Bike speed: average speed in miles per hour (MPH), rounded to 2 decimal places
    Bike_Speed_MPH = round(bike_distance_miles / (period_to_seconds(Bike_Time) / 3600), 2),
    
    # Run pace: average time per mile in HMS format
    Run_Pace_per_Mile = seconds_to_period(round(period_to_seconds(Run_Time) / run_distance_miles))
  )

# Display the updated data structure to confirm new columns
print(ironman_data |> select(Swim_Pace_100m, Bike_Speed_MPH, Run_Pace_per_Mile))
str(ironman_data)
```
![image](https://github.com/user-attachments/assets/9cb124c5-37e3-4b91-bc14-ceaa22139c92)


## Predicting the Future of Ironman Performance
To prepare for the 2025 Ironman 70.3 race, I conducted a regression analysis to forecast performance times for two groups: the top 23 athletes in the M18-24 division and the top 1,000 athletes overall. By analyzing historical data, I identified trends in average swim, bike, run, and overall times. These trends allowed me to predict 2025 performance metrics, offering insights into the race's increasing competitiveness and helping set realistic expectations for my training goals. Below, I break down the key steps and processes in this forecasting analysis specifically for the Top 23 M18-24 forecast. I have not included the code for the Top 1000 Overall, however it was forecasted using the exact same methodology, simply different filters.

### Part 1: Forecasting Segment Times for the Top 23 Athletes in Division M18-24
#### 1. Data Filtering
We began by filtering the dataset for the top 23 athletes in the M18-24 division, grouped by year, to ensure the analysis was focused on elite athletes.
```R
top_23_m18_24 <- ironman_data |> 
  filter(Division == "M18-24") |> 
  group_by(Year_of_Race) |> 
  arrange(Division_Rank) |> 
  filter(Division_Rank %in% 1:23) |> 
  ungroup()

top_23_m18_24 <- top_23_m18_24 |> 
  mutate(Division_Rank = as.numeric(Division_Rank))
````
#### 2. Converting HMS to Seconds
To facilitate time-based calculations, all times (swim, bike, run, and overall) were converted from the hms format into seconds using the period_to_seconds function. This standardization was necessary for regression analysis and forecasting.
```R
top_23_times_in_secs <- top_23_m18_24 |> 
  group_by(Division_Rank, Year_of_Race) |> 
  summarize(
    Avg_Swim_Time = mean(period_to_seconds(Swim_Time), na.rm = TRUE),
    Avg_Bike_Time = mean(period_to_seconds(Bike_Time), na.rm = TRUE),
    Avg_Run_Time = mean(period_to_seconds(Run_Time), na.rm = TRUE),
    Avg_Overall_Time = mean(period_to_seconds(Overall_Time), na.rm = TRUE)
  ) |> 
  filter(!is.na(Avg_Swim_Time), !is.na(Avg_Bike_Time), !is.na(Avg_Run_Time), !is.na(Avg_Overall_Time)) |> 
  ungroup()
```
#### 3. Trend Calculation Function
A custom function, calculate_trend, was designed to compute year-over-year changes and predict future values. This function calculates the average annual change in times and uses it to forecast 2025 metrics.
```R
calculate_trend <- function(time_column) {
  # Get the changes in time year-over-year for each rank
  time_diffs <- diff(time_column, lag = 1)
  # Calculate the average year-over-year change
  avg_change_per_year <- mean(time_diffs, na.rm = TRUE)
  # Forecast 2025 time by adding this average change to the last known time
  forecast_2025 <- last(time_column) + avg_change_per_year
  # Convert to HMS format if forecast is positive; otherwise, NA
  if (forecast_2025 > 0) {
    return(seconds_to_period(round(forecast_2025)))
  } else {
    return(NA)
  }
}
```
#### 4. Forecasting 2025 Performance
Using the calculate_trend function, we forecasted the swim, bike, run, and overall times for each rank in 2025.
```R
pred_top23_m1824 <- top_23_times_in_secs |> 
  group_by(Division_Rank) |> 
  summarize(
    Predicted_Swim_Time_2025 = calculate_trend(Avg_Swim_Time),
    Predicted_Bike_Time_2025 = calculate_trend(Avg_Bike_Time),
    Predicted_Run_Time_2025 = calculate_trend(Avg_Run_Time),
    Predicted_Overall_Time_2025 = calculate_trend(Avg_Overall_Time)
  ) |> 
  ungroup()

# Display the predicted times for 2025 for each rank in HMS format
print(pred_top23_m1824)
```
#### Result:
![image](https://github.com/user-attachments/assets/5d86a606-e7a7-4251-b634-28cfe0aa85ea)

### Part 2: Mutating Predicted Data
To enhance the interpretability of the forecasted data, several columns were added to include predicted rankings and metrics.
#### 1. Adding Division Rank Column
```R
pred_top23_m1824 <- pred_top23_m1824 |> 
  arrange(Predicted_Overall_Time_2025) |>  # Arrange by Predicted Overall Time
  mutate(Division_Rank = row_number())     # Create Division Rank based on sorted Predicted Overall Time
```
#### 2. Adding Segment Ranks
```R
pred_top23_m1824 <- pred_top23_m1824 |> 
  mutate(
    Predicted_Swim_Rank_2025 = rank(Predicted_Swim_Time_2025, ties.method = "first"),
    Predicted_Bike_Rank_2025 = rank(Predicted_Bike_Time_2025, ties.method = "first"),
    Predicted_Run_Rank_2025 = rank(Predicted_Run_Time_2025, ties.method = "first")
  )
```
#### 3. Add Transition Time Column
```R
# Convert all relevant times to seconds for accurate calculation
pred_top23_m1824 <- pred_top23_m1824 |> 
  mutate(
    Predicted_Overall_Time_2025_seconds = period_to_seconds(Predicted_Overall_Time_2025),
    Predicted_Swim_Time_2025_seconds = period_to_seconds(Predicted_Swim_Time_2025),
    Predicted_Bike_Time_2025_seconds = period_to_seconds(Predicted_Bike_Time_2025),
    Predicted_Run_Time_2025_seconds = period_to_seconds(Predicted_Run_Time_2025),
    
    # Calculate Transition Time in seconds and then convert back to HMS
    Predicted_Total_Transition_Time_seconds = Predicted_Overall_Time_2025_seconds - Predicted_Swim_Time_2025_seconds - Predicted_Bike_Time_2025_seconds - Predicted_Run_Time_2025_seconds,
    
    # Convert the transition time from seconds back to HMS format
    Predicted_Total_Transition_Time = seconds_to_period(Predicted_Total_Transition_Time_seconds)
  ) |> 
  select(-ends_with("_seconds"))  # Remove the columns in seconds for a cleaner table
```
#### 4. Add Pace Metrics
```R
# Step 4: Add swim pace, bike speed, and run pace columns based on the same formulas used earlier

# Constants for distances
swim_distance_m <- half_ironman_distances |> filter(Distance == "Swim_Distance") |> pull(Meters)
bike_distance_miles <- half_ironman_distances |> filter(Distance == "Bike_Distance") |> pull(Miles)
run_distance_miles <- half_ironman_distances |> filter(Distance == "Run_Distance") |> pull(Miles)

# Adding calculated pace and speed columns to the predicted tables
pred_top23_m1824 <- pred_top23_m1824 |> 
  mutate(
    Swim_Pace_100m = seconds_to_period(round((period_to_seconds(Predicted_Swim_Time_2025) / swim_distance_m) * 100)),
    Bike_Speed_MPH = round(bike_distance_miles / (period_to_seconds(Predicted_Bike_Time_2025) / 3600), 2),
    Run_Pace_per_Mile = seconds_to_period(round(period_to_seconds(Predicted_Run_Time_2025) / run_distance_miles))
  )


# Display the mutated tables
print(pred_top23_m1824)
```
#### New Table With Mutated Columns:
![image](https://github.com/user-attachments/assets/de4540ec-3592-4b49-898d-3db8fecbe651)

### Visualizing the Trend Over Time
Here is now what the trend for the Top 23 Atheletes in the M18-24 division from 2017 to 2025 looks like:

![image](https://github.com/user-attachments/assets/364dfd7e-e1a6-4748-b030-5af22b037926)


## Where Do I Stand? Placing Myself in the Field
To understand where I stand competitively, I used the forecasted rankings to place myself in both my age division (M18-24) and the overall rankings for the 2025 Ironman 70.3 Rockford. By inputting my current swimming, biking, and running metrics, I calculated my predicted total race time and compared it to the forecasted top athletes. This analysis allows me to identify the gap between my current performance and my competitive goals. Although I followed the same steps for both rankings, this section focuses on my placement in the M18-24 division for simplicity.

### Code Walkthrough
#### 1. Define My Metrics/Pace
First, I defined my current paces for swimming (seconds per 100 meters), biking (miles per hour), and running (seconds per mile). These serve as the inputs to calculate my expected performance during the race.
```R
my_name <- "Jesus Mendoza-Garmendia"
my_swim_pace_100m <- 110    # Swim pace in seconds per 100 meters
my_bike_speed_mph <- 21     # Bike speed in miles per hour
my_run_pace_per_mile <- 570 # Run pace in seconds per mile
```
#### 2. Calculate My Segment Times
Using the race distances for each segment (swim, bike, run), I calculated the time required for each discipline based on my current metrics. The distances were extracted from a pre-defined dataset (half_ironman_distances).
```R
# Extract Distances from `half_ironman_distances` dataframe
swim_distance_m <- half_ironman_distances |> filter(Distance == "Swim_Distance") |> pull(Meters)
bike_distance_miles <- half_ironman_distances |> filter(Distance == "Bike_Distance") |> pull(Miles)
run_distance_miles <- half_ironman_distances |> filter(Distance == "Run_Distance") |> pull(Miles)

# Calculate segment times in seconds
my_swim_time <- round((swim_distance_m / 100) * my_swim_pace_100m)
my_bike_time <- round((bike_distance_miles / my_bike_speed_mph) * 3600)  # Convert hours to seconds
my_run_time <- round(run_distance_miles * my_run_pace_per_mile)
my_transition_time <- 6 * 60  # 6 minutes in seconds
```
#### 3. Calculate Total Race Time
I summed the segment times (swim, bike, run) with the transition time to estimate my total race time.

```R
my_total_time <- my_swim_time + my_bike_time + my_run_time + my_transition_time
```
#### 4. Prepare Forecasted Tables
To integrate my results, I ensured the forecasted tables (pred_top23_m1824 and pred_top1000) included all necessary columns and converted key columns to numeric types for accurate ranking.
```R
pred_top23_m1824 <- pred_top23_m1824 |> mutate(
  Swim_Pace_100m = as.numeric(Swim_Pace_100m),
  Bike_Speed_MPH = as.numeric(Bike_Speed_MPH),
  Run_Pace_per_Mile = as.numeric(Run_Pace_per_Mile)
)
```
#### 5. Insert My Data
I added my metrics and predicted times into the Top 23 (M18-24) table. Each row includes segment paces, times, and the total race time in human-readable format (HH:MM:SS). After inserting the data, I recalculated rankings to show where I would stand.
```R
# Convert times to HMS format for display
my_swim_time_hms <- seconds_to_period(my_swim_time)
my_bike_time_hms <- seconds_to_period(my_bike_time)
my_run_time_hms <- seconds_to_period(my_run_time)
my_transition_time_hms <- seconds_to_period(my_transition_time)
my_total_time_hms <- seconds_to_period(my_total_time)

# Insert into the M18-24 division table
my_m1824_placement <- pred_top23_m1824 |> 
  add_row(
    Name = my_name,
    Division_Rank = NA,  # Placeholder, will be recalculated
    Swim_Pace_100m = my_swim_pace_100m,
    Bike_Speed_MPH = my_bike_speed_mph,
    Run_Pace_per_Mile = my_run_pace_per_mile,
    Predicted_Swim_Time_2025 = my_swim_time_hms,
    Predicted_Bike_Time_2025 = my_bike_time_hms,
    Predicted_Run_Time_2025 = my_run_time_hms,
    Predicted_Total_Transition_Time = my_transition_time_hms,
    Predicted_Overall_Time_2025 = my_total_time_hms
  ) |> 
  arrange(Predicted_Overall_Time_2025) |> 
  mutate(
    Division_Rank = row_number(),
    Predicted_Swim_Rank_2025 = rank(Predicted_Swim_Time_2025),
    Predicted_Bike_Rank_2025 = rank(Predicted_Bike_Time_2025),
    Predicted_Run_Rank_2025 = rank(Predicted_Run_Time_2025)
  )
```
#### 6. Reorder Columns and Format
```R
# Reorder columns to place "Name" first
my_m1824_placement <- my_m1824_placement[, c("Name", setdiff(names(my_m1824_placement), "Name"))]

# Convert numeric pace values back to time format
my_m1824_placement <- my_m1824_placement |> mutate(
  Swim_Pace_100m = seconds_to_period(Swim_Pace_100m),
  Run_Pace_per_Mile = seconds_to_period(Run_Pace_per_Mile)
)

# Display the final placement tables
print("Predicted Rank in M18-24 Division:")
print(my_m1824_placement |> filter(Name == my_name))
```

### How I Stacked Up Against the Overall Competition

![image](https://github.com/user-attachments/assets/09a1ad4e-1022-4336-9967-0a277586324b)

### How I Stacked Up Against My Division

![image](https://github.com/user-attachments/assets/8e28aa60-0144-4dd7-adfb-df586584a373)



## Striving for the Podium: Scenario Simulation and Optimization
In this section, I explored how to achieve a podium finish (1st, 2nd, or 3rd place) in my division by simulating realistic performance scenarios. To do this, I analyzed the segment times (swim, bike, run, and transitions) of the top 3 athletes in my division and created a range of simulated performance metrics. These simulations helped identify the optimal paces and speeds required to surpass these athletes. By comparing the predicted top 3 times against these simulated results, I pinpointed actionable performance goals to aim for in training. This process required creating custom functions to handle the complex calculations and merging forecasted data with simulated results to provide a clear path for optimization. Below, I break down the steps of the code to help understand its purpose and functionality.
### Code Walkthrough
#### 1. Preparing Data for Simulation
The first step is to convert key columns in the forecasted table into seconds. This ensures accurate calculations of total times, which is essential for meaningful comparisons.
```R
sorted_div_placements <- pred_top23_m1824 |> 
  mutate(
    Swim_Pace_100m_sec = period_to_seconds(Swim_Pace_100m), 
    Run_Pace_per_Mile_sec = period_to_seconds(Run_Pace_per_Mile),
    Total_Time = period_to_seconds(Predicted_Overall_Time_2025)
  )
```
#### 2. Extracting Top 3 Finishers' Data
We isolate data for the top 3 athletes and assign placeholder names (P1, P2, P3) for easier reference.
```R
top3_div_athletes <- sorted_div_placements |>
  filter(Division_Rank == 1:3) |> 
  mutate(
    Name = case_when(
      Division_Rank == 1 ~ "P1",
      Division_Rank == 2 ~ "P2", 
      Division_Rank == 3 ~ "P3"
    )
  )
```
#### 3. Simulating Performance Scenarios
Here, we generate a range of swim paces, bike speeds, and run paces around the performance metrics of the top 3 athletes.
```R
# Adjust ranges to focus on realistic scenarios
simulated_paces <- expand.grid(
  Swim_Pace_100m_sec = seq(min(top3_div_athletes$Swim_Pace_100m_sec) + 6, 
                           max(top3_div_athletes$Swim_Pace_100m_sec) + 10, by = 2),
  Bike_Speed_MPH = seq(max(top3_div_athletes$Bike_Speed_MPH) - 2, 
                       max(top3_div_athletes$Bike_Speed_MPH) + 1, by = 0.5),
  Run_Pace_per_Mile_sec = seq(min(top3_div_athletes$Run_Pace_per_Mile_sec) + 19, 
                              max(top3_div_athletes$Run_Pace_per_Mile_sec) + 39, by = 10)
)
```
#### 4. Calculating Total Times for Each Scenario
The simulated segment paces are converted into total times (including transitions).
```R
simulated_results <- simulated_paces |>
  mutate(
    Swim_Time = (swim_distance_m / 100) * Swim_Pace_100m_sec,
    Bike_Time = (bike_distance_miles / Bike_Speed_MPH) * 3600,
    Run_Time = run_distance_miles * Run_Pace_per_Mile_sec,
    Transition_Time = 6 * 60,  # Keep transition time constant
    Total_Time = Swim_Time + Bike_Time + Run_Time + Transition_Time
  ) |>
  arrange(Total_Time)  # Sort by fastest to slowest
```
#### 5. Merging Simulations with Top Athletes’ Data
We merge the simulated scenarios with the top 3 athletes' data and recalculate rankings based on total times.
```R
optimized_rankings <- top3_div_athletes |>
  bind_rows(
    simulated_results |>
      mutate(Name = "Simulated Scenario")
  ) |>
  arrange(Total_Time) |>
  mutate(Division_Rank = row_number()) |> 
  mutate(
    Swim_Pace_100m = seconds_to_period(Swim_Pace_100m_sec),
    Run_Pace_per_Mile = seconds_to_period(Run_Pace_per_Mile_sec)
  )
```
#### 6. Identifying Times to Beat Each Top Rank
This step pinpoints the slowest times needed to surpass the current top 3 athletes.
```R
p1_row <- which(optimized_rankings$Name == "P1")
p2_row <- which(optimized_rankings$Name == "P2")
p3_row <- which(optimized_rankings$Name == "P3")

# Select the rows directly above P1, P2, and P3
slowest_for_top3 <- optimized_rankings |> 
  slice(c(p1_row - 1, p2_row - 1, p3_row - 1)) |> 
  mutate(Placement_I_Get = rank(Division_Rank, ties.method = "first")) |> 
  mutate(Predicted_Overall_Time_2025 = round(seconds_to_period(Total_Time)))
```
### Breaking Down the Podium: Segment Goals by Placement
![image](https://github.com/user-attachments/assets/275e3347-55a9-459d-865b-1d7db96c8028)

### Closing the Gap: Swim Pace Improvement
#### -------- Current Swim Pace -------- VS -------- Required Swim Pace

<img width="800" src="https://github-production-user-asset-6210df.s3.amazonaws.com/181696165/391376761-21e9b0b5-e059-4541-ac4b-50ac60ec2a20.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20241202%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241202T004411Z&X-Amz-Expires=300&X-Amz-Signature=bd31e111205fdfc6ef6e30e5d4c2171b881b6e4bd3cf1da5e30be1b049505c4c&X-Amz-SignedHeaders=host" />

### Pedaling to the Podium: Bike Speed Progression
####      Current Bike Pace      VS      Required Bike Pace
 
<img width="800" src="https://private-user-images.githubusercontent.com/181696165/391381333-c29207b9-febc-4aab-b10e-e7b37ad3c31e.gif?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MzMxMDI4NjAsIm5iZiI6MTczMzEwMjU2MCwicGF0aCI6Ii8xODE2OTYxNjUvMzkxMzgxMzMzLWMyOTIwN2I5LWZlYmMtNGFhYi1iMTBlLWU3YjM3YWQzYzMxZS5naWY_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQxMjAyJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MTIwMlQwMTIyNDBaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT00ZTA1MTBlM2I5NDY4NjIxZTQ1NjdhN2M1ZTM2NDI0MzI5M2MxYjQ5OGVmNzk4N2I4NDRhMTUzMmJhNzUzNTM5JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.6PcinIfI9ox5wRKSRIa4InkSt_TibH81IuALCI3zp68" />

### The Final Stretch: Running Pace Refinement
####      Current Run Pace      VS      Required Run Pace

<img width="800" src="https://github-production-user-asset-6210df.s3.amazonaws.com/181696165/391377930-0dcaf18b-69fa-4da7-b362-ae9af831830a.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20241202%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241202T005423Z&X-Amz-Expires=300&X-Amz-Signature=894bd1a54e019e69a672061e98ecb742f042e71359d8865be8f2b7abd1020ebb&X-Amz-SignedHeaders=host" />

## Prioritizing Training: Segment Sensitivity Analysis
Using simulated results, I conducted a sensitivity analysis to determine which segment improvements would yield the greatest overall time reductions. This analysis revealed which disciplines—swimming, biking, or running—warrant the most focus, helping prioritize training efforts for maximum impact on race performance.
```R
# Sensitivity Analysis: Rank Change by Segment
sensitivity_analysis <- optimized_rankings |>
  pivot_longer(cols = c(Swim_Pace_100m_sec, Bike_Speed_MPH, Run_Pace_per_Mile_sec),
               names_to = "Segment", values_to = "Value") |>
  mutate(Value = case_when(
    Segment == "Run_Pace_per_Mile_sec" ~ -Value,
    Segment == "Swim_Pace_100m_sec" ~ -Value,
    TRUE ~ Value  
  )) |>
  group_by(Segment) |>
  summarise(
    Correlation_with_Total_Time = cor(Value, Total_Time)  # Correlation with total time
  )

print(sensitivity_analysis)
```
![image](https://github.com/user-attachments/assets/9a4f651c-acca-49c8-97c7-69d83fede9d2)
