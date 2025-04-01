# Career Marketplace Setup

## SharePoint Setup

### Taxonomy

1. **Create the term group "Career Marketplace"** at the tenant level.
2. **Add a term set** for the following:
   - **Job Type**
   - **Program Area**
3. **Add terms** to each of the term sets created.
4. **Add translations** for each term set and any child terms.

### Job Opportunity List

1. **Create an SPFx list** in the Career Marketplace site called `JobOpportunity`.
2. **Add columns** for the following properties:

| Column Name             | Type                   | Source List         | Selected Column   | Additional Columns         | Additional Details            |
|-------------------------|------------------------|---------------------|-------------------|----------------------------|-------------------------------|
| **ContactObjectId**     | Single line of text    |                     |                   |                            |                               |
| **ContactName**         | Single line of text    |                     |                   |                            |                               |
| **Department**          | Lookup List Column     | Department          | NameEn            | NameFr, ID                 |                               |
| **ContactEmail**        | Single line of text    |                     |                   |                            |                               |
| **JobTitleEn**          | Single line of text    |                     |                   |                            |                               |
| **JobTitleFr**          | Single line of text    |                     |                   |                            |                               |
| **JobType**             | Managed Metadata       | Job Type            |                   |                            | Allow multiple selections     |
| **ProgramArea**         | Managed Metadata       | Program Area        |                   |                            |                               |
| **ClassificationCode**  | Lookup List Column     | ClassificationCode  | NameEn            | NameFr, ID                 |                               |
| **ClassificationLevel** | Lookup List Column     | ClassificationLevel | NameEn            | NameFr, ID                 |                               |
| **NumberOfOpportunities** | Number               |                     |                   |                            |                               |
| **Duration**            | Lookup List Column     | Duration            | NameEn            | NameFr, ID                 |                               |
| **ApplicationDeadlineDate** | Date and Time     |                     |                   |                            |                               |
| **JobDescriptionEn**    | Multiple lines of text |                     |                   |                            |                               |
| **JobDescriptionFr**    | Multiple lines of text |                     |                   |                            |                               |
| **WorkSchedule**        | Lookup List Column     | WorkSchedule        | NameEn            | NameFr                     |                               |
| **SecurityClearance**   | Lookup List Column     | SecurityClearance   | NameEn            | NameFr                     |                               |
| **LanguageComprehension** | Single line of text  |                     |                   |                            |                               |
| **LanguageRequirement** | Lookup List Column     | LanguageRequirement | NameEn            | NameFr, ID                 |                               |
| **WorkArrangement**     | Lookup List Column     | WorkArrangement     | NameEn            | NameFr                     |                               |
| **ApprovedStaffing**    | Yes/No                 |                     |                   |                            |                               |
| **Skills**              | Lookup List Column     | Skills              | NameEn            | NameFr                     | Allow multiple selections     |
| **City**                | Lookup List Column     | City                | NameEn            | NameFr, ID                 |                               |
| **DurationQuantity**    | Number               |                     |                   |                            |                               |
| **DurationInDays**      | Number               |                     |                   |                            |                               |

3. **Reindex the site** after adding columns:
   - Go to **Site Settings** -> **Search and offline availability** -> **Reindex site**.
   - This process may take some time before the new list columns appear in the crawled properties.

### Search Schema

1. **Create managed properties** for the columns in the `JobOpportunity` list:
   - Go to **SharePoint Admin Center** -> **More Features** -> **Search** -> **Manage Search Schema**.
   - Create a new managed property for each column/additional column(s) with the naming convention: `CM-{ColumnName}En`, `CM-{ColumnName}Fr`, or `CM-{ColumnName}Id`.
   - For **DurationInDays** map to one of the  existing `RefinableInt{num}` managed properties.
   - **Crawled Property Formats:**
     - Managed metadata: `OWS_TAXID_{ColumnName}`
     - Other fields: `OWS_{ColumnName}`

2. **Managed Property Settings:**
   - **Type:** Text (Yes/No for `ApprovedStaffing`)
   - **Searchable:** True
   - **Queryable:** True
   - **Retrievable:** True
   - **Multiple Values:** False (True for `JobType` and `Skills`)

