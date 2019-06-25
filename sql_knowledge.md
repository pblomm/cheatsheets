# SQL KNOWLEDGE

## BASIC SQL

### TABLES

#### CREATING

```
create table table_name (
	id	number,
	name 	varchar2(10)
	);
```

#### INSERTING ROWS

```
insert into table_name (id)
values (50);

insert into table_name
values (50, 'pascal');
```

#### DELETING ROWS

```
delete from table_name
where id = 100;
```

#### UPDATING ROWS

```
update table_name
set name = 'tom'
where id = 50;
```

#### DROPPING

```
drop table table_name;
```

#### CONSTRAINTS

## PL/SQL

### SYSTEM TABLES

user_source		bevat de zelfgeschreven procedures/functies/... van de gebruiker

### RUN PL/SQL

- in SQL developer via het Run.. command in het procedure scherm (CTRL+F10)
- door de PL/SQL commandos tussen begin..end te zetten

```
begin
	procedure_name;
end;
```

### MELDINGEN NAAR HET SCHERM

De console.log() van PL/SQL.
Soms nodig om 'set serveroutput on' eerst uit te voeren om output te zien.

```
dbms_output.put_line('hello world');
```

### BENADEREN VAN TABELLEN

De select moet exact 1 rij opleveren.
Indien meer moet men gebruik maken van bvb collections.
Steeds evenveel attribuutnamen als er variabelen zijn om in te steken.

```
select attr_name into v_attr
from table_name;
```

### BULK DML

Om context-switchen tot een minimum te herleiden. Deze wegen op de performantie.
Normaal kan men via een loop alle elementen van een collection overlopen.
Dit is niet performant, want veel context-switch.
Oplossing is de forall.

```
forall v_index in indices of t_coll
save exceptions
insert into table_name values (100, 50, 'pascal');
```

Door 'save exceptions' zullen rijen die geen problemen geven (geen exceptions geven) wel worden uitgevoerd.

### CURSORS

1. Impliciete

Door Oracle beheerd.
Cursorattributen:

sql%found		boolean		true als rijen verwerkt
sql%notfound		boolean		true als geen rijen verwerkt
sql%rowcount		number		aantal rijen dat is verwerkt
sql%bulk_rowcount()	number		het aantal rijen dat via bulk dml is aangepast	
sql%bulk_exceptions()			collection, bevat error_index en error_code		

2. Expliciete

Door de gebruiker gedefinieerd.
In het declaratiegedeelte van het programma.

```
cursor cursor_name is
query;
```

De actieve set wordt bepaald. Query wordt uitgevoerd.
Bij elke fetch haalt men een rij op uit de actieve set.
De ruimte wordt terug vrijgegeven.

```
open cursor_name;
fetch cursor_name into r_record
close cursor_name;
```

#### FETCH BULK COLLECT INTO

Om de gehele actieve set in een collection te pompen.

```
fetch cursor_naam bulk collect into t_coll
```

#### CURSORATTRIBUTEN

%found
%notfound
%rowcount
%isopen

#### CURSOR FOR LOOP

Cursor wordt hier automatisch geopend, gefetchd en gesloten!

```
for r_record in cur_cursor
loop
end loop;
```

Bij DML instructie.
Verband leggen tussen gefetchte rij en rij in database.
Met 'current of cur_name'.

```
update medewerkers
set maandsal = 5000
where current of cur_name;
```

#### CURSOR MET PARAMETERS

Parameters mogelijk bij cursors.

### NAAMCONVENTIES

variabelen	v_naam
constanten	cons_naam
parameters	p_naam
cursors		cur_naam
collections	t_naam
records		r_naam
types		type_naam
triggers	tr_naam

### TRIGGERS

Een trigger is code die aan een bepaalde tabel gekoppeld is en uitgevoerd wordt wanneer die tabel een
bepaalde DML instructie ondergaat.

```
create or replace trigger tr_naam
before insert or update on table_naam
end tr_naam;
```

Een trigger kan worden uitgevoerd 'before' of 'after' de DML.
Verschillende DML kunnen worden samen gecombineerd: 'before insert or update'.
Triggers worden automatisch uitgevoerd bij DML. Enkel gebruikers met DML object privilege zullen de trigger
in werking kunnen doen treden.

#### INSERTING, DELETING, UPDATING

Om verfijnder te kunnen reageren op verschillende DML instructies


#### RIJTRIGGER

De trigger wordt voor elke rij betrokken bij de DML instructie uitgevoerd.

```
create or replace trigger tr_naam
before insert on table_naam
for each row
end tr_naam;
```

Men kan ook gebruik maken van 'when()' na de 'for each row' om een conditie mee te geven.
De code van de trigger wordt dan enkel uitgevoerd wanneer aan die conditie is voldaan.
Opgelet: in 'when()' mag men geen ':' gebruiken. Dus 'when(old.attrib < new.attrib)'.

