# Analysis of US adult guardianship filing counts

This repository and document describes BuzzFeed News’ analysis of data on adult guardianship filings in US courts, supporting [this investigative article](https://buzzfeednews.com/article/heidiblake/conservatorship-investigation-free-britney-spears), published September 17, 2021. Please see that article to understand the full context.

Authors: Ethan Corey and Jeremy Singer-Vine. 

With special thanks to statisticians [Regina Nuzzo](https://www.reginanuzzo.com/) and [Michael Lavine](https://people.math.umass.edu/~lavine/) for their advice.

## Overview

For decades, experts and advocates have bemoaned the lack of consistent data on guardianships. So BuzzFeed News filed public records requests to all 50 states and the District of Columbia seeking detailed statistics about guardianships in their jurisdictions. Many ignored our requests, or pointed to published reports with limited figures.

Much of the data we did collect was unusable for our primary goal, which was to estimate the annual number of adult guardianship cases filed. Some states, for instance, did not separate guardianships of adults from those of minors. Other states’ figures were unusably incomplete or displayed extreme variation from year to year without any clear cause or explanation.

Ultimately, 23 jurisdictions (22 states and DC) had data fully usable for the main analyses presented here. Although this number represents fewer than half the states in the US, it appears to be the largest published sample of such filing counts to date. Another 12 states had data we could use for a sub-analysis. Below we describe the data collected, how we analyzed it, and the caveats associated with the estimates. This repository also contains [the data we used](data/) and [a computational notebook](notebooks/analyze-filing-counts.ipynb) written in Python.

## Data description

Our analysis focuses on the reporting years that ended in 2019, the most recent year not affected by the COVID-19 pandemic. (For some states this is calendar year 2019, while for others it is a fiscal year.) Where possible, we also collected data for other years, and include those figures in this repository so that other researchers may use them. Similarly, we have also collected and included other data points we are not using directly in the analysis. (The records we are not using, however, have not been as carefully vetted, and should be examined closely before being used.)

These records are presented in the [`data/raw/filing-counts.csv`](data/raw/filing-counts.csv) file, which contains the variables in the table below. Each row corresponds to a *categorization* of cases, disaggregated where possible. As a result, there are often multiple rows per state for any given year.

| Column | Description |
|--------|-------------|
| `state` | The name of the state (or the District of Columbia). |
| `year` | For fiscal years, this variable represents the ending calendar year. (All fiscal years in this particular dataset begin in July or later.)|
| `case_action` | Either `Initiated ONLY`, `Reopened ONLY`, `Initiated+Reopened`, or `Ambiguous`. See section below for details. |
| `case_type` | Either `Guardianship ONLY`, `Conservatorship ONLY`, `Joint G/C ONLY`, or `G+C+Joint`. See section below for details. |
| `age_group` | Either `Adults ONLY`, `Minors ONLY`, `Adults+Minors`, or `Ambiguous`. Ultimately, our estimates focus on counts for `Adults ONLY` — i.e., counts that cleanly separate out filings of adult guardianship cases. We also use counts for `Adults+Minors` in a sub-analysis, and record the other counts for completeness and public benefit. |
| `count` | The number of cases filed fitting these criteria.\* |
| `for_total` | This indicates whether a count can be used toward either the total adult filings count or total adults+minors filings count, based on BuzzFeed News' research. |

\* Two states, Virginia and Washington, provided data that BuzzFeed News determined were usable but which explicitly excluded major counties/courts: The Fairfax Circuit Court and Alexandria Circuit Court in Virginia and King County in Washington. For these states, BuzzFeed News applied an adjustment factor proportional to the adult populations in the missing jurisdictions. In Kentucky, the data provided referred to cases *disposed* (rather than filed); BuzzFeed News observed that this figure, however, is likely to track case filings relatively closely.

Additional state-specific details, including links to sources, can be found in [this document](https://docs.google.com/document/d/10IzTOxrUds3i-vzVbM0BxJ-HG9JSj__vykIhDZmuL50/edit).

## States analyzed

States provided various categorizations of cases, even among the jurisdictions we considered to be usable for our analysis. There are two particular distinctions worth highlighting:

For some jurisdictions, we are able to count both cases *initiated* and cases *reopened*, which comes closest to the count we want. Here we follow the Court Statistics Project's "[Guide to Statistical Reporting](https://www.courtstatistics.org/__data/assets/pdf_file/0026/23984/state-court-guide-to-statistical-reporting.pdf)," which calls for including both new cases and reopened cases in the total count of incoming cases. For others, we have only cases initiated, or the state’s figures do not make clear whether they include case reopenings.

The definition of guardianship also varies from state to state, with some states making clearer distinctions than others — in nomenclature or legal status — between various arrangements. The most common distinction is between managing the *personal affairs* of a ward and managing their *financial affairs*, what many states call “conservatorship.” In these states, guardians and conservators are often appointed in the same case. For some states, these distinctions are represented in the data; for others they are not. To avoid double-counting, we have excluded conservatorships where the states’ figures explicitly count those types of cases separately.  

Here’s how our data for each of the 23 jurisdictions differs in these respects:

| Jurisdiction         | Includes Reopenings?\* | Excludes Conservatorships? |
|----------------------|------------------------|----------------------------|
| Arizona              | ?                      | Y                          |
| Arkansas             | ?                      | N                          |
| Colorado             | Y                      | Y                          |
| Connecticut          | ?                      | N                          |
| District of Columbia | Y                      | N                          |
| Idaho                | N                      | Y                          |
| Kansas               | ?                      | Y                          |
| Kentucky             | Y                      | N                          |
| Massachusetts        | Y                      | Y                          |
| Michigan             | Y\*\*                  | Y                          |
| Minnesota            | N                      | N                          |
| Missouri             | ?                      | N                          |
| Nebraska             | ?                      | N                          |
| New Hampshire        | N                      | N                          |
| New Mexico           | Y                      | Y                          |
| North Dakota         | ?                      | Y                          |
| Ohio                 | Y                      | Y                          |
| Oregon               | Y                      | N                          |
| Pennsylvania         | Y                      | N                          |
| Utah                 | Y                      | Y                          |
| Vermont              | Y                      | N                          |
| Virginia             | N                      | Y                          |
| Washington           | N                      | N                          |


\* For jurisdictions that follow the Court Statistics Project reporting guidelines, we set `Includes Reopenings?` to `Y`, even if the relevant data source does not explicitly say the count includes reopenings. 

\*\* Michigan’s figures explicitly indicate that zero cases were reopened in the years we examined. 

Additionally, the various ways that jurisdictions *count* guardianship cases may differ, and may not capture the totality of relevant cases; in some states, for instance, guardians may be appointed in cases that are not formally labeled as such.
## Methodology

The primary goal of our analysis is to estimate a national count of adult guardianship filings in recent years. Extrapolating a national estimate from a relatively small sample can be difficult, so we sought advice from statisticians, and used their feedback to develop the methodology, which we sketch in general terms below and in more detail in [our computational notebook](notebooks/analyze-filing-counts.ipynb).

Our two main approaches involve calculating the annual __rate of filings per capita__ (specifically, per 100,000 adults) for each of our 23 jurisdictions, and then using those rates to estimate a range of possible counts for the remaining 28 states.

The first approach is to use those rates directly in a “bootstrap” analysis. Each “unknown” state has its adult population size multiplied by a per capita rate *randomly* selected from one of the 23 known rates, producing a hypothetical filing count for that state. Those filing counts are summed to produce a total and that total is added to the total for our 23 known jurisdictions. We repeat this procedure 10,000 times to produce a distribution of likely results.

Our second approach uses a statistical model to try smoothing out the choppiness inherent in our relatively small sample. We do so by fitting a negative binomial distribution to our observed data, and then using that distribution to generate, randomly, the hypothetical filing rates for the remaining 28 states. Whereas the bootstrap allows those rates only to be one of the 23 we have observed, this approach allows an infinite possibility of rates, but with their probabilities shaped by our observed data.

Although we don't have fully usable data from states that do not disambiguate between guardianships of adults and those of minors, we can still use this information to construct some bounds. There cannot be, for instance, more adult guardianship filings in these states than there are total adult+minor filings combined. And adult guardianship filings are likely to be some substantial portion of those filings. We conduct an analysis that tests the effect of various such proportions on our final estimates.

## Caveats and areas warranting further research

The approaches described above treat each state as a blank slate, without accounting for its particular characteristics.

We know, however, that various factors likely influence the number of guardianship cases filed in a given state. Some state laws make filing guardianship cases harder than others, for instance by requiring certain types of documentation. Others try to create alternatives to guardianship, such as through [Supported Decision-Making Agreements](https://www.americanbar.org/groups/law_aging/publications/bifocal/vol-41/volume-41-issue-1/where-states-stand-on-supported-decision-making/). We collected data on four such aspects of guardianship law for each jurisdiction, and compared them to the observed filing rates. Although it would be reasonable to assume these laws have some effect, we saw no clear pattern we could confidently incorporate into our estimates, perhaps due to our small sample size or perhaps because other factors outweigh these.

Another caveat to consider is whether the availability and quality of court data affect our estimates. Some states may report higher-quality, more comprehensive numbers than others — overcounting or undercounting the true number of cases filed. And *whether* usable numbers are available in the first place may be correlated, if indirectly, with a state’s frequency of guardianship filings.

Our computational notebook also explores the possibility that a state’s population may be inversely correlated with its per capita filing rate. We see a potential such trend, but it does not meet traditional levels of statistical significance (i.e., p<0.05), and we do not know whether the trend would generalize to the full set of jurisdictions. It is also unclear what would drive this correlation, if it did exist; perhaps large states may have more judicial bottlenecks, or tend to have different legal regimes. Nevertheless, our notebook explores what such a correlation would mean for our estimates.

Another area warranting further research is whether filing rates are spatially correlated — i.e., whether states near one another tend to have similar rates. We test this with neighboring states and find no statistically significant pattern we would feel comfortable generalizing to the full set of jurisdictions. A larger sample size or other approaches, however, may provide more definitive findings on this question.

## Reproducibility

Reproducing the notebook’s calculations requires having Python 3 installed on your computer, and also installing the Python libraries defined in this repository’s [`requirements.txt`](requirements.txt) file. If you already have Python 3 installed, you may automate this process by running `make venv` to install the requirements in a local virtual environment, and `make reproduce` to rerun the notebook. You can also open the notebook in Jupyter and run the notebook manually.

## Licensing

All code in this repository is available under the [MIT License](https://opensource.org/licenses/MIT). All data in this repository are available under the [Creative Commons Attribution 4.0 International (CC BY 4.0) license](https://creativecommons.org/licenses/by/4.0/), with the exception of the Census data, which is in the public domain.

## Contact

If you have any questions or comments about this repository, please contact Jeremy Singer-Vine at `jeremy.singer-vine@buzzfeed.com`.

Looking for more from BuzzFeed News? [Click here for a list of our open-sourced projects, data, and code](https://github.com/BuzzFeedNews/everything).
