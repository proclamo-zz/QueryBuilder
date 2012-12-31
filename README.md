QueryBuilder
============

A very simple query builder tool for Caché ObjectScript from Intersystems, inspired by QueryBuilder interface from Doctrine2.

------------------------

Classe genèrica per construïr querys sql a parts. No permet consultes complexes amb joins ni amb subconsultes.
Té una interfaç fluida i es pot afegir noves columnes o condicions en qualsevol moment

## Example ##

	s query=##class(UTILS.QueryBuilder).%New()	
	
    d query.select("ID,Name,Date").top(3).from("SQLUser.Demo").andWhere("Birthday IS NULL").andWhere("Age IS NOT NULL").orderBy("ID").desc().addSelect("Person->Name")		
	
    w query.getQuery()  
    ; prints SELECT TOP 3 ID,Name,Date,Person->Name FROM SQLUser.Demo WHERE Birthday IS NULL AND Age IS NOT NULL ORDER BY ID desc


La principal ventatja és la reutilització

    ClassMethod MyQuery1()
    {	
    s query=##class(UTILS.QueryBuilder).%New("SQLUser.Demo")
    d query.select("id,Name").andWhere("Date>?")
    quit query
    }
    
    ClassMethod MyQuery2()
    {
    s query=..MyQuery1()	
    d query.addSelect("count(*) as cont").addSelect("Type").groupBy("Type")
    
    s sql=query.getQuery()
    w sql,!   
    ; SELECT id,Name,count(*) as cont,Type FROM SQLUser.Demo WHERE Date>? GROUP BY Type
    }

Si no s'indica cap columna, agafa el %ID per defecte:

    s query=##class(UTILS.QueryBuilder).%New("SQLUser.Demo")
    w query.getQuery(),!    ; SELECT %ID FROM SQLUser.Demo

## TODO ##

- Afegir alias a la taula principal per poder fer subqueries
- Afegir filtres:
	- equals(columna,valor) => WHERE columna = valor</li>
	- startsWith(columna,valor) => WHERE columna %STARTSWITH valor</li>
	- between(columna,start,end) => WHERE columna BETWEEN start AND end</li>
	- in(columna,valors) => WHERE columna IN ( $P(valors,",",1), $P(valors,",",2), ... )

- Suport per a JOIN
