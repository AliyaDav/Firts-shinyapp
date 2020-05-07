---
title: "Final_project"
author: "Aliya Davletshina"
date: "5/5/2019"
output:
  html_document:
    df_print: paged
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, out.width = "100%", fig.align = "center")
```

```{r message = FALSE, warning = FALSE, comment = '>'}
library(tidyverse)
library(Hmisc)
library(magrittr)
library(base)
library(stringr)
library(ggplot2)
library(ggthemes)
library(ggfortify)
library(reshape2)
library(plyr)
library(RColorBrewer)
library(viridis)
library(rworldmap)
library(readxl)

fileurls <- c(
  "https://docs.google.com/spreadsheet/pub?key=t9GL1nIZdtxszJbjKErN2Hg&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=0AkBd6lyS3EmpdHo5S0J6ekhVOF9QaVhod05QSGV4T3c&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=0AkBd6lyS3EmpdFJTUEVleTM0cE5jTnlTMk41ajBGclE&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=phAwcNAVuyj2tPLxKvvnNPA&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=0ArfEDsV3bBwCdE4tekJPYkR4WmJqYTRPWjc3OTl4WUE&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=0ArfEDsV3bBwCcGhBd2NOQVZ1eWowNVpSNjl1c3lRSWc&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=t1YAVXUoD3iJKy2mSq2Padw&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=0AkBd6lyS3EmpdEhTN2hlZ05ZczVwZDdVZlF5cUxJb2c&output=xlsx",
  "https://docs.google.com/spreadsheet/pub?key=phAwcNAVuyj3XYThRy0yJMA&output=xlsx", 
  "https://docs.google.com/spreadsheet/pub?key=phAwcNAVuyj0XOoBL_n5tAQ&output=xlsx"
)

if(!file.exists("./data")) {dir.create("./data")}
var_names <- c("aid", "income", "edu_cost", "life_exp",
               "sanit_access", "child_mortality", "poverty", 
               "prime_school", "health_spend", "pop")

get_clean <- function (url, var) {
  download.file(url, destfile = "./tmp.xlsx",  mode = "wb")
  data <- read_excel("./tmp.xlsx")
  names(data)[1] <- "country"
  data <- data %>% 
    gather(key = "year", value = !!var, -country, na.rm = TRUE, convert = TRUE)
  data
}

all_data <- map2(fileurls, var_names, get_clean)
new_data = all_data[[4]][,-3]
for (i in seq_along(all_data)) {
  new_data <- left_join(new_data, all_data[[i]], by = c("year", "country"))
}
new_data$year = as.integer(new_data$year)

# write.csv(new_data, file = "all_data.csv", sep = ",", row.names = FALSE)

# adding status for countries based on income 
new_data = new_data %>% mutate(
  status = case_when(
    income <= 1000 ~ "low",
    income <= 4000 & income > 1000 ~ "lower_middle",
    income <= 12000 & income > 4000 ~ "upper_middle",
    income > 12000 ~ "high",
    TRUE ~ as.character(income)
  )
)
new_data$status = factor(new_data$status, 
                         levels = c("high","upper_middle","lower_middle","low"), 
                         labels = c("high","upper_middle","lower_middle","low"))
