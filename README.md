# SocialVulnR
SOCIAL VULNERABILITY MAPPING IMPLEMENTED IN TIDYCENSUS FOR R

---
title: "Building an Adaptive Capacity Index Using TidyCensus in R"
author: "Anthony Holmes - Prepared on Behalf of The Nature Conservancy - Oregon"
copyright: 2020 - The Nature Conservancy
date: "March, 2020"
output:
  html_document:
    toc: true
    toc_float: 
      toc_float: true
      collapsed: false
    toc_depth: 2
    theme: journal
editor_options: 
  chunk_output_type: console
---

#Introduction#

<p>According to the CDC, Social vulnerability refers to the "resilience of communities when confronted by external stresses on humanhealth, stresses such as natural or human-caused disasters, or disease outbreaks." These methods implement an index similar to the one maintained by the CDC which are adapted from methods published by Flanagan et al. (2011) in "A Social Vulnerability Index for Disaster Management," which describes 13 metrics for assesing the 'social vulnerability' or 'adaptive capacity' (Davies et al.2018) of a census tract. This study attempts to replicate these metrics as closely as possible at a block group scale using 5 year ACS data in order to quantify the capacity of a block-group to respond to a given disturbance, minimizing risks to health, safety, property and other essential services. This index could be used to identify areas and communities at an increased risk and target appropriate response, recovery and mitigation efforts.

The SVI/ACI is <b>just one</b> component of a vulnerability assesment which would  include assesing the impact of the hazard itself (disease, fire, flood hurricane, etc.), the vulnerability of the physical infrastructure, and community assets/resources that could diminsish the impact of a given hazard and/or aid in recovery efforts in a given community (Flanagan et al. 2011).

The index comprises 4 domains including:

> Socioeconomic Status

    Percent of Persons Below Poverty Level
    Percent of Persons (age 16+) Unemployed
    Per Capita Income

> Language & Education

    Percent of Persons With No 12th Grade Education
    Percent of Persons Do Not Who Speak English

> Demographics

    Percent of Persons 65 Years of Age or Older
    Percent of Persons 17 Years of Age or Younger
    Percent of Persons 5 Years of Age or Older With a Disability
    Percent of Single Parent Households

> Housing & Transportation

    Percent of Persons Living in Multi-Unit Structure
    Percent of Persons Living in a Mobile Home
    Percent of Persons Living in ‘Crowded’ Conditions - more than one person per room
    Percent of Household With No Vehicle Available
    Percent of Persons Residing in Group Quarters

The index is constructed according to Flanagan et al. (2011) by assigning a percentile rank to each of the above named variables, ranked from highest to lowest (excluding per capita income which is ranked from lowest to highest). To calculate the final SVI, the sum for each of the previously calculated percentile ranks is taken for a given blockgroup and the percentile rank of these sums across a given geographic area results in the final SVI score, accordingly, the geographic area and enumeration units used in this equation will effect the final SVI derived for a given area so it is advised that the user consider these factors carefully.

Additional variables were brought in based on a review of the existing literature, however, these were not included in the index in this script since we were attempting to implement the existing literature of Flanagan et al. as closely as possible, however, the literature cited here supports their application in future iterations of the SVI index and their contribution to social vulnerability is worthy of further review.

#Works Cited#

Davies, Ian P., Haugo, Ryan D., Robertson, James C., & Levin, Phillip S. (2018). The unequal vulnerability of communities of color to wildfire. PLoS ONE, 13(11), E0205825.

Flanagan BE, Gregory EW, Hallisey EJ, Heitgerd JL, Lewis B. A Social Vulnerability Index for Disaster Management. J Homel Secur Emerg Manag. 2011;8. https://doi.org/10.2202/1547-7355.1792

#Tidy Census#

This script implements the R Package TidyCensus to bring in the necessary variables for this index, this allows the index to be easily reproduced for a vairety of geographies and time periods, many of the variables were found at the block group level and some could not be derived at this scale, there may be alternative variables or indicators available, however, this project only attempts to reproduce the index as published at the finest geographic scale available.

Kyle Walker (2020). tidycensus: Load US Census Boundary and Attribute Data as 'tidyverse' and 'sf'-Ready Data Frames. R package version 0.9.6. https://CRAN.R-project.org/package=tidycensus

#Block Group Level:#

These were the variables available/derived at the block group level

> TOTAL POPULATION
> PER CAPITA INCOME
> PERCENT OF POPULATION 65+
> PERCENT OF HOUSING UNITS W/ 10+ UNITS IN STRUCTURE
> PERCENT OF POPULATION LIVING IN ACCOMODATIONS W/ LESS THAN 1 ROOM PER PERSON (CROWDING)
> PERCENT OF POPULATION UNEMPLOYED
> PERCENT OF POPULATION WITH NO VEHICLE AVAILABLE
> PERCENT OF POPULATION LIVING IN MOBILE HOMES
> PERCENT OF POPULATION 25+ LESS THAN HS GRADUATE
> PERCENT OF POPULATION 65+
> PERCENT OF POPULATION SPEAK ENGLISH LESS THAN “VERY WELL”
> PERCENT OF POPULATION 17 YEARS AND UNDER
> PERCENT MINORITY


#Variables:#

These are all the variables used in the calculations at the blockgroup level

```{r}
c('B25003_001','B25003_003','B25070_007','B25070_008','B25070_009','B25070_010','B25071_001','B11007_001','B11007_003','B25034_001','B25034_008','B25034_009','B25034_01','B25034_011','B01003_001','B19301_001','B25033_001','B25033_006','B25033_007','B25033_012','B25033_013','B25044_001','B25044_003','B25044_010','B23025_003','B23025_005','B25014_001','B25014_005','B25014_006','B25014_007','B25014_011','B25014_012','B25014_013','B25024_001','B25024_007','B25024_008','B25024_009','B09021_022','B09021_001','B01001_020','B01001_021','B01001_022','B01001_023','B01001_024','B01001_025','B01001_044','B01001_045','B01001_046','B01001_047','B01001_048','B01001_049','B99163_001','B99163_005','B01001_003','B01001_004','B01001_005','B01001_006','B01001_027','B01001_028','B01001_029','B01001_030','B03002_003','B02001_004','B02001_00','B02001_003','B03003_003','B02001_006','B02001_007','B02001_008','B03002_003','B03002_001','B02001_001','B25002_001','B25002_003','B15003_001','B15003_016','B15003_017','B15003_018','B15003_019','B15003_020','B15003_021','B15003_022','B15003_023','B15003_024','B15003_025')
```

