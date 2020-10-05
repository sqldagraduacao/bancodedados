# EXERCÍCIOS Parte 2

1) Faça um consulta que retorne o nome e sobrenome do cliente, seu bairro, e os valores das suas movimentações, a data ordenando as movimentações pelas mais recentes.

~~~sql
	SELECT ClienteNome,
       ClienteSobrenome,
       ClienteBairro,
       MovimentoData,
       MovimentoValor
	FROM Clientes,
		 Contas,
		 Movimentos
	WHERE Clientes.ClienteCodigo = Contas.ClienteCodigo
		  AND Contas.ContaNumero = Movimentos.ContaNumero
	ORDER BY MovimentoData DESC;
~~~

2) Mostre o nome do cliente, sobrenome e a sua renda convertida em dolar e euro.

~~~sql
	SELECT ClienteNome,
		   ClienteSobrenome,
		   (ClienteRendaAnual / 3.9) AS Dolar,
		   (ClienteRendaAnual / 4.4) AS Euro
	FROM Clientes;
~~~

3) Traga o nome dos clientes, o sobrenome, o bairro, o estado civil \(descrito\), o sexo \(descrito\) e classifique o cliente de acordo com a sua renda anual, C tem renda menor que 50.000, B tem renda menor que 70.000 e A tem renda acima de 70.000.

~~~sql

	SELECT ClienteNome,
       ClienteSobrenome,
       ClienteBairro,
       ClienteEstadoCivil,
       CASE
           WHEN ClienteEstadoCivil = 'S' THEN
               'Solteiro'
           ELSE
               'Casado'
       END AS ESTADOCIVILDECRITO,
       ClienteSexo,
       CASE
           WHEN ClienteSexo = 'M' THEN
               'Masculino'
           ELSE
               'Feminino'
       END AS SEXODESCRITO,
       ClienteRendaAnual,
       CASE
           WHEN ClienteRendaAnual < 50000 THEN
               'C'
           WHEN ClienteRendaAnual < 70000 THEN
               'B'
           ELSE
               'A'
       END AS 'CLASSIFICAÇÃO'
	FROM Clientes;
~~~

4) Liste todos os clientes que moram no mesmo bairro das agências do banco.

~~~sql
	SELECT ClienteNome,
       ClienteBairro,
       AgenciaBairro,
       AgenciaNome
	FROM Clientes,
		 Agencias
	WHERE ClienteBairro = AgenciaBairro;
~~~

5) Mostre todos os clientes que possuem número no seu e-mail.

~~~sql
	SELECT Clientes.ClienteNome,
		   Clientes.ClienteEmail
	FROM dbo.Clientes
	WHERE Clientes.ClienteEmail LIKE '%[0-9]%';
~~~

6) Mostre todos os clientes em que o nome da rua começa começa com R. e não com RUA.

~~~sql
	SELECT ClienteRua
	FROM dbo.Clientes
	WHERE ClienteRua LIKE 'R.%'
		  AND ClienteRua NOT LIKE 'RUA%';
~~~

7) Mostre o nome do cliente e a renda apenas do 5 melhores clientes com base na sua renda.

~~~sql
	SELECT TOP 5
		   ClienteNome,
		   ClienteRendaAnual
	FROM dbo.Clientes
	ORDER BY ClienteRendaAnual DESC;
~~~

8) Mostre o nome do cliente e a renda apenas do 5 piores clientes com base na sua renda.

~~~sql
	SELECT TOP 5
       ClienteNome,
       ClienteRendaAnual
	FROM dbo.Clientes
	ORDER BY ClienteRendaAnual;
~~~

9) Mostre o nome e a rua dos clientes que moram em residencias cujo número está entre 300 e 500.

~~~sql
	SELECT ClienteNome,
       ClienteRua
	FROM dbo.Clientes
	WHERE ClienteNumero
	BETWEEN 300 AND 500;
~~~

10) Utilizando o conceito de sub consulta, mostre quais clientes não possuem cartão de crédito.

~~~sql
	SELECT *
	FROM dbo.Clientes
	WHERE ClienteCodigo NOT IN
		  (
			  SELECT ClienteCodigo FROM dbo.CartaoCredito
		  );
~~~

11) Mostre o nome do cliente, o nome da agência e o bairro da agência, as movimentações dos clientes e o limite do cartão de crédito deles, somente para os clientes em que a conta foi aberta a partir de 2008.

~~~sql
	SELECT ClienteNome,
       AgenciaNome,
       AgenciaBairro,
       MovimentoValor
	FROM dbo.Clientes,
		 dbo.Agencias,
		 dbo.Contas,
		 dbo.CartaoCredito,
		 dbo.Movimentos
	WHERE clientes.ClienteCodigo = Contas.ClienteCodigo
		  AND agencias.AgenciaCodigo = dbo.Contas.AgenciaCodigo
		  AND CartaoCredito.ClienteCodigo = clientes.ClienteCodigo
		  AND dbo.Contas.ContaNumero = dbo.Movimentos.ContaNumero
		  AND ContaAbertura >= '2008-01-01';
~~~

12) Faça uma consulta que classifique os clientes em Regiões conforme o bairro que moram.

~~~sql
	SELECT dbo.Clientes.ClienteNome,
		   dbo.Clientes.ClienteBairro,
		   CASE
			   WHEN ClienteBairro IN ( 'ITINGA', 'FLORESTA' ) THEN
				   'SUL'
		   END AS 'REGIÃO'
	FROM Clientes;
~~~

13) Mostre o nome do cliente e o tipo de movimentação, apenas para as movimentações de débito.

~~~sql
	SELECT ClienteNome,
		   MovimentoValor,
		   MovimentoTipo,
		   TipoMovimentoDescricao
	FROM Clientes,
		 Contas,
		 Movimentos,
		 TipoMovimento
	WHERE Clientes.ClienteCodigo = Contas.ClienteCodigo
		  AND Contas.ContaNumero = dbo.Movimentos.ContaNumero
		  AND dbo.Movimentos.MovimentoTipo = dbo.TipoMovimento.TipoMovimentoCodigo
		  AND TipoMovimento.TipoMovimentoCodigo = -1;
~~~

