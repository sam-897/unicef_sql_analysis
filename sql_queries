-- BASIC DEMOGRAPHIC DETAILS AND ECONOMIC DETAILS
-- highest population
CREATE OR REPLACE VIEW top_ten_populations AS
SELECT country,
       population_total_2021,
       rank
FROM (
    SELECT country,
           population_total_2021,
           RANK() OVER (ORDER BY TO_NUMBER(REPLACE(population_total_2021, ',', ''), '9999999')::INTEGER DESC) AS rank
    FROM public.demographics
) population_ranking
WHERE rank <= 10;
SELECT * FROM top_ten_populations;

-- lowest population
SELECT country,
       population_total_2021
    FROM public.demographics
ORDER BY TO_NUMBER(REPLACE(population_total_2021, ',', ''), '9999999')::INTEGER 
	 ;
     

-- understanding the composition of population, young or aging and the total economic burden on the working population
--young population
WITH data AS (
    SELECT 
        country,
        population_total_2021, 
        TO_NUMBER(NULLIF(children_proportion, '-'), '9999999')::INTEGER AS children_dependency,
        TO_NUMBER(NULLIF(senior_citzens_proportion, '-'), '9999999')::INTEGER AS old_dependency 
    FROM 
        public.demographics
)
SELECT 
    country,
    population_total_2021,
    children_dependency,
    old_dependency
FROM 
    data
WHERE 
    children_dependency > 40 
    AND old_dependency < 20
    ORDER BY children_dependency desc;

--ageing population
WITH data AS (
    SELECT 
        country,
        population_total_2021, 
        TO_NUMBER(NULLIF(children_proportion, '-'), '9999999')::INTEGER AS children_dependency,
        TO_NUMBER(NULLIF(senior_citzens_proportion, '-'), '9999999')::INTEGER AS old_dependency 
    FROM 
        public.demographics
)
SELECT 
    country,
    population_total_2021,
    children_dependency,
    old_dependency
FROM 
    data
WHERE 
    children_dependency < 30 
    AND old_dependency > 25
ORDER BY old_dependency desc;

--urban composition of population
SELECT country,population_total_2021,urban_population_percentage,migration_rate_2021
FROM public.demographics
WHERE urban_population_percentage NOT LIKE '-'
ORDER BY TO_NUMBER(NULLIF(urban_population_percentage, '-'), '9999999')::INTEGER desc;

--gdp per capita
SELECT country, 
    TO_NUMBER(REPLACE(NULLIF(gdp_per_capita, '-'), ',', ''), '9999999')::INTEGER AS gdp_per_capita
FROM 
    public.social_protection
WHERE 
    gdp_per_capita NOT LIKE '-'
ORDER BY 
    gdp_per_capita DESC;



--Gross domestic product and income inequality
WITH economic_details AS 
	(
	SELECT d.country as country,
	TO_NUMBER(REPLACE(NULLIF(d.population_total_2021, '-'), ',', ''), '9999999')::INTEGER as population,
	TO_NUMBER(REPLACE(NULLIF(s.gdp_per_capita, '-'), ',', ''), '9999999')::INTEGER as gdp_per_capita,
	TO_NUMBER(NULLIF(s.gini, '-'), '9999999')::INTEGER as gini,
	TO_NUMBER(NULLIF(s.palma, '-'), '9999999')::INTEGER as palma
	FROM public.demographics d INNER JOIN public.social_protection s
	ON d.country=s.country
	)
SELECT country,population, gdp_per_capita,gini,palma
FROM economic_details
WHERE 
population IS NOT NULL
AND gdp_per_capita IS NOT NULL
AND gini IS NOT NULL
AND palma IS NOT NULL
ORDER BY gini desc,palma desc;

--countries with gdp belowa verage and population above average
WITH economic_details AS (
    SELECT 
        d.country AS country,
        TO_NUMBER(REPLACE(NULLIF(d.population_total_2021, '-'), ',', ''), '9999999')::INTEGER AS population,
        TO_NUMBER(REPLACE(NULLIF(s.gdp_per_capita, '-'), ',', ''), '9999999')::INTEGER AS gdp_per_capita,
        TO_NUMBER(NULLIF(s.gini, '-'), '9999999')::INTEGER AS gini,
        TO_NUMBER(NULLIF(s.palma, '-'), '9999999')::INTEGER AS palma
    FROM 
        public.demographics d 
    INNER JOIN 
        public.social_protection s ON d.country = s.country
),
avg_population AS (
    SELECT AVG(population) AS avg_population FROM economic_details
),
avg_gdp AS (
    SELECT AVG(gdp_per_capita) AS avg_gdp FROM economic_details
)
SELECT 
    e.country, 
    e.population, 
    e.gdp_per_capita, 
    e.gini, 
    e.palma
FROM 
    economic_details e
CROSS JOIN 
    avg_population
CROSS JOIN 
    avg_gdp
WHERE 
    e.population > avg_population.avg_population
    AND e.gdp_per_capita < avg_gdp.avg_gdp
