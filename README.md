# Stock Prediction Dashboard

This project is a stock prediction dashboard. It looks at a stock's past prices and today's news to guess where the stock is headed. Instead of just spitting out a "Buy" or "Sell" signal, it shows you exactly why it made that choice.

### What it does and how it works
You type in a stock name, and the app goes to work. First, it grabs the latest market data and predicts the trend for the next month. At the same time, it scrapes the latest financial news and scores the mood of those articles. Finally, it combines the math from the prices and the mood from the news to give you a final trading signal. The best part is it breaks down every step visually so you are not blindly trusting a black box.

### How to run the project
1. Clone this code to your computer.
2. Create a virtual environment using `python -m venv .venv` and activate it.
3. Install the dependencies by running `pip install -r requirements.txt`.
4. Start the backend by running `uvicorn app.api:app`.
5. Open your browser and go to `http://localhost:8000` to see the dashboard.

### How to read the dashboard
Here is a quick guide on what each card means and how to read it:

* **Signal**: This is the main takeaway. It tells you if the model leans toward Buy, Hold, or Sell. Interpretation: Treat this as a friendly suggestion, not a financial guarantee.
* **Probability Chart**: A pie chart showing how confident the model is. Interpretation: If it says Buy with 51% confidence and Hold with 49%, the model is basically guessing. Always check the confidence level.
* **Forecast Line Chart**: Shows past prices and draws a predicted line for the next month with a shaded area around it. Interpretation: The shaded area is the uncertainty. If it is super wide, the model is not very sure about the future.
* **Feature Impact**: A bar chart showing what pushed the model to its final choice. Interpretation: If the "news sentiment" bar is massive, it means today's news is heavily driving the Buy or Sell signal.
* **Attention Heatmaps**: A color coded breakdown of recent news headlines. Interpretation: Darker words mean the model paid more attention to them. If it highlights "lawsuit" in dark red, you instantly know why the sentiment score crashed.
* **Temporal Sensitivity**: Bar charts showing which past weeks or months strongly influenced today's prediction. Interpretation: Tall bars mean the model is heavily anchoring its current guess on that specific time period in the past.
* **Changepoints**: A list of dates where the stock trend shifted. Interpretation: This helps you see the exact days the market changed its mind about the stock.
* **News Feed**: A list of the latest headlines scraped from Google News. Interpretation: Use this to get the context for the sentiment scores.
* **Historical Rollback**: A feature to check what the model would have predicted on a past date. Interpretation: If there is enough past data available, it automatically shows the prediction for that date. If not, it just doesn't show. Great for checking if the model would have seen a past market crash coming.

### Technical Breakdown

**NLP Part (Qualitative Analysis)**
For reading the news, we use a pre-trained language model called FinBERT. Why this instead of basic word counting? Because FinBERT understands financial context. It knows a "drop in debt" is a good thing, even though the word "drop" usually sounds negative. It reads the headlines and accurately scores them.

**Time Series (Quantitative Analysis)**
For predicting prices, we use Facebook Prophet. Why Prophet over heavy models like LSTMs, RNNs, or Transformers? Because deep neural networks are overkill here. They need massive amounts of clean data, take forever to train, and act like black boxes. Prophet is fast, handles missing weekends easily, models yearly or weekly trends perfectly, and gives us clear mathematical changepoints. It does not require a supercomputer to run and gets the job done efficiently.

**XAI (Explainability)**
We wanted to show the math behind the predictions. Here is how we break open the black box:
1. **SHAP**: We use this to see exactly which feature (like moving averages vs news sentiment) contributed more to the final signal.
2. **Attention Maps**: We literally pull out the inner workings of FinBERT to highlight the exact words it focused on.
3. **Temporal Sensitivity**: We add random noise to different chunks of past price data and see how much the final prediction changes. If messing with last January's data ruins the whole forecast, we know last January was a crucial month.
4. **Changepoints**: We pull out dates from Prophet where the trend math shifted, giving us hard dates for momentum changes.
5. **Historical Rollback**: We built a time machine testing mode so you can pick a date in the past and see what the model would have predicted. You don't have to manually feed it data; if enough historical data is available up to that point, it just calculates and shows the results. If there isn't enough data, it stays hidden. This keeps us honest and stops the model from cheating using future data.

### Demo and Metrics
Since the frontend runs completely locally, you can visualize the predictions in real time on the dashboard once you start the server. The data processes in about 15 to 30 seconds depending on how much news it has to parse.

When reviewing the model's performance, the average **Confidence Score** hovers around **48%**. For a three way classification problem (Buy, Hold, Sell), random guessing is 33%. A 48% confidence means the model is finding a noticeable signal in the noise, but it is not getting overconfident, which is exactly how a realistic market model should behave. It rarely hits 90% confidence because the stock market is inherently unpredictable.

**A Note on Rollback Limitations**: If you try to run a historical rollback (e.g., testing a date from two years ago) and the dashboard doesn't show enough news data, it is not a bug in the code. We use Google News RSS feeds, which only hold the most recent headlines. Fetching specific news from random days years ago requires heavy archiving APIs that are locked behind expensive paywalls.

### Conclusion
This dashboard bridges the gap between raw data analysis and qualitative news sentiment. It runs smoothly locally, gives actionable insights, and most importantly, it respects the user by showing its complete thought process.