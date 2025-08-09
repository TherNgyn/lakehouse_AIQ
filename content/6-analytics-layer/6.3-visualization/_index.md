---
title : "Data Visualization with QuickSight"
date : 2025-08-10
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

## Air Quality Data Visualization with Amazon QuickSight

In this section, we'll create interactive dashboards to visualize our air quality data from the Gold and Platinum layers. We'll build visualizations similar to those shown below to help identify patterns, correlations, and insights from our data.

### Dashboard Design Overview

Our visualization strategy will include three main dashboards:

1. **General Air Quality Overview Dashboard**
   - Summary statistics of air pollutants
   - Distribution of AQI levels
   - Monthly trends of all environmental indicators

2. **PM2.5 Analysis Dashboard**
   - Temporal analysis of PM2.5, humidity, and temperature
   - Monthly AQI level distribution
   - Time series of key air quality indicators

3. **Correlation and Trend Analysis Dashboard**
   - Monthly average pollutant concentrations
   - Temperature vs. PM2.5 correlation
   - PM2.5 average by month and AQI level

### Creating Your QuickSight Dashboards

#### Prerequisites

Before starting, ensure you have:
- Access to Amazon QuickSight with appropriate permissions
- Your Gold layer data exported to a location accessible by QuickSight (S3 bucket, Athena, etc.)
- Basic understanding of QuickSight interface and visualization types

#### Step 1: Setting Up Your Data Source

1. Log into your AWS account and navigate to the QuickSight service
2. Click on **Datasets** in the left navigation pane
3. Click **New dataset** and select your data source:
   - For direct S3 access: Choose **S3** and specify the path to your parquet files
   - For Athena: Select **Athena** and choose the appropriate database and table
   - For direct connection to Delta Lake: Use the Delta Lake connector (if available)

4. Configure your dataset:
   ```
   - Data source name: AirQualityAnalytics
   - Import option: SPICE (for faster analytics)
   - Table: Select your gold_air_quality_daily_avg, gold_air_quality_hourly_avg, 
     and gold_air_quality_monthly_avg tables
   ```

5. Click **Edit/Preview data** to verify and prepare your dataset

6. Set appropriate data types for each column:
   - Date columns as Date
   - Measurement columns as Decimal
   - AQI_Level as String

7. Click **Save & publish**

#### Step 2: Creating the General Air Quality Overview Dashboard

1. From Datasets, select your AirQualityAnalytics dataset and click **Create analysis**

2. Add a **Table** visualization for summary statistics:
   - Fields: Select all air quality measurement fields (CO, NO2, PM25, SO2, O3, Humidity)
   - Add calculated fields for count, max, mean, min, and stddev
   - Configure table settings to show statistics in rows
   - Use conditional formatting to highlight values above thresholds

3. Add a **Pie chart** for AQI level distribution:
   - Field: AQI_Level
   - Value: Count (records)
   - Colors: Configure custom colors for each AQI level (Good: blue, Moderate: orange, etc.)

4. Add a **Line chart** for monthly environmental indicators:
   - X-axis: month
   - Y-axis: Sum of average values for each pollutant (CO, NO2, PM25, SO2, O3, Temperature)
   - Use different colors for each line to distinguish between pollutants
   - Add a title: "Monthly Environmental Indicator Trends"

5. Format and arrange the visualizations on your dashboard:
   - Resize elements for proper visibility
   - Add descriptive titles and legends
   - Use filter controls to allow interactive filtering by date range

#### Step 3: Creating the PM2.5 Analysis Dashboard

1. Create a new analysis from your dataset

2. Add a **Line chart** for PM2.5, humidity, and temperature over time:
   - X-axis: date
   - Y-axis: avg_PM25, avg_humidity, avg_temp
   - Use different colors for each line
   - Add title: "Air Quality Indicators Over Time"

3. Add a **Stacked bar chart** for monthly AQI distribution:
   - X-axis: month
   - Y-axis: Count of records
   - Group/Color: AQI_Level
   - Display as 100% stacked bar chart
   - Title: "AQI Level Distribution by Month"

4. Add a **KPI visual** to show current period vs. previous period comparison:
   - Primary value: Current period avg_PM25
   - Comparison value: Previous period avg_PM25
   - Format to show percent change
   - Add conditional formatting (green for improvement, red for deterioration)

5. Format and arrange the visualizations:
   - Add dashboard parameters for date range selection
   - Configure synchronization between visualizations
   - Add descriptive insights as text boxes

#### Step 4: Creating the Correlation and Trend Analysis Dashboard

1. Create a new analysis from your dataset

2. Add a **Clustered bar chart** for monthly average pollutants:
   - X-axis: month
   - Y-axis: Sum of avg_PM25, Sum of avg_SO2, Sum of avg_O3
   - Group bars by pollutant type
   - Sort by month ascending
   - Title: "Average Pollutant Concentrations by Month"

