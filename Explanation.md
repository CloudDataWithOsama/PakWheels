

# **FULL DETAILED EXPLANATION (LINE BY LINE)**

Below is your code with *deep explanation* of every step.

---

## **1. Importing Required Libraries**

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
```

### **Explanation**

* `requests`
  Used to send an HTTP request to a webpage and get its HTML code.

* `BeautifulSoup`
  Used to parse (read + extract) the HTML code of the webpage.

* `pandas`
  Used to store data in table form (DataFrame) and later save it as CSV.

---

## **2. Empty List to Store All Cars**

```python
all_cars = []
```

### **Explanation**

* This empty list will store all scraped car dictionaries.
* Each car will be collected as a dictionary → appended to this list.

---

## **3. Target URL**

```python
url = "https://www.pakwheels.com/used-cars/search/-/ct_karachi/"
```

### **Explanation**

* This is the PakWheels page for **used cars in Karachi**.
* You are scraping all car ads from this URL.

---

## **4. Create Fake Browser Header**

```python
headers = {
    "User-Agent": "Mozilla/5.0"
}
```

### **Explanation**

* Websites sometimes block bots/scrapers.
* Adding a **User-Agent** makes your script look like a real browser.
* Prevents server from blocking your request.

---

## **5. Request the Web Page**

```python
response = requests.get(url, headers=headers)
```

### **Explanation**

* Sends an HTTP GET request to the URL.
* Returns the HTML of the webpage in `response`.

---

## **6. Parse the HTML**

```python
soup = BeautifulSoup(response.text, "html.parser")
```

### **Explanation**

* `response.text` → raw HTML code of the page.
* BeautifulSoup reads and structures this HTML.
* `"html.parser"` tells BeautifulSoup how to read HTML.

---

## **7. Select All Car Advertisement Blocks**

```python
ads = soup.select("div.well.search-list.ad-container")
```

### **Explanation**

* On PakWheels, each ad is inside a `<div>` containing these CSS classes:

  * `well`
  * `search-list`
  * `ad-container`
* `soup.select()` finds **all** HTML blocks matching this CSS pattern.
* `ads` becomes a list of all car ad HTML blocks.

---

## **8. Loop Through All Ads**

```python
for ad in ads:
```

### **Explanation**

* Iterates each advertisement block one by one.
* For each ad, extract:

  * car name
  * city
  * price
  * year / mileage / fuel / engine / transmission

---

## **9. Error Handling**

```python
    try:
```

### **Explanation**

* Some ads may be missing data.
* If any error occurs, skip the ad instead of stopping the whole program.

---

## **10. Extract Car Name**

```python
car_name = ad.select_one("a.car-name h3").text.strip()
```

### **Explanation**

* `select_one()` finds the first matching CSS selector.
* `a.car-name h3` is where the car name appears.
* `.text` extracts text only (not HTML).
* `.strip()` removes extra spaces or newline characters.

---

## **11. Extract City Name**

```python
city = ad.select_one("ul.search-vehicle-info li").text.strip()
```

### **Explanation**

* City name is inside the first `<li>` of the list `.search-vehicle-info`.
* Again `.text.strip()` cleans the extracted text.

---

## **12. Extract Price**

```python
price = ad.select_one("div.price-details").text.strip()
```

### **Explanation**

* Price is inside the `<div>` with class `price-details`.

---

## **13. Extract All Details (Year, km, Fuel, Engine, Transmission)**

```python
detail_items = [li.text.strip() for li in ad.select("ul.search-vehicle-info-2 li")]
```

### **Explanation**

* Loops through all `<li>` inside `.search-vehicle-info-2`.
* Extracts each detail:

  * Model year
  * Mileage
  * Fuel type
  * Engine CC
  * Transmission
* Stores them in a list.

---

## **14. Join Details Into One String**

```python
details = " | ".join(detail_items)
```

### **Explanation**

* Converts list → string with `" | "` separator.
* Example:
  `["2018", "52,000 km", "Petrol", "1500 cc", "Automatic"]`
  becomes
  `"2018 | 52,000 km | Petrol | 1500 cc | Automatic"`

---

## **15. Append Car Data Into List**

```python
all_cars.append({
    "Car Name": car_name,
    "City": city,
    "Details": details,
    "Price": price,
})
```

### **Explanation**

* Creates a dictionary for each car.
* Appends it to the `all_cars` list.
* This makes it easy to convert to a DataFrame later.

---

## **16. Skip Errors**

```python
    except:
        pass