#How The Values Were Calculated:#

    1.) TOTAL POPULATION

#B01003_001

    2.) PER CAPITA INCOME

#B19301_001 - Estimate!!Per capita income in the past 12 months (in 2018 inflation-adjusted dollars)

    3.) PERCENT OF POPULATION BELOW POVERTY LEVEL

(C17002_002+C17002_003)/C17002_001

#C17002_001 - Estimate!!Total - RATIO OF INCOME TO POVERTY LEVEL IN THE PAST 12 MONTHS
#C17002_002 - Estimate!!Total!!Under .50
#C17002_003 - Estimate!!Total!!.50 to .99

    4.) PERCENT OF POPULATION 25+ LESS THAN 12TH GRADE EDUCATION

1-(B15003_016+B15003_017+B15003_018+B15003_019+B15003_020+B15003_021+B15003_022+B15003_023+B15003_024+B15003_025)/B15003_001

#B15003_001 - Estimate!!Total
#B15003_016 - Estimate!!Total!!12th grade, no diploma
#B15003_017 - Estimate!!Total!!Regular high school diploma
#B15003_018 - Estimate!!Total!!GED or alternative credential
#B15003_019 - Estimate!!Total!!Some college, less than 1 year
#B15003_020 - Estimate!!Total!!Some college, 1 or more years, no degree
#B15003_021 - Estimate!!Total!!Associate’s degree
#B15003_022 - Estimate!!Total!!Bachelor’s degree
#B15003_023 - Estimate!!Total!!Master’s degree
#B15003_024 - Estimate!!Total!!Professional school degree
#B15003_025 - Estimate!!Total!!Doctorate degree

    5.) PERCENT OF POPULATION LIVING IN MOBILE HOMES (Mobile homes estimate)

(B25033_006+B25033_007+B25033_012+B25033_013)/B25033_001

#B25033_001 - Estimate!!Total
#B25033_006 - Estimate!!Total!!Owner occupied!!Mobile home
#B25033_007 - Estimate!!Total!!Owner occupied!!Boat, RV, van, etc.
#B25033_012 - Estimate!!Total!!Renter occupied!!Mobile home
#B25033_013 - Estimate!!Total!!Renter occupied!!Boat, RV, van, etc.
  
    6.) PERCENT OF POPULATION WITH NO VEHICLE AVAILABLE (Households with no vehicle available estimate)

(B25044_003+B25044_010)/B25044_001

#B25044_001 - Estimate!!Total
#B25044_003 - Estimate!!Total!!Owner occupied!!No vehicle available
#B25044_010 - Estimate!!Total!!Renter occupied!!No vehicle available
  
    7.) PERCENT OF POPULATION UNEMPLOYED (Civilian (age 16+) unemployed estimate)

B23025_005/B23025_003

#B23025_003 - Estimate!!Total!!In labor force!!Civilian labor force
#B23025_005 - Estimate!!Total!!In labor force!!Civilian labor force!!Unemployed

#Notes:
According to the Census Bureau “In Civilian Labor Force” includes all civilians 16 years older.

    8.) PERCENT OF POPULATION LIVING IN ACCOMODATIONS W/ LESS THAN 1 ROOM PER PERSON/CROWDING (At household level, occupied housing units, more people than rooms estimate)

(B25014_005+B25014_006+B25014_007+B25014_011+B25014_012+B25014_013)/B25014_001

#B25014_001 - Estimate!!Total
#B25014_005 - Estimate!!Total!!Owner occupied!!1.01 to 1.50 occupants per room
#B25014_006 - Estimate!!Total!!Owner occupied!!1.51 to 2.00 occupants per room
#B25014_007 - Estimate!!Total!!Owner occupied!!2.01 or more occupants per room
#B25014_011 - Estimate!!Total!!Renter occupied!!1.01 to 1.50 occupants per room
#B25014_012 - Estimate!!Total!!Renter occupied!!1.51 to 2.00 occupants per room
#B25014_013 - Estimate!!Total!!Renter occupied!!2.01 or more occupants per room

    9.) PERCENT OF HOUSING UNITS W/ 10+ UNITS IN STRUCTURE (Housing in structures with 10 or more units estimate)

(B25024_007+B25024_008+B25024_009)/B25024_001

#B25024_001 - Estimate!!Total
#B25024_007 - Estimate!!Total!!10 to 19
#B25024_008 - Estimate!!Total!!20 to 49
#B25024_009 - Estimate!!Total!!50 or more

    10.) PERCENT OF POPULATION 65+ (Persons aged 65 and older estimate)

B09021_022/B09021_001

#B09021_022 - Estimate!!Total!!65 years and over
#B09021_001 - Estimate!!Total

    11.) PERCENT OF POPULATION WHO DO NOT SPEAK ENGLISH

B99163_005/B99163_001

#B99163_001 - Estimate!!Total
#B99163_005 - Estimate!!Total!!Speak other languages!!Ability to speak English –!!Not allocated

#Notes:
According to Flanagan et al. (2011) this should be “percent of persons who speak English ”less than well," however, a suitable variable could not be found to represent this in the 2018 ACS, that is not to say one doesn’t exist.

    12.) PERCENT OF POPULATION 17 YEARS AND UNDER

(B01001_003+B01001_004+B01001_005+B01001_006+B01001_027+B01001_028+B01001_029+B01001_030)/B01003_001

#B01001_003 - Estimate!!Total!!Male!!Under 5 years
#B01001_004 - Estimate!!Total!!Male!!5 to 9 years
#B01001_005 - Estimate!!Total!!Male!!10 to 14 years
#B01001_006 - Estimate!!Total!!Male!!15 to 17 years
#B01001_027 - Estimate!!Total!!Female!!Under 5 years
#B01001_028 - Estimate!!Total!!Female!!5 to 9 years
#B01001_029 - Estimate!!Total!!Female!!10 to 14 years
#B01001_030 - Estimate!!Total!!Female!!15 to 17 years

    13.) PERCENT MINORITY

