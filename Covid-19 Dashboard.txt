--Checking both tables
Select *
From Project1..CovidDeaths
order by 3,4
Select *
From Project1..CovidVaccinations
order by 3,4

--The number of people dying who have reported being been infected (As of 03/22/2020 you have a 2.94% chance of dying after contracting Covid if you live in Afghanistan.)
Select location, date , total_cases,total_deaths, (cast(total_deaths as float)/cast(total_cases as float))*100 as DeathPercentage
From Project1..CovidDeaths
order by 1,2

--Looking for the same information but limiting search to only include locations with 'states' in the name
Select location, date , total_cases,total_deaths, (cast(total_deaths as float)/cast(total_cases as float))*100 as DeathPercentage
From Project1..CovidDeaths
Where location like '%states%'
order by 1,2

--Looking at total cases vs population (The percentage of the population that has gotten covid in the US)
Select location, date , total_cases,population, (cast(total_cases as float)/cast(population as float))*100 as InfectedPopulationPercentage
From Project1..CovidDeaths
Where location like '%states%'
order by 1,2

--Want to determine what countries have the highest infection rate compared to population (Andorra has the highest percentage of it's people infected)
Select location, population, MAX(total_cases) as HighestInfectionCount, Max(cast(total_cases as float)/cast(population as float))*100 as InfectedPopulationPercentage
From Project1..CovidDeaths
Group by location, population
order by InfectedPopulationPercentage desc --ordering based on infected population percentage from highest percentage to lowest

--Want to determine what countries have the highest occurence of death per population
Select location, population, MAX(total_deaths) as HighestDeathCount, Max(cast(total_deaths as float)/cast(population as float))*100 as DeathPercentage
From Project1..CovidDeaths
Group by location, population
order by DeathPercentage desc

--Showing countries with highest death count (US has the highest death count)
Select location, MAX(total_deaths) as TotalDeathCount
From Project1..CovidDeaths
Where continent is not null
Group by location
order by TotalDeathCount desc

--Showing continents with highest death count
Select continent, MAX(total_deaths) as TotalDeathCount
From Project1..CovidDeaths
Where continent is not null
Group by continent
order by TotalDeathCount desc

--Lets us know the death percentages across the world per day 
Select date , SUM(new_cases) as TotalCases, SUM(new_deaths) as TotalDeaths, SUM(cast(new_deaths as float))/SUM(cast(new_cases as float))*100 as DeathPercentage
From Project1..CovidDeaths
Where continent is not null 
group by date
order by 1,2

--Lets us know the total number of cases, deaths, and the death percentage for the entire world 
Select SUM(new_cases) as TotalCases, SUM(new_deaths) as TotalDeaths, SUM(cast(new_deaths as float))/SUM(cast(new_cases as float))*100 as DeathPercentage
From Project1..CovidDeaths
Where continent is not null 
order by 1,2

---Joining two tables together
Select *
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null
order by 2, 3

---Rolling count for each individual location , lets you know total amount of vaccination for each location,
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(cast(vac.new_vaccinations as float)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingNumOfPeopleVaccinated
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null
order by 2, 3

---Using CTE
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(cast(vac.new_vaccinations as float)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingNumOfPeopleVaccinated
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null) 
Select *, (RollingPeopleVaccinated/Population)*100 as PeopleVaccinated --Determining the percent of people vaccinated 
From PopvsVac

--Temp table
Drop Table if exists #PercentPopulationVaccinated
Create table #PercentPopulationVaccinated
(Continent nvarchar (255), location nvarchar (255), date datetime, population numeric, new_vaccinations numeric, RollingNumOfPeopleVaccinated numeric)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(cast(vac.new_vaccinations as float)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingNumOfPeopleVaccinated
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null 
Select *, (RollingNumOfPeopleVaccinated/Population)*100 
From #PercentPopulationVaccinated

--Creating view to store data for later visualizations
Create or alter view PercentageOfPopVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(cast(vac.new_vaccinations as float)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingNumOfPeopleVaccinated
From Project1..CovidDeaths dea
Join Project1..CovidVaccinations vac
on dea.location = vac.location 
and dea.date = vac.date
where dea.continent is not null
