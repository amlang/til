# PL/SQL uses short circuit evaluation

PL/SQL uses short circuit evaluation, when evaluating a logical expression. This means that it will stop evaluating the expression as soons as the result can be determine.

Short example:
If left-hand side of an `OR` expression is `true`, the whole expression is `true` and the right side won't be evaluated.
It behaves the same way with `AND` expressions. If left-hand side of an `AND` expression is `false`, the whole expression is `false` and the right side of the expression won't be evaluated.

This also means that the short circuit evaluation prevents the `OR` and `AND` expression from causing an error under certain circumstances.
That allows us to write expressions that would otherwise results to an error.

```plsql
set serverout on
declare
    l_zero int := 0;

    function raise_error
        return boolean
    is
    begin
        raise_application_error(-20000, 'This error will never be raised');
    end;
begin
    -- the right side of the expression is evaluated only if the left side is not true
    if true or raise_error then
       dbms_output.put_line('Nothing to raise, right side is evaluating first');
    end if;

    -- Example from Oracle documentation
    if (l_zero = 0) or (5 / l_zero > 1) then
         dbms_output.put_line('does not cause as divide by zero exception, right side is true');
    end if;
end;
/
```
Better error handling is not the only benefit.
It can also potentially improve the performance of our code. But how?
As mentioned above PL/SQL will stop evaluating the expression if e.g. the first value of an `OR` expression is `true`. Because the whole expression is true. 

So if we imagine that we have a slow executing function that returns a Boolean value, we can write our logical expression in the "normal" way, which means we let the `slow_function` evaluate first and then the Boolean value, or in the "short-circuit" way, which means we evaluate the Boolean value first and then the function. Spoiler alert, the expressions in the type of "short circuit", end the evaluation prematurely and cost zero time.
 
```plsql
set serverout on
-- Thanks @ oracle-base.com
-- https://oracle-base.com/articles/misc/short-circuit-evaluation-in-plsql 
declare
    l_start    number;
    
    function slow_function
        return boolean
    is
    begin
        DBMS_LOCK.sleep(0.500);
        return true;
    end;
begin
    -- Time "normal" OR with TRUE
    l_start := DBMS_UTILITY.get_time;
    if slow_function OR true then
        -- Do nothing.
        null;
    end if;
    dbms_output.put_line('Normal OR: ' || to_char((DBMS_UTILITY.get_time - l_start) * 1e-2, '0.999') || ' s');

    -- Time short-circuit OR with TRUE
    l_start := DBMS_UTILITY.get_time;
    if true OR slow_function then
        -- Do nothing.
        null;
    end if;
    dbms_output.put_line('Short circuit OR: ' || to_char((DBMS_UTILITY.get_time - l_start) * 1e-2, '0.999') || ' s');
    
    -- Time "normal" AND with FALSE
    l_start := DBMS_UTILITY.get_time;
    if slow_function OR false then
        -- Do nothing.
        null;
    end if;
    dbms_output.put_line('Normal AND: ' || to_char((DBMS_UTILITY.get_time - l_start) * 1e-2, '0.999') || ' s');

    -- Time short-circuit AND with FALSE
    l_start := DBMS_UTILITY.get_time;
    if false and slow_function then
        -- Do nothing.
        null;
    end if;
    dbms_output.put_line('Short circuit AND: ' || to_char((DBMS_UTILITY.get_time - l_start) * 1e-2, '0.999') || ' s');
end;
/

Normal OR:  0.510 s
Short circuit OR:  0.000 s
Normal AND:  0.510 s
Short circuit AND:  0.000 s
 PL/SQL procedure successfully completed.
```
---
Further reading:
* [PL/SQL Language Fundamentals - Short-Circuit Evaluation](https://docs.oracle.com/cd/E11882_01/appdev.112/e25519/fundamentals.htm#LNPLS258)
* [Short-Circuit Evaluation in PL/SQL](https://oracle-base.com/articles/misc/short-circuit-evaluation-in-plsql)
