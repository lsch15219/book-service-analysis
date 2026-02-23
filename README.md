### Book Service Analytics Project

Overview: Ad hoc analytics on a mobile book service using ClickHouse SQL and Jupyter Notebook. The project explores user behavior, content popularity, platform usage, app metrics, and data quality.

### Tasks / Outputs
| Task | Description | Output Columns |
|-------|--------------|----------------|
| 1 | Top cities & regions by reading & listening hours | City/Region, Total Hours, iOS Hours, Android Hours |
| 2 | Top 5 books by usage; avg reading & listening times (both formats) | Book Title, Author, Total Hours, Avg Text Reading Hours, Avg Audio Listening Hours |
| 3 | Top authors by total reading; unique text books; avg mobile audiobook hours | Author, Total Reading Hours, Unique Text Books, Avg Audio Listening Hours |
| 4 | User segmentation (Reader, Listener, Both) by main platform | Platform, Segment, User Count |
| 5 | Usage patterns by content type & day (weekdays vs weekends) | Content Type, Avg Weekday Hours, Avg Weekend Hours |
| 6 | App version adoption per platform | Platform, % Users on Latest Version |
| 7 | App update frequency per user (update rate) | Platform, Update Rate |
| 8 | Books tagged “Magic” outside fiction | Magic Books Count |
| 9 | Books with “magic” in title but missing “Magic” tag (exclude fiction) | Magic Title Books Without Tag Count |
|10 | Avg number of categories: Magic books vs all books | Avg Categories (Magic Books), Avg Categories (All Books) |
|11 | Session anomalies via coefficient of variation by country & platform | Country, Platform, Coefficient of Variation, Notes |

### Methodology

- Data: Tables audition & content from the book service.
- Queries: ClickHouse SQL.
- Presentation: Structured analytical report in Markdown format with ClickHouse SQL queries, explanations, and pre-calculated result tables.
- Numeric Formatting: Hours rounded (whole numbers or 2 decimals).
- User Segmentation & Main Platform: Based on cumulative session hours.

### Outcomes

- Insights into user behavior, content popularity, platform usage, and app metrics.
- Identification of top cities, books, authors, and usage patterns.
- Detection of data inconsistencies and anomalies in sessions and content categorization.
- Demonstrates practical application of ad hoc analytics for decision-making in a mobile book service.