1-(B03002_003/B03002_001)

#B03002_003 - Estimate!!Total (WHITE ALONE, NOT HISPANIC OR LATINO)

#Notes: 
I used the B03002 table because “Hispanic/Latino” is not considered a “race” by the U.S. Census Bureau, people of Hispanic or Latino origin either don’t answer this question or they select/write-in any number of racial categories and which one they select can vary by geography and other demographic factors. According to the Pew Research Center, “In the 2010 census, 37% of Hispanics — 18.5 million people — said they belonged to “some other race.” Among those who answered the race question this way in the 2010 census, 96.8% were Hispanic. And among those Hispanics who did, 44.3% indicated on the form that Mexican, Mexican American or Mexico was their race, 22.7% wrote in Hispanic or Hispano or Hispana, and 10% wrote in Latin American or Latino or Latin.” The Census Bureau suggests that if you want to treat “Hispanic” as a “race-like” category, you use the B03002 table. So if you were to use the B02001 table for “Percent White”, it could also include minority Hispanic groups that identify as white.

Sources: https://censusreporter.org/topics/race-hispanic/, https://www.pewsocialtrends.org/2015/06/11/chapter-1-race-and-multiracial-americans-in-the-u-s-census/

    13a.) PERCENT AMERICAN INDIAN AND ALASKA NATIVE ALONE

B02001_004/B02001_001

B02001_004 - Estimate!!Total (AMERICAN INDIAN AND ALASKA NATIVE ALONE)

    13b.) PERCENT ASIAN ALONE

B02001_005/B02001_001

B02001_00 - Estimate!!Total (ASIAN ALONE)

    13c.) PERCENT BLACK OF AFRICAN AMERICAN ALONE

B02001_003/B02001_001

B02001_003 - Estimate!!Total (BLACK OR AFRICAN AMERICAN ALONE)

    13d.) PERCENT HISPANIC OR LATINO

B03003_003/B03003_001

B03003_003 - Estimate!!Total (HISPANIC OR LATINO)

    13e.) PERCENT NATIVE HAWAIIAN AND OTHER PACIFIC ISLANDER ALONE

B02001_006/B02001_001

B02001_006 - Estimate!!Total (NATIVE HAWAIIAN AND OTHER PACIFIC ISLANDER ALONE)

    13f.) PERCENT SOME OTHER RACE ALONE

B02001_007/B02001_001

B02001_007 - Estimate!!Total (SOME OTHER RACE ALONE)
    
    13g.) PERCENT TWO OR MORE RACES

B02001_008/B02001_001

B02001_008 - Estimate!!Total (TWO OR MORE RACES)

    13h.) PERCENT WHITE

B03002_003/B03002_001

B03002_003 - Estimate!!Total (WHITE ALONE, NOT HISPANIC OR LATINO)

#OTHER VARIABLES TO INCLUDE (OPTIONAL):#

    PERCENT OF HOMES OCCUPIED

1-B25002_003/B25002_001

B25002_001 - Estimate!!Total (TOTAL NUMBER OF HOUSING UNITS)
B25002_003 - Estimate!!Total!!Vacant

    PERCENT OF HOMES RENTER OCCUPIED

B25003_003/B25003_001

B25003_001 - Estimate!!Total
B25003_003 - Estimate!!Total!!Renter occupied

#Notes: 
According to Lee et al. 2019, “Previous studies have shown that renters lack resources prior to a disaster (preparedness) and continue to do so in post disaster times (recovery). Before a disaster, available resources differ, including available funds and housing condition and location. Renters have limited household, social, and physical resources prior to a disaster, as compared to those of owners. Additionally, renters tend to suffer more severe damage during a disaster. During recovery, renters limited financial resources (i.e., lack of insurance and less governmental assistance; Bolin and Stanford 1998; Comerio 1998), as well as inadequate social and political capital, mean that they have less influence on decisions being made about recovery (Morrow 1999). This causes them to face greater struggles over a longer period of time.” (Lee et al. 2019)

Sources:

Lee, J., & Van Zandt, S. (2019). Housing Tenure and Social Vulnerability to Disasters: A Review of the Evidence. Journal of Planning Literature, 34(2), 156-170.

    PERCENT OF HOUSEHOLDS PAYING MORE THAN 30% OF THEIR INCOME ON RENT:

(B25070_007+B25070_008+B25070_009+B25070_010)/b25070_001

B25070_007 - Estimate!!Total!!30.0 to 34.9 percent
B25070_008 - Estimate!!Total!!35.0 to 39.9 percent
B25070_009 - Estimate!!Total!!40.0 to 49.9 percent
B25070_010 - Estimate!!Total!!50.0 percent or more

    MEDIAN GROSS RENT AS A PERCENTAGE OF HOUSEHOLD INCOME

B25071_001

B25071_001 - Estimate!!Median gross rent as a percentage of household income

#Notes: 
According to the U.S. Department of Housing and Urban Development (HUD) “Families who pay more than 30 percent of their income for housing are considered cost burdened and may have difficulty affording necessities such as food, clothing, transportation and medical care.” They may be above poverty level, but the cost of housing may still be a factor that effects their vulnerability and ability to recover from a crisis (HUD). According to Diaz et al., “Renting, crowding, and unaffordable housing are directly and indirectly linked with negative outcomes for children and adults (e.g., Evans, Lepore, Shejwal, & Palsane, 1998; Haurin, Parcel, & Haurin, 2002; Leventhal & Newman, 2010; Pollack, Griffın, & Lynch, 2010). These housing situations, coupled with other vulnerabilities, also increase exposure to harm and limit the ability of individuals and households to cope with and recover from the impacts of environmental hazards … For many households, spending more on housing costs than is standard suggests housing situations that are unstable, difficult to maintain over the long term, and may reflect stresses that are otherwise difficult to detect” (excerpt from Diaz et al. 2017).

