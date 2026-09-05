# Geog4/6300: Lab 0–Tornadoes and social vulnerability


*Name: Grayson Smith*

This “lab” assignment provides an opportunity to learn the basic
mechanics of Github classroom, loading data, and doing some basic data
manipulation using the tools in the tidyverse. Through this lab, you
will identify counties with the highest levels of social vulnerability
that also had high numbers of severe tornadoes (rated F3 and higher)
during this study period.

This lab assesses the following learning standards:

1.  Classify variables by measurement type (nominal, ordinal, interval,
    ratio) and justify those classifications with reference to the
    characteristics of the data. (Task 2)
2.  Load tabular and spatial data into a code-based environment from
    multiple sources, including local files, remote APIs, and spatial
    file formats. (Task 1)
3.  Filter, aggregate, and transform datasets using grouping and summary
    operations to answer specific analytical questions. (Task 3, 4, and
    5)
4.  Join multiple datasets using appropriate join strategies and explain
    how different join types affect the resulting output. (Task 6 and 7)
5.  Use Git and Github to create and share project materials in a
    repository format. (Lab submission)

## Loading the data

We’ll be combining two datasets for this assignment: NOAA’s database of
tornadoes from 1950-2026 (so far) and the CDC’s Social Vulnerability
Index (SVI): https://www.atsdr.cdc.gov/place-health/php/svi/index.html

You can load the tornado data (`noaa_stormevent_tornado_2026_08.csv` in
the folder `data/stormevents/`) using the `read_csv` function. The SVI
data is located at `data/SVI_2022_US_county.csv`. Load the tornado
dataset into an object called `tornadoes` and the SVI dataset into an
object called `svi`.

(Side note: you can see how we downloaded this data directly from NWS in
the “stormdata_download” script in the data folder.)

**Task 1:** *Load the tornado data and the SVI data using `read_csv`.*

``` r
# Your code goes here.

svi <- read_csv("data/SVI_2022_US_county.csv")
```

    Rows: 3144 Columns: 158
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr   (7): ST, STATE, ST_ABBR, STCNTY, COUNTY, FIPS, LOCATION
    dbl (151): AREA_SQMI, E_TOTPOP, M_TOTPOP, E_HU, M_HU, E_HH, M_HH, E_POV150, ...

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
tornadoes <- read_csv("data/stormevents/noaa_stormevent_tornado_2026_08.csv")
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 80996 Columns: 52
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (25): STATE, MONTH_NAME, EVENT_TYPE, CZ_TYPE, CZ_NAME, cty_fips, WFO, BE...
    dbl (24): BEGIN_YEARMONTH, BEGIN_DAY, BEGIN_TIME, END_YEARMONTH, END_DAY, EN...
    lgl  (3): MAGNITUDE_TYPE, FLOOD_CAUSE, CATEGORY

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#view(tornadoes)
```

**Task 2:** *Pick two variables from either dataset that use different
levels of measurement (nominal, ordinal, interval or ratio). Explain
which level best describes each variable and why. Also identify one
variable that would be more efficiently read as a factor, and explain
why.*

*Response*: The two variables that I chose to analyze where the CZ_name
(County Name) from the Tornado database and E_UNEMP (Number of
unemployeed people) in the SVI database. I think that the CZ_name data
are nominal because it is using categorical labeling data with no
natural order or rank. E_UNEMP data appear to be ratio data, as it is
quantitative data that has a natural rank and order and has a true zero
value (i.e., 0 people being unemployed truly means there is nobody
unemployed).

Concerning a variable that might be better read as a factor, it appears
that the TOR_F_SCALE – which is currently read as a character despite
likely being an ordinal variable in nature – might be better read as a
factor. This is likely the case because this would allow us to first
order the information from F0 to F5 (in the order that makes the most
sense) and perhaps to also group that data together according to that
rank (i.e., all the F0s could be grouped together). This would help us
take the currently character (and often repeating) F-scale data and put
it into a format where its ordering is better recognized and repeating
values are better grouped together.

## Filtering and/or summarizing the data

Open up the tornado data frame so you can look at it. There’s multiple
variables here related to the timing of the tornado, its location, its
magnitude, and the number of injuries and fatalities. You can open the
“Storm-Data-Bulk-csv_Format” pdf in the data folder to learn more about
them.

For this part of the lab, you want to count the number of tornadoes
rated F3 or higher (the `TOR_F_SCALE` variable) within each county (the
`cty_fips` and `CZ_NAME` variables). Your resulting data frame should
have one row per county and state combination, with a variable that
stores the total number of severe tornadoes.

To do so, you’ll need to:

1.  Filter the `TOR_F_SCALE` variable to only include observations rated
    F3, F4, or F5.
2.  Use `group_by()` to group the data by the county and state fips
    codes and names (`STATE_FIPS`, `STATE`, `cty_fips`, and `CZ_NAME`).
3.  Use `summarise()` to count the number of these severe tornadoes
    within these groups.

**Task 3:** *Filter the storm data to include tornadoes rated F3 or
higher and then count the number of events by county and state.*

``` r
# Your code goes here.