all_data = new_data
```

### World data 

The world is changing very fast, some countries experiencing unprecedented economic growth, others remaining at the same position they were 50 years ago. Today we live in a world which is more diversified then ever and the gap between the poorest and the wealthiest seems to keep growing.  The question is, should we look for the ways to reduce this gap and promote economic development in the poorest countries or should we stand aside? 
  
The first meeting of a newly established Development Assistance Group (DAG) took place in in Washington, D.C. (U.S.A.) on 9–11 March 1960. A primary concern was to keep track of the financial flows to developing countries and for that reason the term ODA (official development aid) was introduced. Now it is defined as official foreign financial aid with the main objective to promote economic development and welfare in developing countries. Throughout more than half a century data has been collected and now it is possible to look and see whether ODA is an effective measure that assists developing countries or it has little impact on the economic situation.  
  
The data used in this analysis is taken from gapminder.org and it consists of the following variables:

```{r message = FALSE, warning = FALSE, comment = '>'}
names(all_data)
```

1. aid – Aid received per person (current US\$)
Net official development assistance (ODA) per capita consists of disbursements of loans made on concessional terms (net of repayments of principal) and grants by official agencies of the members of the Development Assistance Committee (DAC), by multilateral institutions, and by non-DAC countries to promote economic development and welfare in countries and territories in the DAC list of ODA recipients; and is calculated by dividing net ODA received by the midyear population estimate. It includes loans with a grant element of at least 25 percent (calculated at a rate of discount of 10 percent).
2. income - Income per cap - GDP/capita (US$, inflation-adjusted)
Gross Domestic Product per capita in constant 2000 US\$. The inflation but not the differences in the cost of living between countries has been taken into account.
3. edu_cost - Expenditure per student, primary (% of GDP per person).
Public expenditure per student is the public current spending on education divided by the total number of students by level, as a percentage of GDP per capita. Public expenditure (current and capital) includes government spending on educational institutions (both public and private), education administration as well as subsidies for private entities (students/households and other private entities). 
4. life_exp - Life expectancy (years)
	The average number of years a newborn child would live if current mortality patterns were to stay the same.
5. sanit_access – Proportion of the population using improved sanitation facilities, total
	Access to improved sanitation facilities refers to the percentage of the population with at least adequate access to excreta disposal facilities that can effectively prevent human, animal, and insect contact with excreta. Improved facilities range from simple but protected pit latrines to flush toilets with a sewerage connection.
6. child_mortality - Child mortality (0-5 year-olds dying per 1,000 born)
	The probability that a child born in a specific year will die before reaching the age of five if subject to current age-specific mortality rates. Expressed as a rate per 1,000 live births.
7. health_spend - Total health spending (% of GDP)
8. prime_school - Primary completion rate, total (% of relevant age group)
	Primary completion rate is the percentage of students completing the last year of primary school. It is calculated by taking the total number of students in the last grade of primary school, minus the number of repeaters in that grade, divided by the total number of children of official graduation age. The ratio can exceed 100% due to over-aged and under-aged children who enter primary school late/early and/or repeat grades. 
9. poverty - Extreme poverty (% people below $1.25 a day)
	Population below 1.25 dollar a day is the percentage of the population living on less than 1.25 dollar a day at 2005 international prices. 
10. pop - Population, total
11. status - Status of a country according the level of income per capita. "Low" for under \$1,000, "lower-middle" for \$1,000-\$4,000, "upper-middle" for \$4,000-$12,000, "high" for above \$12,000.

The analysis consist of two parts. Firstly, I've taken the data for all the countried, compared the progress in different status groups by looking at how various economic indicators were changing over time and tried to find out if there is any difference in correlation between the variables among different status groups. I've also studied which status group and which particular countries have received most aid over the period of 50 years. Secondly, I've chosen 3 regions -- Latin America, West Asia and Africa to study in more detail. The main goal was to find out if there is any significant difference between the countries which received most ODA and the others.  
The link to my shiny app is [here](https://aliyaadav.shinyapps.io/Economic_development/)


```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
theme_set(theme_classic(base_size = 12))

# How income has changed over time?
attach(all_data)
all_data %>% 
      dplyr::filter(year >= 1960) %>% 
      dplyr::select(year, status) %>% 
      drop_na() %>%
      dplyr::group_by(year) %>% plyr::count() %>% 
      ggplot(aes(x = year, y = freq, fill = status)) + geom_col(position = "fill") +
      theme(axis.text.x = element_text(size = 12, angle = 45, hjust = 1),
            axis.text.y = element_text(size = 12),
            plot.title = element_text(lineheight = 1, hjust = 0.5, size = 12, face = "bold")) +
      scale_fill_viridis_d() +
      labs(y = "Proportion") + xlab(NULL)+
      labs(title = " The number of high-income countries had been \n increasing till 1980")
```

The chart shows that the number of high-income countries had been increasing till 1980, little changed over the next 20 years and in the beginning of the 21st century the number of low-income countries has slightly dropped. 
  
```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
all_data %>% 
      filter(year %in% 1960:2011 & !is.na(status) & !is.na(income)) %>%
      dplyr::group_by(year, status) %>% 
      dplyr::mutate(avg_inc = mean(income)) %>%
      ggplot(aes(x = year, y = avg_inc, group = status)) + 
      geom_line(aes(color = status)) +
      theme(axis.text.x = element_text(angle = 45, hjust = 1),
            plot.title = element_text(lineheight = 1, hjust = 0.5, size = 12, face = "bold"),
            legend.position = "none") +
      facet_wrap(~ status, scales = "free_y") + 
      scale_color_viridis_d()+
      labs(title = "Change in average income level \n among different groups", 
           y = "Average income") + xlab(NULL)
