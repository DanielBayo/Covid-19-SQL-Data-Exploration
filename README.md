# COVID-19 SQL DATA EXPLORATION
This is an exploratory analysis of Covid-19 Data, in this analysis, I will try to answer some questions related to the Covid-19 in Nigeria. Some of the questions we will try to answer are:

1. What are the total Covid-19 Cases, New Cases, total Death and the population of each Country and daily?

2. What is the Probability that Someone with Covid-19 will die based on their location?

3. What Percentage of each Country''s population contracted Covid-19?

4. What is the Covid-19 Per Capital in Nigeria?


### Select the Data that we are going to be using

```sql
SELECT *
FROM portfolioproject..CovidDeaths
where continent is not null
order by 3,4
```

--SELECT *
--FROM portfolioproject..Covidvaccination
--order by 3,4

### Total cases vs  Total Deaths
```sql
SELECT 
Location,
date,
total_cases,
new_cases,
total_deaths,
population
FROM portfolioproject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2
```

### This shows the likelihood of dying if you contract covid in your country
```sql
SELECT 
Location,
date,
total_cases,
total_deaths,
round((total_deaths/total_cases)*100,2) as DeathPercent
FROM portfolioproject..CovidDeaths
where continent is not null and location like '%Nigeria%'
order by 1,2
```

### Looking at the Total Cases vs Population,Shows what percentage of the population got Covid
```sql
SELECT 
Location,
date,
total_cases,population,round((total_cases/population)*100,2) as PopulationPercent
FROM portfolioproject..CovidDeaths
where location like '%Nigeria%' and continent is not null
order by 1,2
```

### Looking at Countries  with Highest Infection Rate compared to Population
```sql
SELECT 
Location,
population,
max(total_cases) as HighestCases,
round(max(total_cases/population)*100,2) as PercentPopulationInfected
FROM portfolioproject..CovidDeaths
where continent is not null
--where location like '%Nigeria%'
Group by location,population
order by PercentPopulationInfected desc
```

### Showing Countries with the Highest Death Count per population
```sql
SELECT 
Location,
population,
max(cast(total_deaths as int)) as TotalDeathCount
FROM portfolioproject..CovidDeaths
where continent is not null
--where location like '%Nigeria%'
Group by location,population
order by TotalDeathCount desc
```


## LET''S ANALYZE THE DATA BY CONTINENT

### Showing continent with the highest cases per population
```sql
SELECT 
location,
population,
max(total_cases) as TotalCases,
round((max(total_cases)/population)*100,2) as PercentInfected
FROM portfolioproject..CovidDeaths
where continent is null and location not like 'World%' and location not like 'European%' and location not like 'International%'
--where location like '%Nigeria%'
Group by location,population
order by PercentInfected desc
```

### Showing continent with the highest death count per population
```sql
SELECT 
location,
population,
max(cast(total_deaths as int)) as TotalDeathCount,
round((max(cast(total_deaths as int))/population)*100,2) as PercentDead
FROM portfolioproject..CovidDeaths
where continent is null and location not like 'World%' and location not like 'European%' and location not like 'International%'
--where location like '%Nigeria%'
Group by location,population
order by PercentDead desc
```



## Analyzing by the Global Number

### This shows the number of infected people and dead people globally on Daily Basis
```sql
select 
date,
sum(new_cases) as GlobalCasebyDay,
sum(cast(new_deaths as int)) as GlobalDeathbyDay,
round((sum(cast(new_deaths as int))/sum(new_cases))*100,2) as GlobalDeathPercent
from portfolioproject..CovidDeaths
where continent is not null and new_cases is not null
Group by date
order by 1,2
```


### This shows the total cases,the total deaths
```sql
select 
sum(new_cases) as GlobalCasebyDay,
sum(cast(new_deaths as int)) as GlobalDeathbyDay,
round((sum(cast(new_deaths as int))/sum(new_cases))*100,2) as GlobalDeathPercent
from portfolioproject..CovidDeaths
where continent is not null and new_cases is not null
--Group by date
order by 1,2
```

### Looking at Total Population Vs Vaccinations
```sql
select 
dea.continent,
dea.location,
dea.date,
dea.population,
vac.new_vaccinations,
sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as CummVac
from portfolioproject..CovidDeaths dea
join portfolioproject..Covidvaccination vac
on dea.location=vac.location
and dea.date=vac.date
where dea.continent is not null and vac.new_vaccinations is not null
order by 1,2,3


--USE CTE
with popvsvac (continent,location,date,population,new_vaccinations,Cummvac)
as
(
select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
,sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as CummVac
from portfolioproject..CovidDeaths dea
join portfolioproject..Covidvaccination vac
on dea.location=vac.location
and dea.date=vac.date
where dea.continent is not null and vac.new_vaccinations is not null
--order by 2,3
)
select *,round((Cummvac/population)*100,2)as PercentVaccinated
from popvsvac
--where location like '%Nigeria%'


--TEMP TABLE
Drop table if exists #percentvac
create table #Percentvac
(
continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
Cummvac numeric
)
insert into #Percentvac
select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
,sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as CummVac
from portfolioproject..CovidDeaths dea
join portfolioproject..Covidvaccination vac
on dea.location=vac.location
and dea.date=vac.date
where dea.continent is not null and vac.new_vaccinations is not null
--order by 2,3
select *,round((Cummvac/population)*100,2)as PercentVaccinated
from #Percentvac

--creating view to store data for later vizualizations
drop view if exists Percentagepopulationvaccinated
create view Percentagepopulationvaccinated as
select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
,sum(cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location,dea.date) as CummVac
from portfolioproject..CovidDeaths dea
join portfolioproject..Covidvaccination vac
on dea.location=vac.location
and dea.date=vac.date
where dea.continent is not null and vac.new_vaccinations is not null
--order by 2,3

select *
from Percentagepopulationvaccinated