violent_tors <- tornadoes %>%
  filter(str_detect(TOR_F_SCALE,"F3") | str_detect(TOR_F_SCALE,"F4") | str_detect(TOR_F_SCALE,"F5"))
#view(violent_tors) #used this to make sure it worked

grouped_violent_tors <- violent_tors %>%
  group_by(STATE_FIPS, STATE, cty_fips,CZ_NAME)
#view(grouped_violent_tors)

vio_tors_by_cty_st <- grouped_violent_tors %>%
  group_by(cty_fips,STATE_FIPS,CZ_NAME,STATE) %>%
  summarise(tor_count = n())
```

    `summarise()` has regrouped the output.
    ℹ Summaries were computed grouped by cty_fips, STATE_FIPS, CZ_NAME, and STATE.
    ℹ Output is grouped by cty_fips, STATE_FIPS, and CZ_NAME.
    ℹ Use `summarise(.groups = "drop_last")` to silence this message.
    ℹ Use `summarise(.by = c(cty_fips, STATE_FIPS, CZ_NAME, STATE))` for
      per-operation grouping (`?dplyr::dplyr_by`) instead.

``` r
#view(vio_tors_by_cty_st)
```

You also want to filter the social vulnerability data so that it only
includes counties with a population over 25,000 people. This allows us
to only track counties with a moderate or large population. Use the SVI
Documentation pdf in the data folder to determine which variable this
is, recognizing that you want the **estimate** and not the margin of
error (MOE).

Filter the data below.

**Task 4:** *Filter the SVI data to include counties with a population
of 25,000 or more.*

``` r
# Your code goes here.

revised_counties_pop <- svi %>%
  filter(E_TOTPOP > 25000)
```

Let’s add one more filter based on social vulnerability. When you look
at the documentation there are a number of “dummy” variables (0/1) that
flag counties that are at or above the 90th percentile for a number of
factors. For example, `F_DISABL` flags counties in the top 10% for
percentage of persons with a disability. There are also cumulative
flags: `F_THEME2` sums all the flagged variables within the “Household
Characteristics” category.

Pick one of those flags. It could be for a specific variable or a
cumulative one. Your selected flag should have a plausible connection to
tornado preparedness, evacuation, sheltering, recovery, or access to
warnings.

Filter the dataset you created in Task 4 so that it only includes
counties you are considering vulnerable. Then list that flagged variable
and explain how you chose it and (if the flag is cumulative) how you
picked a numeric cutoff.

**Task 5:** *Filter the data from Task 4 to only vulnerable counties and
explain your decision.*

``` r
# Your code goes here.
mobile_home_filter <- revised_counties_pop %>%
  filter(F_MOBILE == 1)
```

Describe your variable, why you chose it, and (if applicable) why you
chose the cutoff you did.

*Response*: I chose the flag used to delineate those counties with a
percentage of mobile homes that is in the 90th (or above) percentile. I
chose this individual variable for a number of reasons, chief among
those the fact that mobile homes are one of the most dangerous places to
be during a tornadic event – with extensive literature indicating that
the chance of fatalities is much higher in mobile (and manufactured)
homes than in other structures. Further, the percentage of mobile homes
in a county might also be a good proxy for a lower socioeconomic status
in that county (given that mobile homes are typically a more affordable
alternative to other, more expensive \[but safer\] types of housing),
which might also increase the vulnerability of that county to tornadoes
and their effects (especially in the recovery stage after the storm). I
also considered using the F_THEME4 flag for Housing Type/Transporation
characteristics to consider other types of vulnerable housing situations
(like people living in crowded homes); however, given the very
well-established correlation between mobile homes and tornado effects, I
decided to go with that flag to better measure counties that fall within
that well-known vulnerable situation. Further research could certainly
focus on a broader look at a wider breadth of potentially vulnerable
housing situations; however, this might even be more effectively done by
singularly examining vulnerability per structure (like using the flag
for crowded houses) to determine vulnerability per housing type rather
than examining potentially vulnerable housing types as a whole (since
using whole-sale housing vulnerability might not give as helpful
information on what types of structures are actually most vulnerable).

## Connecting the data

Next you’ll need to join these tornado counts to SVI population data
using the `inner_join()` function. To do so you’ll need to have two
fields with the same name in each dataset. The FIPS codes are in both
datasets, but they have different names (`FIPS` and `cty_fips`). The
following code creates a new variable called `cty_fips` in the SVI data
using `rename()`.

**NOTE:** You may need to change the name of the population data below
to match the object you used for the SVI data.

``` r
revised_counties_pop_corr <- mobile_home_filter %>%
  rename(cty_fips = FIPS)
