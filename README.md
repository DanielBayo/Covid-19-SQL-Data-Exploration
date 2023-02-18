# COVID-19 SQL DATA EXPLORATION
This is an exploratory analysis of Covid-19 Data, in this analysis, I will try to answer some questions related to the Covid-19 in Nigeria. Some of the questions we will try to answer are:

1. What are the total Covid-19 Cases, New Cases, total Death and the population of each Country and daily?

2. What is the Probability that Someone with Covid-19 will die based on their location?

3. What Percentage of each Country''s population contracted Covid-19?

4. What is the Covid-19 Per Capital in Nigeria?


### Select the Data that we are going to be using

```sql
SELECT *
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
ORDER  BY 3,4
```

### Total cases vs  Total Deaths
```sql
SELECT location,
       date,
       total_cases,
       new_cases,
       total_deaths,
       population
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
ORDER  BY 1,
          2 
```

### This shows the likelihood of dying if you contract covid in your country
```sql
SELECT location,
       date,
       total_cases,
       total_deaths,
       Round(( total_deaths / total_cases ) * 100, 2) AS DeathPercent
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
       AND location LIKE '%Nigeria%'
ORDER  BY 1,
          2 
```

### Looking at the Total Cases vs Population,Shows what percentage of the population got Covid
```sql
SELECT location,
       date,
       total_cases,
       population,
       Round(( total_cases / population ) * 100, 2) AS PopulationPercent
FROM   portfolioproject..coviddeaths
WHERE  location LIKE '%Nigeria%'
       AND continent IS NOT NULL
ORDER  BY 1,
          2 
```

### Looking at Countries  with Highest Infection Rate compared to Population
```sql
SELECT location,
       population,
       Max(total_cases)                              AS HighestCases,
       Round(Max(total_cases / population) * 100, 2) AS
       PercentPopulationInfected
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
--where location like '%Nigeria%'
GROUP  BY location,
          population
ORDER  BY percentpopulationinfected DESC 
```

### Showing Countries with the Highest Death Count per population
```sql
SELECT location,
       population,
       Max(Cast(total_deaths AS INT)) AS TotalDeathCount
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
--where location like '%Nigeria%'
GROUP  BY location,
          population
ORDER  BY totaldeathcount DESC 
```


## LET''S ANALYZE THE DATA BY CONTINENT

### Showing continent with the highest cases per population
```sql
SELECT   location,
         population,
         Max(total_cases)                           AS totalcases,
         Round((Max(total_cases)/population)*100,2) AS percentinfected
FROM     portfolioproject..coviddeaths
WHERE    continent IS NULL
AND      location NOT LIKE 'World%'
AND      location NOT LIKE 'European%'
AND      location NOT LIKE 'International%'
         --where location like '%Nigeria%'
GROUP BY location,
         population
ORDER BY percentinfected DESC 
```

### Showing continent with the highest death count per population
```sql
SELECT location,
       population,
       Max(Cast(total_deaths AS INT))                                  AS
       TotalDeathCount,
       Round(( Max(Cast(total_deaths AS INT)) / population ) * 100, 2) AS
       PercentDead
FROM   portfolioproject..coviddeaths
WHERE  continent IS NULL
       AND location NOT LIKE 'World%'
       AND location NOT LIKE 'European%'
       AND location NOT LIKE 'International%'
--where location like '%Nigeria%'
GROUP  BY location,
          population
ORDER  BY percentdead DESC 
```



## Analyzing by the Global Number

### This shows the number of infected people and dead people globally on Daily Basis
```sql
SELECT date,
       Sum(new_cases)                                                    AS
       GlobalCasebyDay,
       Sum(Cast(new_deaths AS INT))                                      AS
       GlobalDeathbyDay,
       Round(( Sum(Cast(new_deaths AS INT)) / Sum(new_cases) ) * 100, 2) AS
       GlobalDeathPercent
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
       AND new_cases IS NOT NULL
GROUP  BY date
ORDER  BY 1,
          2 
```


