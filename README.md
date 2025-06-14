# Layoffs-Data-Cleaning

-- sql project- data cleaning
-- https://www.kaggle.com/datasets/swaptr/layoffs-2022


CREATE DATABASE world_layoffs;

select*
from world_layoffs.layoffs;



-- first we want to do is create a staging table. this is the one we will work in and clean the data. 

create table world_layoffs.layoffs_staging
like world_layoffs.layoffs;

select *
from layoffs_staging;



insert layoffs_staging
select * 
from world_layoffs.layoffs;


-- now when we are data cleaning we usually follow a few steps
-- 1. check for duplicates and remove any
-- 2. standardize data and fix errors
-- 3. Look at null values and see what 
-- 4. remove any columns and rows that are not necessary - few ways



-- 1. Remove Duplicates

# First let's check for duplicates



select *
from layoffs_staging;



select *,
Row_Number() over(
partition by company, industry, total_laid_off, percentage_laid_off, `date`) as row_num
from layoffs_staging;



with duplicate_cte as
(
select *,
Row_Number() over(
partition by company, industry, total_laid_off, percentage_laid_off, `date`) as row_num
from layoffs_staging)
select * 
from duplicate_cte
where row_num > 1;


-- let's just look at oda to confirm 
select *
from world_layoffs.layoffs_staging
where company = 'oda';



with duplicate_cte as
(
select *,
Row_Number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, `date`
, stage, country, funds_raised_millions) as row_num
from layoffs_staging)
select * 
from duplicate_cte
where row_num > 1;
 
 
 -- another idea Is to create a new column and add those row numbers in. Then delete where row numbers are over 2, then delete that column
 
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
 
 
select *
from layoffs_staging2;

INSERT INTO layoffs_staging2
select *,
Row_Number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, `date`
, stage, country, funds_raised_millions) as row_num
from layoffs_staging;
 
 
 select *
 from layoffs_staging2
 where row_num > 1;
 
 -- these are the ones we want to delete where the row number is > 1 or 2or greater essentially

 delete
 from layoffs_staging2
 where row_num > 1;
 
 
 -- 2. Standardize Data
 
 select distinct (company)
 from layoffs_staging2;
 
 select distinct (trim(company))
 from layoffs_staging2;
 
 select company,trim(company)
 from layoffs_staging2;
 
 update layoffs_staging2
 set company= trim(company);
 
 
  select distinct industry
 from layoffs_staging2
 order by 1;
 
 update layoffs_staging2
 set industry='crypto'
 where industry like 'crypto%';
 
 select *
 from layoffs_staging2
 where industry like 'crypto%';
 
 
select distinct location
 from layoffs_staging2
 order by 1;
 
 
select distinct country
 from layoffs_staging2
 order by 1;
 

select distinct country, trim(trailing '.' from country)
 from layoffs_staging2
 order by 1;
 
 update layoffs_staging2
 set country= trim(trailing '.' from country)
 where country like 'united states%';
 
 
 select `date`,
 str_to_date(`date`, '%m/%d/%Y')
 from layoffs_staging2;
 
 
 update layoffs_staging2
 set `date`=  str_to_date(`date`, '%m/%d/%Y');
 
 select `date`
 from layoffs_staging2;
 
 alter table layoffs_staging2
 modify column `date` date;
 
 
 
 
 -- 3. Look at Null Values
 select *
 from layoffs_staging2
 where total_laid_off is null
 and percentage_laid_off is null;
 
 
 update layoffs_staging2
 set industry=null 
 where industry='';
 
 
  select *
 from layoffs_staging2
 where industry is null
 or industry= '';
 
 
 select *
 from layoffs_staging2
 where company='airbnb';
 
 select *
 from layoffs_staging2 t1
 join layoffs_staging2 t2
    on t1.company=t2.company
    and t1.location=t2.location
where (t1.industry is null or t1.industry='')
and t2.industry is not null;


select t1.industry,t2.industry
 from layoffs_staging2 t1
 join layoffs_staging2 t2
    on t1.company=t2.company
    and t1.location=t2.location
where (t1.industry is null or t1.industry='')
and t2.industry is not null;


update layoffs_staging2 t1
join layoffs_staging2 t2
on t1.company=t2.company
set t1.industry=t2.industry
where t1.industry is null
and t2.industry is not null;


select *
 from layoffs_staging2
 where company like 'bally%';
 
 
 
 
 -- 4. remove any columns and rows we need to

 select *
 from layoffs_staging2
 where total_laid_off is null
 and percentage_laid_off is null; 
 
 
 delete
 from layoffs_staging2
 where total_laid_off is null
 and percentage_laid_off is null;
 
 
  select *
 from layoffs_staging2;
 
 alter table layoffs_staging2
 drop column row_num;
 
 
