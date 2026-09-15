# ☕ Café REST API with Amazon API Gateway

This project demonstrates how to build a serverless REST API for a café ordering website using **Amazon API Gateway**, with mock endpoints that simulate data from a DynamoDB backend. The API is connected to a static website hosted on **Amazon S3**.

This was completed as part of an AWS hands-on training lab (*Lab 6.1: Developing REST APIs with Amazon API Gateway*).

---

## 📋 Project Overview

**Goal:** Build a REST API using API Gateway with mock data endpoints, then connect a café website (hosted on S3) to consume that API instead of static hardcoded JSON files.

### Architecture

```
┌────────────────────┐          ┌──────────────────┐          ┌───────────────────┐
│   S3 Website       │ ───────▶     API Gateway     ───────▶  Mock Integration  
│  (index.html,      │  HTTP    │  (ProductsApi)   │          │  (hardcoded JSON) │
│   main.js, etc.)   │          │                  │          │                   │
└────────────────────┘          └──────────────────┘          └───────────────────┘
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

<img width="1920" height="875" alt="01-aws-console-home" src="https://github.com/user-attachments/assets/b25b8d29-2668-4843-837b-75953330abf6" />

*AWS Management Console home page after logging into the lab account.*
<img width="1920" height="876" alt="02-website-initial-mock-json" src="https://github.com/user-attachments/assets/9dca04d2-f385-4df9-bb24-d1971c549b61" />

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

<img width="1920" height="806" alt="03-products-api-get-method" src="https://github.com/user-attachments/assets/00da2fd4-6b3e-4a81-af11-2ee522a90c15" />

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

<img width="1920" height="871" alt="04-on-offer-resource-get-method" src="https://github.com/user-attachments/assets/92595d4d-8499-4166-99d1-055d19236f0a" />

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
<img width="1920" height="838" alt="05-resources-tree-all-endpoints" src="https://github.com/user-attachments/assets/ce262af7-b26b-4892-8ed2-550ecacf12d8" />

*Final resource tree in API Gateway showing all three endpoints (`/create_report`, `/products`, `/products/on_offer`) before deployment.*
<img width="1920" height="881" alt="06-api-deployed-invoke-url" src="https://github.com/user-attachments/assets/52600a4b-755e-4bc7-9ab8-56ed3ac2c3cd" />

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

<img width="1920" height="872" alt="07-website-on-offer-via-apigateway" src="https://github.com/user-attachments/assets/ac720d9e-5c91-4450-aa95-862e365a35c7" />

*Website now shows only 1 item under "on offer" — sourced live from the `/products/on_offer` API Gateway mock endpoint.*
<img width="1920" height="877" alt="08-website-view-all-via-apigateway" src="https://github.com/user-attachments/assets/c386d620-93d3-4756-8eca-ecc6f63e328c" />

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

## 📝 Notes

- This project was built inside a temporary AWS Skill Builder lab sandbox account — resource IDs (API Gateway ID, S3 bucket name) shown above are specific to that session.
- All endpoints in this stage return **static mock data** for development purposes; no live database calls are made yet.
