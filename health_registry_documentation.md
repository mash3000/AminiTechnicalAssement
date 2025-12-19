# Health Registry Data Documentation

## Executive Summary

This document summarizes the profiling and cleaning of the National Health Facility Registry dataset. The dataset contains information about hospitals, clinics, and community health centres across the country.

### Key Findings

- **Initial Dataset**: 13,366 rows, 8 columns
- **Final Cleaned Dataset**: 10,318 rows, 12 columns
- **Data Retention**: 79.3% (2,695 rows removed)
- **Major Quality Issues**: 
  - Multiple date formats across records
  - Inconsistent facility IDs (numeric, hex, prefixed, missing)
  - Region names with reversed text, case variations, and encoding issues
  - GPS coordinates in 4+ different formats
  - Significant duplicate entries
  - Missing facility IDs, capacity values, and GPS coordinates

---

## Data Profiling Results

### Initial vs. Cleaned Dataset

| Metric | Initial | Cleaned | Change |
|--------|---------|---------|--------|
| **Rows** | 13,366 | 10,318 | -2,695 (20.7% removed) |
| **Columns** | 8 | 12 | +4 (added latitude, longitude, capacity_unit, date_anomaly) |
| **Unique Facilities** | ~9,000-10,000 (estimated) | 10,318 | After deduplication |

### Missingness Analysis

#### Before Cleaning
The raw data contained multiple representations of missing values:
- Null values (NaN)
- Empty strings ('')
- String 'NULL'
- 'N/A' values
- '-' symbols

#### After Cleaning
Missing values are now standardized as NaN:

| Column | Missing Count | Missing Percentage |
|--------|---------------|-------------------|
| facility_id | 2,945 | 28.5% |
| facility_name | 2 | <0.1% |
| facility_type | 1,571 | 15.2% |
| capacity_numeric | 3,215 | 31.2% |
| capacity_unit | 3,215 | 31.2% |
| region | 2 | <0.1% |
| licence_issue_date | 109 | 1.1% |
| inspection_date | 115 | 1.1% |
| latitude | 1,789 | 17.3% |
| longitude | 1,789 | 17.3% |
| remarks | 3,432 | 33.3% |
| date_anomaly | 0 | 0% |

**Key Observations:**
- Facility IDs are missing for ~28.5% of records
- Capacity information is missing for ~31% of records
- GPS coordinates are missing for ~17% of records
- Facility names and regions are nearly complete (>99%)

### Duplicate Statistics

- **Exact Duplicates Removed**: ~2,600+ rows
- **Near-Duplicates Identified**: Facilities with same name but different IDs or slight variations
- **Deduplication Strategy**: Kept the most complete record based on completeness scoring

### Data Quality Issues Found

