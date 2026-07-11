# SDMX Information Model Explained Using a Real Database Example

## Let's build a real system from scratch

# Step 1: Imagine an Institution wants to publish Inflation Statistics

Suppose the Institution wants to publish the following dataset.

| Country | Indicator | Frequency | Time     | Inflation |
| ------- | --------- | --------- | -------- | --------: |
| India   | CPI       | Monthly   | Jan-2025 |       3.4 |
| India   | CPI       | Monthly   | Feb-2025 |       3.6 |
| India   | CPI       | Monthly   | Mar-2025 |       3.8 |
| USA     | CPI       | Monthly   | Jan-2025 |       2.8 |
| USA     | CPI       | Monthly   | Feb-2025 |       2.9 |
| Japan   | CPI       | Monthly   | Jan-2025 |       1.7 |


Looks simple.

But now let's design the database.

# Step 2: Traditional Database Design

A data engineer rarely stores everything in one table. Instead, we'd
normalize it.

### COUNTRY

| country_id | code | name          |
| ---------- | ---- | ------------- |
| 1          | IN   | India         |
| 2          | US   | United States |
| 3          | JP   | Japan         |


### INDICATOR
| indicator_id | code | description          |
| ------------ | ---- | -------------------- |
| 1            | CPI  | Consumer Price Index |
| 2            | PPI  | Producer Price Index |


### FREQUENCY
| freq_id | code | description |
| ------- | ---- | ----------- |
| 1       | M    | Monthly     |
| 2       | Q    | Quarterly   |
| 3       | A    | Annual      |



### FACT TABLE
| country | indicator | frequency | time    | value |
| ------- | --------- | --------- | ------- | ----: |
| IN      | CPI       | M         | 2025-01 |   3.4 |
| IN      | CPI       | M         | 2025-02 |   3.6 |
| US      | CPI       | M         | 2025-01 |   2.8 |


This is a classic star schema.

------------------------------------------------------------------------

# Translating the Database into SDMX

This is where people often get confused.

SDMX does **not** replace your data.

Instead, it replaces **how everyone agrees to describe the data**.

Think of SDMX as adding a universal **contract** around your database.



### SDMX Architecture

``` text
                    DATAFLOW
                        │
                        ▼
         DATA STRUCTURE DEFINITION (DSD)
                        │
      ┌─────────────────┼──────────────────┐
      ▼                 ▼                  ▼
  Concepts          Codelists         Constraints
                        │
                        ▼
                   Actual Data
                        │
          Attributes & Metadata
                        │
                 REST / XML / JSON
```


Let's go through every box.


## 1. Dataflow

A **Dataflow** represents a published statistical dataset.

Examples:

-   Inflation Statistics
-   Employment Statistics
-   GDP
-   Bank Credit
-   Foreign Exchange
-   Debt Securities

Think of it like Netflix categories:

-   Movies
-   TV Shows
-   Sports

Each category is independent.

Likewise, **Inflation Statistics** is one Dataflow.

It answers:

"What dataset am I looking at?"

**Database analogy:** Business Dataset or Schema.

------------------------------------------------------------------------

## 2. Data Structure Definition (DSD)

This is the heart of SDMX.

Imagine someone asks

"How is Inflation Statistics structured?"

The answer is the DSD.

The DSD defines the structure of a dataset.

It specifies:

-   Dimensions
    -   Country
    -   Frequency
    -   Indicator
    -   Time
-   Measure
    -   OBS_VALUE
-   Attributes
    -   Unit
    -   Status
    -   Decimals
The DSD defines structure, not the actual observations.



It also specifies which codelists must be used.

For example the DSD also says: 

Country must use Country Codelist

Frequency must use Frequency Codelist

Indicator must use Indicator Codelist


SQL equivalent:

``` sql
CREATE TABLE inflation(
 country,
 indicator,
 frequency,
 time,
 value
);
```

The DSD is similar, but much richer.


------------------------------------------------------------------------


## 3. Concepts

Concepts define the business vocabulary.

Examples:

-   Country
-   Indicator
-   Frequency
-   Time
-   Unit
-   Observation Value
-   Status

Notice something.

None of these are actual values.

They are field definitions.

**Database analogy:** Column names.

------------------------------------------------------------------------

## 4. Codelists



Now we define allowed values.

### Country
| Code | Meaning       |
| ---- | ------------- |
| IN   | India         |
| US   | United States |
| JP   | Japan         |

#### Frequency
| Code | Meaning   |
| ---- | --------- |
| M    | Monthly   |
| Q    | Quarterly |
| A    | Annual    |


### Indicator
| Code | Meaning              |
| ---- | -------------------- |
| CPI  | Consumer Price Index |
| PPI  | Producer Price Index |


These are reference tables.

Database analogy : Foreign key lookup tables.

------------------------------------------------------------------------


## 5. Dimensions

Suppose I ask

Which inflation value?

You answer:

- India
- Monthly
- CPI
- Jan-2025


Those four fields uniquely identify one observation.

These are the Dimensions.


Dimensions uniquely identify an observation.

Example:

-   Country = India
-   Frequency = Monthly
-   Indicator = CPI
-   Time = Jan-2025

