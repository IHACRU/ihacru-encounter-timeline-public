---
title: "Service Terrain"
output:
  html_document:
    css: ../../libs/css/sidebar.css
    highlight: tango
    keep_md: yes
    number_sections: yes
    theme: spacelab
    toc: yes
---

<!-- These two chunks should be added in the beginning of every .Rmd that you want to source an .R script -->
<!--  The 1st mandatory chunck  -->
<!--  Set the working directory to the repository's base directory -->


<!--  The 2nd mandatory chunck  -->
<!-- Set the report-wide options, and point to the external code file. -->


This report demonstrates the structure of the Clinical Context Coding Scheme (CCCS) with respect to the target cohort.


# Environment

This section will introduce you to the linguistic environment of the report. Use this section for reference as you interpret the code and its output in the present document. 
<!-- Load 'sourced' R files.  Suppress the output when loading packages. --> 

```r
# Attach these packages so their functions don't need to be qualified
library(magrittr) # pipes
library(readxl)   # excel input
library(ggplot2)  # graphing
library(dplyr)    # data wrangling
```


<!-- Load the sources.  Suppress the output when loading sources. --> 

```r
# Call in functions defined on other scripts 
base::source("./scripts/common-functions.R") # personal utilities
```

<!-- Load any Global functions and variables declared in the R file.  Suppress the output. --> 

```r
# where we stored an analytic product containing computed measures of service engagement
path_input <- "./data-public/raw/addiction-cohort-summative-2019-04-05.xlsx"

# SERVICE CLASS - a group of programs, homogenous with respect to services they provide
# the fundamental unit of analysis
service_class <- c(
  "service_class_code"         # numeric id
  ,"service_class_description" # label
)
# Available quantifications for service engagements registered by VIHA
# computed for each of service class in CCCS(6)
measures <- c(  # over "Addiction" cohort,  for Jan 1, 2007 - Sep 1, 2017
  "n_people"      # n of unique patients that engaged this service class at least once
  ,"n_encounters" # n of times the service was engaged by this cohort
)
# Dimensions of the Clinical Context Coding Scheme (CCCS)
cccs_dimensions <- c(
  "intensity_type"             # intensity level-1
  ,"intensity_severity_risk"   # intensity level-2
  ,"population_age"            # age of people targeted by this health service 
  ,"service_location"          # where service takes place 
  ,"clinical_focus"            # the nature of health conditions
  ,"service_type"              # type of service provided
)
variables_selected_for_analysis <- c(
  service_class      # a group of programs, homogeneous with respect to services they provide
  ,"beyond_ed_acute" # T/FALSE - service is in (ED + Acute) set, typically captured by an EHR
  ,measures          # quantifications of service engagement (n_people, n_encounters)
  ,cccs_dimensions   # Clinical Context Coding Scheme (6)
)  
```

<!-- Declare any global functions specific to a Rmd output.  Suppress the output. --> 


# Cohort
The cohort of `4,067` persons was selected on the basis of transactional data. Specifically, persons were included if they had at least one encounter with any of the following VIHA programs*:  