Voor alle attributen van de betrokken rij beschik je over de oude en nieuwe waarden.

```
:old.attrib_naam
:new.attrib_naam
```

Bij delete is :new gelijk aan null.
Bij insert is :old gelijk aan null.

#### MUTATING TABLE PROBLEEM

Mutating table is de tabel waar de trigger op gebouwd is.
In de code van de rijtrigger mag men nooit lezen of schrijven naar de mutating table.

! Uitzondering, rij triggers met triggering event 'insert'

Oplossing:

1. Globale variabele definieren
2. Statement trigger creeeren die globale variabele declareert
3. Rijtrigger creeeren die gebruik maakt van die globale variabele

Andere oplossing:

Compound trigger definieren.

1. Geen gebruik van globale variabele
2. Definitie van 1 trigger, die rij en statement trigger bevat.

```
create or replace trigger tr_naam
for insert or update on table_naam
	compound trigger
		-- informatie die tussen de triggers wordt gedeeld, een variabele bvb.
	before statement
	is
	begin
		-- code statement trigger
	end before statement;

	before each row
	is
	begin
		-- code row trigger
	end before each row;
end tr_naam;
```

#### MISC

Een trigger kan nooit een commit, rollback of procedures bevatten die een commit, rollback bevatten.

Wie kan triggers aanmaken?

```
grant create trigger on user_name;
```

Hoe kan men een trigger in/uitschakelen?

```
alter trigger tr_naam disable
alter trigger tr_naam enable
```

Alle triggers op een tabel uitschakelen.

```
alter table table_naam disable all triggers;
```

Trigger droppen.

```
drop trigger tr_naam;
```



### COLLECTIONS

#### ASSOCIATIEVE ARRAY

Bevat 2 kolommen.
Eerste kolom bevat de index en doet dienst als PK.
Het datatype hiervan is pls_integer.
Tweede kolom bevat de gegevens (enkelevoudige variabele of record).

1. Structuur definieren (indien record, deze structuur ook definieren en koppelen)
2. Variabele koppelen aan die structuur.

Dit gebeurt allemaal in het 'is' vak van een PL/SQL block.

```
type type_t_naam is table of number index by pls_integer;
t_naam type_t_naam;
```

```
type type_r_naam is record (
	dit	number,
	dat	number
	);
type type_t_naam is table of type_r_naam index by pls_integer;
t_naam type_t_naam;
```

##### DECLAREREN

Het toewijzen van waarden aan een bepaald veld uit de collection met :=.

```
t_coll(5).naam := 'pascal'
```

Als men de collection wilt vullen met informatie uit de tabel gebruik 'bulk collect into'.

```
type type_t_namen is table of varchar2(20) index by pls_integer;
t_namen type_t_namen;
select naam bulk collect into t_namen
from table_name;
```


##### METHODEN

count		het totaal aantal gegevens in de collection
exists(n)	true als er een nde gegeven bestaat in de collection
first		index van het eerste gegeven
last		index van het laatste gegeven

#### NESTED TABLES

Kan net als een een associatieve array worden gevuld met 'bulk collect into'.

Nested table als zelfstandig object:

```
create type naam is table of datatype;
```

#### VARRAYS

Kan net als een een associatieve array worden gevuld met 'bulk collect into'.

Varray als zelfstandig object:

```
create type naam is varray(n) of datatype;
```

Vullen van een varray.
Complexer dan een gewone associatieve array.
Er wordt gebruik gemaakt van een constructor.

```
is
	type type_t_varray is varray(4) of number;
	t_varray type_t_varray;
begin
	t_varray := type_t_varray();
	t_varray.extend(4);
	-- hier de varray vullen
```

### RECORDS

1. Structuur definieren
2. Record variabele koppelen aan die structuur

Dit gebeurt allemaal in het 'is' vak van een PL/SQL block.

```
type type_address is record (
	street		varchar(20),
	street_number	number(3),
	city_zip	varchar(4)
	);

r_address type_address;
```

Men kan ook tabel%rowtype gebruiken van een tabel als type voor een record.
Dan neemt men alle attributen van die tabel in het record.

#### DECLAREREN

Het toewijzen van waarden aan een record met := of via 'select into'.

```
r_address.street := 'kipdorp';
r_address.city_zip := 2000;
```

Het resultaat van een DML instructie kan worden weggeschreven met 'returning'.
Kan ook in een record worden gestoken.

```
insert into medewerkers_tabel
values (100, 50)
returning id, naam into r_medewerker;
```

### VARIABELEN

#### AANMAKEN

v_variabele_naam [constant]  datatype [not null] [{:=| default} expressie];

#### VOORBEELDEN