### This shows the total cases,the total deaths
```sql
SELECT Sum(new_cases)                                                    AS
       GlobalCasebyDay,
       Sum(Cast(new_deaths AS INT))                                      AS
       GlobalDeathbyDay,
       Round(( Sum(Cast(new_deaths AS INT)) / Sum(new_cases) ) * 100, 2) AS
       GlobalDeathPercent
FROM   portfolioproject..coviddeaths
WHERE  continent IS NOT NULL
       AND new_cases IS NOT NULL
--Group by date
ORDER  BY 1,
          2 
```

### Looking at Total Population Vs Vaccinations
```sql
SELECT dea.continent,
       dea.location,
       dea.date,
       dea.population,
       vac.new_vaccinations,
       Sum(Cast(vac.new_vaccinations AS INT))
         OVER (
           partition BY dea.location
           ORDER BY dea.location, dea.date) AS CummVac
FROM   portfolioproject..coviddeaths dea
       JOIN portfolioproject..covidvaccination vac
         ON dea.location = vac.location
            AND dea.date = vac.date
WHERE  dea.continent IS NOT NULL
       AND vac.new_vaccinations IS NOT NULL
ORDER  BY 1,
          2,
          3 
```


### USE CTE
```sql
WITH popvsvac (continent, location, date, population, new_vaccinations, cummvac)
     AS (SELECT dea.continent,
                dea.location,
                dea.date,
                dea.population,
                vac.new_vaccinations,
                Sum(Cast(vac.new_vaccinations AS INT))
                  OVER (
                    partition BY dea.location
                    ORDER BY dea.location, dea.date) AS CummVac
         FROM   portfolioproject..coviddeaths dea
                JOIN portfolioproject..covidvaccination vac
                  ON dea.location = vac.location
                     AND dea.date = vac.date
         WHERE  dea.continent IS NOT NULL
                AND vac.new_vaccinations IS NOT NULL
        --order by 2,3
        )
SELECT *,
       Round(( cummvac / population ) * 100, 2)AS PercentVaccinated
FROM   popvsvac
--where location like '%Nigeria%'
```


## TEMP TABLE
```sql
DROP TABLE IF EXISTS #percentvac
CREATE TABLE #percentvac
             (
                          continent        nvarchar(255),
                          location         nvarchar(255),
                                           date datetime,
                          population       numeric,
                          new_vaccinations numeric,
                          cummvac          numeric
             )INSERT INTO #percentvac
SELECT   dea.continent,
         dea.location,
         dea.date,
         dea.population,
         vac.new_vaccinations,
         Sum(Cast(vac.new_vaccinations AS INT)) OVER (partition BY dea.location ORDER BY dea.location,dea.date) AS CummVac
FROM     portfolioproject..coviddeaths dea
JOIN     portfolioproject..covidvaccination vac
ON       dea.location=vac.location
AND      dea.date=vac.date
WHERE    dea.continent IS NOT NULL
AND      vac.new_vaccinations IS NOT NULL
--order by 2,3SELECT *,
       Round((cummvac/population)*100,2)AS PercentVaccinated
FROM   #percentvac
```

### creating view to store data for later vizualizations
```sql
DROP VIEW IF EXISTS percentagepopulationvaccinated 
CREATE VIEW percentagepopulationvaccinated AS 
SELECT 
  dea.continent, 
  dea.location, 
  dea.DATE,
  dea.population,
  vac.new_vaccinations,
  sum(CAST(vac.new_vaccinations AS int))
OVER (PARTITION BY dea.location ORDER BY dea.location, dea.DATE) AS cummvac 
FROM portfolioproject..coviddeaths dea 
JOIN portfolioproject..covidvaccination vac ON
dea.location=vac.location AND dea.DATE=vac.DATE 
WHERE dea.continent IS NOT NULL
AND vac.new_vaccinations IS NOT NULL
--order by 2,3
```

### Percentage of Population that has been vaccinated
```sql
SELECT *
FROM Percentagepopulationvaccinated
```