Sources: Díaz Mcconnell, E. (2017). Rented, Crowded, and Unaffordable? Social Vulnerabilities and the Accumulation of Precarious Housing Conditions in Los Angeles. Housing Policy Debate, 27(1), 60-79.

    PERCENT OF HOUSEHOLDS WITH SENIORS (65+) LIVING ALONE

B11007_003/B11007_001

B11007_001 - Estimate!!Total
B11007_003 - Estimate!!Total!!Households with one or more people 65 years and over!!1-person household

#Notes: 
“That seniors are more vulnerable to disasters is a proposition that is supported by a growing body of literature. Experimental research confirms that the elderly and disabled confront unique difficulties during periods of evacuation.” (Donner et al. 2008). For example: “Half of the deaths from Hurricane Katrina were adults age 75 and older (Brunkard, Namulanda, and Ratard, 2008), and 63 percent of the deaths after the 1995 heat wave in Chicago were adults age 65 or older (Whitman et al., 1997)” … A 2012 survey found that 15 percent of U.S. adults age 50 or older would not be able to evacuate their homes without help, and half of this group would need help from someone outside the household (National Association of Area Agencies on Aging, National Council on Aging, and UnitedHealthcare, 2012) (Shih et al. 2018)”. According to Chau et al. “Living alone increases the risk of social isolation, which may, in turn, be associated with poorer mental and physical health, and leads to problems in escape and recovery from emergency situations” (Chau et al. 2014).

Sources: Chau, P., Gusmano, H., Cheng, M., Cheung, K., & Woo, J. (2014). Social Vulnerability Index for the Older People—Hong Kong and New York City as Examples. Journal of Urban Health, 91(6), 1048-1064.

Shih RA, Acosta JD, Chen EK, et al. Improving Disaster Resilience Among Older Adults: Insights from Public Health Departments and Aging-in-Place Efforts. Rand Health Q. 2018;8(1):3. Published 2018 Aug 2.


    PERCENT OF HOMES BUILT BEFORE 1969
    
(B25034_008+B25034_009+B25034_010+B25034_011)/B25034_001

B25034_001 - Estimate!!Total
B25034_008 - Estimate!!Total!!Built 1960 to 1969
B25034_009 - Estimate!!Total!!Built 1950 to 1959
B25034_010 - Estimate!!Total!!Built 1940 to 1949
B25034_011 - Estimate!!Total!!Built 1939 or earlier

#Notes: 
“Congress created the National Flood Insurance Program (NFIP) in 1968 In order to participate, jurisdictions are required to adopt a floodplain management ordinance. Units that pre-date the local adoption of those ordinances are more likely to be out of compliance with the design and construction standards they require. Thus, the age of the buildings in floodplains may offer some insight into whether buildings are designed or retrofitted for flooding.” (Furman Center)

>Tract Level:

These Variables were only found to be available at the Tract level, that is not to say a suitable variable does not exist at the block-group level

> PERCENT OF POPULATION BELOW POVERTY LEVEL
> PERCENT OF POPULATION 5< W/ A DISABILITY
> PERCENT OF POPULATION LIVING IN GROUP QUARTERS
> PERCENT OF CHILDREN LIVING IN SINGLE PARENT HOUSEHOLDS

Tract Level Variables:

```{r}
c('B18101_025','B18101_026','B18101_006','B18101_007','C18130_009','C18130_010','C18130_016','C18130_017','B17020_001','B17020_002','B26001_001','B11001_001','B11004_012','B11004_018','B11001_001','B09008_010','B09008_011','B09008_012','B17023_001','B17023_016','B17023_017','B17023_018')
```

How The Values Were Calculated:

    14.) PERCENT OF POPULATION 5< W/ A DISABILITY (Civilian noninstitutionalized population with a disability estimate)

(B18101_026+B18101_007+C18130_010+C18130_017)/(B18101_025+B18101_006+C18130_009+C18130_016)

#B18101_025 - Estimate!!Total!!Female!!5 to 17 years
#B18101_026 - Estimate!!Total!!Female!!5 to 17 years!!With a disability
#B18101_006 - Estimate!!Total!!Male!!5 to 17 years
#B18101_007 - Estimate!!Total!!Male!!5 to 17 years!!With a disability
#C18130_009 - Estimate!!Total!!18 to 64 years
#C18130_010 - Estimate!!Total!!18 to 64 years!!With a disability
#C18130_016 - Estimate!!Total!!65 years and over
#C18130_017 - Estimate!!Total!!65 years and over!!With a disability

    15.) PERCENT OF POPULATION LIVING IN GROUP QUARTERS

B26001_001/B01003_001

#B26001_001 - Estimate!!Total!!Group quarters population

    16.) PERCENT SINGLE PARENT HOUSEHOLD

(B09008_010+B09008_011+B09008_012)/B09008_001

#B09008_001 - Estimate!!Total
#B09008_010 - Estimate!!Total!!No unmarried partner of householder present!!In family households!!In male householder, no wife present, family
#B09008_011 - Estimate!!Total!!No unmarried partner of householder present!!In family households!!In female householder, no husband present, family
#B09008_012 - Estimate!!Total!!No unmarried partner of householder present!!In nonfamily households

#Notes:

According to Flanagan et al. this should be an estimate of Percent male or female householder, no spouse present, with children under 18. There may be a variety of approximations available, including, but not limited to B23008, however, these table fails to represent a variety of housing situations in which a child may be under the guardianship of a single adult caretaker and may also include situations in which a household is classified as a single parent household where there is the presence of an unmarried partner who is likely to provide support in the responsibilities of parenting, diminishing the unique risk faced by a single parent in a disaster scenario. According the Census Bureau, “A nonfamily householder” is a “householder living alone or with non-relatives only.” A family is a group of two people or more (one of whom is the householder) related by birth, marriage, or adoption and residing together so If there is 1.) No “unmarried partner of householder present” and 2.) It is a nonfamily household so the absence of a married partner is assumed and 3.) There is a child present then I assumed it would include children living with unmarried non-relatives where there is no unmarried partner present so although the child is unrelated to their adult caretaker, their adult care taker would function essentially as a “single parent.”

