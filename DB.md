## DB Schema

We prepend table names with h in order to maintain convention of singularity. 

### User
```
create table huser (
	id serial primary key,
    name varchar(100),
    password varchar(100),
    email varchar(50),
    enabled bool,
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Role
```
create table hrole (
	id serial primary key,
    name varchar(100),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Right
```
create table hright (
	id serial primary key,
    name varchar(100),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Charter
The charter defines the rights of each individual based on the roles that they inhabit.

```
create table hcharter (
	hrole_id integer references hrole(id),
    hright_id integer references hright(id),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Hierarchy
The hierarchy specifies which roles each user inhabits.
```
create table hhierarchy (
	hrole_id integer references hrole(id),
    huser_id integer references huser(id),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Life

- Assume that any life added to this table has already paid premium in full and in advance


```
create table hlife (
	id serial primary key,
    huser_id integer references huser(id),
    date_of_birth date, 
    date_of_death date,
    hpolicy_id integer references hpolicy(id),
    addition_date date,
    deletion_date date,
    sum_insured numeric,
    premium numeric,
    claimed_amount numeric,
    paid_out bool,
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Policy

- We store policy wording in JSONB format to allow indexing. Cost for input is low because new policies are not added often.

```
create table hpolicy (
	id serial primary key,
    name varchar(100),
    wording jsonb,
    currency char(3),
    issue_date date,
    expiry_date date,
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Question
Stores list of questions possible in a questionnaire
```
create table hquestion (
	id serial primary key,
    description varchar(200),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Answer
Stores list of possible answers per question

```
create table hanswer (
	id serial primary key,
    description varchar(200),
    hquestion_id integer references hquestion(id),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Questionnaire
Stores list of questions available per policy

```
create table hquestionnaire (
	hpolicy_id integer references hpolicy(id),
    hquestion_id integer references hquestion(id),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Profile
Stores list of answers of each life
```
create table hprofile (
	hanswer_id integer references hanswer(id),
    hlife_id integer references hlife(id),
   	created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
   	updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
)
```

### Trigger to set updated_at

```
CREATE OR REPLACE FUNCTION trigger_set_timestamp()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

```
CREATE TRIGGER set_timestamp
BEFORE UPDATE ON huser
FOR EACH ROW
EXECUTE PROCEDURE trigger_set_timestamp();
```

### Schema Diagram
![db.png](#file:4676e500-48e3-3927-01db-901421068ea9)

### Data Generator

```
insert into huser (
    name, password, email, enabled, horg_id
)
select
    md5(random()::text),
	left(md5(random()::text), 4),
    md5(random()::text),
    true,
    1
from generate_series(1, 100) s(i)
```

```
insert into hlife (
    huser_id, date_of_birth, hpolicy_id, addition_date, sum_insured, premium
)
select
    i,
    timestamp '1980-01-01' - random() * (timestamp '1980-01-01' - timestamp '2000-01-01'),
    1,
    '2018-05-24',
    10,
    1
from generate_series(1, 100) s(i)
```

```
insert into hprofile (
    hanswer_id, hlife_id
)
select
    trunc(random() * 2 + 1),
    i
from generate_series(101, 200) s(i);
```

### Death Probability Calculation

```
-- What is the probability that someone exactly like Bart Simpson will die next year?
select * from hlife
inner join huser on hlife.huser_id = huser.id
where huser.name = 'Bart Simpson';

-- Probability of a Bart Simpson dying at age 31 = P(Death|Age 31) = P(D|X) = P(X|D).P(D)/P(X)
select 
	((cast(
		(select
		count(date_part('year', age(date_of_death,date_of_birth)))
	from hlife
	where date_part('year', age(date_of_death,date_of_birth)) = 31) as decimal) / count(date_of_death)) *
	(cast(count(date_of_death) as decimal) / count(id)) / 
	(cast (
	(
	select
		count(date_part('year', age(coalesce(date_of_death, now()) ,date_of_birth))) -- if dead then calculate date at death
	from hlife
	where date_part('year', age(coalesce(date_of_death, now()) ,date_of_birth)) = 31) as decimal) 
	/ count(id))) as prob_d_given_x
from hlife;

-- Multiply above by (select sum_insured from hlife where huser_id=1) to get base premium without expenses

-- P(X|D) = Probability of a life in the population being 31 years old given that he is dead
select cast(
	(select
	count(date_part('year', age(date_of_death,date_of_birth)))
from hlife
where date_part('year', age(date_of_death,date_of_birth)) = 31) as decimal) / count(date_of_death) from hlife;

-- P(D) = Probability of Death in entire population
select cast(count(date_of_death) as decimal) / count(id)
from hlife;

-- P(X) = Probability of a life being 31 years old in the entire population, not limited to our pool. 
-- i.e. What percentage of the population is 31 years old, whether they are alive or dead
select cast (
	(
	select
		count(date_part('year', age(coalesce(date_of_death, now()) ,date_of_birth))) -- if dead then calculate date at death
	from hlife
	where date_part('year', age(coalesce(date_of_death, now()) ,date_of_birth)) = 31) as decimal) / count(id) as prob_of_age_x
from hlife;

```