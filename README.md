# Career Marketplace Setup

## SharePoint Setup

### Taxonomy

1. **Create the term group "Career Marketplace"** at the tenant level.
2. **Add a term set** for the following:
   - **Classification Code**
   - **Department**
   - **Duration**
   - **Job Type**
   - **Language Requirement**
   - **Location**
     - Hierarchy for this term set: `Province -> Region -> City`
     - Example: `Location -> Ontario -> Eastern Ontario -> Ottawa`
   - **Program Area**
   - **Security Clearance**
   - **Work Arrangement**
   - **Work Schedule**
3. **Add terms** to each of the term sets created.
4. **Add translations** for each term set and all child terms.

### Job Opportunity List

1. **Create an SPFx list** in the Career Marketplace site called `JobOpportunity`.
2. **Add columns** for the following properties:

| Column Name             | Type                   | Details                                                         |
|--------------------------|------------------------|-----------------------------------------------------------------|
| **ContactObjectId**     | Single line of text    |                                                                 |
| **ContactName**         | Single line of text    |                                                                 |
| **Department**          | Managed Metadata       | (Department term set)                                           |
| **ContactEmail**        | Single line of text    |                                                                 |
| **JobTitleEn**          | Single line of text    |                                                                 |
| **JobTitleFr**          | Single line of text    |                                                                 |
| **JobType**             | Managed Metadata       | (Job Type - allow multiple selections)                          |
| **ProgramArea**         | Managed Metadata       | (Program Area)                                                  |
| **ClassificationCode**  | Managed Metadata       | (Classification Code term set)                                  |
| **ClassificationLevel** | Number                 | Min: 1, Max: 5                                                  |
| **NumberOfOpportunities** | Number               | Default: 1, Min: 1                                              |
| **Duration**            | Managed Metadata       | (Duration term set)                                             |
| **ApplicationDeadlineDate** | Date and Time     | Default to 3 months from creation date.<br>Formula: `=DATE(YEAR(Created), MONTH(Created) + 3, DAY(Created))` |
| **JobDescriptionEn**    | Multiple lines of text |                                                                 |
| **JobDescriptionFr**    | Multiple lines of text |                                                                 |
| **WorkSchedule**        | Managed Metadata       | (Work Schedule term set)                                        |
| **Location**            | Managed Metadata       | (Location term set)                                             |
| **SecurityClearance**   | Managed Metadata       | (Security Clearance term set)                                    |
| **LanguageRequirement** | Managed Metadata       | (Language Requirement term set)                                 |
| **WorkArrangement**     | Managed Metadata       | (Work Arrangement term set)                                     |
| **ApprovedStaffing**    | Yes/No                 | Default: No                                                     |
| **AssetSkills**         | Single line of text    | TODO: Consider handling translations                            |
| **EssentialSkills**     | Single line of text    | TODO: Consider handling translations                            |

3. **Reindex the site** after adding columns:
   - Go to **Site Settings** -> **Search and offline availability** -> **Reindex site**.
   - This process may take some time before the new list columns appear in the crawled properties.

### Search Schema

1. **Create managed properties** for the columns in the `JobOpportunity` list:
   - Go to **SharePoint Admin Center** -> **More Features** -> **Search** -> **Manage Search Schema**.
   - Create a new managed property for each column with the naming convention: `CM-{ColumnName}`.
   - **Crawled Property Formats:**
     - Managed metadata: `OWS_TAXID_{ColumnName}`
     - Other fields: `OWS_{ColumnName}`

2. **Managed Property Settings:**
   - **Type:** Text (Yes/No for `ApprovedStaffing`)
   - **Searchable:** True
   - **Queryable:** True
   - **Retrievable:** True
   - **Multiple Values:** False (True for `JobType`)

3. **Create managed properties for PnP Search Filters:**
   - For **date filters:** Use `RefinableDateFirst{num}`.
   - For **managed metadata filters:** Use `RefinableString{num}`.

4. **Reindex the site** after adding managed properties.

### App Catalogue

**Build and upload** the following projects:

- [PnP Modern Search (v4.14.0)](https://microsoft-search.github.io/pnp-modern-search/)
- [spfx-pnp-modern-search-career-marketplace](https://github.com/gcxchange-gcechange/spfx-pnp-modern-search-career-marketplace)
- [spfx-advanced-search](https://github.com/gcxchange-gcechange/spfx-advanced-search)
- [spfx-careerMarketplace](https://github.com/gcxchange-gcechange/spfx-careerMarketplace)

## Webpart Setup

### PnP - Search Box

- Add the web part to the page. No configuration needed.

### Advanced Search Web Part

1. Add the web part to the page.
2. Set the language to English or French in the control panel.

### PnP - Search Filters

1. Add the web part to the page.
2. Set the data connection to the **PnP - Search Results** web part.
3. **Customize Filters:**
   - Add filters and set the **Filter field** to the managed properties (e.g., `RefinableString{num}`).
   - **Template:**
     - Use **Check box** for most filters.
     - Use **Date range** for date filters.
   - For filters allowing multiple values (e.g., **Work Schedule**), enable multi-value options with the **OR** operator.
4. Set the **operator** between filters to **AND**.
5. Layout: **Horizontal**.

### PnP - Search Results

#### Config Page 1

- **Data Source:** SharePoint Search
- **Query Template:** Leave blank with a single space " " and apply.
- **Result Source Id:** LocalSharePointResults
- **Selected Properties:**
  - Add all managed properties created for the `JobOpportunity` list (`CM-{ColumnName}`).
  - Add: `Path`, `Title`, `UniqueID`.
- **Enable localization:** ON
- **Show paging:** ON
- **Items per page:** 9
- **Pages to display in range:** 5

#### Config Page 2

- **Layout:** "Job Opportunity" custom type with briefcase icon.
- **Show results count:** ON
- **Show selected filters:** ON
- **Language:** `en` or `fr`

#### Config Page 3

- **Use input query text:** ON
- **Dynamic Value:** Connected to **PnP - Search Box**.
- **Configure Query Modifiers:**
  - Enable **Advanced Search**.
- **Connect to Filters Web Part:** Target the **PnP - Search Filters** web part.

#### Advanced Search Settings

- **List Path:** `https://{tenant}.sharepoint.com/sites/{SiteName}/Lists/{JobOpportunityListName}/`
- **Search Box Selector:** `[data-sp-feature-tag="pnpSearchBoxWebPart"] input`
- **Search Button ID:** `advancedSearch-Search`
- **Clear Button ID:** `advancedSearch-Clear`

### Final Steps

- Refresh the page after applying settings.
- Repeat the process for both English and French.

## Function App Setup

### `appsvc-function-dev-cm-listmgmt-dotnet001`

1. **List hidden columns** in the `JobOpportunity` list:
   ```http
   GET /v1.0/sites/{site-id}/lists/{list-id}/columns?$select=id,name,hidden,displayName
   ```
2. Identify the hidden columns corresponding to the metadata fields. <br> - They're usually formatted like `{ColumnName}_0` and you want the name property of that object. <br> - Something like `ha560ef4634b48b49bcb4e0358a668ed`
3. Add the hidden column names to the **app settings** in the format: `{columnName}HiddenColName` (first character lowercase).
4. Complete the Function App setup as per the project documentation.

For more details, refer to the `README.md` of the function app.
