UCPD Incident Report Data-set
================
Adam Shelton

## About

The University of Chicago Police Department, one of the largest private
police forces in the United States, maintains a jurisdiction of tens of
thousands of people in Hyde Park and surrounding areas. The university
publishes incident records publicly to their website [which are archived
back to
July 1, 2010](https://incidentreports.uchicago.edu/incidentReportArchive.php).
Using the `rvest` package in R, these records were scraped from the site
and compiled into the data-set available here with the code below.
Snapshots of the data are also available to download in the
[Releases](https://github.com/tonofshell/ucpd-incident-data/releases)
section of this repo.

## Scraping Data

The archive website includes a form to specify a date range to display
(at 5 observations at a time). However, the URL follows a consistent
pattern to access the archive through a URL query, which is actually
much easier. The dates in the URL query are in [Unix epoch
time](https://www.epochconverter.com/) Starting from July 1, 2010 12:00
AM CST to current time (although there appears to be a delay in reports
being published to the website). Also available in the URL query is an
‘offset’ which goes to a specific observation at an index. As each
page displays five observations, we can increment this offset by five
until the scraper reaches the last page. The current page number and
total number of pages is scraped with each page to keep track of the
scraper’s progress.

## Exporting Data

The scraped data is appended to a tibble on the fly, so from there, it
is simple to export the tibble as a CSV file. Each observation is a
reported incident and includes and incident category, a location, the
time the incident was reported, the time the reported incident occurred,
comments/notes, status of the report, and an ID number. As of the last
scraping at Tue Sep 10 12:18:38 2019 there were 12505 observations.

``` r
write_csv(crime_data, here(paste0("ucpd_crime_data_scraped_", Sys.Date(), ".csv")))
```