```
v_aantal number(3) := 1;
v_pi constant number(3, 2) default 3.14;
v_id medewerkers.id%type;
v_leeftijd number(2) not null;
```

### PROGRAMMABESTURING

#### IF STRUCTUUR

##### VOORBEELD

```
if v_age < 18 then
	-- do this
elsif v_age < 65 then
	-- do this
else
	-- do that
end if;
```

#### CASE STRUCTUUR

##### VOORBEELD

```
case v_name
when 'max' then
	-- do this
when 'tom' then
	-- do this
else
	-- do that
end case;
```

#### LOOP STRUCTUREN

Het is mogelijk om loop structuren te nesten.
In elke loop zijn de volgende commando's mogelijk: 'exit', 'continue'.
Men kan loops labelen: << label >> 

```
<< label_name >>
loop
end loop label_name;
```

##### GEWONE LOOP

###### VOORBEELD

```
loop
	
	exit when v_count > 9;
	v_count := v_count + 1;
	-- do something
end loop;
```

##### WHILE LOOP

###### VOORBEELD

```
while v_count < 11
loop
	v_count := v_count + 1;
	-- do something
end loop;
```

##### FOR LOOP

###### VOORBEELD

```
for v_counter in 1..10
loop
	-- do something
end loop;
```

### PROCEDURES

De code wordt gecompileerd en opgeslagen in systeemtabellen.

#### PARAMETERS

Geen precisie meegeven bij het datatype!
De mode geeft aan of de parameter door het programma nog gewijzigd kan worden of niet.

```
p_name in number
```

#### AANMAKEN

```
create or replace procedure procedure_naam (p_naam in number)
is
begin
exception
end procedure_naam;
```

#### AANROEPEN

Named notation verdient de voorkeur.
De volgorde van parameters speelt hier geen rol en defaults kunnen ook gebruikt worden.

- named notation

```
procedure_naam(p_naam => 'pascal');
```

- positional notation

```
procedure_naam('pascal');
```

### FUNCTIES

De code wordt gecompileerd en opgeslagen in systeemtabellen.
Gebruik 'return' in het uitvoeringsdeel van het programma.
Geen komma na de return voor de 'is' zetten.

```
create or replace function functie_naam (p_naam in number)
return number
is
begin
exception
end functie_naam;
```

### PACKAGES

Packages worden gebruikt om code te bundelen.
De declaraties in dit stuk zijn allemaal public.
Gebruiker kan gebruik maken van alles wat in de package is gedeclareerd indien rechten.

```
grant execute on package_naam;
```

```
create or replace package package_naam
as
	-- declaraties
end package_naam;
```

Nadat de package specification is gebouwd moet de package body worden gedefinieerd.
Hier staat de echte code van de procedures, functies, cursors.
Kan niet worden gezien door diegene die een grant heeft gekregen op de package.
Enkel de maker van de package kan de body zien.

```
create or replace package body package_naam
as
	-- defenities
end package_naam;
```

Wie kan een package aanmaken?

```
grant create procedure to user_naam;
```

Wie kan een package oproepen?
De maker. En iemand met rechten.

```
grant execute on package_naam to user_naam;
```

Droppen van package

```
drop package package_naam	-- volledige package wordt verwijderd
drop package body package_naam	-- enkel de body wordt verwijderd
```

#### OPROEPEN

```
package_naam.procedure_naam();
```

#### INITIALISATIECODE

Code die moet worden uitgevoerd bij eerste aanroep van de package binnen een sessie.
Staat onderaan de package body en zit tussen 'begin' en 'end package_naam'.

#### OVERLOADING

Binnen een package meerdere programma's (functies, procedures) mogelijk met zelfde naam.
Maar met verschillende parameters.
Enkele mogelijk binnen een package!

#### DE STANDARD PACKAGE

SQL Oracle heeft een standard package gedefinieerd met voorgebouwde functies/procedures..
Men mag de naam 'standard' weglaten indien men er een functie van gebruikt.


### EXCEPTIONS

'sqlerrm' bevat de tekst van de foutboodschap

1. Compilatiefouten
2. Runtime Errors

#### HET EXCEPTION CODE BLOCK

```
exception
when e_naam_een then
	-- commandos
when e_naam_twee then
	-- commmandos
when others then
	-- commmandos
```

#### SOORTEN EXCEPTIONS

1. Predefined
2. User defined gekoppeld aan ORA Runtime Error

In declaratiedeel:

```
e_naam exception;
pragma exception_init(e_naam, foutcode);
```

3. User defined niet gekoppeld

Programmeur kan de exception raisen.

#### MISC

Door blokken te nesten kan men exceptions een eigen scope geven.
Verfijnde besturing. Eenzelfde soort fout kan verschillende exceptions oproepen nu.

Indien men het programma wil laten stoppen:

raise_application_error(foutcode, fouttekst);

to_date(sysdate, 'DY')









