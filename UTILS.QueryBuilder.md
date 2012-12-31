    
    Class UTILS.QueryBuilder Extends %RegisteredObject
    {
    
    Property info As UTILS.QueryInfo;
    
    Property alias As %String [ Internal ];
    
    Property columns As %List [ Internal ];
    
    Property orderBys As %List [ Internal ];
    
    Property ands As %List [ Internal ];
    
    Property ors As %List [ Internal ];
    
    Property whereClause As %String [ Internal ];
    
    Property havingClause As %String [ Internal ];
    
    /*
     TODO
    Property equals As %List [ Internal ];
    
    Property startsWith As %List [ Internal ];
    
    Property between As %List [ Internal ];
    */
    Method %OnNew(tableName As %String = "", alias As %String = "") As %Status [ Private, ProcedureBlock = 1, ServerOnly = 1 ]
    {
        s ..info=##class(UTILS.QueryInfo).%New() 
        
        d:tableName'="" ..from(tableName, alias)
        
        Quit $$$OK
    }
    
    Method setWhere(where As %String)
    {
        s ..whereClause=where
        
        q ##this
    }
    
    Method setHaving(having As %String)
    {
        s ..havingClause=having
        
        q ##this
    }
    
    Method addOr(or As %String) [ Internal ]
    {
        s:'$LF(..ors,or) ..ors=..ors_$LB(or)
    }
    
    Method addAnd(and As %String) [ Internal ]
    {
        s:'$LF(..ands,and) ..ands=..ands_$LB(and)
    }
    
    Method addSelect(column As %String)
    {
        s:'$LF(..columns,column) ..columns=..columns_$LB(column)	
        
        q ##this
    }
    
    Method addOrderBy(orderBy As %String)
    {
        s:'$LF(..orderBys,orderBy) ..orderBys=..orderBys_$LB(orderBy)
        
        q ##this
    }
    
    Method orderBy(column As %String)
    {
        d ..addOrderBy(column)
        
        q ##this
    }
    
    /// table name
    Method from(name As %String, alias As %String = "")
    {
        s ..info.tableName=name_" "_alias
        s ..alias=alias
        
        q ##this
    }
    
    /// limit
    Method limit(limit As %Integer)
    {
        s ..info.rows=limit
        
        q ##this
    }
    
    Method top(top As %Integer)
    {
        q ..limit(top)
    }
    
    /// columnes que es consulten (la part del SELECT)
    Method select(columns As %String)
    {
        s cols=$LFS(columns,",")
        
        F i=1:1:$LL(cols) {
            d ..addSelect($LG(cols,i))
        }
        
        q ##this
    }
    
    /// afegeix condició AND
    Method andWhere(where As %String)
    {
        d ..addAnd(where)
        
        q ##this
    }
    
    Method orWhere(where As %String)
    {
        d ..addOr(where)
        
        q ##this
    }
    
    /// ORDER BY column ASC
    Method asc()
    {
        s ..info.sortOrder="asc"
        
        q ##this
    }
    
    /// ORDER BY column DESC
    Method desc()
    {
        s ..info.sortOrder="desc"
        
        q ##this
    }
    
    /* TODO
    Method equals(column As %String, value As %String)
    {
        s:'$LF(..equals,column_"#"_value) ..equals=..equals_$LB(column_"#"_value)
        
        q ##this
    }
    
    Method startsWith(column As %String, value As %String)
    {
        s:'$LF(..startsWith,column_"#"_value) ..startsWith=..startsWith_$LB(column_"#"_value)
        
        q ##this
    }
    
    
    Method between(column As %String, value As %String)
    {
        s:'$LF(..between,column_"#"_value) ..between=..between_$LB(column_"#"_value)
        
        q ##this
    }
    */
    Method groupBy(groupBy As %String)
    {
        s ..info.groupByClause=groupBy
        
        q ##this
    }
    
    /// retorna el SQL resultant
    Method getQuery() As %String
    {
        /*
        s prefix=$S(..alias'="":..alias_".",1:"")
        
        if prefix'="" {
            F i=1:1:$LL(..columns) {
                s column=$LG(..columns,i)	
            }	
        }*/
            
        ; selects	
        F i=1:1:$LL(..columns) {
            s select($I(select))=$LG(..columns,i)	
        }
        
        merge ..info.columns=select
        
        if ..whereClause="" {
            s and=$LTS(..ands," AND ")
            s or="("_$LTS(..ors, " OR ")_")"	
        
            s ..info.whereClause=$S($LL(..ands)>0&&($LL(..ors)=0):and,$LL(..ands)>0&&($LL(..ors)>0):and_" AND "_or,$LL(..ands)=0&&($LL(..ors)>0):or,1:"")
        }
        else {
            s ..info.whereClause=..whereClause	
        }
            
        s ..info.havingClause=..havingClause	
                
        s:$LL(..orderBys)>1 ..info.orderByClause=$LTS(..orderBys,",")
        s:$LL(..orderBys)=1 ..info.sortColumn=$LG(..orderBys,1)
        
        s st=..info.%CreateSQL(0,..info)
        q $REPLACE(..info.queryText,"  "," ")
    }
    
    Method Dump()
    {
        w "   Select: "_$LTS(..columns)_$S($LL(..columns):" ("_$LL(..columns)_")",1:""),!
        w "     From: "_..info.tableName,!
        w "    Where: ",!
        w "      and: "_$LTS(..ands)_$S($LL(..ands)>0:" ("_$LL(..ands)_")",1:""),!
        w "       or: "_$LTS(..ors)_$S($LL(..ors)>0:" ("_$LL(..ors)_")",1:""),!
        w " order by: "_$LTS(..orderBys)_" ("_$LL(..orderBys)_")",!
        w " group by: "_..info.groupByClause,!
        w "     sort: "_..info.sortOrder,!
        w "    limit: "_..info.rows,!!
    }
    
    }