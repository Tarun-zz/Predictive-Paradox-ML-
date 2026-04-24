# Predictive Paradox

## 1. The Big Picture
The goal of this project was to build a practical machine learning model to predict short-term power demand on the national grid. Since the rules restricted us to classical machine learning, I chose **LightGBM**. It’s incredibly fast and great at understanding complex, non-linear patterns—like how power demand spikes when it gets either extremely hot or extremely cold.

---

## 2. Cleaning Up the Data (Fixing the Paradoxes)
Real-world data is messy, and the PGCB dataset had some serious issues that would have broken any model if left unchecked:

* **Handling Impossible Spikes:** There were massive typos in the demand data, including an impossible spike of 156,050 MW (way beyond the country's actual capacity). I replaced these extreme outliers with empty values so they wouldn't skew the model's understanding of normal behavior.
* **Fixing the Broken Timeline:** The sensors didn't always record data perfectly every hour. Sometimes there were 30-minute gaps, duplicates, or missing hours. I forced the data onto a strict 1-hour grid to keep the timeline consistent.
* **Smooth Bridging:** To fill in the empty values created by capping the spikes and fixing the timeline, I used linear interpolation. This smoothly connected the dots between known values so the model still had a continuous trend to learn from.

---

## 3. Feature Engineering: Teaching the Model How Time Works
Tree-based models like LightGBM treat every row in isolation. If it's looking at 2 PM, it has no idea what happened at 1 PM. I had to manually engineer "memory" and "context" into the dataset. 

### Giving the Model a Memory
* **Recent History:** I created columns for the demand 1, 2, and 3 hours ago to capture the immediate momentum of the grid.  
* **Daily and Weekly Habits:** Human behavior is repetitive. I added a feature for "Demand exactly 24 hours ago" and "Demand exactly 7 days ago" to capture daily routines and weekday/weekend differences.
* **The Baseline Trend:** I calculated 24-hour and 7-day rolling averages. This acts like an anchor, helping the model realize if it's currently in the middle of a high-demand summer heatwave or a low-demand winter week.

### Translating Time and Weather
* **Breaking Down the Clock:** Instead of treating "Hour" as a simple number from 0 to 23, I split it into 24 distinct True/False flags. This helps the model easily trigger specific rules (like anticipating an evening peak when the clock hits 6 PM).
* **The "Feels-Like" Factor:** I prioritized 'apparent_temperature' over raw temperature. People turn on their ACs based on how hot and humid it actually feels, making this a much stronger trigger for power spikes.
* **Macro Growth:** Over 10 years, baseline power demand naturally grows. I linked yearly World Bank data (like GDP and Population) to the hourly rows so the model understood the country's gradual expansion.

---

## 4. What the Model Actually Learned (Feature Importance)
After training the model, I analyzed the Feature Importance plot to see which columns it relied on the most. The results were highly intuitive:

1. **Yesterday is the Best Clue:** The most powerful predictor was 'demand_lag_24'. Knowing exactly what happened at this exact hour yesterday is the strongest indicator of what will happen today.
2. **Comfort Drives the Grid:** 'apparant_temperature' was consistently one of the top triggers. Sudden shifts in weather are the main cause of unexpected demand spikes.
3. **The Weekly Vibe:** The 7-day rolling average was heavily utilized, proving that the model needed to understand the broader seasonal context to make accurate short-term guesses.
4. **Sunset Surges:** The specific hour flags for early evening (6 PM - 8 PM) ranked very high, capturing the daily surge when people return home and turn on their lights and appliances.


<img width="705" height="468" alt="image" src="https://github.com/user-attachments/assets/17b23220-a1a0-4ea3-995f-563762147ccd" />