ORDER BY e.gdp_per_capita asc,e.population desc;

-- gdp alone below average
WITH economic_details AS (
    SELECT 
        d.country AS country,
        TO_NUMBER(REPLACE(NULLIF(d.population_total_2021, '-'), ',', ''), '9999999')::INTEGER AS population,
        TO_NUMBER(REPLACE(NULLIF(s.gdp_per_capita, '-'), ',', ''), '9999999')::INTEGER AS gdp_per_capita,
        TO_NUMBER(NULLIF(s.gini, '-'), '9999999')::INTEGER AS gini,
        TO_NUMBER(NULLIF(s.palma, '-'), '9999999')::INTEGER AS palma
    FROM 
        public.demographics d 
    INNER JOIN 
        public.social_protection s ON d.country = s.country
),
avg_population AS (
    SELECT AVG(population) AS avg_population FROM economic_details
),
avg_gdp AS (
    SELECT AVG(gdp_per_capita) AS avg_gdp FROM economic_details
)
SELECT 
    e.country, 
    e.population, 
    e.gdp_per_capita, 
    e.gini, 
    e.palma
FROM 
    economic_details e
CROSS JOIN 
    avg_gdp
WHERE 
    e.gdp_per_capita < avg_gdp.avg_gdp
ORDER BY e.gdp_per_capita asc;

--HEALTH INDICATORS
--life expectancy
SELECT 
    country, 
    life_expectancy_2021
FROM 
    public.demographics
ORDER BY 
    life_expectancy_2021;

--Comparing births and mortality
SELECT 
    d.country as country, 
    TO_NUMBER(NULLIF(d.annual_number_of_births_2021, '-'), '9999999')::INTEGER as annual_births_2021,
	TO_NUMBER(NULLIF(m.infant_mortality_rate_2021, '-'), '9999999')::INTEGER as mortality_rate_infants
FROM 
    public.demographics d INNER JOIN public.child_mortality m
	ON d.country=m.country
WHERE d.annual_number_of_births_2021 NOT LIKE '-'
	AND m.infant_mortality_rate_2021 NOT LIKE '-'
ORDER BY annual_births_2021 asc,mortality_rate_infants desc;

--child mortality rates above average
WITH averages AS
(SELECT 
	AVG(TO_NUMBER(NULLIF(under_five_mortality_2021, '-'), '9999999')::INTEGER) as avg_under_five_mortality_rate_2021,
	AVG(TO_NUMBER(NULLIF(mortality_rate_five_to_fourteen, '-'), '9999999')::INTEGER) as avg_five_to_fourteen_mortality_rate_2021
FROM 
    public.child_mortality 
WHERE 
	 under_five_mortality_2021 NOT LIKE '-'
	AND mortality_rate_five_to_fourteen NOT LIKE '-'
	)
SELECT m.country,m.under_five_mortality_2021,m.mortality_rate_five_to_fourteen
FROM child_mortality m
	CROSS JOIN
	averages a
WHERE TO_NUMBER(NULLIF(m.under_five_mortality_2021, '-'), '9999999')::INTEGER>a.avg_under_five_mortality_rate_2021
OR TO_NUMBER(NULLIF(m.mortality_rate_five_to_fourteen, '-'), '9999999')::INTEGER > a.avg_five_to_fourteen_mortality_rate_2021;


--influence of social protection on child mortality
SELECT 
    s.country as country, 
    TO_NUMBER(NULLIF(s.social_protection_children_prportion, '-'), '9999999')::INTEGER as social_protection_children_proportion,
	TO_NUMBER(NULLIF(m.under_five_mortality_2021, '-'), '9999999')::INTEGER as under_five_mortality_rate_2021,
	TO_NUMBER(NULLIF(m.mortality_rate_five_to_fourteen, '-'), '9999999')::INTEGER as five_to_fourteen_mortality_rate_2021
FROM 
    public.social_protection s INNER JOIN public.child_mortality m
	ON s.country=m.country
WHERE s.social_protection_children_prportion NOT LIKE '-'
	AND m.under_five_mortality_2021 NOT LIKE '-'
	AND m.mortality_rate_five_to_fourteen NOT LIKE '-'
ORDER BY social_protection_children_proportion asc,under_five_mortality_rate_2021 desc,five_to_fourteen_mortality_rate_2021 desc;

--EARLY EDUCATION
-- attendance in early education 
SELECT country,attendance_early_education_total,attendance_early_education_male,attendance_early_education_female
FROM public.childhood_development
ORDER BY TO_NUMBER(NULLIF(attendance_early_education_total, '-'), '9999999')::INTEGER asc
	
-- comparing attendance in early education to stimulation and availability of learning materials
SELECT country,attendance_early_education_total,early_stimulation_total, learning_materials_books,learning_materials_play
FROM public.childhood_development
ORDER BY TO_NUMBER(NULLIF(attendance_early_education_total, '-'), '9999999')::INTEGER asc
	














