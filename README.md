# electricity-demand-forecasting
Electricity demand forecasting using XGBoost | IITG.ai Recruitment Task
1. How I handled missing data and outliers

Demand data:
I first sorted the data by time and removed duplicate timestamps to ensure consistency. After that, I resampled the data to an hourly frequency so that every hour was present.
To handle missing values, I used time based interpolation. For any remaining gaps at the beginning, I used backward filling. This ensured continuity without introducing unrealistic jumps. Before filling missing values, I handled outliers using a rolling IQR method over a 24-hour window. This approach detects extreme values relative to their local context rather than the entire dataset. It works better than a global filter because electricity demand naturally varies throughout the day, and this method preserves genuine peaks while removing abnormal spikes.

Weather data:
The weather dataset had improperly placed headers. I loaded it without column names, identified the row starting with "time", and used that as the header.
After that,I converted the time column to datetime, converted all other columns to numeric, and resampled to hourly frequency using forward fill. This ensured alignment with the demand dataset.

Economic data:
I selected five relevant macroeconomic indicators:
Population, GDP growth, GDP per capita, Access to electricity, Urban population
The dataset was originally in wide format, so I reshaped it into a yearly format where each row represents a year.
To avoid data leakage, I shifted the years by +1. This means that for any prediction in year t, the model only uses economic data from year t-1. This reflects real world conditions, where current-year economic data is not available at prediction time.


2. Temporal features I created and why
Since the model does not inherently understand time, I explicitly engineered features to represent temporal patterns.
Calendar features:
hour,day_of_week,month,is_weekend
These help capture daily, weekly, and seasonal patterns. For example, demand is typically higher during working hours and weekdays.

Lag features:
demand_1h_ago,demand_2h_ago,demand_24h_ago,demand_168h_ago
These allow the model to use recent history and recurring patterns. The 24-hour lag captures daily cycles, while the 168-hour lag captures weekly repetition.

Rolling features:
avg_demand_3h,avg_demand_24h
These provide a smoothed representation of recent trends. All rolling features were calculated using shifted data to ensure only past information is used.

Target variable:
 The target was defined as the next hour’s demand using a shift of -1. This ensures the model predicts future values based only on past data.

3. What the model learned
I trained an XGBoost regressor using data from 2015 to 2022 and evaluated it on a strictly chronological test set from 2023 to 2025.
The model achieved MAPE=2.48%, indicating strong predictive performance.

Feature importance insights:
The most important feature is the current demand (demand_mw), which makes sense because demand changes gradually.
hour and avg_demand_24h are also highly influential, confirming the presence of strong daily cycles.
Lag features contribute significantly by capturing short-term trends.
Economic indicators have lower importance but still provide useful long-term context.

Model behavior:
The model closely follows actual demand patterns, especially daily fluctuations. It performs well in capturing both short-term changes and periodic trends.











 
