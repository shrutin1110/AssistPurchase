# AssistPurchase

A REST API that helps customers select the right patient monitoring equipment through a guided, multi-step questionnaire. Built with ASP.NET Core 3.1.

## Overview

AssistPurchase walks a customer through a three-layer question flow — Features → Services → Display Size — and returns a filtered list of matching patient monitors. It also exposes endpoints for managing the product catalog and recording order confirmations.

The product catalog includes Philips IntelliVue monitors and similar devices (12 products pre-loaded in-memory).

## Project Structure

```
AssistPurchaseCaseStudy/
├── Controllers/
│   ├── ProductController.cs          # Guided question flow + product recommendation
│   ├── ConfigureProductsController.cs # Product catalog CRUD
│   └── AlertController.cs            # Order confirmation and consumer lookup
├── Models/
│   ├── Products.cs                   # Product entity (ID, Name, Features, Services, DisplaySize)
│   ├── RequestResponse.cs            # Question/answer state passed between requests
│   └── AlertDataModel.cs             # Customer confirmation record
├── Repository/
│   ├── ProductRepository.cs          # In-memory product store
│   ├── AlertRepository.cs            # In-memory customer/alert store
│   └── SuggestionPaths.cs            # Question tree logic (layer hierarchy + valid members)
└── Utility/
    ├── Validation.cs                 # Alert data validation
    └── RequestResponseValidation.cs  # Question request validation

AssistPurchaseTests/
├── ProductControllerTests.cs         # Unit tests for question flow
├── AlertControllerTests.cs           # Unit tests for alert/confirmation
├── ConfigureProductsControllerTests.cs
└── AutomatedTests/
    ├── AlertIntegrationTests.cs      # Integration tests against live server
    └── ConfigureProductsIntegrationTests.cs
```

## API Endpoints

### Product Guidance (`/api/Product`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/Product/questions` | Returns a sample `RequestResponse` showing the expected payload shape |
| POST | `/api/Product/questions` | Accepts the current layer + selected members, returns the next layer and its valid options |
| GET | `/api/Product/questions/showproducts` | Returns products matching the completed choice dictionary |

**Question flow layers:**

1. `Features` — `Touch_Screen`, `Handle`, `Alarm`, `Battery`
2. `Services` — derived from Feature selections (e.g. `ESN`, `Resp`, `CO2`, `Only Spo2`, `Additional Display`, `Others`)
3. `DisplaySize` — `upto 10`, `10-15`, `above 15`

**Sample POST body for `/api/Product/questions`:**
```json
{
  "layer": "Features",
  "layerMembers": ["Touch_Screen", "Battery"],
  "choiceDictionary": {}
}
```

**Sample response:**
```json
{
  "layer": "Services",
  "layerMembers": ["Only Spo2", "ESN", "Resp", "CO2", "Others"],
  "choiceDictionary": {
    "Features": ["Touch_Screen", "Battery"]
  }
}
```

---

### Product Catalog (`/api/ConfigureProducts`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/ConfigureProducts` | List all products |
| GET | `/api/ConfigureProducts/{productId}` | Get a specific product by ID |
| POST | `/api/ConfigureProducts/AddProduct` | Add a new product |
| PUT | `/api/ConfigureProducts/UpdateProduct/{productId}` | Update an existing product |
| DELETE | `/api/ConfigureProducts/RemoveProduct/{productId}` | Remove a product |

**Product payload:**
```json
{
  "id": "P113",
  "name": "Example Monitor",
  "features": ["Touch_Screen", "Battery"],
  "services": ["ESN", "Resp"],
  "displaySize": "10-15"
}
```

---

### Alerts / Order Confirmation (`/api/Alert`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/Alert/Consumers` | List all customers who confirmed an order |
| GET | `/api/Alert/Consumers/{region}` | Get customers filtered by region |
| POST | `/api/Alert/ConfirmationAlert` | Record a customer's product confirmation |

**Confirmation payload:**
```json
{
  "customerName": "Jane Doe",
  "customerContactNo": "9876543210",
  "customerRegion": "Pune",
  "customerEmailId": "jane@example.com",
  "productIdConfirmed": "P104"
}
```

Validation rules: all fields required, contact number must be 10 digits, product ID must exist in the catalog.

## Prerequisites

- [.NET Core SDK 3.1](https://dotnet.microsoft.com/en-us/download/dotnet/3.1)

## Running the API

```bash
cd AssistPurchaseCaseStudy
dotnet run
```

The API will be available at `https://localhost:5001` and `http://localhost:5000` by default.

## Running Tests

**Unit tests:**
```bash
cd AssistPurchaseTests
dotnet test
```

**Integration tests** require the API to be running at `https://localhost:5001` before executing.

The test suite uses xUnit and covers:
- Valid and invalid question-layer transitions
- Product filtering by criteria
- Alert creation with valid/invalid data
- Region-based consumer lookup

## Data Storage

All data is stored **in-memory** and resets on restart. There is no database dependency — the `ProductRepository` and `AlertRepository` singletons are registered via DI in `Startup.cs`.

## Tech Stack

- ASP.NET Core 3.1 Web API
- xUnit for unit and integration tests
- Microsoft.AspNetCore.TestHost for integration testing
- Newtonsoft.Json