#### 1. Facility ID Issues
- **Multiple Formats**: HF-0000, numeric (033477), hex codes (333db8), prefixed (#2), NULL strings, empty values
- **Resolution**: Standardized to consistent formats, normalized for deduplication

#### 2. Facility Name Issues
- **Emojis**: 🏥, 💊, ❤ (♥) characters
- **Encoding Issues**: Special Unicode characters, corrupted text
- **Line Breaks**: Newline characters within names
- **Resolution**: Removed emojis, normalized whitespace, handled encoding

#### 3. Facility Type Inconsistencies
- **Variants Found**: Hosp./Hospital, Health Ctr./Health Centre, CHC/Community Health Centre
- **Abbreviations**: Multiple abbreviation styles
- **Resolution**: Mapped to standardized categories

#### 4. Capacity Format Issues
- **Text Values**: "ten beds", "N/A", "unknown"
- **Inconsistent Units**: beds, cots, patients, capacity
- **Number Formats**: "117Beds", "250 beds", numeric only
- **Resolution**: Extracted numeric values and standardized units

#### 5. Region Normalization Challenges
- **Case Variations**: St. James, ST. JAMES, st. james, st.james
- **Reversed Text**: "semaJ .tS" = St. James (OCR errors)
- **Parish Suffix**: Inconsistent use of "Parish" suffix
- **Resolution**: Mapped to canonical region names, fixed reversed text patterns

#### 6. Date Format Complexity
- **Formats Found**: 
  - DD-MM-YY (04-01-21)
  - YYYY-MM-DD (2016-02-18)
  - DD/MM/YY (18/05/18)
  - Month DD YYYY (November 21 2020)
  - YYYYMMDD (20160217)
  - And many more variations
- **Issues**: Invalid dates ("not a date"), future dates, inspection dates before licence dates
- **Resolution**: Parsed to datetime, flagged anomalies

#### 7. GPS Coordinate Formats
- **POINT Format**: POINT(-58.84001 12.87196)
- **Comma-Separated**: "13.08576, -58.75331" (order varies)
- **Semicolon-Separated**: -58.71602;12.98428
- **Degree-Minute-Second**: 13°12′43″N 58°51′50″W
- **Resolution**: Converted all to decimal degrees (latitude, longitude), validated ranges

---

## Major Findings

### Cleaning Decisions Made

1. **Removed Test/Invalid Rows**: Eliminated rows with patterns like "???", "not a date", or completely empty key fields (42 rows)

2. **Facility ID Standardization**:
   - Extracted numeric digits only (removed all letters and special characters)
   - Converted to integer type
   - Converted 'NULL' strings and empty values to NaN

3. **Name Cleaning**: 
   - Fixed abbreviations: Hosp. → Hospital, Clin. → Clinic
   - Fixed backwards region prefixes in parentheses: (Semaj) → St. James, (nhoj) → St. John
   - Moved suffixes like (St.) to front of name and removed brackets
   - Removed emojis, special characters, and Unicode characters

4. **Capacity Parsing**: 
   - Converted "ten beds" to numeric 10
   - Extracted numbers from mixed text
   - Standardized units (beds, cots, patients, capacity, units)

5. **Region Mapping**: Created canonical mapping for all region variants, including reversed text patterns from OCR errors. Implemented text reversal logic to handle backwards region names.

6. **Date Parsing**: Implemented multi-format parser with fallback to pandas flexible parsing

7. **GPS Standardization**: Parsed all formats to decimal degrees, validated against Barbados geographic bounds (lat: 12.5-13.5, lon: -59.7 to -58.5)

8. **Remarks Cleaning**: Removed emojis and special characters, normalized whitespace

9. **Duplicate Handling**: 
   - Removed exact duplicates
   - Identified near-duplicates by normalized name + region
   - Kept most complete record based on completeness scoring

### Edge Cases Handled

1. **Reversed Region Names**: Fixed OCR errors like "semaJ .tS" → "St. James"
2. **Future Dates**: Preserved but flagged for review
3. **Date Anomalies**: Flagged cases where inspection_date < licence_issue_date
4. **GPS Coordinate Order**: Handled cases where lat/lon order was ambiguous
5. **Mixed Capacity Units**: Preserved unit information separately from numeric value
6. **Partial Records**: Kept records with missing IDs if other information was valuable

### Limitations

1. **Missing Facility IDs**: 28.5% of records lack facility IDs, making deduplication challenging
2. **Capacity Data**: ~31% missing, limiting capacity analysis
3. **GPS Coverage**: ~17% missing coordinates, affecting geographic analysis
4. **Date Anomalies**: Some records have logical inconsistencies (inspection before licence)
5. **Region Normalization**: A few edge cases may not have been fully normalized (e.g., some reversed text patterns)

---

## Column Documentation

### Cleaned Dataset Schema

| Column | Data Type | Description | Constraints/Notes |
|--------|-----------|-------------|-------------------|
| **facility_id** | integer | Unique identifier for facility | Extracted numeric portion only, converted to int. May be missing (28.5%) |
| **facility_name** | string | Name of the health facility | Nearly complete (>99%). Cleaned of emojis and special characters |
| **facility_type** | string | Type of facility | Standardized categories: Hospital, Health Centre, Community Health Centre, Polyclinic, Clinic, Infirmary |
| **capacity_numeric** | float | Numeric capacity value | Missing for 31.2% of records |
| **capacity_unit** | string | Unit of capacity | Values: beds, cots, patients, capacity, units. Missing when capacity_numeric is missing |
| **region** | string | Geographic region/parish | Standardized to canonical names (St. James, St. Lucy, etc.) |
| **licence_issue_date** | datetime | Date facility licence was issued | Parsed to datetime. Missing for 1.1% |
| **inspection_date** | datetime | Date of last inspection | Parsed to datetime. Missing for 1.1% |
| **latitude** | float | Geographic latitude (decimal degrees) | Missing for 17.3%. Validated to Barbados bounds (12.5-13.5) |
| **longitude** | float | Geographic longitude (decimal degrees) | Missing for 17.3%. Validated to Barbados bounds (-59.7 to -58.5) |
| **remarks** | string | Additional notes/status | Values include: Good, Follow-up 2023, Reinspection due, Needs upgrade, etc. Missing for 33.3% |
| **date_anomaly** | boolean | Flag for date logic issues | True if inspection_date < licence_issue_date |

### Data Distributions

**Facility Types** (top categories):
- Health Centre
- Community Health Centre
- Hospital
- Clinic
- Polyclinic
- Infirmary

**Regions** (Barbados Parishes):
- St. James
- St. Lucy
- St. Peter
- St. Andrew
- St. Michael
- St. Joseph
- St. John
- St. George
- Christ Church

**Capacity Units**:
- beds (most common)
- cots
- patients
- capacity
- units

---

## Usage Recommendations

### Suitable Use Cases

1. **Facility Inventory**: Count and categorize health facilities by type and region
2. **Geographic Analysis**: Map facility locations (for records with GPS coordinates)
3. **Capacity Planning**: Analyze capacity distribution by region/facility type (consider missing data)
4. **Regulatory Compliance**: Track licence issue dates and inspection dates
5. **Time Series Analysis**: Analyze trends in facility registrations over time

### Limitations for Analytics

1. **Missing IDs**: Cannot reliably link facilities across datasets without IDs
2. **Incomplete Capacity Data**: ~31% missing limits capacity analysis
3. **GPS Coverage**: ~17% missing coordinates affects mapping precision
4. **Date Quality**: Some dates may be invalid or anomalous (check date_anomaly flag)
5. **Duplicate Risk**: Some near-duplicates may remain despite cleaning efforts

### Data Quality Recommendations

1. **For Production Use**: 
   - Reconcile missing facility IDs with source systems
   - Verify date anomalies with source documentation
   - Validate GPS coordinates for critical analyses

2. **For Reporting**: 
   - Clearly document missing data percentages
   - Use date_anomaly flag to exclude questionable records if needed
   - Consider imputation strategies for missing capacity/GPS if appropriate

3. **For Integration**: 
   - Use facility_name + region as composite key if IDs are missing
   - Validate date ranges before merging with other datasets
   - Handle timezone considerations for datetime columns

---

## Knowledge Graph & RAG Applications

### Knowledge Graph Modeling

#### Entities to Extract

1. **Facility** (primary entity)
   - Properties: facility_id, facility_name, facility_type, capacity_numeric, capacity_unit
   - Identifier: facility_id (or facility_name + region if ID missing)

2. **Region** (geographic entity)
   - Properties: region_name
   - Relationships: Contains facilities

3. **Licence** (regulatory entity)
   - Properties: licence_issue_date
   - Relationships: Issued_to Facility

4. **Inspection** (regulatory entity)
   - Properties: inspection_date, remarks
   - Relationships: Performed_on Facility

5. **Location** (geospatial entity)
   - Properties: latitude, longitude
   - Relationships: Location_of Facility

#### Relationships

- `Facility` --[LOCATED_IN]--> `Region`
- `Facility` --[HAS_TYPE]--> `FacilityType` (enumerated: Hospital, Health Centre, etc.)
- `Facility` --[HAS_LICENCE]--> `Licence`
- `Facility` --[HAS_INSPECTION]--> `Inspection`
- `Facility` --[HAS_LOCATION]--> `Location`
- `Region` --[CONTAINS]--> `Facility`

#### Ontology Sketch

**Core Classes:**
- `HealthFacility` (class)
  - Properties: id (string), name (string), type (enum), capacity (integer), capacityUnit (enum)
- `Region` (class)
  - Properties: name (string), type (enum: Parish)
- `Licence` (class)
  - Properties: issueDate (date), facilityId (string, foreign key)
- `Inspection` (class)
  - Properties: inspectionDate (date), remarks (string), facilityId (string, foreign key)
- `Location` (class)
  - Properties: latitude (float), longitude (float), facilityId (string, foreign key)

**Identifiers:**
- `HealthFacility.id`: Primary identifier (may be null, use composite key)
- `HealthFacility.name + HealthFacility.region`: Composite key for facilities without IDs

**Properties:**
- All properties are optional (nullable) due to missing data
- Capacity unit is an enumerated type
- Facility type is an enumerated type

### RAG (Retrieval-Augmented Generation) Recommendations

#### Open-Source Stack

1. **Vector Database**:
   - **ChromaDB**: Lightweight, easy to integrate, good for small-medium datasets
   - **Pinecone** (managed): Better for production scale, managed service
   - **Qdrant**: Open-source, high performance, good for geospatial queries

2. **LLM Framework**:
   - **LangChain**: Comprehensive framework for building RAG applications
   - **LlamaIndex**: Optimized for data indexing and retrieval

3. **Graph Database** (for structured relationships):
   - **Neo4j**: Industry standard, excellent Cypher query language
   - **ArangoDB**: Multi-model (document + graph), good flexibility

#### Recommended Architecture

**Hybrid Approach:**
- **Graph Database (Neo4j/ArangoDB)**: Store structured relationships (Facility → Region, Facility → Licence, etc.)
- **Vector Database (ChromaDB/Qdrant)**: Store embeddings of facility descriptions, remarks, and text fields
- **LLM Framework (LangChain)**: Orchestrate retrieval from both sources

**Why This Stack:**

1. **Graph Database Benefits**:
   - Efficient relationship queries ("Find all hospitals in St. James with licences expiring soon")
   - Traversal queries ("Find facilities near facility X")
   - Relationship-based reasoning

2. **Vector Database Benefits**:
   - Semantic search over text fields (facility names, remarks)
   - Similarity matching (find facilities similar to query)
   - Handles natural language queries better

3. **LangChain Benefits**:
   - Unified interface for multiple data sources
   - Built-in RAG patterns and best practices
   - Easy integration with OpenAI, Anthropic, or open-source LLMs

#### Managed Services (Alternative)

1. **AWS Bedrock**: 
   - Access to multiple LLM models (Claude, Llama, etc.)
   - Integrated with AWS services (RDS, Neptune for graph)
   - Good for AWS-native deployments

2. **Azure OpenAI Service**:
   - Enterprise-grade OpenAI API access
   - Integration with Azure Cognitive Search
   - Good for Microsoft ecosystem

3. **Google Vertex AI**:
   - Google's LLM offerings (PaLM, Gemini)
   - Integrated with BigQuery and other GCP services
   - Good for analytics-heavy workflows

#### Specific Recommendations for This Dataset

**For Knowledge Graph + RAG:**
1. **Ingest cleaned data into Neo4j** for relationship queries
2. **Generate embeddings** for facility names, types, and remarks using sentence transformers
3. **Index embeddings in ChromaDB** for semantic search
4. **Use LangChain** to:
   - Query graph for structured information
   - Query vector DB for semantic similarity
   - Combine results for comprehensive answers
   - Generate natural language responses

**Example Use Cases:**
- "Find all community health centres in St. Lucy that need upgrades"
- "Show me facilities similar to [facility name]"
- "What facilities are located within 10km of [location]?"
- "List hospitals with inspection dates in 2024"

This hybrid approach leverages both structured relationships (graph) and semantic understanding (vectors) for comprehensive health facility information retrieval.