```

### **Explanation**

* If any ad fails, ignore it and continue.
* Prevents script from stopping halfway.

---

## **17. Create Pandas DataFrame**

```python
df = pd.DataFrame(all_cars)
```

### **Explanation**

* Converts the list of dictionaries into a DataFrame (table).
* Each key becomes a column.

---


## **18. Splitting the Details String**

```python
temp_df = df['Details'].str.split('|', expand=True)

for i in range(temp_df.shape[1]):
    col_names = ['Year', 'Mileage', 'Fuel', 'Engine', 'Transmission', 'Extra']
    if i < len(col_names):
        df[col_names[i]] = temp_df[i].str.strip()
```

### **Explanation**

* `str.split('|', expand=True)`
  The "Details" column contained multiple data points (Year, Mileage, etc.) merged into one string. This command splits that string wherever a pipe (`|`) symbol appears and expands them into individual columns.
* `for loop`
  Since not all car ads have the same amount of information (e.g., Electric cars may not have engine CC), this loop dynamically checks how many columns were created and assigns the correct headers (`Year`, `Mileage`, etc.) accordingly.
* `.str.strip()`
  Removes any leading or trailing whitespace that remains after the split.

---

## **19. Numeric Price Conversion**

```python
def clean_price(price):
    price = price.replace('PKR', '').replace(',', '').strip()
    if 'lacs' in price:
        return float(price.replace('lacs', '').strip()) * 100000
    elif 'crore' in price:
        return float(price.replace('crore', '').strip()) * 10000000
    return price

df['Price (PKR)'] = df['Price'].apply(clean_price)
```

### **Explanation**

* `clean_price` function
  On PakWheels, prices are listed as text (e.g., "PKR 15.5 lacs"). A computer cannot perform calculations on text. This function:
  1. Removes the `PKR` currency label and commas (`,`).
  2. If the word `lacs` is present, it multiplies the number by **100,000**.
  3. If the word `crore` is present, it multiplies the number by **10,000,000**.
* `.apply()`
  Runs this logic across the entire "Price" column, converting strings into actual **integers/floats** so we can calculate averages or find the cheapest car.

---

## **20. Cleaning Mileage and Engine Units**

```python
df['Mileage'] = df['Mileage'].str.replace('km', '').str.replace(',', '').str.strip().astype(float)
df['Engine'] = df['Engine'].str.replace('cc', '').str.replace('kWh', '').str.strip().astype(float)
```

### **Explanation**

* `.str.replace()`
  Removes the unit labels **'km'** from mileage and **'cc'** or **'kWh'** (for electric vehicles) from the engine column.
* `.astype(float)`
  Changes the data type of these columns from "Object/String" to **"Float"** (decimal numbers). This makes the data ready for statistical analysis or machine learning models.

---

## **21. Refining Car Names (Regex)**

```python
df['Car Name'] = df['Car Name'].str.replace('for Sale', '', case=False).str.strip()
df['Car Name'] = df['Car Name'].str.replace(r'\d{4}', '', regex=True).str.strip()
df['Car Name'] = df['Car Name'].str.replace(r'\s+', ' ', regex=True)
```

### **Explanation**

* `'for Sale'` removal
  This phrase appeared at the end of every car title and was redundant.
* `r'\d{4}'` (Regex)
  Uses a **Regular Expression** to find any 4-digit number (the year) within the car name and removes it. We do this because the "Year" is already stored in its own dedicated column.
* `r'\s+'`
  Fixes any double or triple spaces that might have been created during the cleaning process, ensuring the names are neat and single-spaced.

---

## **22. Final Cleanup and Export**

```python
columns_to_remove = ['Details', 'Price', 'Extra']
df.drop(columns=columns_to_remove, inplace=True)

df.to_csv("PakWheels_Karachi_Cleaned.csv", index=False)
```

### **Explanation**

* `.drop()`
  Now that we have extracted all the useful information into clean columns (Year, Price PKR, etc.), we delete the original "raw" columns to keep the dataset structured and efficient.
* `.to_csv()`
  Exports the final, cleaned table into a "PakWheels_Karachi_Cleaned.csv" file.
* `index=False`
  Tells Pandas not to include the automatic row numbers (0, 1, 2...) as a separate column in the final CSV file.