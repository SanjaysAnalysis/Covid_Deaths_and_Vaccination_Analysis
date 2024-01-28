# Covid_Deaths_and_Vaccination_Analysis

SELECT * FROM Covid_Deaths
Order by location, date

SELECT * FROM Covid_Vaccinations
Order by location, date

SELECT * 
FROM Projects..Covid_Deaths 
where continent is not null
order by 3,4

Select Location, date, total_cases, new_cases, total_deaths, Population
from Covid_Deaths
where continent is not null
Order by 1,2

--Total cases vs Total deaths	

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as Death_percentage
from Covid_Deaths
where location like '%India%'
and continent is not null
order by 1,2

--Total Cases vs Population

select location, date, total_cases, population, (total_cases/population)*100 as Infection_percentage
from Covid_Deaths
where location like '%India%'
and continent is not null
order by 1, 2

--Highest infected countries compared to their population

select location, population, max(total_cases) as Highest_Infection_Count, max((total_cases/population))*100 as Percentage_of_popu_infected
from Covid_Deaths
where continent is not null
group by location, population
order by Percentage_of_popu_infected desc

--datewise
select location, population,date, max(total_cases) as Highest_Infection_Count, max((total_cases/population))*100 as Percentage_of_popu_infected
from Covid_Deaths
where continent is not null
group by location, population, date
order by Percentage_of_popu_infected desc

--Highest death count compared to their population

select Location, max(cast(total_deaths as int)) as Total_death_count 
from Covid_Deaths
where continent is not null
group by location
order by total_death_count desc

--Highest death count based on continent

select Location, max(cast(total_deaths as int)) as Total_death_count 
from Covid_Deaths
where continent is null
and location in ('Africa','Asia', 'Europe', 'North America', 'South America', 'Oceania')
group by location
order by total_death_count desc

/*
select continent, max(cast(total_deaths as int)) as Total_death_count 
from Covid_Deaths
where continent is not null
group by continent
order by total_death_count desc
*/

--cases on Global level

select date, sum(new_cases) as Total_Cases, sum(cast(new_deaths as float)) as Total_deaths, 
sum(cast(new_deaths as float))/nullif(sum(new_cases),0) *100 as Death_Percentage
from Covid_Deaths
group by date
order by 3


select sum(new_cases) as Total_Cases, sum(cast(new_deaths as float)) as Total_deaths, 
sum(cast(new_deaths as float))/nullif(sum(new_cases),0) * 100 as Death_Percentage
from Covid_Deaths
where continent is not null
order by 1, 2


--Vaccination data

select * from Covid_Vaccinations

select * 
from Covid_Deaths d
join Covid_Vaccinations v on d.location = v.location and d.date = v.date
;

--population vs people vaccinated

with popu_vs_vacc as (
select d.continent, d.location, d.date, d.population, v.new_vaccinations, 
sum(convert(float, v.new_vaccinations)) over (partition by d.location order by d.location, d.date) as total_vaccinated_by_date
from Covid_Deaths d 
join Covid_Vaccinations v 
on d.location = v.location and d.date = v.date
where d.continent is not null
)
select *, (total_vaccinated_by_date/population) * 100 as percentage_vaccinated
from popu_vs_vacc

--Views for analysis

create view population_vaccinated_by_percentage 
as (
select d.continent, d.location, d.date, d.population, v.new_vaccinations, 
sum(convert(float, v.new_vaccinations)) over (partition by d.location order by d.location, d.date) as total_vaccinated_by_date
from Covid_Deaths d 
join Covid_Vaccinations v 
on d.location = v.location and d.date = v.date
where d.continent is not null
)

create view Global_numbers as 
select Location, max(cast(total_deaths as int)) as Total_death_count 
from Covid_Deaths
where continent is null
and location in ('Africa','Asia', 'Europe', 'North America', 'South America', 'Oceania')
group by location
order by Total_death_count desc
