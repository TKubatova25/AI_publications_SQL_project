# AI_publications_SQL_project
The aim of this project is to learn to make a database using PostgreSQL, Docker and DBeaver and practice queries while analyzing data.

A) Project goal

Analyze AI research intensity and economic context across countries (2015–2022) using OWID datasets and SQL.

B) Datasets (with links)

I used publicly available data from ourworldindata.org (OWID):

Scholarly publications on artificial intelligence per million people
https://ourworldindata.org/grapher/scholarly-publications-on-artificial-intelligence-per-million-people?tab=table&time=latest&tableFilter=countries

Annual articles published in scientific and technical journals per million people, 2022
https://ourworldindata.org/grapher/scientific-publications-per-million

GDP per capitaIn constant international-$ – World Bank
https://ourworldindata.org/grapher/gdp-per-capita-worldbank?tab=table

Share of the population using the Internet
https://ourworldindata.org/grapher/share-of-individuals-using-the-internet?tab=table&time=earliest..2023

C) Tech stack

PostgreSQL + Docker + DBeaver

D) Basic workflow

- find data
- donload CVS files
- clean data (split columns, set delimiter to ;)
- create yaml file and initialize Docker
- create tables in PostgreSQL database in DBeaver
- import CVSs
- run queries
- analyze resutls

E) Key findings

Among major AI-producing economies, India exhibits the highest AI specialization in 2022, with AI accounting for 20% of its scientific output — suggesting a research ecosystem more concentrated in computing and digital technologies compared to diversified research systems like the United States.

F) Docker Compose file

docker-compose.yml

services:

  db:
  
    image: postgres:16
    
    container_name: ai_economy_db
    
    restart: unless-stopped
    
    environment:
    
      POSTGRES_USER: ${POSTGRES_USER}
      
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      
      POSTGRES_DB: ${POSTGRES_DB}
      
    ports:
    
      - "5432:5432"
      
    volumes:
    
      - pgdata:/var/lib/postgresql/data

volumes:

  pgdata:
  
G) SQL scripts

1 scripts to create tables: 

CREATE TABLE ai_publications_per_million (
  country TEXT NOT NULL,
  year INT NOT NULL,
  ai_pubs_per_million NUMERIC,
  PRIMARY KEY (country, year)
);

CREATE TABLE scientific_publications_per_million (
  country TEXT NOT NULL,
  year INT NOT NULL,
  sci_pubs_per_million NUMERIC,
  PRIMARY KEY (country, year)
);

CREATE TABLE gdp_per_capita (
  country TEXT NOT NULL,
  year INT NOT NULL,
  gdp_pc_usd NUMERIC,
  PRIMARY KEY (country, year)
);

CREATE TABLE internet_users (
  country TEXT NOT NULL,
  year INT NOT NULL,
  internet_users_pct NUMERIC,
  PRIMARY KEY (country, year)
);

2 scripts for analysis

-- Top AI publications-producing economies (selected list) - 2022 snapshot

SELECT
  a.country,
  a.ai_pubs_per_million,
  s.sci_pubs_per_million,
  g.gdp_pc_usd,
  i.internet_users_pct,
  
  ROUND(a.ai_pubs_per_million / NULLIF(s.sci_pubs_per_million,0) * 100, 2) AS ai_share_pct
  
FROM ai_publications_per_million a

JOIN scientific_publications_per_million s

  ON a.country = s.country AND a.year = s.year
  
LEFT JOIN gdp_per_capita g

  ON a.country = g.country AND a.year = g.year
  
LEFT JOIN internet_users i

  ON a.country = i.country AND a.year = i.year
  
WHERE a.year = 2022

  AND a.country IN (
    'China','United States','India','United Kingdom','Germany',
    'Japan','South Korea','Italy','Canada','France'
  )
  
ORDER BY ai_share_pct DESC;