#view(revised_counties_pop)
```

Call the function above to rename your data. Now you’re ready to join
the data.

**Task 6:** *Use `inner_join()` to connect the filtered tornado data to
your county data from Task 5.*

``` r
# Your code goes here.

# Found out that I might need to change the state names in the SVI data to be all caps so it can better join with the tornado data, so I am going to try and use "toupper."

revised_count_pop_cor_C <- revised_counties_pop_corr %>%
  mutate(STATE = toupper(STATE))
svi_tornado_joined <- inner_join(revised_count_pop_cor_C,vio_tors_by_cty_st)
```

    Joining with `by = join_by(STATE, cty_fips)`

**Task 7:** *Find some documentation on the `inner_join()` function
online or using help in R. Describe how it works, and explain how the
results would have been different if you used `full_join()` instead.*

I used the “?” feature in R to find documentation on the differences in
these two functions:

?inner_join() ?full_join()

Response: The inner_join() function works by only keeping the
observations from the original dataset that have a matching
characteristic with the secondary dataset (i.e., keeping the
observations in df “x” that coincide directly with a characteristic in
df “y”). Conversely, the full_join() function works by keeping all the
observations in both df x and df y. In this case, this would have made a
much larger dataset with all of the observations (i.e., all of the
counties) instead of the very limited ones we obtained with our
inner_join(). The R documentation also states that the inner_join() is
generally not ideal, given that it easily excludes observations that
might be needed; however, it was certainly serviceable in this case
given that we were looking at a very specific research question with
this data join.

## Answering your research question

**Task 8:** *Open up your joined dataset. Which three vulnerable
counties have the highest number of severe tornadoes and how many were
in each? Are there cities or other notable geographic features located
in these counties?*

The three most vulnerable counties – according to my analysis – were
Walker County (AL), De Soto County (LA), and Copiah County (MS). There
were 11 significant tornadoes in Walker County, 9 in De Soto Parish, and
7 in Copiah County. The county seat of Walker County is Jasper, AL, and
although there are no major apparent features in this county, it is home
to over 65,000 people. De Soto Parish’s county seat is Mansfield, and
this county has over 27,000 people living therein (despite having no
other major features). Finally, Walker County’s city seat is Hazlehurst,
and although this county did not have any major features either, it
still has a meaningful population of over 27,000. Together, although
each of these counties did not appear to have major features (aside from
Interstate(s)), they did have meaningful populations that – according to
this analysis – would be at great risk of harm to life and property from
the very strong tornadoes that plague the Southeast.

## Challenge question

Find a peer-reviewed article published in the last ten years that uses
this NWS/NOAA tornado dataset. What research question was it trying to
answer? What methods were used? What are the most notable findings? Give
the full citation and a summary of at least 100 words below.

The article I examined, written by Nouri et al. (2021), looked at the
relationships between human factors (like population density changes and
observational technology improvements) and climate teleconnection
factors (like the El Niño Southern Oscillation \[ENSO\]) in explaining
changes in tornado frequency from 1950-2018 in specific regions (i.e.,
Tornado Alley, Dixie Alley, and elsewhere) of the US. The authors
answered this question by first obtaining their data, utilizing the
Storm Data dataset (as used in this lab) for their tornado data,
population density from the US Census Bureau, and climate oscillation
data from time series provided by NOAA’s National Centers for
Environmental Prediction (NCEP). From there, they performed a principal
component analysis that (in largely over-simplified terms) appears to
have allowed them to ascertain which variables (“modes”) had the
greatest influence on variability of tornado data in the studied
regions. Using that data, they then performed a “wavelet analysis” that
allowed them to ascertain potential cycles present within the data
(seemiingly especially those that are not evident from simple analysis
alone). Finally, they then trained two models (which are, again, greatly
oversimplfied here) to determine how various factors explained tornado
frequency variance: one that only examined anthropogenic influences on
tornado frequency (i.e., changes in popualtion density and better
observing systems) and another that combined these human-driven changes
with climate oscillation variables. Interestingly enough, they used R
(and other programs) to conduct their modeling, analysis, and
visualization.

Using the methods described (albeit in very simple terms) above, the
authors reached a number of fascinating and notable conclusions. In
terms of human impacts, they found that changes in population density
were the most responsible for tornado frequency variance in the
Southeast (i.e., Dixie Alley) where as observational network changes
(i.e., due to the introduction of moden radar technology) was most
responsible for variance in the Central US (i.e., Tornado Alley). They
found mixed results outside of these two major regions in terms of human
forcings on reported tornado frequency. In terms of climate, they found
a wide range of results that suggested that numerous climate
oscillations might have influences on environmental characteristics that
then make tornadoes (especially tornado outbreaks) more common in the
studied regions. Specifically, they found that climate oscillations like
ENSO and the North Atlantic Oscillation (NAO) had greater affect on
Tornado Alley than Dixie Alley, whereas other oscillations (like the
Arctic Oscillation) influence variability in Dixie Alley. The mechanisms
for exactly how each of these oscillations affected these areas was not
fully discussed (or completely understood by the author of this
summary), much of this variability appears to be rooted in jet stream
dynamics (and resultant changes in wind shear necessary for tornado
environments) and local wind changes (like a change in wind flow that
either increases flow from moisture sources \[like the Gulf of Mexico\]
or decreases that flow \[as happens during El Niņo in Tornado Alley\]).

Together, they found that human and climate factors could meaningfully,
but certainly not completely, explain variance in tornado frequency –
suggesting (at least to me) that future work is necessary in this area
and that, very simply, tornadoes (and tornado outbreaks) are phenomena
dependent on very specific environmental circumstances that may not be
fully dependent on a set of extenuating circumstances in some cases
(i.e., that tornadoes are inherently especially chaotic, localized, and
difficult to forecast and thus might be difficult to very confidently
tie to any single \[or even collection of\] human or climate
connection\[s\]).

Citation: Nouri, N., Devineni, N., Were, V., & Khanbilvardi, R. (2021).
Explaining the trends and variability in the United States tornado
records using climate teleconnections and shifts in observational
practices. Scientific Reports, 11(1).
https://doi.org/10.1038/s41598-021-81143-5

## Final submission stuff

**Disclosure of assistance:** Besides class materials, what other
sources of assistance did you use while completing this lab? These can
include input from classmates, relevant material identified through web
searches (e.g., Stack Overflow), or assistance from ChatGPT or other AI
tools. How did these sources support your own learning in completing
this lab?

*Response*: One of my primary resources when completing this assignment
was the data transformation with dplyr cheatsheet that we discussed in
class. This was then supplemented by some input from my classmates at
the end of Friday’s (4 September) class session and some help from
Google Gemini on trouble shooting various issues – esepcially on helping
me discover errors in Task 3 that were making my code ineffective and
taking quite some time to fix. Both Gemini and my classmates were
supportive of my learning in that they helped me to realize small errors
in my coding that I can now avoid in the future. Gemini was also
especially helpful at providing me additional tools that I had not yet
gained from class, such as the use of %in% in Task 3 (although that
certainly might have been covered and I simply missed it), that I also
plan to use on future labs to create more effective code. Further, since
I also made a point of asking Gemini to explain why the strategies it
provided were more effective than those I attempted, this tool also
allowed me to become more familiar with how each function worked (rather
than just learning what words to type in to get a result).

In terms of the dplyr cheatsheet, this resource primairly helped me by
providing a basic guide as to what functions to chose for each task –
with great assistance from our in-class discussion on those tools – and
what parameters needed to be entered in those functions to make them
produce the desired outcome. Together, these outside resources served a
dual purpose: making my work more effective and timely while also
providing me with additional information that I can use to become a
better R programmer in subsequent labs and other activities.

**Lab reflection:** How do you feel about the work you did on this lab?
Was it easy, moderate, or hard? What are the biggest things you learned
by completing it?

I thought that this lab was of moderate difficulty, primarily because
this was my first major foray into notable R programming since my
Biostats class (and likely more extensive already than most of my
experiences in that class). Despite that moderate difficulty, I did
think that I learned quite a lot. Specifically, I learned more about the
use of the data wrangling functions we discussed in class – including
how to better use them (like with the %in% in Task 3 \[as mentioned in
the “Disclosure of assistance” section\]) to manipulate data. Further, I
also learned how to troubleshoot problems – including with the
assistance of others – and how to interpret the data that R outputs
(even if it is not always clear). Finally, also I learned (or
re-learned) some of the small sytax features of R (like the need for the
“==” instead of “=” in Task 5) that are necessary for effective coding.
Put together, although each of these lessons are relatively minor, I am
hoping that the culmination of these lessons will help me continue
building a strong foundation in R that I can use for the rest of this
class (and hopefully into my future endeavours).

That’s it! When you’re done with this lab, use the Render command in
Quarto to create a GitHub markdown document. Then push it to GitHub
using the procedure outlined in this week’s videos.
