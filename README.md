# Telia test task data engineer
Repository for Telia 2026 data engineer test task

## Database structure information 


```CREATE TABLE "clean_weather_daily" (
"country" TEXT,
  "capital" TEXT,
  "date" TIMESTAMP,
  "temperature_2m_max" REAL,
  "temperature_2m_min" REAL,
  "precipitation_sum" REAL,
  "wind_speed_10m_max" REAL,
  "sunshine_duration" REAL,
  "sunshine_duration_hours" REAL
);

CREATE TABLE "raw_countries" (
"country" TEXT,
  "capital" TEXT,
  "capital_latitude" REAL,
  "capital_longitude" REAL,
  "population" INTEGER,
  "area" REAL
);

CREATE TABLE "raw_weather_daily" (
"country" TEXT,
  "capital" TEXT,
  "date" TEXT,
  "temperature_2m_max" REAL,
  "temperature_2m_min" REAL,
  "precipitation_sum" REAL,
  "wind_speed_10m_max" REAL,
  "sunshine_duration" REAL
);

-- Views

CREATE VIEW v_30day_summary AS
SELECT 
    country, capital,
    COUNT(date) AS days_count,
    ROUND(AVG(temperature_2m_max), 2) AS avg_temp_max,
    ROUND(AVG(temperature_2m_min), 2) AS avg_temp_min,
    ROUND(SUM(precipitation_sum), 2) AS total_rainfall,
    ROUND(MAX(wind_speed_10m_max), 2) AS max_wind,
    ROUND(SUM(sunshine_duration_hours), 2) AS total_sunshine_hours
FROM clean_weather_daily
GROUP BY country, capital
ORDER BY country;

CREATE VIEW v_capitals_ranked_by_avg_temp AS
SELECT country, capital, ROUND(AVG(temperature_2m_max), 2) AS avg_temperature_2m_max
FROM clean_weather_daily
GROUP BY country, capital
ORDER BY avg_temperature_2m_max DESC;```

CREATE VIEW v_countries_most_rainfall AS
SELECT country, capital, ROUND(SUM(precipitation_sum), 2) AS total_precipitation_sum
FROM clean_weather_daily
GROUP BY country, capital
ORDER BY total_precipitation_sum DESC;`
