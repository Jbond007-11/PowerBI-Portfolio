# NHL Player Statistics in Power BI - Complete Guide

## Table of Contents
1. [Overview](#overview)
2. [The Challenge: Why Web Scraping Failed](#the-challenge)
3. [The Solution: NHL API](#the-solution)
4. [Complete Implementation Steps](#implementation-steps)
5. [Code Logic Breakdown](#code-logic-breakdown)
6. [Why This Method Works](#why-this-works)
7. [Troubleshooting](#troubleshooting)

---

## Overview

This guide explains how to extract NHL player statistics into Power BI using the official NHL API. We'll cover why traditional web scraping failed and how the API method provides a reliable, complete dataset.

**Final Result:** ~775 rows of NHL player data with 23 statistics columns, automatically refreshable.

---

## The Challenge: Why Web Scraping Failed

### Initial Approach: Web.BrowserContents()

We initially tried using Power Query's `Web.BrowserContents()` function to scrape data directly from the NHL website:

```powerquery
Web.BrowserContents("https://www.nhl.com/stats/skaters?page=0&pageSize=50")
```

### Problems Encountered

1. **JavaScript Rendering Issues**
   - `Web.BrowserContents()` only captures the initial HTML
   - The NHL website loads data dynamically via JavaScript
   - Power Query doesn't execute JavaScript, so much of the data was missing

2. **Rate Limiting & Bot Detection**
   - The NHL website detected automated requests
   - After ~5-6 pages, the server stopped serving data
   - Pages returned 0 rows or incomplete data
   - Result: Only ~650 out of 775 players were captured

3. **Inconsistent Results**
   - Different pages would fail on different refresh attempts
   - No way to add delays between requests in Power Query
   - Data gaps made the dataset unreliable

### Diagnostic Results

When we tested each page individually:
```
Page 0: 51 rows ✓
Page 1: 51 rows ✓
Page 2: 51 rows ✓
...
Page 6: 0 rows ✗ (blocked)
Page 7: 0 rows ✗ (blocked)
Page 8: 0 rows ✗ (blocked)
```

**Conclusion:** Web scraping wasn't viable for this use case.

---

## The Solution: NHL API

### What is an API?

**API (Application Programming Interface)** is a structured way to request data from a server. Instead of scraping HTML, we request data in a clean, machine-readable format (JSON).

### NHL API Endpoint

The NHL provides a public API for statistics:

```
https://api.nhle.com/stats/rest/en/skater/summary
```

**Key Parameters:**
- `start`: Starting position (0, 50, 100, etc.)
- `limit`: Number of records per request (50)
- `seasonId`: Filter by season (20252026 = 2025-26 season)
- `gameTypeId`: Game type (2 = regular season)
- `sort`: Sorting criteria (by points descending, games played ascending)

### Example API Call

```
https://api.nhle.com/stats/rest/en/skater/summary?start=0&limit=50&seasonId<=20252026&seasonId>=20252026&gameTypeId=2
```

**Returns:** Clean JSON with player statistics

---

## Implementation Steps

### Starting from a Blank Report

#### Step 1: Open Power BI and Access Power Query

1. Open **Power BI Desktop**
2. Click **"Blank report"** on the home screen
3. In the ribbon, click **"Transform data"**
4. Power Query Editor opens in a new window

---

### Step 2: Create the API Function

This function will handle fetching data for a single page.

#### 2.1 Create a Blank Query

1. In Power Query Editor: **Home tab** → **New Source** dropdown → **Blank Query**
2. A new query called "Query1" appears in the left panel

#### 2.2 Open Advanced Editor

1. Right-click on **"Query1"** → Select **"Advanced Editor"**
2. Delete all existing code

#### 2.3 Paste the Function Code

```powerquery
(pageNum as number) =>
let
    // Calculate start position (NHL API uses 'start' parameter, not 'page')
    startPos = pageNum * 50,
    
    // API endpoint with parameters
    url = "https://api.nhle.com/stats/rest/en/skater/summary?isAggregate=false&isGame=false&sort=%5B%7B%22property%22:%22points%22,%22direction%22:%22DESC%22%7D,%7B%22property%22:%22gamesPlayed%22,%22direction%22:%22ASC%22%7D%5D&start=" & Text.From(startPos) & "&limit=50&factCayenneExp=gamesPlayed%3E=1&cayenneExp=gameTypeId=2%20and%20seasonId%3C=20252026%20and%20seasonId%3E=20252026",
    
    // Fetch JSON data from API
    Source = Json.Document(Web.Contents(url)),
    
    // Extract the 'data' array from the JSON response
    data = Source[data],
    
    // Convert the array to a table
    DataTable = Table.FromList(data, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    
    // Expand the record column to show all player statistics
    ExpandedData = Table.ExpandRecordColumn(DataTable, "Column1", 
        {"playerId", "skaterFullName", "seasonId", "teamAbbrevs", "positionCode", 
         "gamesPlayed", "goals", "assists", "points", "plusMinus", 
         "penaltyMinutes", "pointsPerGame", "evGoals", "evPoints", 
         "ppGoals", "ppPoints", "shGoals", "shPoints", "otGoals", 
         "gameWinningGoals", "shots", "shootingPct", "timeOnIcePerGame"},
        {"playerId", "Player", "Season", "Team", "Position", 
         "GP", "G", "A", "P", "+/-", 
         "PIM", "P/GP", "EVG", "EVP", 
         "PPG", "PPP", "SHG", "SHP", "OTG", 
         "GWG", "S", "S%", "TOI/GP"})
in
    ExpandedData
```

#### 2.4 Save and Rename the Function

1. Click **"Done"**
2. Right-click on "Query1" → **Rename**
3. Type: **fxGetNHLAPI**
4. Press **Enter**

**What you should see:** A function icon next to the query name with parameters: `(pageNum as number) => table`

---

### Step 3: Create the Main Query

This query will call the function for all pages and combine the results.

#### 3.1 Create Another Blank Query

1. **Home tab** → **New Source** → **Blank Query**
2. A new "Query1" appears

#### 3.2 Open Advanced Editor

1. Right-click on "Query1" → **Advanced Editor**
2. Delete all code

#### 3.3 Paste the Main Query Code

```powerquery
let
    // Create a list of page numbers (0 to 15 = 16 pages for ~775 players)
    Source = {0..15},
    
    // Convert the list to a table
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    
    // Rename the column to something meaningful
    #"Renamed Columns" = Table.RenameColumns(#"Converted to Table",{{"Column1", "PageNumber"}}),
    
    // Call our API function for each page number
    #"Invoked Custom Function" = Table.AddColumn(#"Renamed Columns", "Data", each fxGetNHLAPI([PageNumber])),
    
    // Remove the PageNumber column (we don't need it anymore)
    #"Removed Columns" = Table.RemoveColumns(#"Invoked Custom Function",{"PageNumber"}),
    
    // Expand the nested tables into one large table
    #"Expanded Data" = Table.ExpandTableColumn(#"Removed Columns", "Data", 
        {"playerId", "Player", "Season", "Team", "Position", "GP", "G", "A", "P", "+/-", 
         "PIM", "P/GP", "EVG", "EVP", "PPG", "PPP", "SHG", "SHP", "OTG", "GWG", "S", "S%", "TOI/GP"},
        {"playerId", "Player", "Season", "Team", "Position", "GP", "G", "A", "P", "+/-", 
         "PIM", "P/GP", "EVG", "EVP", "PPG", "PPP", "SHG", "SHP", "OTG", "GWG", "S", "S%", "TOI/GP"}),
    
    // Set proper data types for each column
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Data",{
        {"playerId", Int64.Type}, 
        {"Player", type text}, 
        {"Season", Int64.Type}, 
        {"Team", type text}, 
        {"Position", type text}, 
        {"GP", Int64.Type}, 
        {"G", Int64.Type}, 
        {"A", Int64.Type}, 
        {"P", Int64.Type}, 
        {"+/-", Int64.Type}, 
        {"PIM", Int64.Type}, 
        {"P/GP", type number}, 
        {"EVG", Int64.Type}, 
        {"EVP", Int64.Type}, 
        {"PPG", Int64.Type}, 
        {"PPP", Int64.Type}, 
        {"SHG", Int64.Type}, 
        {"SHP", Int64.Type}, 
        {"OTG", Int64.Type}, 
        {"GWG", Int64.Type}, 
        {"S", Int64.Type}, 
        {"S%", type number}, 
        {"TOI/GP", type number}})
in
    #"Changed Type"
```

#### 3.4 Execute and Rename

1. Click **"Done"**
2. Wait 30-60 seconds for data to load
3. **Verify:** Check bottom-left corner - should show ~775 rows
4. Right-click on "Query1" → **Rename** → Type: **NHL_Stats**

---

### Step 4: Load to Power BI

1. In Power Query Editor: **Home tab** → **Close & Apply**
2. Wait for data to load into Power BI
3. In the **Fields panel** (right side), you should see **NHL_Stats** with all columns

---

## Code Logic Breakdown

### Function: fxGetNHLAPI

Let's break down what each line does:

#### Input Parameter
```powerquery
(pageNum as number) =>
```
- **Purpose:** Defines a function that accepts one parameter
- **pageNum:** A number (0, 1, 2, etc.) representing which page to fetch
- **Returns:** A table of player statistics

#### Calculate Starting Position
```powerquery
startPos = pageNum * 50,
```
- **Why:** The API uses `start` parameter, not `page`
- **Logic:** 
  - Page 0 → start=0 (players 1-50)
  - Page 1 → start=50 (players 51-100)
  - Page 2 → start=100 (players 101-150)
- **Formula:** start = pageNum × 50

#### Build the API URL
```powerquery
url = "https://api.nhle.com/stats/rest/en/skater/summary?..." & Text.From(startPos) & "..."
```
- **Text.From(startPos):** Converts the number to text for URL concatenation
- **URL Parameters:**
  - `start=`: Starting position (0, 50, 100...)
  - `limit=50`: Get 50 players per request
  - `seasonId<=20252026 and seasonId>=20252026`: Filter for 2025-26 season only
  - `gameTypeId=2`: Regular season games only
  - `sort=...`: Order by points (descending), then games played (ascending)

#### Fetch and Parse JSON
```powerquery
Source = Json.Document(Web.Contents(url)),
```
- **Web.Contents(url):** Makes HTTP request to API
- **Json.Document():** Parses the JSON response into a Power Query record

**JSON Structure Returned:**
```json
{
  "data": [
    {
      "playerId": 8478402,
      "skaterFullName": "Nathan MacKinnon",
      "gamesPlayed": 21,
      "goals": 17,
      "assists": 20,
      ...
    },
    ...
  ],
  "total": 775
}
```

#### Extract the Data Array
```powerquery
data = Source[data],
```
- **Purpose:** Access the `data` field from the JSON
- **Result:** A list of player records

#### Convert List to Table
```powerquery
DataTable = Table.FromList(data, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
```
- **Table.FromList():** Converts the list to a table
- **Result:** A table with one column containing records

#### Expand Records into Columns
```powerquery
ExpandedData = Table.ExpandRecordColumn(DataTable, "Column1", 
    {"playerId", "skaterFullName", "seasonId", ...},
    {"playerId", "Player", "Season", ...})
```
- **First parameter:** Table to expand
- **Second parameter:** Column name to expand ("Column1")
- **Third parameter:** Fields to extract from each record
- **Fourth parameter:** New column names (renaming for clarity)

**Before Expansion:**
| Column1 (Record) |
|-----------------|
| {playerId: 8478402, skaterFullName: "Nathan MacKinnon", ...} |
| {playerId: 8477934, skaterFullName: "Connor McDavid", ...} |

**After Expansion:**
| playerId | Player | Season | Team | GP | G | A |
|----------|--------|--------|------|----|----|---|
| 8478402 | Nathan MacKinnon | 20252026 | COL | 21 | 17 | 20 |
| 8477934 | Connor McDavid | 20252026 | EDM | 24 | 10 | 23 |

---

### Main Query: NHL_Stats

#### Create Page Number List
```powerquery
Source = {0..15},
```
- **{0..15}:** Creates a list: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
- **Why 0-15?** 
  - 775 total players ÷ 50 per page = 15.5 pages
  - Need 16 pages (0-15) to get all players

#### Convert to Table
```powerquery
#"Converted to Table" = Table.FromList(Source, ...)
```
**Result:**
| Column1 |
|---------|
| 0 |
| 1 |
| 2 |
| ... |
| 15 |

#### Rename Column
```powerquery
#"Renamed Columns" = Table.RenameColumns(...{{"Column1", "PageNumber"}})
```
**Result:**
| PageNumber |
|------------|
| 0 |
| 1 |
| 2 |
| ... |

#### Invoke Function for Each Page
```powerquery
#"Invoked Custom Function" = Table.AddColumn(#"Renamed Columns", "Data", each fxGetNHLAPI([PageNumber]))
```
- **Table.AddColumn:** Adds a new column called "Data"
- **each:** For each row...
- **fxGetNHLAPI([PageNumber]):** Call our function with the page number from that row
- **[PageNumber]:** References the PageNumber column value for that row

**Result:**
| PageNumber | Data (Table) |
|------------|--------------|
| 0 | [Table: 50 rows, 23 columns] |
| 1 | [Table: 50 rows, 23 columns] |
| 2 | [Table: 50 rows, 23 columns] |
| ... | ... |

#### Remove Page Number Column
```powerquery
#"Removed Columns" = Table.RemoveColumns(...{"PageNumber"})
```
- **Why:** We don't need the page numbers anymore
- **Result:** Only the "Data" column remains

#### Expand Nested Tables
```powerquery
#"Expanded Data" = Table.ExpandTableColumn(#"Removed Columns", "Data", ...)
```
- **Purpose:** Combine all 16 tables into one large table
- **Table.ExpandTableColumn:** "Unpacks" nested tables

**Before:**
| Data (Table) |
|--------------|
| [Table: 50 rows] |
| [Table: 50 rows] |
| ... |

**After:**
| playerId | Player | Team | GP | G | A | ... |
|----------|--------|------|----|----|---|-----|
| 8478402 | Nathan MacKinnon | COL | 21 | 17 | 20 | ... |
| 8477934 | Connor McDavid | EDM | 24 | 10 | 23 | ... |
| ... (775 rows total) |

#### Set Data Types
```powerquery
#"Changed Type" = Table.TransformColumnTypes(...{
    {"playerId", Int64.Type}, 
    {"Player", type text}, 
    {"GP", Int64.Type},
    ...
})
```
- **Purpose:** Tell Power BI what kind of data each column contains
- **Int64.Type:** Whole numbers (games, goals, assists)
- **type number:** Decimal numbers (percentages, per-game stats)
- **type text:** Text strings (names, positions)

**Why This Matters:**
- Enables proper sorting (numeric vs alphabetic)
- Allows mathematical operations
- Optimizes memory usage
- Prevents errors in visualizations

---

## Why This Method Works

### Comparison: Web Scraping vs API

| Aspect | Web Scraping | API |
|--------|--------------|-----|
| **Data Format** | HTML (designed for browsers) | JSON (designed for machines) |
| **JavaScript** | Required but not executed | Not needed |
| **Rate Limiting** | Aggressive bot detection | Designed for programmatic access |
| **Reliability** | Inconsistent (650/775 rows) | 100% reliable (775/775 rows) |
| **Speed** | Slow (HTML parsing) | Fast (lightweight JSON) |
| **Maintenance** | Breaks if website changes | Stable API endpoint |
| **Data Cleanliness** | Requires heavy cleaning | Clean, structured data |

### Technical Advantages

#### 1. **JSON is Machine-Readable**
```json
{
  "skaterFullName": "Nathan MacKinnon",
  "goals": 17
}
```
vs HTML:
```html
<td class="sc-gYEVde iITVLX">Nathan MacKinnon</td>
<td class="sc-gYEVde iITVLX">17</td>
```
- JSON explicitly labels each field
- HTML requires CSS selectors that can break

#### 2. **No JavaScript Execution Required**
- Web scraping needs the page to fully render
- API returns data directly, no rendering needed
- Power Query can handle JSON natively

#### 3. **Designed for Automation**
- APIs are built for programmatic access
- No CAPTCHA, no bot detection
- Expected usage pattern

#### 4. **Pagination Built-In**
- Clean `start` and `limit` parameters
- Predictable, consistent behavior
- Easy to iterate through all pages

#### 5. **Complete Data in Every Request**
- Each API call returns full player records
- No missing fields or partial data
- Guaranteed structure

---

## Column Definitions

| Column | Description | Type | Example |
|--------|-------------|------|---------|
| **playerId** | Unique NHL player identifier | Integer | 8478402 |
| **Player** | Full player name | Text | Nathan MacKinnon |
| **Season** | Season ID (YYYYZZZZ format) | Integer | 20252026 |
| **Team** | Team abbreviation | Text | COL |
| **Position** | Player position code | Text | C |
| **GP** | Games Played | Integer | 21 |
| **G** | Goals | Integer | 17 |
| **A** | Assists | Integer | 20 |
| **P** | Points (G + A) | Integer | 37 |
| **+/-** | Plus/Minus rating | Integer | 23 |
| **PIM** | Penalty Minutes | Integer | 12 |
| **P/GP** | Points Per Game | Decimal | 1.76 |
| **EVG** | Even Strength Goals | Integer | 12 |
| **EVP** | Even Strength Points | Integer | 28 |
| **PPG** | Power Play Goals | Integer | 5 |
| **PPP** | Power Play Points | Integer | 9 |
| **SHG** | Short-Handed Goals | Integer | 0 |
| **SHP** | Short-Handed Points | Integer | 0 |
| **OTG** | Overtime Goals | Integer | 0 |
| **GWG** | Game-Winning Goals | Integer | 2 |
| **S** | Shots on Goal | Integer | 90 |
| **S%** | Shooting Percentage | Decimal | 18.9 |
| **TOI/GP** | Time On Ice Per Game (minutes) | Decimal | 21.37 |

---

## Refreshing the Data

### How to Refresh in Power BI

1. **In Power BI Desktop:**
   - Click **"Refresh"** button in the Home ribbon
   - Or right-click the dataset → **Refresh**

2. **Automatic Refresh:**
   - Publish to Power BI Service
   - Configure scheduled refresh (daily, weekly, etc.)

### What Happens on Refresh

1. Power Query re-executes both queries
2. Makes 16 API calls (one per page)
3. Fetches current player statistics
4. Combines into updated dataset
5. Updates all visualizations

**Refresh Time:** ~30-60 seconds

---

## Troubleshooting

### Issue: "Expression.Error: The column 'playerId' wasn't found"

**Cause:** API response structure changed or connection failed

**Solution:**
1. Test the API URL in a browser:
   ```
   https://api.nhle.com/stats/rest/en/skater/summary?start=0&limit=50
   ```
2. Verify JSON structure matches expected format
3. Check if NHL has updated their API

---

### Issue: "DataSource.Error: Web.Contents failed to get contents"

**Cause:** Network connection or API temporarily unavailable

**Solution:**
1. Check your internet connection
2. Try accessing the NHL website in a browser
3. Wait a few minutes and try again
4. Check if NHL API is undergoing maintenance

---

### Issue: Fewer than 775 rows returned

**Cause:** Not enough pages in the range

**Solution:**
1. Check the total player count on NHL.com
2. Update the page range in the main query:
   ```powerquery
   Source = {0..X},  // where X = (total_players ÷ 50) rounded up
   ```

**Example:** If 825 players exist:
- 825 ÷ 50 = 16.5 pages
- Use `{0..16}` (17 pages)

---

### Issue: "The field 'data' of the record wasn't found"

**Cause:** API response format changed

**Solution:**
1. Test the API in a browser
2. Look at the JSON structure
3. Update the function to match new structure:
   ```powerquery
   // If structure changed from Source[data] to Source[results]
   data = Source[results],  // Update this line
   ```

---

### Issue: Refresh takes too long

**Cause:** Too many pages or slow connection

**Solution:**
1. Increase `limit` parameter to 100 (fetch more per request):
   ```powerquery
   url = "...&limit=100..."
   startPos = pageNum * 100,
   ```
2. Reduce page range accordingly:
   ```powerquery
   Source = {0..7},  // 8 pages × 100 = 800 players
   ```

---

## Advanced Customization

### Filter by Team

Add team filter to the API URL:

```powerquery
url = "...&cayenneExp=gameTypeId=2 and seasonId=20252026 and teamAbbrevs='TOR'"
```

### Multiple Seasons

Create separate queries for each season and combine:

```powerquery
// Season 2024-25
url = "...seasonId=20242025..."

// Season 2025-26
url = "...seasonId=20252026..."
```

Then use `Table.Combine()` to merge both seasons.

### Add Custom Calculations

In Power Query, add calculated columns:

```powerquery
#"Added Custom" = Table.AddColumn(#"Changed Type", "Goals Per Game", each [G] / [GP])
```

---

## Best Practices

### 1. **Keep Functions Separate**
- Always create the function as a separate query
- Makes it reusable and easier to debug

### 2. **Use Meaningful Names**
- Rename queries: `fxGetNHLAPI`, `NHL_Stats`
- Rename columns: `Player` instead of `skaterFullName`

### 3. **Set Data Types**
- Always specify data types explicitly
- Prevents errors in visualizations
- Improves performance

### 4. **Document Your Code**
- Add comments explaining complex logic
- Note the source of the API endpoint
- Include date of implementation

### 5. **Test Incrementally**
- Test the function with one page first
- Verify data structure before expanding to all pages
- Check row counts at each step

### 6. **Handle Errors Gracefully**
- Use `try...otherwise` for robust error handling:
```powerquery
data = try Source[data] otherwise {}
```

---

## Conclusion

### What We Learned

1. **Web scraping has limitations:**
   - JavaScript rendering issues
   - Rate limiting and bot detection
   - Inconsistent results

2. **APIs are superior for data extraction:**
   - Designed for programmatic access
   - Clean, structured data
   - Reliable and fast

3. **Power Query can handle complex scenarios:**
   - Custom functions
   - JSON parsing
   - Table expansion and transformation

### Final Solution Benefits

**Complete Data:** All 775 players, 100% reliable  
**Fast Refresh:** 30-60 seconds  
**Maintainable:** Won't break if website design changes  
**Expandable:** Easy to add more seasons or statistics  
**Professional:** Uses official NHL data source  

---

## Additional Resources

- **NHL API Documentation:** [NHL_API_Docs](https://github.com/Zmalski/NHL-API-Reference)
- **Power Query M Reference:** [Microsoft Docs](https://docs.microsoft.com/en-us/powerquery-m/)
- **JSON Structure Viewer:** [jsonviewer.stack.hu](https://jsonviewer.stack.hu/)

---

## Version History

- **v1.0** - Initial implementation (November 2025)
- Uses NHL API endpoint: `api.nhle.com/stats/rest/en/skater/summary`
- Season: 2025-26 (20252026)
- Records: ~775 players

---

**Created by:** Mirzet  
**Date:** November 23, 2025  
**Purpose:** Data Analytics Portfolio Project

---

*Note: This solution uses publicly available NHL API endpoints. Always respect API rate limits and terms of service.*
