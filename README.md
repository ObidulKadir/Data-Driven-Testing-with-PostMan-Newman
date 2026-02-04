# Data Driven Testing with Postman & Newman

This repository contains examples and setup for **Data Driven Testing** using **Postman** and **Newman**. It demonstrates how to run Postman collections with dynamic data from CSV/JSON files.

---

## 🛠 Prerequisites

1. [Node.js](https://nodejs.org/en/download/) installed
2. [Postman](https://www.postman.com/downloads/) installed
3. Newman installed globally:

```bash
npm install -g newman
```
## 🚀 Running Tests in Postman

1. Open Postman
2. Import the collection Data_Driven_Testing.postman_collection.json
3. Import the environment DataDrivenTesting.postman_environment.json
4. Go to Runner
5. Select the collection
6. Choose the data file (CSV or JSON) from data/ folder
7. Run the collection

## 🔄 Test Flow

The data-driven testing follows this flow:

1. **Authentication (Auth)**
   - Run the Auth request first.
   - Get the token from the response.
   - Save the token in an environment variable `{{auth_token}}`.

2. **Booking**
   - Use the `{{auth_token}}` from the Auth request in the Booking request header.
   - Create a booking using data from the CSV file (`SingleUpdateBooking.csv`).
   - Validate the response for each iteration.
  
```bash
[CSV Data] --> [Auth Request] --> [Save Token] --> [Booking Request] --> [Validate Response]
```
## How to run an API on Newman
## 1. Auth- get token
```newman
newman run "Data Driven Testing.postman_collection.json" -e "DataDrivenTesting.postman_environment.json" -d "authSingleEntries.csv" --folder "auth" --export-environment token_with_env.json
**format: newman run "collection" -e "environment" -d "dataset" --folder "folderName"**
```

## 2. Create & Update api run
```newman
newman run "Data Driven Testing.postman_collection.json" -e "token_with_env.json" -d "SingleUpdateBooking.csv" --folder "booking"

```

## How to generate a Report
```
1. JSON file report
newman run "Data Driven Testing.postman_collection.json" -e "token_with_env.json" -d "SingleUpdateBooking.csv" --folder "booking" --reporters cli,json --reporter-json-export "FullBookingRunReport.json"
2.html file report
newman run "Data Driven Testing.postman_collection.json" -e "token_with_env.json" -d "SingleUpdateBooking.csv" --folder "booking" -r cli,htmlextra

```
![ER1](https://github.com/ObidulKadir/Data-Driven-Testing-with-PostMan-Newman/blob/31181814375e40907ca07cb197d2fbf1f51a66f5/AuthReport.png)
![ER1](https://github.com/ObidulKadir/Data-Driven-Testing-with-PostMan-Newman/blob/db2bafe28f71801314f57b9b0d7498045a114e5e/Create%26UpdateReport.png)

## 📌 Newman & Data Driven Testing Notes

### 1️⃣ Running Collections in Newman

- Newman can run **collections** in two ways:
  1. **Collection-wise** – Run the entire collection at once.
  2. **Folder-wise** – Run a specific folder inside a collection.
     - **Note:** Running a specific **request** directly is **not possible** in Newman.  

---

### 2️⃣ Boolean Value Matching with CSV Data

- Issue: In Postman, boolean values may match with CSV data differently when run in Newman.  
- **Solution:**
  1. Always apply `trim()` and `toLowerCase()` on CSV data.
  2. Convert CSV value to boolean.
  3. Then match it with response value in the test.

```javascript
let expected = pm.iterationData.get("isActive").trim().toLowerCase() === 'true';
pm.test("Boolean value match", () => {
    pm.expect(responseJson.isActive).to.eql(expected);
});
```
### 3️⃣ Running Multiple Folders

- Newman **cannot run multiple folders at once**.  
- **Solution:**  
  - Run requests **separately**.  
  - Or keep multiple requests inside **one folder** and run it with CSV data.  
  - Maintain CSV columns according to the requests inside the folder.

---

### 4️⃣ Limitations of Running Separate Folders

- Running a separate folder in Newman is tricky because:  
  - Environment variables set at runtime (like IDs from previous requests) **may not persist** across folders.  
  - The next folder may **not get required values** automatically.

---

### 5️⃣ Merging Folders with CSV Data

- **Solution:**  
  - Combine related requests like **Create** and **Update** inside **one folder**.  
  - Maintain **one CSV file** for that folder.  
  - Manage separate columns in CSV for different requests (e.g., `create_name`, `update_name`).  
  - Update request body dynamically based on CSV column names.  

💡 **Tip:** Always map CSV column names to request body fields to avoid runtime issues.

