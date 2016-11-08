# MySql
Querys e rotinas que facilitam nosso dia a dia.


#Verifica dependencias de uma tabela
Basta informar o nome da tabela que deseja verificar as dependencias.
```sh
select TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, 
REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME from 
INFORMATION_SCHEMA.KEY_COLUMN_USAGE where 
REFERENCED_TABLE_NAME = '<table>'; 

```
