# Sales Dashboard - Power BI Project

## 📊 Overview

This Power BI project provides a comprehensive sales analytics dashboard that tracks sales performance, targets, and vendor metrics. The dashboard enables users to monitor sales trends, compare actual performance against targets, and analyze data across products, vendors, and time periods.

## 🗂️ Data Model Architecture

### Data Model Structure

The project implements a **Star Schema** data model with:
- **2 Fact Tables**: `fSales`, `fTargets`
- **3 Dimension Tables**: `dCalendar`, `dProducts`, `dVendors`

### Data Model Diagram

```
            ┌──────────────┐
            │  dCalendar   │
            │              │
            │ - Date (PK)  │
            │ - Year       │
            │ - Month      │
            │ - Month Name │
            └──────┬───────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   ┌────▼─────┐         ┌────▼─────┐
   │  fSales  │         │ fTargets │
   │          │         │          │
   │ - Date   │         │ - Date   │
   │ - Total  │         │ - Target │
   └─┬──────┬─┘         └──────────┘
     │      │
     │      │
┌────▼───┐  └─────►┌──────────┐
│dVendors│         │dProducts │
│        │         │          │
│ -Code  │         │  -Code   │
│ -Seller│         │ -Product │
└────────┘         └──────────┘
```

---

## 📋 Table Specifications

### Fact Tables

#### 1. **fSales** (Sales Transactions)
Contains transactional sales data with details on invoices, products, sellers, and amounts.

| Column Name     | Data Type | Description                           |
|-----------------|-----------|---------------------------------------|
| IssueDate       | Date      | Transaction date                      |
| InvoiceNumber   | Integer   | Invoice number                        |
| ProductCode     | Integer   | Product identifier (FK to dProducts)  |
| SellerCode      | Integer   | Seller identifier (FK to dVendors)    |
| Quantity        | Integer   | Quantity sold                         |
| UnityValue      | Decimal   | Unit price                            |
| Total           | Decimal   | Calculated total (Quantity × UnityValue) |

**Relationships:**
- `fSales[IssueDate]` → `dCalendar[Date]` (Many-to-One)
- `fSales[ProductCode]` → `dProducts[ProductCode]` (Many-to-One)
- `fSales[SellerCode]` → `dVendors[SellerCode]` (Many-to-One)

#### 2. **fTargets** (Sales Targets)
Contains monthly sales targets by seller.

| Column Name | Data Type | Description                        |
|-------------|-----------|-------------------------------------|
| Date        | Date      | Target period (FK to dCalendar)     |
| sellerCode  | Integer   | Seller identifier                   |
| Target      | Decimal   | Target sales amount                 |

**Relationships:**
- `fTargets[Date]` → `dCalendar[Date]` (Many-to-One)

---

### Dimension Tables

#### 3. **dCalendar** (Date Dimension)
Custom calendar table with date attributes for time-based analysis.

| Column Name  | Data Type | Description                           |
|--------------|-----------|---------------------------------------|
| Date         | Date      | Unique date (Primary Key)             |
| Year         | Integer   | Year (YYYY)                           |
| Month        | Integer   | Month number (1-12)                   |
| Month Name   | Text      | Full month name (e.g., "January")     |
| Month short  | Text      | 3-letter month abbreviation (e.g., "Jan") |
| Month/Year   | Text      | Formatted as "MMM/YYYY" (e.g., "Jan/2021") |

**Date Range:** Dynamically generated from minimum to maximum date in `fSales`

#### 4. **dProducts** (Products Dimension)
Product master data with hierarchical categorization.

| Column Name     | Data Type | Description                      |
|-----------------|-----------|----------------------------------|
| ProductCode     | Integer   | Unique product identifier (PK)   |
| Product         | Text      | Product name                     |
| Product Group   | Text      | Product group classification     |
| Product Segment | Text      | Product segment classification   |

#### 5. **dVendors** (Vendors/Sellers Dimension)
Sales team organizational hierarchy.

| Column Name  | Data Type | Description                    |
|--------------|-----------|--------------------------------|
| SellerCode   | Integer   | Unique seller identifier (PK)  |
| Seller       | Text      | Seller name                    |
| Supervisor   | Text      | Supervisor name                |
| Manager      | Text      | Manager name                   |
| Sales Team   | Text      | Sales team name                |

---

## 🔗 Relationships

The model uses **one-way filtering** from dimension to fact tables:

1. **Date Relationships:**
   - `dCalendar[Date]` ↔ `fSales[IssueDate]` (1:Many)
   - `dCalendar[Date]` ↔ `fTargets[Date]` (1:Many)

2. **Product Relationships:**
   - `dProducts[ProductCode]` ↔ `fSales[ProductCode]` (1:Many)

