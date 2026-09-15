# ☕ Café REST API with Amazon API Gateway

This project demonstrates how to build a serverless REST API for a café ordering website using **Amazon API Gateway**, with mock endpoints that simulate data from a DynamoDB backend. The API is connected to a static website hosted on **Amazon S3**.

This was completed as part of an AWS hands-on training lab (*Lab 6.1: Developing REST APIs with Amazon API Gateway*).

---

## 📋 Project Overview

**Goal:** Build a REST API using API Gateway with mock data endpoints, then connect a café website (hosted on S3) to consume that API instead of static hardcoded JSON files.

### Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌───────────────────┐
│   S3 Website     │ ───────▶│  API Gateway      │ ───────▶│  Mock Integration  │
│  (index.html,    │  HTTP   │  (ProductsApi)    │         │  (hardcoded JSON)  │
│   main.js, etc.)  │         │                   │         │                    │
└─────────────────┘         └──────────────────┘         └───────────────────┘
```

### API Endpoints Created

| Method | Path                  | Purpose                                           |
|--------|-----------------------|----------------------------------------------------|
| GET    | `/products`           | Returns all café menu items                        |
| GET    | `/products/on_offer`  | Returns only items currently on offer               |
| POST   | `/create_report`      | Triggers an inventory report (mock, no auth yet)    |

---

## 🛠️ Tech Stack

- **Amazon API Gateway** – REST API with mock integrations
- **Amazon S3** – Static website hosting
- **AWS SDK for Python (Boto3)** – Used to programmatically create API Gateway resources
- **AWS CLI v2**
- **VS Code (code-server)** – Cloud-based IDE for the lab environment

---

## 🚀 Steps I Followed

### Step 1 — Environment Setup
- Connected to a VS Code IDE (code-server) in the AWS lab environment.
- Downloaded and extracted the lab starter files (`code.zip`).
- Ran `setup.sh` to install **boto3** and provision the café website + S3 bucket.
- Verified `aws --version` (v2) and `pip3 show boto3` were correctly installed.
- Verified the café website loaded from its S3 **Object URL**, showing 6 "on offer" hardcoded menu items.

![AWS Console Home](./screenshots/01-aws-console-home.png)
*AWS Management Console home page after logging into the lab account.*

![Website with hardcoded JSON data](./screenshots/02-website-initial-mock-json.png)
*Café website initially displaying data read directly from static JSON files (`all_products.json`, `all_products_on_offer.json`) in S3 — before API Gateway was connected.*

---

### Step 2 — Created the First API Endpoint (`GET /products`)
- Opened `create_products_api.py` and created a boto3 API Gateway client:
  ```python
  client = boto3.client('apigateway', region_name='us-east-1')
  ```
- The script created:
  - A REST API named **ProductsApi**
  - A resource: `/products`
  - A **GET** method with a **MOCK** integration returning hardcoded product data (matching the DynamoDB attribute structure used in the previous lab)
- Ran the script:
  ```bash
  cd python_3
  python3 create_products_api.py
  ```
- Verified in the API Gateway console that the `GET /products` method returned a `200` status with 3 mock products.

![GET /products method execution](./screenshots/03-products-api-get-method.png)
*API Gateway console showing the `/products` GET method execution flow: Client → Method Request → Integration Request (MOCK) → Integration Response → Method Response.*

---

### Step 3 — Created the Second API Endpoint (`GET /products/on_offer`)
- Opened `create_on_offer_api.py` and filled in:
  ```python
  api_id = 'nmiox8sjoj'      # ProductsApi's REST API ID
  parent_id = 'xgqdf7'       # Resource ID of /products
  ```
- This created a **nested resource** `/products/on_offer` with its own GET method and MOCK integration, returning a single "on offer" mock item.
- Ran the script and verified via Test in the console — returned `200` with 1 mock item.

![on_offer nested resource GET method](./screenshots/04-on-offer-resource-get-method.png)
*The `/products/on_offer` resource nested under `/products`, with its GET method configured and ready to test.*

---

### Step 4 — Created the Third API Endpoint (`POST /create_report`)
- Opened `create_report_api.py` and filled in:
  ```python
  api_id = 'nmiox8sjoj'
  ```
- Unlike the previous two endpoints, this one:
  - Used **POST** instead of GET
  - Was created at the **root level** (`/create_report`, not nested under `/products`)
  - Did **not** enable CORS response parameters (this will be handled once Cognito auth is added in a future lab)
  - Returned a hardcoded mock message: `"report requested, check your phone shortly"`
- Ran the script successfully.

---

### Step 5 — Deployed the API
- From the API Gateway console, selected the root resource `/`.
- Clicked **Deploy API** → created a **new stage** named `prod`.
- All three endpoints were now live under a single **Invoke URL**:
  ```
  https://nmiox8sjoj.execute-api.us-east-1.amazonaws.com/prod
  ```

![All resources created + API deployment](./screenshots/05-resources-tree-all-endpoints.png)
*Final resource tree in API Gateway showing all three endpoints (`/create_report`, `/products`, `/products/on_offer`) before deployment.*

![API successfully deployed to prod stage](./screenshots/06-api-deployed-invoke-url.png)
*Successful deployment confirmation with the generated Invoke URL for the `prod` stage.*

---

### Step 6 — Connected the Website to the Deployed API
- Updated `resources/website/config.js`:
  ```javascript
  window.COFFEE_CONFIG = {
      API_GW_BASE_URL_STR: "https://nmiox8sjoj.execute-api.us-east-1.amazonaws.com/prod",
      COGNITO_LOGIN_BASE_URL_STR: null
  };
  ```
- Updated `python_3/update_config.py` with the S3 bucket name:
  ```python
  bucket_name = "c220889a5570640l16844295t1w713327901072-s3bucket-bwyt8j8lr3yd"
  ```
- Ran the script to re-upload the updated `config.js` to S3:
  ```bash
  python3 update_config.py
  ```
- Reloaded the website — it now pulled data **live from API Gateway** instead of static JSON files:
  - **"on offer"** view showed **1 item** (matching the `/products/on_offer` mock response)
  - **"view all"** view showed **3 items** (matching the `/products` mock response)

![Website "on offer" view via API Gateway](./screenshots/07-website-on-offer-via-apigateway.png)
*Website now shows only 1 item under "on offer" — sourced live from the `/products/on_offer` API Gateway mock endpoint.*

![Website "view all" view via API Gateway](./screenshots/08-website-view-all-via-apigateway.png)
*Website "view all" now shows 3 products — sourced live from the `/products` API Gateway mock endpoint.*

---

## ✅ Verification Summary

| Task                                             | Status |
|---------------------------------------------------|:------:|
| Environment setup (VS Code, boto3, AWS CLI)        | ✅     |
| `GET /products` endpoint (mock, 3 items)           | ✅     |
| `GET /products/on_offer` endpoint (mock, 1 item)   | ✅     |
| `POST /create_report` endpoint (mock, no CORS)     | ✅     |
| API deployed to `prod` stage                       | ✅     |
| Website connected to live API Gateway endpoint     | ✅     |

---

## 📂 Project Structure

```
.
├── python_3/
│   ├── create_products_api.py     # Creates GET /products (mock)
│   ├── create_on_offer_api.py     # Creates GET /products/on_offer (mock)
│   ├── create_report_api.py       # Creates POST /create_report (mock)
│   └── update_config.py           # Uploads updated config.js to S3
├── resources/
│   └── website/
│       ├── config.js              # Holds the API Gateway Invoke URL
│       ├── index.html
│       ├── scripts/
│       ├── styles/
│       └── images/
├── screenshots/                   # Screenshots documenting each step
└── README.md
```

---

## 🔜 Next Steps (Future Labs)

- Replace the **MOCK** integrations with **AWS Lambda** functions that query the actual **DynamoDB** table (`FoodProducts`).
- Implement authentication with **Amazon Cognito** so `/create_report` can be securely called by logged-in café staff.
- Enable proper CORS handling once Cognito auth is in place.

---

## 📝 Notes

- This project was built inside a temporary AWS Skill Builder lab sandbox account — resource IDs (API Gateway ID, S3 bucket name) shown above are specific to that session and will differ if you redeploy this yourself.
- All endpoints in this stage return **static mock data** for development purposes; no live database calls are made yet.
