Depois queria falar contigo para perceber se esta otimização seria viável. Mas no contexto de shipping costs, é nos sempre benéfico termos menos intervalos, na nossa tabela, correto? Supondo que a premissa anterior é verdadeira, ontem fiquei com isto na cabeça e acho que temos uma forma de fazer isto.

Ou seja, isto seria a nossa forma de definirmos os intervalos pelo ficheiro
```sql
INSERT INTO pre_shipment_prices_outbound
SELECT
	* EXCEPT(LOWER_BOUND),
	CASE 
		LOWER_BOUND IS NULL THEN 
			0
		ELSE 
			LOWERBOUND
	END AS LOWER BOUND
FROM (
	SELECT 
		* EXCEPT(weigth_interval_kg), 
		MAX(weight_interval_kg) AS LOWER_BOUND,
		LEAD(weight_interval_kg) AS UPPER_BOUND,
	FROM file
)
WHERE UPPER_BOUND IS NOT NULL
```

Para percebermos quais sao os intervalos que temos de inativar, teriamos de fazer o seguinte:
```sql
SELECT 
	client, 
	type, 
	service_level, 
	warehouse, 
	currency, 
	margin_estimated, 
	shipping_price, 
	country,
	lower_bound, 
	upper_bound 
FROM SHIPMENT_PRICES_OUTBOUND
WHERE ACTIVE = TRUE
	AND CLIENT = @client
EXCEPT DISTINCT
SELECT 
	client, 
	type, 
	service_level, 
	warehouse, 
	currency, 
	margin_estimated, 
	shipping_price, 
	country,
	lower_bound, 
	upper_bound 
FROM pre_shipment_prices_outbound
```
Isto retornaria, os intervalos que estao ativos na tabela que nao existem no nosso ficheiro. Ou seja, estes intervalos teriamos de dar update do `SET ACTIVE = FALSE`

Ao contrário, se queremos saber quais sao os novos intervalos que temos de dar insert:
```sql
SELECT 
	client, 
	type, 
	service_level, 
	warehouse, 
	currency, 
	margin_estimated, 
	shipping_price, 
	country,
	lower_bound, 
	upper_bound 
FROM pre_shipment_prices_outbound
EXCEPT DISTINCT
SELECT 
	client, 
	type, 
	service_level, 
	warehouse, 
	currency, 
	margin_estimated, 
	shipping_price, 
	country,
	lower_bound, 
	upper_bound 
FROM SHIPMENT_PRICES_OUTBOUND
WHERE ACTIVE = TRUE
	AND CLIENT = @client
```
e assim sabemos quais sao as linhas que temos de dar INSERT.

Qualquer update que ocorresse no ficheiro a um preço seria contemplado também por este select except. 




# Importing File