```

The graphs show how average income per capita has changed in different status groups. There was a huge progress in high-income countries whereas income in upper-middle group has been decreasing. The other groups show fluctuations which indicates instability. Overall, we may conclude that the gap between high-income and low-income countries has increased even further. 
  
```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

sPDF <- getMap()  
map_income_1960 = joinCountryData2Map(all_data[year == "1960",], joinCode = "NAME", 
                                      nameJoinColumn = "country")
inc_1960 = mapCountryData(map_income_1960, nameColumnToPlot = "income", 
               catMethod = "pretty",
               colourPalette = rev(viridis(256)),
               mapTitle = "Income per cap in 1960")
do.call( addMapLegend, c(inc_1960, legendLabels = "all" ))

map_income_2011 = joinCountryData2Map(all_data[year == "2010",], joinCode = "NAME", 
                                 nameJoinColumn = "country")
inc2011 = mapCountryData(map_income_2011, nameColumnToPlot = "income", 
               catMethod = "pretty",
               colourPalette = rev(viridis(256)),
               mapTitle = "Income per cap in 2010")
do.call( addMapLegend, c(inc2011, legendLabels = "all" ))

```

The maps for 1960 and 2010 illustrate that little has changed in terms of the leading countries -- North America, Europe, Australia and Japan are still among the wealthiest. It is worth pointing out that the highest average income per capita has increased 5 times compared to 1960, which means that on average, we expect a country to have 5 times bigger GDP in 2010 than in 1960. 

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
# Let's look at how life expectancy has changed over the years 

library(maps)
library(mapdata)
library(mapproj)
library(rworldmap)

attach(new_data)
map_lifeEx_1960 = joinCountryData2Map(new_data[year == 1960,], joinCode = "NAME", 
                                      nameJoinColumn = "country")
map_1960 = mapCountryData(map_lifeEx_1960, nameColumnToPlot = "life_exp", catMethod = "pretty",
                           colourPalette = viridis(256, option = "A",direction = -1),
                           mapTitle = "Life expectancy in 1960", addLegend = F)
do.call( addMapLegend, c(map_1960, legendLabels = "all" ))
map_lifeEx_2010 = joinCountryData2Map(new_data[year == 2010,], joinCode = "NAME", 
                                      nameJoinColumn = "country")
map_2010 = mapCountryData(map_lifeEx_2010, nameColumnToPlot = "life_exp", catMethod = "pretty",
                          colourPalette = viridis(256, option = "A",direction = -1), 
                          mapTitle = "Life expectancy in 2010")
do.call( addMapLegend, c(map_2010, legendLabels = "all" ))
```

The maps show that the upper boundary of life expentancy has increased by 15 years while the lower boundary only by 5 years. Overall, life expentancy has increased everywhere but African countries are still falling benind.  
  
```{r message = FALSE, warning = FALSE, comment = '>'}
#How are variables correlated with each other in different income groups?

noNA = new_data %>% filter(status == "low") %>%
  select(-country,-status,-pop,-child_mortality,-year) %>%  
  drop_na() %>% cor()
corrplot::corrplot(noNA, order = "AOE", col = viridis(100), 
                        tl.col = "black", tl.cex = 0.8, cl.cex = 0.75, number.cex = 0.8, tl.srt=45,
                        addCoef.col = "black", type = "upper", diag = FALSE)

noNA = new_data %>% filter(status == "lower_middle") %>% drop_na() %>%
  select(-country,-status,-pop,-child_mortality,-year) %>% cor()
corrplot::corrplot(noNA, order = "AOE", col = viridis(100), 
                        tl.col = "black", tl.cex = 0.8, cl.cex = 0.75, number.cex = 0.8, tl.srt=45,
                        addCoef.col = "black", type = "upper", diag = FALSE)
noNA = new_data %>% filter(status == "upper_middle") %>% drop_na() %>%
  select(-country,-status,-pop,-child_mortality,-year) %>% cor()
corrplot::corrplot(noNA, order = "AOE", col = viridis(100), 
                        tl.col = "black", tl.cex = 0.8, cl.cex = 0.75, number.cex = 0.8, tl.srt=45,
                        addCoef.col = "black", type = "upper", diag = FALSE)
```
  
