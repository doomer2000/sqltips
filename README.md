# sqltips
# goes through all columns that are defined as UUID and then returns the values for those columns. You can optionally pass a UUID value that should be searched for
create or replace function find_uuids(p_id uuid default null)
  returns table(schema_name text, table_name text, column_name text, uid uuid)
as
$$
declare
  l_row record;
  l_sql text;
begin
  for l_row in select n.nspname, tb.relname, col.attname
               from pg_attribute col
                 join pg_type ty on ty.oid = col.atttypid
                 join pg_class tb on tb.oid = col.attrelid
                 join pg_namespace n on n.oid = tb.relnamespace
               where col.attnum > 0
                 and not col.attisdropped
                 and ty.typname = 'uuid' 
                 and n.nspname not in ('pg_catalog', 'information_schema')
                 and n.nspname not like 'temp%'
                 and tb.reltype = 'r'
  loop
    l_sql := format('select %L as schema_name, 
                            %L as table_name, 
                            %L as column_name, 
                            %I as uid 
                     from %I.%I', l_row.nspname, l_row.relname, l_row.attname, l_row.attname, l_row.nspname, l_row.relname);
    if p_id is not null then 
       l_sql := l_sql || format(' where %I = %L', l_row.attname, p_id);
    end if;
    return query execute l_sql;
  end loop;
end;
$$
language plpgsql;

select *
from find_uuids('19d2efcc-45c1-4371-addc-45028a054b85');
