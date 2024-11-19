# Ironman_Triathlon_Analysis
Ironman 70.3: A Data-Driven Approach to Performance Prediction - This repository contains a 6-year analysis of the Ironman 70.3 Augusta race. The analysis leverages historical data to predict performance metrics for my upcoming Ironman 70.3 Rockford race in 2025. By understanding past trends and patterns, this project aims to provide valuable insights for myself and other athletes.

## Efficient Data Extraction: Building the Foundation
With limited publicly available datasets, I began my analysis by web scraping performance data for Ironman 70.3 Augusta races from 2017, 2018, 2019, 2021, 2022, and 2023. These races were selected due to their similarity to the Rockford Ironman 70.3 event I plan to compete in for 2025. Using Python, I extracted critical metrics for thousands of athletes, ensuring a comprehensive dataset. The data was exported to a CSV file and underwent preliminary cleaning to prepare for further analysis. (Note: Web scraping code is stored separately in the repository Ironman_70.3_DataExtraction.)

## Transitioning Data into RStudio for Analytical Precision
The cleaned dataset was imported into RStudio using the read.csv function, with additional formatting applied to time columns to ensure accurate analysis. Additionally, I created a reference table detailing the Half Ironman segment distances (in miles, yards, and meters) to facilitate metric calculations and ensure consistency in future analyses.
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