> OTHER VARIABLES TO INCLUDE (OPTIONAL):

    PERCENT SINGLE MOTHER HOUSEHOLDS BELOW POVERTY LINE

(B17023_016+B17023_017+B17023_018)/B22002_001

#B17023_001 - Estimate!!Total
#B17023_016 - Estimate!!Total!!Income in the past 12 months below poverty level!!Other families!!Female householder, no husband present!!1 or 2 own children of the householder
#B17023_017 - Estimate!!Total!!Income in the past 12 months below poverty level!!Other families!!Female householder, no husband present!!3 or 4 own children of the householder
#B17023_018 - Estimate!!Total!!Income in the past 12 months below poverty level!!Other families!!Female householder, no husband present!!5 or more own children of the householder

#Notes: 

“Demographic data shows that single mothers tend to be less educated and poorer than the general population, thus placing them at greater risk to disasters … Morrow and Enarson (1996) observed, during the long-term recovery period following Hurricane Andrew, that poor and minority women experienced significant difficulties in accessing relief and recovery aid given that policy programs were reportedly set up only with small, nuclear households in mind. Moreover, Morrow and Enarson argue that women were victims of exploitation and fraud during the recovery period. In another study, Donner (2003) shows that census tracts in the United States with larger percentages of single mothers are more likely to experience a tornado death and/or injury. Rodriguez and Russell (2006) also report that women and children were disproportionately affected by the 2004 Indian Ocean Tsunami and were more likely to be victims (e.g., fatalities) of this catastrophic event relative to their male counterparts.” (excerpt from Donner et al. 2008)

Sources: Donner, W., & Rodríguez, H. (2008). Population Composition, Migration and Inequality: The Influence of Demographic Changes on Disaster Risk and Vulnerability. Social Forces, 87(2), 1089-1114. Retrieved February 20, 2020, from www.jstor.org/stable/20430904

>Setup

Install and load required packages

```{r}
    ReqPkgs <- c('knitr','sp','sf','spdep','tidycensus','dplyr','tidyr','mapview','RColorBrewer','leaflet','leafpop','ggplot2')
    ReqPkgs <- as.list(ReqPkgs)
    #suppressMessages(lapply(ReqPkgs, install.packages, character.only = TRUE))
    suppressMessages(lapply(ReqPkgs, require, character.only = TRUE))
```

> TidyCensus

For this section you will use the `tidycensus` package to read in data from the American Community Survey, including geometry.

Get a Free Census Api Key Here: https://api.census.gov/data/key_signup.html

```{r}
    #tidycensus::census_api_key(key = 'YOUR CENSUS API KEY HERE', install = TRUE, overwrite = TRUE)
    readRenviron("~/.Renviron")
```

> List Counties in State

Now were making a simple character vector to store the names of all the counties we'll be using to pull in the census data since no option exists to pull out blockgroup level data for the whole state.

```{r}
Counties <- tigris::list_counties(state = 'Oregon')
Counties <- Counties$county
print(Counties)
```

> List Variables To Be Pulled In

Now were making a simple character vector to store the names of all the variables we'll be pulling in at the blockgroup and tract level, these were selected through a long process of trial an error

```{r}
varsBG <- c('B25003_001','B25003_003','B25070_007','B25070_008','B25070_009','B25070_010','B25071_001','B11007_001','B11007_003','B25034_001','B25034_008','B25034_009','B25034_010','B25034_011','B01003_001','B19301_001','B25033_001','B25033_006','B25033_007','B25033_012','B25033_013','B25044_001','B25044_003','B25044_010','B23025_003','B23025_005','B25014_001','B25014_005','B25014_006','B25014_007','B25014_011','B25014_012','B25014_013','B25024_001','B25024_007','B25024_008','B25024_009','B09021_022','B09021_001','B01001_020','B01001_021','B01001_022','B01001_023','B01001_024','B01001_025','B01001_044','B01001_045','B01001_046','B01001_047','B01001_048','B01001_049','B99163_001','B99163_005','B01001_003','B01001_004','B01001_005','B01001_006','B01001_027','B01001_028','B01001_029','B01001_030','B03002_003','B02001_004','B02001_001','B02001_003','B03003_003','B02001_006','B02001_007','B02001_008','B03002_003','B03002_001','B02001_001','B25002_001','B25002_003','B15003_001','B15003_016','B15003_017','B15003_018','B15003_019','B15003_020','B15003_021','B15003_022','B15003_023','B15003_024','B15003_025','B02001_005','B03003_001','B25070_001','B17020_001','C17002_001','C17002_002','C17002_003','C17002_004')

varsCT <- c('B18101_025','B18101_026','B18101_006','B18101_007','C18130_009','C18130_010','C18130_016','C18130_017','B26001_001','B11004_012','B11004_018','B09008_001','B09008_010','B09008_011','B09008_012','B17023_001','B17023_016','B17023_017','B17023_018','B22002_001')
```

> Pulling in Block Group Level 

This chunk of code is pulling in all the blockgroup level variables described previously

```{r}
 CBG18_1 <- tidycensus::get_acs(

#get_decentennial() pulls in data from the decentennial census 1990-2010
  
  geography = 'block group', #other options include us, region, division, state, county subdivision, census tract, block, place, alaska native regional corporation, american indian area/alaska native area/hawaiian home land, american indian area/alaska native area (reservation or statistical entity only), american indian area (off-reservation trust land only)/hawaiian home land, metropolitan statistical area/micropolitan statistical area, combined statistical area, new england city and town area, combined new england city and town area, urban area, congressional district, school district (elementary, secondary or unified), public use microdata area, zip code tabulation area, and state legislative district (upper or lower chamber).
  
  state = 'OR',
  
  county = Counties, #The county list created in the previous step
  
  survey = 'acs5', #could include the ACS 1, 3 or 5 year surveys
  
  year = 2018, #2009 through 2018 are available. Defaults to 2018
  
  variables = varsBG, #The variable list created in the previous step, use tidycensus::load_variables to see what variables are available for the survey and or geography, there may be alternatives or others you want to add!

  geometry = FALSE, #if TRUE, uses the tigris package to return an sf tibble with simple feature geometry in the 'geometry' column. We use Tigris later to pull the geometry in.
  
  output = 'wide',
  
  show_call = FALSE
  
)

#Separate Place Names#

CBG18_1 <- tidyr::separate(data = CBG18_1, col = "NAME", into = c("BLOCK_GROUP","CENSUS_TRACT","COUNTY","STATE"), sep = ",", remove = FALSE)

CBG18_1$TRACT_GEOID <- substring(CBG18_1$GEOID, 1, 11)

print(dim(CBG18_1)) #The dimensions should match this for Oregon: 2,634 x 187
```