Comparing these three matrixis it is clear that the correlation between the variables is higher in low-income countries than in other status groups. This might indicate that the more developed a country is, the more complicated is the relation between all the indicators because many other macroeconomic factors (interest rate, tax rate, exchange rate) become significant. 

```{r message = FALSE, warning = FALSE, comment = '>'}
attach(all_data)

avg_aid = all_data %>% 
  filter(year %in% 1960:2010 & !is.na(status) & !is.na(aid)) %>%
  dplyr::group_by(year, status) %>% 
  dplyr::mutate(avg_aid = mean(aid))
avg_aid %>%
    ggplot(aes(year, avg_aid, group = status)) + 
    geom_line(aes(color = status), lwd = 1, alpha = 0.8) +
    theme(axis.text.x = element_text(angle = 45, hjust = 1),
          plot.title = element_text(lineheight = 1, hjust = 0.5, size = 12, face = "bold"), 
          legend.position = "none") +
    facet_wrap(~ status) + 
    scale_color_viridis_d()+
    labs(title = "Foreign aid distribution",
         y = "Average aid per capita") + xlab(NULL)

most_aid = all_data %>% filter(pop > 1000000) %>% dplyr::count(country, wt = aid, sort = T) %>% 
  top_n(20) %>% arrange(n) # countries received most aid
most_aid$country = factor(most_aid$country, levels = most_aid$country)
most_aid %>% 
    ggplot(aes(country,n, fill = country)) + geom_col() + 
    coord_flip() + 
    theme(legend.position = "none", 
          plot.title = element_text(lineheight = 1, hjust = 0.5, size = 12, face = "bold")) + 
    viridis::scale_fill_viridis(discrete = T) + 
    labs(title = "Countries which received most aid per cap \n from 1960 to 2010") +
      xlab(NULL) + ylab("Cumulative aid")
```

It may seem contradictory but the graphs show that most aid was received by high-income countries whereas low-income group received the least. My hypothesis is that foreign aid is given to a country with an intention to develop a new market and countries with higher income need less assistance to become potential consumer markets for the goods of a donor country.

### Data by region 