3. Add a **Scatter plot** for temperature vs. PM2.5 correlation:
   - X-axis: avg_temp
   - Y-axis: avg_PM25
   - Size: record count (optional)
   - Add a trend line
   - Title: "Relationship Between Temperature and PM2.5"

4. Add a **Line chart** for PM2.5 by month and AQI level:
   - X-axis: month
   - Y-axis: avg_PM25
   - Color/Group: AQI_Level
   - Use appropriate colors for AQI levels
   - Title: "PM2.5 Average by Month and AQI Level"

5. Format and arrange the visualizations:
   - Add insights as text boxes
   - Configure dashboard interactivity
   - Add filters for AQI level and date range

#### Step 5: Enhancing Your Dashboards

1. **Add interactivity**:
   - Configure cross-filtering between visualizations
   - Add action controls for drilling down into data
   - Create calculated fields for advanced metrics

2. **Improve formatting**:
   - Use consistent color schemes that match AQI levels
   - Add appropriate axis labels and formats
   - Include explanatory text for complex visualizations

3. **Create dashboard controls**:
   - Add date range selectors
   - Create dropdown filters for stations, AQI levels
   - Include parameter controls for threshold adjustments

4. **Add insights and narratives**:
   - Use text boxes to highlight key findings
   - Add conditional alerts for threshold violations
   - Include reference lines for regulatory standards

### Analyzing Your Dashboards

Once your dashboards are complete, you can perform detailed analysis:

#### General Air Quality Dashboard Analysis

1. **Highest Environmental Indicator**:
   - CO shows the highest average value (993.92)
   - SO₂ follows at 224.61, then O₃ (94.23), NO₂ (96.44)
   - CO also shows high variability (standard deviation of 560.39)

2. **Monthly Trends**:
   - CO peaks in April and November, drops significantly in July-August
   - PM2.5 and SO2 decrease during rainy season (middle of year)
   - Air quality improves during rainy months due to particulate washout

3. **AQI Distribution**:
   - Air quality tends to be worse in dry months
   - Better air quality occurs during rainy season

#### PM2.5 Analysis Dashboard Insights

1. **Temporal Patterns**:
   - PM2.5 fluctuates significantly, with peaks in early and late 2021
   - During rainy season (June-September), PM2.5 drops to 10-20 µg/m³
   - Inverse relationship with humidity: when humidity increases, PM2.5 decreases

2. **Temperature Effects**:
   - PM2.5 tends to increase during periods of lower temperature
   - This aligns with particulate accumulation in cold, dry air

3. **AQI Distribution by Month**:
   - Summer months (June-September) show highest percentage of "Good" AQI
   - January and December show more "Moderate" AQI levels
   - Clear seasonal pattern in air quality

#### Correlation and Trends Dashboard Findings

1. **Pollutant Distribution**:
   - SO₂ is highest by volume (200-280 range)
   - PM2.5 increases in months 1, 2, 4, 5, 10, 11, 12
   - Decreases significantly during rainy months 6-9

2. **Temperature-PM2.5 Relationship**:
   - Inverse relationship: as temperature increases, PM2.5 decreases
   - Peak PM2.5 concentrations occur at lower temperatures (26.5-27.2°C)
   - At temperatures above 28°C, PM2.5 drops to 15-20 µg/m³

3. **PM2.5 and AQI Correlation**:
   - Clear stratification between AQI levels and PM2.5 values
   - January, March, December show highest "Moderate" PM2.5 levels (~40 µg/m³)
   - Summer months show lower PM2.5, corresponding to "Good" or "Average" AQI

### Dashboard Publication and Sharing

Once your dashboards are finalized:

1. Click **Publish dashboard** to save your work
2. Set up scheduled refreshes to keep data current
3. Share with stakeholders by:
   - Using direct dashboard sharing
   - Embedding in internal portals
   - Scheduling email reports
   - Exporting as PDF for offline use

### Best Practices for QuickSight Air Quality Dashboards

1. **Use appropriate visualizations**:
   - Line charts for temporal trends
   - Bar charts for comparisons
   - Scatter plots for correlations
   - Tables for detailed statistics

2. **Apply consistent color coding**:
   - Green/blue for good air quality
   - Yellow/orange for moderate conditions
   - Red for unhealthy conditions

3. **Provide context**:
   - Add reference lines for air quality standards
   - Include explanatory text for interpreting metrics
   - Use annotations to highlight significant events

4. **Enable self-service exploration**:
   - Add interactive filters and controls
   - Include drill-down capabilities
   - Provide parameter controls for scenario analysis

By following these steps, you'll create comprehensive air quality dashboards that provide actionable insights about air quality patterns, correlations between environmental factors, and seasonal variations in pollution levels.
