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

#Listar todas as tabelas que estão vazias(sem registros):
```sh
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_ROWS = 0 AND TABLE_SCHEMA = 'Nome_do_Banco_Dados'

```

#Quero as tabelas vazias que tem um determinado prefixo em seu nome
```sh
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_ROWS = 0 AND TABLE_SCHEMA = 'Nome_do_Banco_Dados'
AND table_name LIKE '%prefixo_%'

```

#Exibir todas as colunas de uma tabela, acompanhada pelo tipo
```sh
SELECT `COLUMN_NAME` as NomeColuna, `COLUMN_TYPE` as TipoColuna
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA='Nome_do_Banco_Dados'
AND TABLE_NAME='Nome_da_Tabela';

```
#Descobrir quais tabelas tem um determinado campo
```sh
SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES T
WHERE TABLE_TYPE='BASE TABLE'
AND table_name in('Colaborador','Linha','CodigoBrasileiroOcupacao','FuncaoColaborador','UnidadeOperacional','TipoVeiculo','Veiculo','PessoaFisica')
and table_schema like 'linave_desenv'
AND  EXISTS (
				select 1 FROM INFORMATION_SCHEMA.COLUMNS C 
				Where COLUMN_NAME = 'DataImportacao'
				AND T.TABLE_NAME = C.TABLE_NAME 
                AND T.TABLE_SCHEMA = C.TABLE_SCHEMA);
    
```
