# Postgresql

## psql output

Sometimes when using psql, lines are really long.

One option is to enable column wrapping. This will wrap lines in columns keeping the max width of
the total columns as long as the terminal total width. To enable:

```
\pset format wrapped
```

Another options is to enable extended view. This will show each row column in a separate row.
To enable:

```
\x on
```

Docs link for all psql options: [https://www.postgresql.org/docs/13/app-psql.html](https://www.postgresql.org/docs/13/app-psql.html)