3. **Create managed properties for PnP Search Filters:**
   - For **date filters:** Use `RefinableDateFirst{num}`.
   - For **managed metadata filters:** Use`RefinableString{num}`.

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
- **Query Template:** Leave blank with a single space ` ` and apply.
- **Result Source Id:** LocalSharePointResults
- **Selected Properties:**
  - **Add the following managed properties**
  - CM-ApplicationDeadlineDate
  - CM-City
  - CM-CityFr
  - CM-ClassificationLevel
  - CM-ClassificationCode
  - CM-ClassificationCodeFr
  - CM-ContactEmail
  - CM-ContactName
  - CM-ContactObjectId
  - CM-Duration
  - CM-DurationFr
  - CM-DurationQuantity
  - CM-JobDescriptionEn
  - CM-JobDescriptionFr
  - CM-JobTitleEn
  - CM-JobTitleFr
  - CM-JobType
  - Path
- **Language of search request:** English (1033)
- **Enable localization:** ON
- **Show paging:** ON
- **Items per page:** 9
- **Pages to display in range:** 5

#### Config Page 2

- **Layout:** "Job Opportunity" custom type with briefcase icon.
- **Show results count:** ON
- **Show selected filters:** ON
- **Selected language:** `en` or `fr`
- **Job opportunity page URL:** The URL to the job opportunity page up intil where ID would be `https://devgcx.sharepoint.com/sites/CM-test/SitePages/Job-Opportunity.aspx?JobOpportunityId=`
- **ApplicationDeadlineDate Managed Property:** `CM-ApplicationDeadlineDate`
- **CityEn Managed Property:** `CM-City`
- **CityFr Managed Property:** `CM-CityFr`
- **ClassificationLevel Managed Property:** `CM-ClassificationLevel`
- **ContactEmail Managed Property:** `CM-ContactEmail`
- **ContactName Managed Property:** `CM-ContactName`
- **ContactObjectId Managed Property:** `CM-ContactObjectId`
- **DurationEn Managed Property:** `CM-Duration`
- **DurationFr Managed Property:** `CM-DurationFr`
- **JobDescriptionEn Managed Property:** `CM-JobDescriptionEn`
- **JobDescriptionFr Managed Property:** `CM-JobDescriptionFr`
- **JobTitleEn Managed Property:** `CM-JobTitleEn`
- **JobTitleFr Managed Property:** `CM-JobTitleFr`
- **JobType Managed Property:** `CM-JobType`
- **DurationQuantity Managed Property:** `CM-DurationQuantity`
- **JobType term set GUID:** The term set GUID for JobType

#### Config Page 3

- **Use input query text:** ON
- Dynamic value
- Connected to **PnP - Search Box**.
- **Use as default value** ON
- **Default value** Blank ` `
- **Configure Query Modifiers:**
  - Enable **Advanced Search**.
- **Connect to Filters Web Part:** Target the **PnP - Search Filters** web part.

#### Advanced Search Settings (page 3 cont.)

- **JobOpportunity List Path:** `https://{tenant}.sharepoint.com/sites/{SiteName}/Lists/{JobOpportunityListName}/`
- **PnP Search Box Selector:** `[data-sp-feature-tag="pnpSearchBoxWebPart"] input`
- **Advanced Search - Search Button ID:** `advancedSearch-Search`
- **Advanced Search - Clear Button ID:** `advancedSearch-Clear`
- **English JobTitle Managed Property:** `CM-JobTitleEn`
- **French JobTitle Managed Property:** `CM-JobTitleFr`
- **Department Managed Property:** `CM-DepartmentId`
- **ClassificationCode Managed Property:** `CM-ClassificationCodeId`
- **ClassificationLevel Managed Property:** `CM-ClassificationLevelId`
- **LanguageRequirement Managed Property:** `CM-LanguageRequirementId`
- **City Managed Property:** `CM-CityId`
- **Duration Managed Property:** `CM-DurationId`
- **Duartion Quantity Managed Property:** The `RefineableInt{num}` you setup earlier.
- **ApplicationDeadline Filter Managed Property:** The `RefinableDateFirst{num}` you setup earlier.

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