```{r message = FALSE, warning = FALSE, comment = '>'}
income_col = function(df) {
  df %>% 
    dplyr::filter(year >= 1960) %>% 
    dplyr::select(year, status) %>% 
    drop_na() %>%
    dplyr::group_by(year) %>% plyr::count() %>% 
    ggplot(aes(x = year, y = freq, fill = status)) +
    geom_col(position = "fill") +
    scale_fill_viridis_d() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5, size = 12)) +
    labs(title = "Changes within income groups \n in 1960-2011", 
         y = "Proportion")+ xlab(NULL)
}

df_for_gg = function(region_df) {
  region_df$country <- as.factor(region_df$country)  # for right ordering of the dumbells
  region_df %>% 
    select(country, year, income) %>% 
    drop_na(income) %>% 
    spread(key = year, value = income) %>%
    mutate(differ = round((`2011` - `1995`)*100/`1995`,2))
}

gg = function(df) {
  ggplot(df, aes(x = df$`1995`, 
                    xend = df$`2011`, y=country, group=country)) + 
    ggalt::geom_dumbbell(color="#23888EFF", 
                         size=0.75, 
                         point.colour.l="#31688EFF") + 
    geom_text(aes(x = (max(df$`2011`, na.rm = T)), label = paste0(df$differ, "%")),
              color = ifelse(df$differ< 0, "red", "black"), hjust = -0.5, size=2) +
    labs(x=NULL, y=NULL, title="Income per cap: 1995 vs 2011") +
    theme(plot.title = element_text(size = 1, hjust=0.5, face="bold"),
          plot.background=element_rect(fill="white"),
          panel.background=element_rect(fill="white"),
          panel.grid.minor=element_blank(),
          panel.grid.major.y=element_blank(),
          panel.grid.major.x=element_line(size = 0.5, color = "grey"),
          axis.ticks=element_blank(),
          legend.position="top",
          panel.border=element_blank(), 
          axis.text.y = element_text(lineheight = 1, size = 8)) +
    xlim(min(df$`1995`, na.rm = T), max(df[!is.na(df$`1995`),]$`2011`, na.rm = T)+3000)
}

df_for_barchart = function(region) {
  data_dev = region %>%
    filter(year == "2009") %>% 
    drop_na(income) %>%
    mutate(income_z = round((income - mean(income))/sd(income), 2),
           type = ifelse(income_z < 0, "below", "above")) 
  data_dev = data_dev[order(data_dev$income_z),]  
  data_dev = data_dev %>%
    mutate(country = factor(country, levels = unique(country)))
  data_dev
}

# Diverging Barcharts
divchart = function(df) {
  ggplot(df, aes(x=country, y=income_z)) +
  geom_bar(stat='identity', aes(fill=df$type), width=.5)  +
  scale_fill_manual(name="Income",
                    labels = c("Above average", "Below average"),
                    values = c("above"="#B4DE2CFF", "below"="#440154FF")) +
  labs(subtitle="Year 2009",
       title= "Income differences", y ="Normalized income")+
    theme(plot.title = element_text(lineheight = 1, face="bold"),
          axis.text.x = element_text(lineheight = 1, size = 8)) +
  coord_flip() + xlab(NULL)
}

aid_by_groups = function(df) {
  df %>% 
    filter(year %in% c(1975:2010)) %>%
    drop_na(aid, status) %>%
    group_by(year, status) %>% 
    dplyr:: mutate(avg_aid = mean(aid)) %>% 
    ggplot(aes(x = year, y = avg_aid, group = status)) +
    geom_line(aes(color = status), lwd = 1, alpha = 0.8) + 
    scale_colour_viridis_d() +
    labs(title = "Aid received by different \n status groups in 1975-2010",
         y = "Average aid per capita") + xlab(NULL) + 
    theme(plot.title = element_text(lineheight = 1, hjust = 0.5, face="bold"))
}
aid_col_sum = function(df) {
  df %>% 
    filter(year %in% c(1975:2010)) %>%
    drop_na(aid, status) %>%
    group_by(country) %>%
    dplyr:: mutate(sum_aid = sum(aid)) %>%
    select(country, sum_aid) %>%
    distinct() %>%
    arrange(desc(sum_aid)) %>%
    ggplot(aes(country, sum_aid, group = country)) + 
    geom_col(aes(fill = country)) + 
    scale_fill_viridis_d() +
    coord_flip()+
    labs(title = "Total aid received in 1975-2010") + ylab("Cumulative aid") +
    xlab(NULL) + 
    theme(plot.title = element_text(lineheight = 1, hjust = 0.5, face="bold"), 
          legend.position = "none")
}

most_aid_table = function(df) {
  most_aid = df %>% 
    filter(year %in% c(1975:2010)) %>%
    drop_na(aid, status) %>%
    group_by(country) %>%
    dplyr:: mutate(sum_aid = sum(aid)) %>%
    select(country, sum_aid) %>%
    distinct() %>%
    arrange(desc(sum_aid))
  (most_aid = most_aid[1:6,])
}
top6_aid = function(df) {
  df %>% 
    filter(country %in% most_aid_table(df)$country & year %in% c(1975:2010)) %>%
    ggplot(aes(x = year)) +
    geom_line(aes(y = aid, color = "Aid received"), color = "#B4DE2CFF", lwd = 1) +
    geom_line(aes(y = income, color = "Income per cap"), color = "#440154FF", lwd = 1) +
    facet_wrap(~country) +
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5, face = "bold"), 
          legend.position = "right") +
    labs(title = "Aid and income in 1975-2010", 
         y = "Aid per capita") + xlab(NULL)
}
top6_health = function(df) {
  df %>% 
    filter(country %in% most_aid_table(df)$country & year %in% c(1995:2010)) %>%
    ggplot(aes(x = year, y = health_spend)) +
    geom_line(aes(color = country), lwd = 1, alpha = 0.8) + 
    scale_colour_viridis_d() +
    facet_wrap(~country) + 
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5, face = "bold"), 
          plot.subtitle = element_text(hjust = 0.5),
          legend.position = "none") +
    labs(title = "Total health spending (% of GDP)", 
         subtitle = "Years 1995-2010") + xlab(NULL) + ylab(NULL)
}
top6_sanit = function(df) {
  df %>% 
    filter(country %in% most_aid_table(df)$country & year %in% c(1990:2010)) %>%
    ggplot(aes(x = year, y = sanit_access)) +
    geom_line(aes(color = country), lwd = 1, alpha = 0.8) + 
    scale_colour_viridis_d() +
    facet_wrap(~country) + 
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5, face ="bold"), 
          plot.subtitle = element_text(hjust = 0.5),
          legend.position = "none") +
    labs(title = "Proportion of the population using improved \n sanitation facilities, total", 
         subtitle = "Years 1970-2011") + xlab(NULL)+ ylab(NULL)
}
top6_child = function(df) {
  df %>% 
    filter(country %in% most_aid_table(df)$country & year %in% c(1980:2015)) %>%
    ggplot(aes(x = year, y = child_mortality)) +
    geom_line(aes(color = country), lwd = 1, alpha = 0.8) + 
    scale_colour_viridis_d() +
    facet_wrap(~country) + 
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5,  face ="bold"),
          plot.subtitle = element_text(hjust = 0.5),
          legend.position = "none") +
    labs(title = "Child mortality rate",
         subtitle =  "Years 1980-2015") + xlab(NULL)+ ylab(NULL)
}
top6_life = function(df) {
  df %>% 
    filter(country %in% most_aid_table(df)$country & year %in% c(1970:2011)) %>%
    ggplot(aes(x = year, y = life_exp)) +
    geom_line(aes(color = country), lwd = 1, alpha = 0.8) + 
    scale_colour_viridis_d() +
    facet_wrap(~country) + 
    theme(axis.text.x = element_text(angle = 45, hjust = 1), 
          plot.title = element_text(lineheight = 1, hjust = 0.5,  face ="bold"),
          plot.subtitle = element_text(hjust = 0.5),
          legend.position = "none") +
    labs(title = "Life expentancy",
         subtitle =  "Years 1980-2015") + xlab(NULL)+ ylab(NULL)
}

```

