UCPD Incident Report Data-set
================
Adam Shelton

## About

The University of Chicago Police Department, one of the largest private
police forces in the United States, maintains a jurisdiction of tens of
thousands of people in Hyde Park and surrounding areas. The university
publishes data on UCPD interactions with community members publicly to
their website. This includes [incident
reports](https://safety-security.uchicago.edu/police/data_information/daily_crime_fire_log/)
[which are archived back to
July 1, 2010](https://incidentreports.uchicago.edu/incidentReportArchive.php),
[traffic
stops](https://safety-security.uchicago.edu/police/data_information/traffic_stops/)
[which are archived back to
July 1, 2015](https://incidentreports.uchicago.edu/trafficStopsArchive.php),
and [field
interviews](https://safety-security.uchicago.edu/police/data_information/field_interviews/)
[which are archived back to
July 1, 2015](https://incidentreports.uchicago.edu/fieldInterviewsArchive.php).
Using the `rvest` package in R, these records were scraped from the site
and compiled into the data-set available here with the code below.
Snapshots of the data are also available to download in the
[Releases](https://github.com/tonofshell/ucpd-incident-data/releases)
section of this repo. View the analysis of this data
[here](analysis.md).

## Scraping Data

The archive website includes a form to specify a date range to display
(at 5 observations at a time). However, the URL follows a consistent
pattern to access the archive through a URL query, which is actually
much easier. The dates in the URL query are in [Unix epoch
time](https://www.epochconverter.com/) starting from 12:00 AM CST on the
first day in the archive to current time (although there appears to be a
delay in reports being published to the website). Also available in the
URL query is an ‘offset’ which goes to a specific observation at an
index. As each page displays five observations, we can increment this
offset by five until the scraper reaches the last page. The current page
number and total number of pages is scraped with each page to keep track
of the scraper’s progress.

``` r
scrape_ucpd_data = function(url, start_date, end_date = Sys.Date()) {
  obs_index = 0
  page_counts = c(0, 1)

  scraped_data = NULL
  prog_bar = NULL
  
  page = str_split(url, "/") %>% unlist() %>% .[length(.)]
  message(paste("Scraping", page, "- this may take awhile..."))
  while (page_counts[1] < page_counts[2]) {
    ucpd_url = paste0(url, "?startDate=", as.numeric(as.POSIXct(start_date)), "&endDate=", as.numeric(as.POSIXct(end_date)), "&offset=", obs_index)
    ucpd_page = read_html(ucpd_url)
    page_counts = ucpd_page %>% html_nodes(".page-count span") %>% html_text() %>% str_split("/") %>% unlist() %>% str_squish() %>% as.numeric()
    scrape_table = function(html_page) {
      html_page %>% html_nodes(".ucpd") %>% html_table() %>% .[[1]] %>% as_tibble() %>% mutate_all(as.character)
    }
    if (is.null(scraped_data)) {
      scraped_data = ucpd_page %>% scrape_table()
      prog_bar = txtProgressBar(min = 1, max = page_counts[2], style = 3)
    } else {
      scraped_data = bind_rows(scraped_data, ucpd_page %>% scrape_table())
      setTxtProgressBar(prog_bar, page_counts[1])
    }
    obs_index = obs_index + 5
  }
  return(scraped_data)
}

incident_data = scrape_ucpd_data("https://incidentreports.uchicago.edu/incidentReportArchive.php", "2010-07-01")

traffic_stop_data = scrape_ucpd_data("https://incidentreports.uchicago.edu/trafficStopsArchive.php", "2015-07-01")

field_interview_data = scrape_ucpd_data("https://incidentreports.uchicago.edu/fieldInterviewsArchive.php", "2015-07-01")
```

## Exporting Data

The scraped data is appended to a tibble on the fly, so from there, it
is simple to export the tibble as a CSV file. Some days will not have
any reports, with an observation specifying as such.

  - For the incident reports, each observation is a reported incident
    and includes and incident category, a location, the time the
    incident was reported, the time the reported incident occurred,
    comments/notes, status of the report, and an ID number. As of the
    last scraping at Mon Dec 02 13:57:04 2019 there were 12833
    observations.
  - For the traffic stops, each observation is a stop and includes the
    location and time the stop occurred, race and gender of the driver,
    IDOT classification of the stop, the reason for the stop, whether
    any citations/violations were issued, the disposition of the stop,
    and whether a search of the vehicle and/or its occupants was
    conducted. As of the last scraping there were 4395 observations.
  - For the field interviews, each observation is an interview and
    includes the location and time the interview occurred, who the
    interview was initiated by, race and gender of the person stopped,
    the reason for the stop, the disposition of the stop, and whether a
    search of the person stopped was conducted. As of the last scraping
    there were 1718 observations.

<!-- end list -->

``` r
write_csv(incident_data, here(paste0("ucpd_incident_data_scraped_", Sys.Date(), ".csv")))
write_csv(traffic_stop_data, here(paste0("ucpd_traffic_stop_data_scraped_", Sys.Date(), ".csv")))
write_csv(field_interview_data, here(paste0("ucpd_field_interview_data_scraped_", Sys.Date(), ".csv")))
```