- **Detox**  
- **Stabilization Unit**  
- **Holly** (post-withdrawal stabilization unit)  
- **Grove** (post-withdrawal stabilization unit)  
- **Intensive Case Management Service**, Johnson Street (713 Outreach)    
- Intensive Case Management Service, **PES** (SAMI)    
- **Sobering and Assessment Centre** (maximum 23 hour stay - for persons who are under the influence of alcohol or other drugs.  

*Please note that these programs (`~1,700`) are what CCCS(6) groups into `~150` "service classes" using (6) dimensions of service description.  

These criteria are 'biased' in favour of ensuring inclusion of persons who have serious/chronic problems with abuse of **alcohol**. However, by including the Sobering and Assessment Centre in the inclusion criteria for defining the cohort, there will be some cases where the person uses drugs other than alcohol. In future analyses, we can refine the inclusion/exclusion criteria on the basis of the Substance Use Profile from the Minimum Reporting Requirements (MRR).

# Data 

For each individual in the cohort, we have extracted the complete record of engagement with VIHA services for the period between January 1, 2007 and September 1, 2017. We call these records **transactional data**(1), because they keep track of patients' encounters with the healthcare system. Encounter data for the Addiction cohort ( _N_ = `4,067`) were mapped onto 6 categories of CCCS(6) classification system (`./manipulation/1-greeter-transactions.R`), producing `124` distinct service classes, unique on the features of each CCCS(6) dimension.

 - (1) older documentation refers to this as *encounter data*

extracted from VIHA's CERNER-Millenium EHR system 
<!-- Load the datasets.   -->

```r
ds <- path_input %>% 
  readxl::read_excel() %>% 
  dplyr::select(variables_selected_for_analysis)
```

<!-- Inspect the datasets.   -->

```r
ds %>% dplyr::glimpse(80)
```

```
Observations: 125
Variables: 11
$ service_class_code        <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13...
$ service_class_description <chr> "Dedicated Psychiatric Emergency Settings...
$ beyond_ed_acute           <lgl> FALSE, TRUE, TRUE, TRUE, FALSE, FALSE, FA...
$ n_people                  <dbl> 564, 838, 165, 66, 447, 8, 1, 79, 2, 29, ...
$ n_encounters              <dbl> 940, 2085, 1103, 109, 754, 8, 1, 129, 2, ...
$ intensity_type            <chr> "ED, Urgent Care, Acute", "ED, Urgent Car...
$ intensity_severity_risk   <chr> "Emergent-Hospital", "Emergent-Community"...
$ population_age            <chr> "Mixed Ages", "Mixed Ages", "Adults, some...
$ service_location          <chr> "Hospital-ED", "Community", "Community", ...
$ clinical_focus            <chr> "MHSU", "MHSU", "MHSU", "Frailty, Non-Spe...
$ service_type              <chr> "ED-PES or Psychiatric Bed", "Crisis Resp...
```
<!-- Tweak the datasets.   -->


In this data frame (`ds`):   

* Each row is a class of health services (aka service class)  
* Service class is identified by `service_class_code` and/or `service_class_description` 
* Engagement with VIHA services is quantified as number of unique persons _`n_people`_ or a number of unique transactions `n_encounters`  
* Clinical Context Coding Scheme places each transaction 6 dimensions:   
  + intensity_type          (intensity level-1)
  + intensity_severity_risk (intensity level-2)
  + population_age          (age of people targeted by this health service)
  + service_location        (where service takes place)
  + clinical_focus          (the nature of health conditions)
  + service_type            (type of services provided)
  

# CCCS

Let us ask this dataframe a few questions to better understand its structure and contents.


```r
# How many different service classes have been engaged by this cohort of 4,067?
ds %>% dplyr::distinct(service_class_code) %>% dplyr::count() 
```

```
# A tibble: 1 x 1
      n
  <int>
1   124
```

```r
# How many categories are there in each dimension of CCCS(6)?
ds %>% 
  tidyr::gather(
    key    = "cccs_dimension"
    ,value = "possible_categories"
    ,c(
      "intensity_type"             # intensity level-1
      ,"intensity_severity_risk"   # intensity level-2
      ,"population_age"            # age of people targeted by this health service 
      ,"service_location"          # where service takes place 
      ,"clinical_focus"            # the nature of health conditions
      ,"service_type"              #
    )
  ) %>% 
  dplyr::group_by(cccs_dimension) %>% 
  dplyr::summarize(
    n_categories  = length(unique(possible_categories)) # N categories in this dimension
    ,n_encounters = sum(n_encounters)                   # N of unique transactions by the cohort
  ) %>% 
  dplyr::ungroup() %>% 
  neat() # custom table formatting, from `./scripts/common-functions.R`
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; ">
 <thead>
  <tr>
   <th style="text-align:left;"> cccs_dimension </th>
   <th style="text-align:right;"> n_categories </th>
   <th style="text-align:right;"> n_encounters </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> clinical_focus </td>
   <td style="text-align:right;"> 43 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> intensity_severity_risk </td>
   <td style="text-align:right;"> 36 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> intensity_type </td>
   <td style="text-align:right;"> 14 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> population_age </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> service_location </td>
   <td style="text-align:right;"> 14 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> service_type </td>
   <td style="text-align:right;"> 53 </td>
   <td style="text-align:right;"> 160318 </td>
  </tr>
</tbody>
</table>

Each dimension of CCCS(6) groups the same set of encounters into different number of categories. 

## Dimensions  

1. **Intensity_Type** - this is a set of categories that are used to classify the 1700 service locations into groups that are relatively homogeneous with respect to the manner in which service intensity would be coded. _(e.g. "Ambulatory_Chronic" would refer to ambulatory
services provided over an extended period of time)_  

2. **Intensity_Severity_Risk** - this set of categories is nested within Intensity_Type and is used to characterize the intensity of services provided within an Intensity_Type class. _(e.g.  three levels of intensity within the "Ambulatory_Chronic" class)_ 

3. **Population_Age** - this set of categories refers to the age range of the patients/clients who access a particular service. _(e.g.  "Infants" or "Older Adults Exclusively" or "Mixed")_  

4. **Service_Location** - this set of categories refers to the physical location where a given service is provided. _(e.g.  "Ambulatory Clinic" or "Hospital" or "Home")_  

5. **Clinical_Focus** - this set of categories refers to the predominant clinical focus of a service associated with a location in the Cerner location build. _(e.g.  "Diabetes" or "Frailty/Neurocognitive, Psychiatric")_  

6. **Service_Type** - this set of categories refers to the type of service provided. _(e.g.  "Assertive Community Treatment" or "Acute Care - Tertiary")_  


# Custom views


```r
t1 <- ds %>% 
  dplyr::filter(clinical_focus == "Med-Surg") %>% 
  dplyr::group_by(beyond_ed_acute, service_class_description, service_location) %>% 
  dplyr::summarize(
    n_encounters = sum(n_encounters)
  ) 

t1 %>% 
  neat() # custom table formatting, from `./scripts/common-functions.R`
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; ">
 <thead>
  <tr>
   <th style="text-align:left;"> beyond_ed_acute </th>
   <th style="text-align:left;"> service_class_description </th>
   <th style="text-align:left;"> service_location </th>
   <th style="text-align:right;"> n_encounters </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> FALSE </td>
   <td style="text-align:left;"> Acute Care - Children, Adolescents </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FALSE </td>
   <td style="text-align:left;"> Acute Care - Infants </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FALSE </td>
   <td style="text-align:left;"> Acute Care - Med-Surg - Mixed Ages </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 3320 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> FALSE </td>
   <td style="text-align:left;"> Acute Care - Med-Surg ED - Mixed Ages </td>
   <td style="text-align:left;"> Hospital-ED </td>
   <td style="text-align:right;"> 89 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> Ambulatory Episodic - Treatment - Medical Day Care </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> 103 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> Ambulatory Episodic - Urgent Assessment </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> 184 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> Med-Surg - Ambulatory Episodic - Mixed Ages </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> 371 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> Med-Surg - Ambulatory Mixed Episodic - Chronic - Child &amp; Youth </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> Med-Surg - Ambulatory Mixed Episodic - Chronic - Mixed Ages </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> 611 </td>
  </tr>
</tbody>
</table>

```r
t1 %>%  
  tidyr::spread( key = "beyond_ed_acute", value = "n_encounters") %>% 
  neat()
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; ">
 <thead>
  <tr>
   <th style="text-align:left;"> service_class_description </th>
   <th style="text-align:left;"> service_location </th>
   <th style="text-align:right;"> FALSE </th>
   <th style="text-align:right;"> TRUE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Acute Care - Children, Adolescents </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 46 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Acute Care - Infants </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Acute Care - Med-Surg - Mixed Ages </td>
   <td style="text-align:left;"> Hospital </td>
   <td style="text-align:right;"> 3320 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Acute Care - Med-Surg ED - Mixed Ages </td>
   <td style="text-align:left;"> Hospital-ED </td>
   <td style="text-align:right;"> 89 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ambulatory Episodic - Treatment - Medical Day Care </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 103 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ambulatory Episodic - Urgent Assessment </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 184 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Med-Surg - Ambulatory Episodic - Mixed Ages </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 371 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Med-Surg - Ambulatory Mixed Episodic - Chronic - Child &amp; Youth </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Med-Surg - Ambulatory Mixed Episodic - Chronic - Mixed Ages </td>
   <td style="text-align:left;"> Ambulatory Clinic </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 611 </td>
  </tr>
</tbody>
</table>


<!-- Basic table view.   -->


<!-- Basic graph view.   -->