3. **Vendor Relationships:**
   - `dVendors[SellerCode]` ↔ `fSales[SellerCode]` (1:Many)

All relationships use **single-direction filtering** to optimize performance.

---

## 📂 Data Sources

### Source Files
The dashboard reads data from Excel files located in a configurable directory:

- **Sales.xlsx**: Contains raw sales transaction data
- **Targets.xlsx**: Contains monthly sales targets by seller

### Parameter Configuration

**Parameter Name:** `filePath`
- **Type:** Text
- **Description:** Path of Excel files
- **Default Value:** `C:\Users\G4\Documents\Power BI Training`
- **Purpose:** Allows flexible file path configuration for different environments

---

## 🔄 Data Transformation (Power Query)

### fSales Transformations
1. Load data from `Sales.xlsx`
2. Promote headers
3. Set column data types
4. Calculate `Total` column (Quantity × UnityValue)
5. Sort by IssueDate (descending)
6. Remove unnecessary columns (keep only fact table fields)

### fTargets Transformations
1. Load data from `Targets.xlsx`
2. Promote headers and filter valid rows
3. **Unpivot** date columns (transform from wide to long format)
4. Filter out "Total" rows
5. Convert date strings to date type
6. Rename "Value" column to "Target"

### dCalendar Transformations
1. Dynamically calculate date range from fSales
2. Generate continuous date list from min to max year
3. Extract date attributes (Year, Month, Month Name)
4. Create abbreviated month name
5. Format Month/Year display column

### dProducts Transformations
1. Load product data from `Sales.xlsx`
2. Extract product dimension columns
3. Remove duplicates based on ProductCode

### dVendors Transformations
1. Load vendor/seller data from `Sales.xlsx`
2. Extract vendor hierarchy columns
3. Remove duplicates based on SellerCode

---

## 🎯 Key Features

- **Sales Performance Tracking**: Monitor actual sales vs targets
- **Time Intelligence**: Analyze trends by year, month, and custom periods
- **Product Analysis**: Break down sales by product, group, and segment
- **Vendor Performance**: Track individual and team performance
- **Organizational Hierarchy**: View sales through supervisor and manager lenses
- **Dynamic Date Range**: Calendar automatically adjusts to data range

---

## 📈 Model Statistics

- **Tables:** 7 (5 main tables + 2 auto-generated date tables)
- **Relationships:** 5 active relationships
- **Measures:** 0 (dashboard may use implicit measures or visuals)
- **Compatibility Level:** 1567 (Power BI Desktop)
- **Last Schema Update:** January 31, 2026
- **Last Data Refresh:** February 9, 2026

---

## 🛠️ Setup Instructions

### Prerequisites
- Power BI Desktop (compatible with model compatibility level 1567+)
- Excel files (Sales.xlsx, Targets.xlsx) in the specified directory

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   ```

2. **Configure file path**
   - Open the .pbix file in Power BI Desktop
   - Navigate to Transform Data > Manage Parameters
   - Update the `filePath` parameter to point to your data directory
   - Ensure Sales.xlsx and Targets.xlsx are in that directory

3. **Refresh the data**
   - Click "Refresh" in Power BI Desktop to load the latest data
   - Verify all tables load successfully

4. **Customize (optional)**
   - Add custom measures and calculations
   - Modify visuals to meet specific requirements
   - Create additional pages or drill-through pages

---

## 📊 Data Requirements

### Sales.xlsx Format
Expected columns in the source file:
- IssueDate, InvoiceNumber, ProductCode, Product, Product Group, Product Segment
- SellerCode, Seller, Supervisor, Manager, Sales Team
- Quantity, UnityValue

### Targets.xlsx Format
Expected structure:
- First column: sellerCode
- Subsequent columns: Monthly targets (formatted as dates)
- Data organized in a cross-tab format (will be unpivoted)

---

## 🔒 Model Properties

- **Default Mode:** Import
- **Culture:** English (United States) - en-US
- **Time Intelligence:** Enabled
- **Source Query Culture:** Bosnian (Latin, Bosnia and Herzegovina) - bs-Latn-BA
- **Discourage Implicit Measures:** No
- **Discourage Report Measures:** No

---

## 📝 Notes

- The model uses **Import mode** for all tables, loading data into memory for fast performance
- Date tables include both a custom `dCalendar` and auto-generated date tables
- All dimension tables are deduplicated to ensure referential integrity
- The `Total` column in fSales is calculated during ETL for performance optimization

---

## 🔄 Version History

- **v1.0** - Initial release with sales tracking and target comparison functionality

---

*Last Updated: February 9, 2026*