![image](https://github.com/user-attachments/assets/c94e0bc9-f6b4-409f-b772-4d5564bd4956)


## Transforming Raw Times into Actionable Metrics
To interpret athlete performance effectively, I calculated pace and speed metrics for each segment. Swimming speed was expressed as 100m pace, biking speed in miles per hour (MPH), and running speed in minutes per mile. These derived metrics provided deeper insights into performance patterns, enabling meaningful comparisons across athletes and years.
```R
# Extract the segment distances in the correct units from `half_ironman_distances`
swim_distance_m <- half_ironman_distances |> filter(Distance == "Swim_Distance") |> pull(Meters)
bike_distance_miles <- half_ironman_distances |> filter(Distance == "Bike_Distance") |> pull(Miles)
run_distance_miles <- half_ironman_distances |> filter(Distance == "Run_Distance") |> pull(Miles)

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


## Unveiling Trends with Summary Tables
To understand performance benchmarks, I summarized average segment times by year for the top 3 athletes in the M18-24 division. This exploration provided a clear view of historical performance trends, serving as a baseline to evaluate my own metrics and identify areas for improvement.
```R
## Step 1: Filter the data for the M18-24 division and select only the top 3 ranks for each year
top_3_m18_24 <- ironman_data |> 
  filter(Division == "M18-24") |> 
  group_by(Year_of_Race) |> 
  filter(Division_Rank == 1:3)

## Step 2: Calculate the average times per segment for each year in HMS format
top3_m18_24_yearly_avg <- top_3_m18_24 |> 
  summarize(
    Avg_Swim_Time = seconds_to_period(round(mean(period_to_seconds(Swim_Time), na.rm = TRUE))),
    Avg_Bike_Time = seconds_to_period(round(mean(period_to_seconds(Bike_Time), na.rm = TRUE))),
    Avg_Run_Time = seconds_to_period(round(mean(period_to_seconds(Run_Time), na.rm = TRUE))),
    Avg_Total_Time = seconds_to_period(round(mean(period_to_seconds(Overall_Time), na.rm = TRUE)))
  )

# Display the structure and content to confirm HMS formatting
print(top3_m18_24_yearly_avg)
str(top3_m18_24_yearly_avg)

## Step 3: Calculate the average segment times for each placement position
top3_m18_24_rank_avg <- top_3_m18_24 |> 
  group_by(Division_Rank) |> 
  summarize(
    Avg_Swim_Time = seconds_to_period(round(mean(period_to_seconds(Swim_Time), na.rm = TRUE))),
    Avg_Bike_Time = seconds_to_period(round(mean(period_to_seconds(Bike_Time), na.rm = TRUE))),
    Avg_Run_Time = seconds_to_period(round(mean(period_to_seconds(Run_Time), na.rm = TRUE))),
    Avg_Total_Time = seconds_to_period(round(mean(period_to_seconds(Overall_Time), na.rm = TRUE)))
  )

# Display the average times per placement
print(top3_m18_24_rank_avg)
str(top3_m18_24_rank_avg)
```
![image](https://github.com/user-attachments/assets/113c5158-1dc0-412f-82ce-2d3ab9d46b13)


## Predicting the Future of Ironman Performance
I used regression analysis to create a forecast for the 2025 Ironman 70.3 race. This analysis focused on predicting times for the top 1,000 overall athletes and the top 23 in the M18-24 division. By identifying trends in performance improvement over the years, the forecast offers insights into the race's increasing competitiveness, helping set realistic expectations for my performance.
```R
# Part 3.1: Forecasting Segment Times for the Top 23 Athletes in Division M18-24

# Step 1: Filter for top 23 athletes in the M18-24 division by rank each year
top_23_m18_24 <- ironman_data |> 
  filter(Division == "M18-24") |> 
  group_by(Year_of_Race) |> 
  arrange(Division_Rank) |> 
  filter(Division_Rank %in% 1:23) |> 
  ungroup()

top_23_m18_24 <- top_23_m18_24 |> 
  mutate(Division_Rank = as.numeric(Division_Rank))

# Step 2: Calculate average times per rank for each year and filter out missing data
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

# Step 3: Define a function to calculate a linear trend and predict the 2025 time
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

# Step 3: Apply the trend function to each rank and segment for 2025
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

#_______________________________________________________________________________
# Part 3.2 (Alternative): Forecasting Segment Times for the Top 1000 Athletes Overall

# Step 1: Filter for top 1000 athletes by overall rank each year
top_1000_overall <- ironman_data |> 
  arrange(Year_of_Race, Overall_Rank) |> # Sort by year and overall rank
  group_by(Year_of_Race) |> 
  filter(Overall_Rank %in% 1:1000) |> # Select top 1000 overall ranks
  ungroup()

# Step 2: Calculate average times per rank for each year and filter out missing data
top_1000_times_in_secs <- top_1000_overall |> 
  group_by(Overall_Rank, Year_of_Race) |> 
  summarize(
    Avg_Swim_Time = mean(period_to_seconds(Swim_Time), na.rm = TRUE),
    Avg_Bike_Time = mean(period_to_seconds(Bike_Time), na.rm = TRUE),
    Avg_Run_Time = mean(period_to_seconds(Run_Time), na.rm = TRUE),
    Avg_Overall_Time = mean(period_to_seconds(Overall_Time), na.rm = TRUE)
  ) |> 
  filter(!is.na(Avg_Swim_Time), !is.na(Avg_Bike_Time), !is.na(Avg_Run_Time), !is.na(Avg_Overall_Time)) |> 
  ungroup()

# Step 3: Apply the trend forecasting function to each rank and segment for 2025
pred_top1000 <- top_1000_times_in_secs |> 
  group_by(Overall_Rank) |> 
  summarize(
    Predicted_Swim_Time_2025 = calculate_trend(Avg_Swim_Time),
    Predicted_Bike_Time_2025 = calculate_trend(Avg_Bike_Time),
    Predicted_Run_Time_2025 = calculate_trend(Avg_Run_Time),
    Predicted_Overall_Time_2025 = calculate_trend(Avg_Overall_Time)
  ) |> 
  ungroup()

# Display the predicted times for 2025 for each rank in HMS format
print(pred_top1000)

#_______________________________________________________________________________
# Part 3.3: Mutate Additional Columns in Predicted Times Data

# Step 1: Update Division Rank based on Predicted Overall Time (in ascending order)
pred_top23_m1824 <- pred_top23_m1824 |> 
  arrange(Predicted_Overall_Time_2025) |>  # Arrange by Predicted Overall Time
  mutate(Division_Rank = row_number())     # Create Division Rank based on sorted Predicted Overall Time

pred_top1000 <- pred_top1000 |> 
  arrange(Predicted_Overall_Time_2025) |>  # Arrange by Predicted Overall Time
  mutate(Overall_Rank = row_number())      # Create Overall Rank based on sorted Predicted Overall Time

# Step 2: Add Predicted Swim, Bike, and Run Ranks
pred_top23_m1824 <- pred_top23_m1824 |> 
  mutate(
    Predicted_Swim_Rank_2025 = rank(Predicted_Swim_Time_2025, ties.method = "first"),
    Predicted_Bike_Rank_2025 = rank(Predicted_Bike_Time_2025, ties.method = "first"),
    Predicted_Run_Rank_2025 = rank(Predicted_Run_Time_2025, ties.method = "first")
  )

pred_top1000 <- pred_top1000 |> 
  mutate(
    Predicted_Swim_Rank_2025 = rank(Predicted_Swim_Time_2025, ties.method = "first"),
    Predicted_Bike_Rank_2025 = rank(Predicted_Bike_Time_2025, ties.method = "first"),
    Predicted_Run_Rank_2025 = rank(Predicted_Run_Time_2025, ties.method = "first")
  )

# Step 3: Calculate Predicted Total Transition Time (Overall Time - Swim Time - Bike Time - Run Time)
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

pred_top1000 <- pred_top1000 |> 
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

pred_top1000 <- pred_top1000 |> 
  mutate(
    Swim_Pace_100m = seconds_to_period(round((period_to_seconds(Predicted_Swim_Time_2025) / swim_distance_m) * 100)),
    Bike_Speed_MPH = round(bike_distance_miles / (period_to_seconds(Predicted_Bike_Time_2025) / 3600), 2),
    Run_Pace_per_Mile = seconds_to_period(round(period_to_seconds(Predicted_Run_Time_2025) / run_distance_miles))
  )

# Display the mutated tables
print(pred_top23_m1824)
print(pred_top1000)
```
![image](https://github.com/user-attachments/assets/364dfd7e-e1a6-4748-b030-5af22b037926)


## Where Do I Stand? Placing Myself in the Field
Using the forecasted tables, I predicted my division (M18-24) and overall rankings for 2025. Inputting my current metrics for swimming, biking, and running, the analysis placed me relative to the top athletes. This step provided a personalized benchmark to understand how close I am to achieving my competitive goals.
```R
# Step 1: Defining my paces (replace these values with my actual paces in seconds per unit)
my_name <- "Jesus Mendoza-Garmendia"
my_swim_pace_100m <- 110    # Swim pace in seconds per 100 meters
my_bike_speed_mph <- 21     # Bike speed in miles per hour
my_run_pace_per_mile <- 570 # Run pace in seconds per mile

# Step 2: Calculate segment times based on distances
# Using the swim, bike, and run distances in `half_ironman_distances` dataframe
swim_distance_m <- half_ironman_distances |> filter(Distance == "Swim_Distance") |> pull(Meters)
bike_distance_miles <- half_ironman_distances |> filter(Distance == "Bike_Distance") |> pull(Miles)
run_distance_miles <- half_ironman_distances |> filter(Distance == "Run_Distance") |> pull(Miles)

# Calculate segment times in seconds
my_swim_time <- round((swim_distance_m / 100) * my_swim_pace_100m)
my_bike_time <- round((bike_distance_miles / my_bike_speed_mph) * 3600)  # Convert hours to seconds
my_run_time <- round(run_distance_miles * my_run_pace_per_mile)
my_transition_time <- 6 * 60  # 6 minutes in seconds

# Step 3: Calculate total time including transition time
my_total_time <- my_swim_time + my_bike_time + my_run_time + my_transition_time

# Step 4: Ensure necessary columns and correct types in both tables
if (!"Name" %in% colnames(pred_top23_m1824)) {
  pred_top23_m1824 <- pred_top23_m1824 |> mutate(Name = NA_character_)
}
if (!"Name" %in% colnames(pred_top1000)) {
  pred_top1000 <- pred_top1000 |> mutate(Name = NA_character_)
}

pred_top23_m1824 <- pred_top23_m1824 |> mutate(
  Swim_Pace_100m = as.numeric(Swim_Pace_100m),
  Bike_Speed_MPH = as.numeric(Bike_Speed_MPH),
  Run_Pace_per_Mile = as.numeric(Run_Pace_per_Mile)
)

pred_top1000 <- pred_top1000 |> mutate(
  Swim_Pace_100m = as.numeric(Swim_Pace_100m),
  Bike_Speed_MPH = as.numeric(Bike_Speed_MPH),
  Run_Pace_per_Mile = as.numeric(Run_Pace_per_Mile)
)

# Step 5: Insert my data into the tables
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

# Insert into the Top 1000 overall table
my_overall_placement <- pred_top1000 |> 
  add_row(
    Name = my_name,
    Overall_Rank = NA,  # Placeholder, will be recalculated
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
    Overall_Rank = row_number(),
    Predicted_Swim_Rank_2025 = rank(Predicted_Swim_Time_2025),
    Predicted_Bike_Rank_2025 = rank(Predicted_Bike_Time_2025),
    Predicted_Run_Rank_2025 = rank(Predicted_Run_Time_2025)
  )

# Step 6: Ensure Name is the first column
reorder_columns <- function(df) {
  df <- df[, c("Name", setdiff(names(df), "Name"))]
  return(df)
}

# Reorder columns in all relevant tables
pred_top23_m1824 <- reorder_columns(pred_top23_m1824)
pred_top1000 <- reorder_columns(pred_top1000)
my_m1824_placement <- reorder_columns(my_m1824_placement)
my_overall_placement <- reorder_columns(my_overall_placement)

# Step 7: Convert Swim_Pace_100m and Run_Pace_per_Mile back to HMS format
convert_paces_to_hms <- function(df) {
  df <- df |> mutate(
    Swim_Pace_100m = seconds_to_period(Swim_Pace_100m),
    Run_Pace_per_Mile = seconds_to_period(Run_Pace_per_Mile)
  )
  return(df)
}

# Apply the conversion to all relevant tables
pred_top23_m1824 <- convert_paces_to_hms(pred_top23_m1824)
pred_top1000 <- convert_paces_to_hms(pred_top1000)
my_m1824_placement <- convert_paces_to_hms(my_m1824_placement)
my_overall_placement <- convert_paces_to_hms(my_overall_placement)

# Display the final placement tables
print("Predicted Rank in M18-24 Division:")
print(my_m1824_placement |> filter(Name == my_name))

print("Predicted Overall Rank in Top 1000:")
print(my_overall_placement |> filter(Name == my_name))
```
![image](https://github.com/user-attachments/assets/09a1ad4e-1022-4336-9967-0a277586324b), ![image](https://github.com/user-attachments/assets/8e28aa60-0144-4dd7-adfb-df586584a373)



## Striving for the Podium: Scenario Simulation and Optimization
To explore paths to success, I developed a simulator for segment times based on sequences of performance metrics. By combining simulated results with forecasted data for the top 3 athletes in my division, I identified optimal scenarios for achieving 1st, 2nd, or 3rd place. A custom function pinpointed the closest faster times needed to meet each target, offering actionable performance goals for training.
```R
sorted_div_placements <- pred_top23_m1824 |> 
  mutate(
    Swim_Pace_100m_sec = period_to_seconds(Swim_Pace_100m), 
    Run_Pace_per_Mile_sec = period_to_seconds(Run_Pace_per_Mile),
    Total_Time = period_to_seconds(Predicted_Overall_Time_2025)
  )

# Extract Top 3 Finishers' Averages
top3_div_athletes <- sorted_div_placements |>
  filter(Division_Rank == 1:3) |> 
  mutate(
    Name = case_when(
      Division_Rank == 1 ~ "P1",
      Division_Rank == 2 ~ "P2", 
      Division_Rank == 3 ~ "P3"
    )
  )

# Adjust ranges to focus on realistic scenarios
simulated_paces <- expand.grid(
  Swim_Pace_100m_sec = seq(min(top3_div_athletes$Swim_Pace_100m_sec) + 6, 
                           max(top3_div_athletes$Swim_Pace_100m_sec) + 10, by = 2),
  Bike_Speed_MPH = seq(max(top3_div_athletes$Bike_Speed_MPH) - 2, 
                       max(top3_div_athletes$Bike_Speed_MPH) + 1, by = 0.5),
  Run_Pace_per_Mile_sec = seq(min(top3_div_athletes$Run_Pace_per_Mile_sec) + 19, 
                              max(top3_div_athletes$Run_Pace_per_Mile_sec) + 39, by = 10)
)

# Calculate total times for scenarios
simulated_results <- simulated_paces |>
  mutate(
    Swim_Time = (swim_distance_m / 100) * Swim_Pace_100m_sec,
    Bike_Time = (bike_distance_miles / Bike_Speed_MPH) * 3600,
    Run_Time = run_distance_miles * Run_Pace_per_Mile_sec,
    Transition_Time = 6 * 60,  # Keep transition time constant
    Total_Time = Swim_Time + Bike_Time + Run_Time + Transition_Time
  ) |>
  arrange(Total_Time)  # Sort by fastest to slowest

# Combine top athletes and simulations
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

# Get slowest times to beat each rank
# Find rows for P1, P2, and P3 in optimized_rankings
p1_row <- which(optimized_rankings$Name == "P1")
p2_row <- which(optimized_rankings$Name == "P2")
p3_row <- which(optimized_rankings$Name == "P3")

# Select the rows directly above P1, P2, and P3
slowest_for_top3 <- optimized_rankings |> 
  slice(c(p1_row - 1, p2_row - 1, p3_row - 1)) |> 
  mutate(Placement_I_Get = rank(Division_Rank, ties.method = "first")) |> 
  mutate(Predicted_Overall_Time_2025 = round(seconds_to_period(Total_Time)))
```
![image](https://github.com/user-attachments/assets/275e3347-55a9-459d-865b-1d7db96c8028)


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
