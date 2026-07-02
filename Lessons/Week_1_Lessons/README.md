# Week 1: Shaking off the Rust & Deep Dive into Core Libraries

I kicked off my Data Science with Python challenge by working through Cisco Networking Academy's **"Data Science Essentials with Python"** course. 

This week was focused on mastering the foundational layers of **Matplotlib** and **Pandas** by breaking down every single line of code until I understood exactly how it worked.

---

## 📊 Matplotlib & Pyplot Insights
* **The Structure**: `matplotlib` is the overall master library, but the actual charting functions (`plot`, `bar`, `scatter`, `show`) live inside `matplotlib.pyplot`. You must import the pyplot module specifically to draw anything.
* **The Visual Metaphor**: `fig, ax = plt.subplots()` finally clicked once I pictured `fig` as the overall wooden picture frame, and `ax` as the blank canvas sitting inside it.
* **Layout Cleanup**: `plt.tight_layout()` stops labels and titles from getting awkwardly cut off by the borders. 
* **Memory Management**: Using `plt.show()` cleanly wipes the memory slate so old figures don't pile up and overlap on your screen.

---

## 🐼 Pandas Data Handling
* **Merging Tables**: `toys.merge(prices, on='toy', how='left')` combines two distinct dataframes on a shared column while safely keeping every row from the left table.
* **The Fast Lane**: Using `df.eval('0.5 * base * height')` calculates new columns quickly, and using the `@` symbol allows me to pull in a normal variable that lives outside the dataframe.
* **Filtering & Aggregating**: Used `.query()` for isolating rows and `.groupby()` for combining totals, like:
  ```python
  df.groupby('product')['revenue'].sum()
  ```

---

## 🧠 My Learning Strategy
Instead of just reading along with the Cisco curriculum, I made a habit of typing out the code in the sandbox environment manually before checking the official solutions. 

I pushed the Cisco exercises further by recreating them inside **Google Colab** to figure out how to add custom elements—like calculating an average target line and stamping text labels onto a bar chart. It involved a lot of bugs and troubleshooting, but that is where the real learning happened!