> Pulling in Tract Level Variables

This chunk of code is pulling in all the tract level variables described previously

```{r}
CT18B <- tidycensus::get_acs(
  
  geography = 'Tract',
  
  state = 'OR',
  
  county = Counties,

    #c('Columbia','Clatsop','Umatilla','Wallowa','Morrow','Union','Gilliam','Tillamook','Washington','Sherman','Multnomah','Hood River','Wasco','Clackamas','Yamhill','Marion','Baker','Polk','Wheeler','Lincoln','Grant','Jefferson','Linn','Benton','Crook','Malheur','Deschutes','Lane','Harney','Douglas','Lake','Klamath','Coos','Jackson','Curry','Josephine'), # This can include multiple counties
  
  survey = 'acs5',
  
  year = 2018,
  
  variables = varsCT,

  geometry = FALSE,
  
  output = 'wide',
  
  show_call = FALSE
  
)

#Separate Place Names#

CT18B <- tidyr::separate(data = CT18B, col = "NAME", into = c("CENSUS_TRACT","COUNTY","STATE"), sep = ",")

CT18B$TRACT_GEOID <- CT18B$GEOID

print(dim(CT18B))

#The dimensions should match this for Oregon: 834 x 45
```

> Join Tables

This chunk uses dplyr to join the tract level variables to the block groups, the variables remain consistent across the block group, this is not ideal and if you find some way to represent these variables more accurately at the block group level, please feel free to change them.

```{r}
JndTbls <- dplyr::left_join(x = CBG18_1, y = CT18B, by = "TRACT_GEOID")

dim(JndTbls) #get dimensions

#The dimensions should match this for Oregon: 2,634 x 231
```

> Now to calculate each of the statistics

