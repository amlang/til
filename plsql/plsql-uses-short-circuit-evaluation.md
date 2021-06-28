# PL/SQL uses short circuit evaluation

PL/SQL uses short circuit evaluation, when evaluating a logical expression. This means that it will stop evaluating the expression if the first expression is true.
This also means that the short circuit evaluation prevents the `OR` expression from causing an error under certain circumstances.
That allows me to write expressions that would otherwise results to an error.

```plsql
set serverout on size unlimited
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

---
Further reading:
* [PL/SQL Language Fundamentals - Short-Circuit Evaluation](https://docs.oracle.com/cd/E11882_01/appdev.112/e25519/fundamentals.htm#LNPLS258)