In order to find out which countries  received most aid and showed most progress, I've looked at three regions and here I start with **Latin America**. 

```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}
latin_countries = c("Belize", "Costa Rica", "El Salvador", "Guatemala", 
                    "Honduras", "Mexico", "Nicaragua", "Panama", "Argentina", "Bolivia", "Brazil", "Honduras", "Panama", "Trinidad and Tobago", "Jamaica","Bahamas",
                    "Chile", "Colombia", "Ecuador", "French Guiana", "Guyana",
                    "Paraguay", "Peru", "Suriname", "Uruguay", "Venezuela", 
                    "Cuba", "Dominican Republic", "Haiti")


LA_data = filter(new_data, country %in% latin_countries)
# attach(LA_data)
income_col(LA_data)
```

Historically, the majority of the countries in Latin America belonged to lower-income group, but by 2011 the proportion of lower- and upper-middle income counties has almost equalized. 

```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}
gg(df_for_gg(LA_data))
```

The graph shows how income level has changed in each country. The highest growth was in Trinidad and Tobago whereas Haiti performed the worst. But one should not forget to take into consideration a fatal earthquake of 2010 which led to hundreds of thousands dealths and had a devastating effect on Haiti's economy.  

```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}

divchart(df_for_barchart(LA_data))
```

The chart shows how income per capita in each country is different from average income per capita in the region. Normally, we would expect half of the countries be on right and the other half on the left, but this chart shows a clear disproportion which means than a couple of countries produce more than a half of the GDP of the region.  

```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}

aid_by_groups(LA_data)
```

In Latin America most ODA was received by low-income countries and there is a spike after the economic crisis of 2008. 
```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}

aid_col_sum(LA_data)
```

Now we look at 6 countries which receved most aid and see if there is a correlation between the amount of aid received and the progress countries made. 

```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}
top6_aid(LA_data)
```

On 6 graphs above dark blue line indicates income per capita and green shows aid per capita. The only country that seems to benefit is Belize, although it is not evident if the aid indeed had any effect on economic growth.  