```{r}
JndTbls$BLANK1 <- " " #These breaks just make it easier to navigate the table

#SOCIOECONOMIC STATUS:

JndTbls$TOTPOP <- JndTbls$B01003_001E #TOTAL_POPULATION - 
JndTbls$POV <- (JndTbls$C17002_002E+JndTbls$C17002_003E)/JndTbls$C17002_001E #PER_POVERTY
JndTbls$UNEMP <- JndTbls$B23025_005E/JndTbls$B23025_003E #PER_UNEMPLOYED 
JndTbls$PCI <- JndTbls$B19301_001E #PER_CAPITA_INCOME

#LANGUAGE AND EDUCATION:

JndTbls$NOHSDP <- 1-((JndTbls$B15003_016E+JndTbls$B15003_017E+JndTbls$B15003_018E+JndTbls$B15003_019E+JndTbls$B15003_020E+JndTbls$B15003_021E+JndTbls$B15003_022E+JndTbls$B15003_023E+JndTbls$B15003_024E+JndTbls$B15003_025E)/JndTbls$B15003_001E) #PER_LESS_HS_GRAD
JndTbls$LIMENG <-  JndTbls$B99163_005E/JndTbls$B99163_001E #PER_POOR_ENGLISH

#DEMOGRAPHICS:

JndTbls$AGE65 <- JndTbls$B09021_022E/JndTbls$B09021_001E #PER_OVER_65 
JndTbls$AGE17 <- (JndTbls$B01001_003E+JndTbls$B01001_004E+JndTbls$B01001_005E+JndTbls$B01001_006E+JndTbls$B01001_027E+JndTbls$B01001_028E+JndTbls$B01001_029E+JndTbls$B01001_030E)/JndTbls$B01003_001E #PER_UNDER_17 
JndTbls$DISABL <- (JndTbls$B18101_026E+JndTbls$B18101_007E+JndTbls$C18130_010E+JndTbls$C18130_017E)/(JndTbls$B18101_025E+JndTbls$B18101_006E+JndTbls$C18130_009E+JndTbls$C18130_016E) #PER_DISABLED
JndTbls$SNGPNT <- (JndTbls$B09008_010E+JndTbls$B09008_011E+JndTbls$B09008_012E)/JndTbls$B09008_001E #PER_SINGL_PRNT

#HOUSING AND TRANSPORTATION:

JndTbls$MUNIT <- (JndTbls$B25024_007E+JndTbls$B25024_008E+JndTbls$B25024_009E)/JndTbls$B25024_001E #PER_MULTI_DWELL
JndTbls$MOBILE <- (JndTbls$B25033_006E+JndTbls$B25033_007E+JndTbls$B25033_012E+JndTbls$B25033_013E)/JndTbls$B25033_001E #PER_MOBILE_DWEL
JndTbls$CROWD <- (JndTbls$B25014_005E+JndTbls$B25014_006E+JndTbls$B25014_007E+JndTbls$B25014_011E+JndTbls$B25014_012E+JndTbls$B25014_013E)/JndTbls$B25014_001E #PER_CROWD_DWELL
JndTbls$NOVEH <- (JndTbls$B25044_003E+JndTbls$B25044_010E)/JndTbls$B25044_001E #PER_NO_VEH_AVAIL
JndTbls$GROUPQ <- JndTbls$B26001_001E/JndTbls$B01003_001E #PER_GROUP_DWELL

#RACIAL AND ETHNIC MAKEUP:

JndTbls$RACIAL_ETHNIC_VARIABLES <- " "

JndTbls$MINORITY <- 1-(JndTbls$B03002_003E/JndTbls$B03002_001E)
JndTbls$NTVAMRCN <- JndTbls$B02001_004E/JndTbls$B02001_001E
JndTbls$ASIAN <- JndTbls$B02001_005E/JndTbls$B02001_001E
JndTbls$BLACK <- JndTbls$B02001_003E/JndTbls$B02001_001E
JndTbls$HISPLATX <- JndTbls$B03003_003E/JndTbls$B03003_001E
JndTbls$PACISL <- JndTbls$B02001_006E/JndTbls$B02001_001E
JndTbls$OTHRRACE <- JndTbls$B02001_007E/JndTbls$B02001_001E
JndTbls$MULTRACE <- JndTbls$B02001_008E/JndTbls$B02001_001E
JndTbls$WHITE <- JndTbls$B03002_003E/JndTbls$B03002_001E

#OPTIONAL VARIABLES:

JndTbls$OPTIONAL_VARIABLES <- " "

JndTbls$HOMESOCCPD <- 1-JndTbls$B25002_003E/JndTbls$B25002_001E
JndTbls$RENTER <- JndTbls$B25003_003E/JndTbls$B25003_001E
JndTbls$RENTBURDEN <- (JndTbls$B25070_007E+JndTbls$B25070_008E+JndTbls$B25070_009E+JndTbls$B25070_010E)/JndTbls$B25070_001E
JndTbls$RENTASPERINCOME <- (JndTbls$B25071_001E/100)
JndTbls$OVR65ALONE <- JndTbls$B11007_003E/JndTbls$B11007_001E
JndTbls$BLTBFR1969 <- (JndTbls$B25034_008E+JndTbls$B25034_009E+JndTbls$B25034_010E+JndTbls$B25034_011E)/JndTbls$B25034_001E
JndTbls$SVRPOV <- JndTbls$C17002_002E/JndTbls$C17002_001E
JndTbls$MODPOV <- JndTbls$C17002_004E/JndTbls$C17002_001E
JndTbls$SINGLMTHRPVRTY <-(JndTbls$B17023_016E+JndTbls$B17023_017E+JndTbls$B17023_018E)/JndTbls$B17023_001E

#RANKING#

#These functions rank each of the variables, variables with matching values across ranks are given the max score, this is the default in excel where the original formulae were derived

a <- JndTbls$RNKPOV <- rank(x = -JndTbls$POV, na.last = "keep", ties.method = "max")
b <- JndTbls$RNKUNEMP <- rank(x = -JndTbls$UNEMP, na.last = "keep", ties.method = "max")
c <- JndTbls$RNKPCI <- rank(x = JndTbls$PCI, na.last = "keep", ties.method = "max") #Note that we are not taking the inverse here because the higher the Per Capita Income, the greater the Adaptive Capacity of a given blockgroup
d <- JndTbls$RNKNOHSDP <- rank(x = -JndTbls$NOHSDP, na.last = "keep", ties.method = "max")
e <- JndTbls$RNKLIMENG <- rank(x = -JndTbls$LIMENG, na.last = "keep", ties.method = "max")
f <- JndTbls$RNKAGE65 <- rank(x = -JndTbls$AGE65, na.last = "keep", ties.method = "max")
g <- JndTbls$RNKAGE17 <- rank(x = -JndTbls$AGE17, na.last = "keep", ties.method = "max")
h <- JndTbls$RNKDISABL <- rank(x = -JndTbls$DISABL, na.last = "keep", ties.method = "max")
i <- JndTbls$RNKSNGPNT <- rank(x = -JndTbls$SNGPNT, na.last = "keep", ties.method = "max")
j <- JndTbls$RNKMUNIT <- rank(x = -JndTbls$MUNIT, na.last = "keep", ties.method = "max")
k <- JndTbls$RNKMOBILE <- rank(x = -JndTbls$MOBILE, na.last = "keep", ties.method = "max")
l <- JndTbls$RNKCROWD <- rank(x = -JndTbls$CROWD, na.last = "keep", ties.method = "max")
m <- JndTbls$RNKNOVEH <- rank(x = -JndTbls$NOVEH, na.last = "keep", ties.method = "max")
n <- JndTbls$RNKGROUPQ <- rank(x = -JndTbls$GROUPQ, na.last = "keep", ties.method = "max")

#Sum The Ranks

JndTbls$SUMRANK = a+b+c+d+e+f+g+h+i+j+k+l+m+n

#Derive the Adaptive Capacity Index

JndTbls$ADPTVCAPACITY <- dplyr::percent_rank(JndTbls$SUMRANK)
```

> This Finds How Much Each Variable Contributed to The Final Percent Rank

```{r}
# This Determines the Percentage Contribution to Final Rank
JndTbls$GEOID <- JndTbls$GEOID.x #Geoid.s was created in the previous join and needs to be renamed before joining it to the geometry
geoid <- which(colnames(JndTbls)=="GEOID")
a <- which(colnames(JndTbls)=="RNKPOV")
z <- which(colnames(JndTbls)=="RNKGROUPQ")
cols <- as.vector(names(JndTbls[a:z]))
Func <- function(x){round((abs(x)/abs(JndTbls$SUMRANK)),2)*100}
RnkPerc <- dplyr::mutate_at(.tbl = JndTbls, .vars = cols, .funs = Func)
RnkPerc <- RnkPerc[c(geoid, a:z)]
JndTbls <- dplyr::right_join(JndTbls, RnkPerc, by = "GEOID")
```

> Now to Bring in Geometry From Tigris

```{r, results = "hide"}
JndTbls$GEOID <- JndTbls$GEOID.x #Geoid.x was created in the previous join and needs to be renamed before joining it to the geometry
options(tigris_use_cache = TRUE)
blockgroup_Geom <- tigris::block_groups(state = 'OR', county = Counties, cb = TRUE) #we are using simplified geometry here, this can be changed by setting cb = FALSE, but takes a little bit longer to download
JndTblsSP <- sp::merge(x = blockgroup_Geom, JndTbls, by = 'GEOID') #Now we're using the GEOID to join the Census Data to the Geometry
```

> Now Let's Map Our Results!