These form the composite primary key.
| Country | Indicator | Frequency | Time |
| ------- | --------- | --------- | ---- |

Together they identify one row.

------------------------------------------------------------------------

## 6. Measure

Now that we've identified the row,

what is the number?
3.4

The Measure is the actual numeric value.

Typically:

`OBS_VALUE`

Our table becomes

| Country | Indicator | Frequency | Time    | OBS_VALUE |
| ------- | --------- | --------- | ------- | --------: |
| IN      | CPI       | M         | 2025-01 |       3.4 |


Everything left of OBS_VALUE identifies the fact.
OBS_VALUE is the fact.

------------------------------------------------------------------------


## 7. Attributes

Attributes describe an observation but do not identify it.

Suppose we also want to say
Examples:

-   Unit = Percent
-   Decimals = 1
-   Status = Preliminary
-   Source = RBI


These do not identify the row.

They merely describe it.

Database
| Country | Time    | Value | Unit    | Status      |
| ------- | ------- | ----: | ------- | ----------- |
| IN      | 2025-01 |   3.4 | Percent | Preliminary |

Unit is not part of the primary key.

Neither is Status.

These are Attributes.

------------------------------------------------------------------------


## 8. Constraints

Suppose this dataset only contains

- India
- USA
- Japan

Although the global country codelist contains about 250 countries.

### Constraint
Allowed Countries
- IN
- US
- JP

Or

Only Monthly data
- Frequency
- M

Or

Only CPI
- Indicator
- CPI

Constraints narrow the valid combinations for a particular dataset.

Constraints restrict valid values.

-   Countries allowed: IN, US, JP
-   Frequency: M only
-   Indicator: CPI only

------------------------------------------------------------------------

## 9. Metadata

This is one of the biggest differences between SDMX and a plain database.

Suppose someone downloads the value
3.4


Questions immediately arise:

Is it month-over-month or year-over-year inflation?
Which methodology was used?
Which institution produced it?
When was it revised?
Is it preliminary or final?
What is the unit?

Metadata answers all of these.

Example:

- Title: Monthly CPI Inflation
- Definition: Consumer Price Index measuring year-on-year inflation.
- Source: Reserve Bank of India
- Release Calendar: 15th of every month
- Methodology: ILO 2023 Standard
- Contact: statistics@institution.org

Metadata makes the numbers interpretable.

Without metadata, numbers lose their meaning.

------------------------------------------------------------------------


## 10. Data

Only now do we reach the actual observations.
| Country | Indicator | Frequency | Time    | Value |
| ------- | --------- | --------- | ------- | ----: |
| IN      | CPI       | M         | 2025-01 |   3.4 |
| IN      | CPI       | M         | 2025-02 |   3.6 |
| US      | CPI       | M         | 2025-01 |   2.8 |

Everything before this point was describing how the data should look.

This table is the actual data.

------------------------------------------------------------------------

## 11. REST API / XML / JSON

The beauty of SDMX is that all these structures can be discovered programmatically.

A typical client workflow looks like this:


``` text
Client
   │
   ▼
Request Dataflows
   │
   ▼
"Show me available datasets."

   │
   ▼
Select Inflation Statistics

   │
   ▼
Request the DSD
   │
   ▼
"How is this dataset structured?"

   │
   ▼
Request Concepts
   │
   ▼
"What fields exist?"

   │
   ▼
Request Codelists
   │
   ▼
"What values are allowed?"

   │
   ▼
Request Constraints
   │
   ▼
"What combinations are valid?"

   │
   ▼
Request Data
   │
   ▼
"Give me CPI for India in 2025."

   │
   ▼
Receive SDMX-JSON or SDMX-XML
```


This is fundamentally different from a typical REST API, where you often have to read separate documentation to understand the schema. In SDMX, the schema itself is machine-readable and discoverable.

------------------------------------------------------------------------

# Putting It All Together


``` text
                    DATAFLOW
        "Monthly Inflation Statistics"
                           │
                           ▼
          DATA STRUCTURE DEFINITION (DSD)
                           │
      ┌────────────────────┼────────────────────┐
      ▼                    ▼                    ▼
  Concepts             Codelists          Constraints
      │                    │                    │
 Country            IN = India          Only M frequency
 Indicator          US = USA            Only CPI
 Frequency          JP = Japan          Only IN/US/JP
 Time               CPI = Consumer Price Index
 OBS_VALUE          M = Monthly
 Unit
 Status
                           │
                           ▼
                     Actual Data
 IN | CPI | M | 2025-01 | 3.4 | Percent | Preliminary
 IN | CPI | M | 2025-02 | 3.6 | Percent | Preliminary
 US | CPI | M | 2025-01 | 2.8 | Percent | Final
                           │
                           ▼
                     Metadata
 Definition • Methodology • Source • Contact • Revisions
                           │
                           ▼
              Exposed as SDMX-JSON / SDMX-XML via REST API
```

This diagram captures the essence of the SDMX Information Model: the observations are just one layer. The real power of SDMX is that every dataset carries a formal, machine-readable description of its structure, meaning, allowed values, and documentation, enabling organizations like BIS, IMF, ECB, and OECD to exchange statistical data without having to negotiate a new schema for every dataset.