```{r message = FALSE, warning = FALSE, comment = '>',include=TRUE, results='hide'}
top6_health(LA_data)
top6_child(LA_data)
top6_life(LA_data)
top6_sanit(LA_data)
```

It is interesting that Bolivia, even with the least of 6 countries access to sanition, showed a most progress in terms of increased life expectancy and decreased child noratlity rate. The other thing which stroke me is a spike in child mortality rate in Honduras. After a little research it turned out there was a hurricane in 1998, and "the President of Honduras estimated that Mitch set back 50 years of economic development" of the country.[^1] [^1]:<https://en.wikipedia.org/wiki/Effects_of_Hurricane_Mitch_in_Honduras>  

  
The next region is **West Asia**. 
```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
west_asia = c("Afghanistan", "Armenia", "Azerbaijan", "Bahrain", "Cyprus", "Georgia", "Iran", "Iraq", "Israel", "Jordan", "Kuwait", "Lebanon", "Oman", "Qatar", "Saudi Arabia", "Syria", "Turkey", "United Arab Emirates", "West Bank and Gaza", "Yemen")
west_asia = filter(new_data, country %in% west_asia)
#attach(west_asia)
income_col(west_asia)
```

There were no high-income countries in West Asia till 1974 -- the year then oil prices soared. Just in 6 years a quarter of the region turned to high-income countries. However, after 1990 things got worse and by 2011 there was almost equal proportion of each status group. 


```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
gg(df_for_gg(west_asia))
```


```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

divchart(df_for_barchart(west_asia))
```

In 2009 three main oil exporters remained the wealthiest countries in the region with Israel (which was experiencing a rapid economic growth) catching up. 

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

aid_by_groups(west_asia)
```

The graph shows no clear pattern in ODA flows.

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

aid_col_sum(west_asia)
```

Jordan, Israel and Bahrain each received at least twice as much as their nearest neighbours. 

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

top6_aid(west_asia)
```

From the previous graph we see that Jordan received more aid than Israel but it never showed such an advancement as Israel or Oman. 

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
top6_health(west_asia)
top6_child(west_asia)
top6_life(west_asia)
top6_sanit(west_asia)
```

The country that definitely stands out is Oman. Steep curves of child mortality rate and life expentancy indicate seem to indicate a considerble improve of the quality of life.  
  
  
The next region is **Africa**.

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}
library(gapminder)
africa = filter(new_data, country %in% gapminder$country[gapminder$continent == "Africa"])
#attach(africa)
income_col(africa)
```

The difference from other regions is rather striking. There was little change in 50 years and three fourths of the continent is still low-income countries. 

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

gg(df_for_gg(africa))
```

Africa is the only continent where the difference among the countries is so striking: income in Zimbabwe has decreased by 32% while Equatorial Guinea showed an increase by 1254%. The rest remained almost at the same level. Such an increase in income in Equatorial Guinea was due to oil and gas exploitation which began in the 1990s.

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

divchart(df_for_barchart(africa))
```

This chart is another prove of a vast diversity in the region. Jist a quarter of the continent has income above average and the rest are within one standard deviation from the mean.

```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

aid_by_groups(africa)

```

The graph shows that historically (from 1975) ODA was mostly to upper-income countries but it dropped for all the groups after 1996 and began to increase again in the 21st century. Probably it is linked to Millennium Development Goals set by UN in 2000. 


```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

aid_col_sum(africa)
top6_aid(africa)
```

The graph shows that the only country where income has increased is Botswana. In other countries like Comoros, Guinea-Bissau and Mauritania the aid practically made up a considerable part of GDP.  
```{r message = FALSE, warning = FALSE, comment = '>', include=TRUE, results='hide'}

top6_health(africa)
top6_child(africa)
top6_life(africa)
top6_sanit(africa)

```

There is a high increase of health expenditures in Botswana from 1995 which resulted in increasing life expectancy in the begining of the 21st century. It turns out that the cause of poor health in the coutry in the first place was HIV/AIDS epidemics which hit Botswana in the 1990s. Even though income in the majority of the countries has not changed, nevertheless there are some signs of improvement -- lower child mortality rate and higher life expectancy. 

Overall, the data shows no direct relation between ODA and economic development in low-income countries. Further research is needed to find out the form of ODA in each particular case, find best practices of how ODA benefitted the countries and see whether the same approach could be implemented in other countries. 