```{r}
suppressPackageStartupMessages(require(leaflet))
suppressPackageStartupMessages(require(dplyr))
suppressPackageStartupMessages(require(leaflet.esri))

pop <- paste0(
  "<h3>","<b>", JndTblsSP$COUNTY.x,"</b>","</h3>",
  "<b>", JndTblsSP$CENSUS_TRACT.x, "</b>","<br>",
  "<b>","TOTAL POPULATION: ", prettyNum(JndTblsSP$TOTPOP, big.mark=","), " +/- ",JndTblsSP$B01003_001M,"</b>","<br>",
  "<b>","ADAPTIVE CAPACITY: ", round(100*(JndTblsSP$ADPTVCAPACITY), 1),"%","</b>","<br>",
  
  "<b><h4>SOCIOECONOMIC STATUS:<b></h4>",
  
  "<b>PCT LIVING IN POVERTY: </b>", round(100*(JndTblsSP$POV), 1), "%","<br>",
  "<b>PCT 16+ UNEMPLOYED: </b>", round(100*(JndTblsSP$UNEMP), 1), "%","<br>",
  "<b>PER CAPITA INCOME: </b>", "$", prettyNum(JndTblsSP$PCI, big.mark=","),"<br>",
  
  "<b><h4>LANGUAGE AND EDUCATION:<b></h4>",
  
  "<b>PCT OF POP 25+ LESS THAN 12th GRADE: </b>", round(100*(JndTblsSP$NOHSDP),1), "%","<br>",
  "<b>PCT NO ENGLISH: </b>", round(100*(JndTblsSP$LIMENG),1), "%","<br>",
  
  "<b><h4>DEMOGRAPHICS:</h4><b>",
  
  "<b>PCT UNDER AGE OF 17: </b>", round(100*(JndTblsSP$AGE17),1), "%","<br>",
  "<b>PCT 65+: </b>", round(100*(JndTblsSP$AGE65),1), "%","<br>",
  "<b>PCT DISABLED: </b>", round(100*(JndTblsSP$DISABL),1), "%","<br>",
  "<b>PCT CHLDRN LVNG IN SNGL PARENT HSHLDS: </b>", round(100*(JndTblsSP$SNGPNT),1), "%","<br>",
  
  "<b><h4>HOUSING AND TRANSPORTATION:</h4><b>",
  
  "<b>PCT LIVING IN MULTI-UNIT STRUCTURE: </b>", round(100*(JndTblsSP$MUNIT),1), "%","<br>",
  "<b>PCT MOBILE DWELLING: </b>", round(100*(JndTblsSP$MOBILE),1), "%","<br>",
  "<b>PCT LIVING IN CROWDED DWELLING: </b>", round(100*(JndTblsSP$CROWD),1), "%","<br>",
  "<b>PCT WITH NO VEHICLE ACCESS: </b>", round(100*(JndTblsSP$NOVEH),1), "%","<br>",
  "<b>PCT LIVING IN GROUP QUARTERS: </b>", round(100*(JndTblsSP$GROUPQ),1), "%","<br>",
  
  "<b><h4>RACIAL AND ETHNIC MAKEUP:<b></h4>",
  "<b>PCT MINORITY: </b>", round(100*(JndTblsSP$MINORITY),1), "%"
) #Here we're creating a popup for our interactive map, include whatever variables you want here!

BRBG <- RColorBrewer::brewer.pal(n = 11, name = "BrBG")

pal <- leaflet::colorQuantile(
  palette = BRBG,
  domain = JndTblsSP$ADPTVCAPACITY, n = 11, reverse = FALSE
) #Creating a Color Pallete, Feel free to choose whatever one you want, see the package Viridis for some cool options

myMap <- leaflet(data = JndTblsSP) %>% addTiles() %>% addPolygons(
    color = "#444444", 
    weight = 1, 
    smoothFactor = 0.5,
    opacity = 0.5, 
    fillOpacity = 0.5,
    fillColor = ~pal(ADPTVCAPACITY),
    highlightOptions = highlightOptions(color = "white", weight = 2, bringToFront = TRUE), 
    popup = pop, popupOptions = popupOptions(maxHeight = 250, maxWidth = 250, )) %>% addLegend("bottomright", 
    pal = pal, 
    values = JndTblsSP$ADPTVCAPACITY,
    title = "Adaptive Capacity Score",
    labFormat = labelFormat(prefix = ""),
    opacity = 0.75) 

myMap
```

Use This Function to Export The Shapefile (or csv) to a Folder of Your Choice!

```{r}
#Export to Shapefile
rgdal::writeOGR(obj = JndTblsSP, dsn = "C:/Users/anthony.holmes/Desktop/New folder/ADAPTIVE_CAPACITY_R", driver = "ESRI Shapefile", layer = "AdaptiveCapacityR", morphToESRI = FALSE)

#Or CSV (The benefit of this is that it preserves the field names, I found it better to export the geometry (blockgroup_Geom) alone and then join the data by GEOID in ArcGIS)
write.csv(x = JndTbls, "C:/Users/anthony.holmes/Desktop/New folder/ADAPTIVE_CAPACITY_R/AdaptiveCapacityR.csv")
```

#Licensing/Disclaimer:# 

THIS SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

THIS SOFTWARE HAS NOT BEEN PEER-REVIEWED AND IS SUBJECT TO REVISION. THE AUTHOR NOR THE NATURE CONSERVANCY MAKE ANY WARRANTY AS TO THE CURRENCY, COMPLETENESS, ACCURACY OR UTILITY OF THIS SOFTWARE. IT IS STRONGLY RECOMMENDED THAT CAREFUL ATTENTION BE PAID TO THE DOCUMENTATION ASSOCIATED WITH THIS SOFTWARE. THE AUTHOR NOR THE NATURE CONSERVANCY SHALL BE HELD LIABLE FOR IMPROPER OR INCORRECT USE OF THIS SOFTWARE OR ANY DATA PRODUCED HEREIN. ALL PARTIES UTILIZING THIS SOFTWARE MUST BE INFORMED OF THESE RESTRICTIONS. THE NATURE CONSERVANCY AND AUTHOR SHALL BE ACKNOWLEDGED CONTRIBUTORS TO ANY REPORTS OR OTHER PRODUCTS DERIVED FROM THIS SOFTWARE. 


And We're Done!


