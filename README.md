# MySql
Querys e rotinas que facilitam nosso dia a dia.

### Consultar tamanho das tabelas e em qual schema se encontra
Basta informar o nome da tabela que deseja verificar as dependencias.
```sql
SELECT 
	TABLE_SCHEMA ,
        table_name AS "Tables", 
	data_length , 
        index_length,
	round(((data_length + index_length) / 1024 / 1024), 2) "Size in MB"
 FROM information_schema.TABLES 
--  WHERE table_NAME = "painel_notificacaooperacional" 
 ORDER BY (data_length + index_length) DESC;


```

### Verifica dependencias de uma tabela
Basta informar o nome da tabela que deseja verificar as dependencias.
```sql
select TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, 
REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME from 
INFORMATION_SCHEMA.KEY_COLUMN_USAGE where 
REFERENCED_TABLE_NAME = '<table>'; 

```

### Listar todas as tabelas que estão vazias(sem registros):
```sql
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_ROWS = 0 AND TABLE_SCHEMA = 'Nome_do_Banco_Dados'

```

### Quero as tabelas vazias que tem um determinado prefixo em seu nome
```sql
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_ROWS = 0 AND TABLE_SCHEMA = 'Nome_do_Banco_Dados'
AND table_name LIKE '%prefixo_%'

```

### Exibir todas as colunas de uma tabela, acompanhada pelo tipo
```sql
SELECT `COLUMN_NAME` as NomeColuna, `COLUMN_TYPE` as TipoColuna
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA='Nome_do_Banco_Dados'
AND TABLE_NAME='Nome_da_Tabela';

```
### Descobrir quais tabelas tem um determinado campo
```sql
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
### Descobrir quais tabelas não tem um determinado campo e gera o script de alter table
```sql
    DROP DATABASE IF EXISTS `temp_database_comandos`;

CREATE DATABASE `temp_database_comandos`;

DELIMITER $$

USE `temp_database_comandos`$$

DROP PROCEDURE IF EXISTS `sp_criarcoluna`$$

CREATE PROCEDURE `sp_criarcoluna`(IN TABELAS VARCHAR(1000), IN NOME_COLUNA VARCHAR(100), IN TIPO_COLUNA VARCHAR(30))
BEGIN
 
 DECLARE v_finished INTEGER DEFAULT 0;
 DECLARE v_comando text DEFAULT "";
   
 DEClARE dataimportacao_cursor CURSOR FOR 
 SELECT concat('ALTER TABLE ', TABLE_SCHEMA, '.', TABLE_NAME,' ADD COLUMN ', NOME_COLUNA, ' ', TIPO_COLUNA, ' null;')
	FROM INFORMATION_SCHEMA.TABLES T
	WHERE TABLE_TYPE='BASE TABLE'
	AND FIND_IN_SET(TABLE_NAME, TABELAS)
	AND NOT EXISTS (
	SELECT TABLE_NAME, TABLE_SCHEMA FROM INFORMATION_SCHEMA.COLUMNS C 
		WHERE COLUMN_NAME = NOME_COLUNA
			AND FIND_IN_SET(TABLE_NAME, TABELAS)
			AND T.TABLE_NAME = C.TABLE_NAME 
			AND T.TABLE_SCHEMA = C.TABLE_SCHEMA);
 
 DECLARE CONTINUE HANDLER 
        FOR NOT FOUND SET v_finished = 1;
 
 DROP TABLE IF EXISTS `temp_database_comandos`.`TEMP_NOVACOLUNA`; 
 CREATE TABLE `temp_database_comandos`.`TEMP_NOVACOLUNA` (comando text);
 
 OPEN dataimportacao_cursor;
 
 get_dataimportacao: LOOP
 FETCH dataimportacao_cursor INTO v_comando;
 
 IF v_finished = 1 THEN 
 LEAVE get_dataimportacao;
 END IF;
  
 
 INSERT INTO `temp_database_comandos`.`TEMP_NOVACOLUNA`(`comando`)VALUES(v_comando);
  
 END LOOP get_dataimportacao;
 
 CLOSE dataimportacao_cursor;
 
END$$
 
DELIMITER ;


call sp_criarcoluna('Colaborador,Linha,CodigoBrasileiroOcupacao,FuncaoColaborador,UnidadeOperacional,TipoVeiculo,Veiculo,PessoaFisica', 'DataImportacao','DateTime');

select * from `temp_database_comandos`.`TEMP_NOVACOLUNA`;

DROP DATABASE IF EXISTS `temp_database_comandos`;

```

## Trabalhando com Eventos (Jobs)
##### VERIFICA SE O EVENTO ESTA HABILITADO
```sql
select @@event_scheduler;
```
##### HABILITA EVENTO
```sql
set global event_scheduler = ON;
```
##### DESABILITA EVENTO
```sql
set global event_scheduler = OFF;
```
##### EXIBE OS EVENTOS
```sql
SHOW EVENTS; 
```

##### EXIBE O FONTE DO EVENTO
```sql
show create event LIMPEZA_TABELAS_DETRO;
```

##### EXCLUI EVENTO
```sql
drop event LIMPEZA_TABELAS_DETRO;
```

##### CRIA EVENTO
```sql
delimiter |
create EVENT LIMPEZA_TABELAS_DETRO
     ON SCHEDULE EVERY 5 minute
    COMMENT 'APAGA MENSAGENS MANTENDO APENAS 2 MESES DE INFORMACAO'
    DO
      BEGIN
         DELETE FROM detro.MENSAGEMENVIADA WHERE DATAHORA <  DATE_SUB(current_timestamp(),INTERVAL 2 MONTH) LIMIT 10000;
		 DELETE FROM detro.mensagemrespondida WHERE DATAHORA <  DATE_SUB(current_timestamp(),INTERVAL 2 MONTH) LIMIT 10000;
		
      END|

delimiter;
```

# VEJA TAMBÉM
## Grupo de Estudo no Telegram
- [Participe gratuitamente do grupo de estudo](https://t.me/blogilovecode)

## Cursos baratos!
- [Meus cursos](https://olha.la/udemy)

## Fique ligado, acesse!
- [Blog ILoveCode](https://ilovecode.com.br)

## Novidades, cupons de descontos e cursos gratuitos
https://olha.la/ilovecode-receber-cupons-novidades
