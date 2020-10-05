# EXERCÍCIOS Parte 3

1) Mostre quais os clientes tem idade superior a média.

~~~sql
SELECT ClienteNome,
       YEAR(GETDATE()) - YEAR(ClienteNascimento) AS idade
FROM Clientes
WHERE YEAR(GETDATE()) - YEAR(ClienteNascimento) >
(
    SELECT AVG(YEAR(GETDATE()) - YEAR(ClienteNascimento)) AS IDADE
    FROM Clientes
);
~~~

2) Mostre qual agência tem quantidade de clientes acima da média.

~~~sql
SELECT AgenciaNome,
       COUNT(ClienteCodigo) AS QDTE
FROM Contas
    INNER JOIN Agencias
        ON Agencias.AgenciaCodigo = Contas.AgenciaCodigo
GROUP BY AgenciaNome
HAVING COUNT(ClienteCodigo) >
(
    SELECT COUNT(DISTINCT ClienteCodigo) / COUNT(DISTINCT AgenciaCodigo)
    FROM Contas
);
~~~



3) Mostre o nome da agência o saldo total, o mínimo, o máximo e a quantidade de clientes de cada agência.

~~~sql
SELECT AgenciaNome,
       SUM(ContaSaldo) AS TOTAL,
       MIN(ContaSaldo) AS MINIMO,
       MAX(ContaSaldo) AS MAXIMO,
       COUNT(Contas.ClienteCodigo) AS QTDE_CLIENTES
FROM Contas
    INNER JOIN dbo.Agencias
        ON Agencias.AgenciaCodigo = Contas.AgenciaCodigo
GROUP BY dbo.Agencias.AgenciaNome;
--ATENCAO AQUI PARA COUNT(*) E COUNT(DISTINT)
~~~



4) Mostre o percentual que cada agencia representa no saldo total do banco.

~~~sql
SELECT AgenciaNome,
       SUM(ContaSaldo) /
       (
           SELECT SUM(ContaSaldo) FROM dbo.Contas
       ) * 100 AS PERCENTUAL
FROM Contas
    INNER JOIN dbo.Agencias
        ON Agencias.AgenciaCodigo = Contas.AgenciaCodigo
GROUP BY dbo.Agencias.AgenciaNome;
~~~



5) Mostre as duas cidades que tem o maior saldo total.

~~~sql
SELECT TOP 2
       AgenciaCidade,
       SUM(ContaSaldo) AS SALDO_TOTAL
FROM Contas
    INNER JOIN Agencias
        ON Agencias.AgenciaCodigo = Contas.AgenciaCodigo
GROUP BY AgenciaCidade
ORDER BY 2 DESC;
~~~



6) Mostre qual a agência tem o maior montante de empréstimo.

~~~sql
SELECT TOP 1
       AgenciaCidade,
       Emprestimos.EmprestimoTotal
FROM dbo.Emprestimos
    INNER JOIN Agencias
        ON Agencias.AgenciaCodigo = Emprestimos.AgenciaCodigo
ORDER BY 2 DESC;
~~~

7) Mostre qual o menor valor devido, o maior e o total devido da tabela devedor.

~~~sql
SELECT MIN(DevedorSaldo) AS MINIMO,
       MAX(DevedorSaldo) AS MAXIMO,
       SUM(DevedorSaldo) AS TOTAL
FROM dbo.Devedores;
~~~

8) Mostre o nome do cliente, se ele tem cartão de crédito, apenas do cliente que é o maior devedor.

~~~sql
SELECT TOP 1 --Experimente remover o TOP 1 para conferir o resultado
       ClienteNome,
       CASE
           WHEN dbo.CartaoCredito.ClienteCodigo IS NULL THEN
               'NÃO TEM CARTÃO CRÉDITO'
           ELSE
               'TEM CARTÃO CRÉDITO'
       END AS 'CARTAO',
       DevedorSaldo
FROM dbo.Clientes
    INNER JOIN dbo.Devedores
        ON Devedores.ClienteCodigo = Clientes.ClienteCodigo
    LEFT JOIN dbo.CartaoCredito
        ON CartaoCredito.ClienteCodigo = Clientes.ClienteCodigo
ORDER BY 3 DESC;
~~~

9) Mostre o nome do cliente, a idade, o saldo total em conta, seu total devido, seu total emprestado e se tem cartão de crédito ou não. Os valores nulos devem aparecer como 0.00. A ordenação dever ser sempre pelo maior devedor.

~~~sql
SELECT Clientes.ClienteNome,
       DATEDIFF(YEAR, Clientes.ClienteNascimento, GETDATE()) AS IDADE,
       ISNULL(Devedores.DevedorSaldo, 0) AS DevedorSaldo,
       ISNULL(Emprestimos.EmprestimoTotal, 0) AS EmprestimoTotal,
       CASE
           WHEN CartaoCredito.CartaoCodigo IS NULL THEN
               'NÃO TEM'
           ELSE
               'TEM'
       END AS CARTAOCREDITO
FROM Clientes
    LEFT JOIN Devedores
        ON Devedores.ClienteCodigo = Clientes.ClienteCodigo
    LEFT JOIN Emprestimos
        ON Emprestimos.ClienteCodigo = Clientes.ClienteCodigo
    LEFT JOIN CartaoCredito
        ON CartaoCredito.ClienteCodigo = Clientes.ClienteCodigo
ORDER BY 3 DESC;
~~~



10) Utilizando a questão anterior, inclua o sexo do cliente e mostre também a diferença entre o que ele emprestou e o que ele está devendo.

~~~sql
SELECT Clientes.ClienteNome,
       DATEDIFF(YEAR, Clientes.ClienteNascimento, GETDATE()) AS IDADE,
       ISNULL(Devedores.DevedorSaldo, 0) AS DevedorSaldo,
       ISNULL(Emprestimos.EmprestimoTotal, 0) AS EmprestimoTotal,
       CASE
           WHEN .CartaoCredito.CartaoCodigo IS NULL THEN
               'NÃO TEM'
           ELSE
               'TEM'
       END AS CARTAOCREDITO,
       CASE
           WHEN ClienteNome LIKE '%a' THEN
               'FEMININO'
           ELSE
               'MASCULINO'
       END AS SEXO,
       ISNULL((Emprestimos.EmprestimoTotal - DevedorSaldo), 0) AS DIFERENÇA
FROM Clientes
    LEFT JOIN Devedores
        ON Devedores.ClienteCodigo = Clientes.ClienteCodigo
    LEFT JOIN Emprestimos
        ON Emprestimos.ClienteCodigo = Clientes.ClienteCodigo
    LEFT JOIN CartaoCredito
        ON CartaoCredito.ClienteCodigo = Clientes.ClienteCodigo
ORDER BY 3 DESC;
~~~

11) Insira um novo cliente chamado Silvio Santos, crie uma conta para ele com saldo de R$ 500,00 na agência Beira Mar. Cadastre um cartão de crédito com limite de 5000,00.

~~~sql
INSERT Clientes
(
    ClienteNome,
    ClienteRua,
    ClienteCidade,
    ClienteNascimento
)
VALUES
('Silvio Santos', 'Rua João Colin, 1234', 'Joinville', '1980-01-01');

SELECT @@IDENTITY; --RETORNA O CÓDIGO DO CLIENTE GERADO PELO AUTO INCREMENTO  IDENTITY

INSERT Contas
(
    AgenciaCodigo,
    ContaNumero,
    ClienteCodigo,
    ContaSaldo,
    ContaAbertura
)
OUTPUT INSERTED.* --RETORNA OS REGISTROS INSERIDOS NA TABELA
VALUES
(5, 'C-999', 14, 500, '2016-01-01');

INSERT CartaoCredito
(
    AgenciaCodigo,
    ClienteCodigo,
    CartaoCodigo,
    CartaoLimite
)
VALUES
(5, 14, '1234-1234-1234-1234', 5000);
~~~


12) Altere a rua do cliente Ana para Rua da Univille.

~~~sql
UPDATE dbo.Clientes
SET ClienteRua = 'Rua da Univille'
WHERE ClienteNome = 'Ana';
~~~


13) Apague todos os registros da cliente Vânia.

~~~sql
DECLARE @ClienteCodigo INT =
        (
            SELECT ClienteCodigo FROM dbo.Clientes WHERE ClienteNome = 'Vânia'
        );

DELETE FROM dbo.Emprestimos
WHERE ClienteCodigo = @ClienteCodigo;
DELETE FROM dbo.Devedores
WHERE ClienteCodigo = @ClienteCodigo;
DELETE FROM dbo.Depositantes
WHERE ClienteCodigo = @ClienteCodigo;
DELETE FROM dbo.CartaoCredito
WHERE ClienteCodigo = @ClienteCodigo;
DELETE FROM dbo.Contas
WHERE ClienteCodigo = @ClienteCodigo;
DELETE FROM dbo.Clientes
WHERE ClienteCodigo = @ClienteCodigo;
~~~

14) Mostre nome e sobrenome e se o cliente for homem, mostre Sr. e se for mulher Sra. na frente do nome. Use o MinhaCaixa\_Beta para resolver essa questão.

~~~sql
USE MinhaCaixa_Beta;
GO
SELECT CASE
           WHEN ClienteSexo = 'M' THEN
               'Sr. ' + dbo.Clientes.ClienteNome + ' ' + dbo.Clientes.ClienteSobrenome
           ELSE
               'Sra. ' + dbo.Clientes.ClienteNome + ' ' + dbo.Clientes.ClienteSobrenome
       END AS Cliente
FROM dbo.Clientes;
~~~


15) Mostre os bairros que tem mais clientes.

~~~sql
USE MinhaCaixa_Beta;
GO
SELECT COUNT(dbo.Clientes.ClienteCodigo) AS Quantidade,
       dbo.Clientes.ClienteBairro
FROM dbo.Clientes
GROUP BY dbo.Clientes.ClienteBairro
ORDER BY 1 DESC;
~~~


16) Mostre a renda de cada cliente convertida em dólar.

~~~sql
USE MinhaCaixa_Beta;
GO
SELECT ClienteNome + ' ' + ClienteSobrenome AS Cliente,
       CONVERT(DECIMAL(10, 2), Clientes.ClienteRendaAnual / 3.25) AS RENDADOLAR
FROM dbo.Clientes;
~~~

17) Mostre o nome do cliente, o número da conta, o saldo da conta, apenas para os 15 melhores clientes.

~~~sql
USE MinhaCaixa_Beta;
GO
SELECT TOP 15
       ClienteNome + ' ' + ClienteSobrenome AS Cliente,
       Contas.ContaNumero,
       SUM(MovimentoValor * MovimentoTipo) AS Saldo
FROM dbo.Clientes
    INNER JOIN dbo.Contas
        ON Contas.ClienteCodigo = Clientes.ClienteCodigo
    INNER JOIN dbo.Movimentos
        ON Movimentos.ContaNumero = Contas.ContaNumero
GROUP BY ClienteNome + ' ' + ClienteSobrenome,
         Contas.ContaNumero
ORDER BY 3 DESC;
~~~


18) Mostre quais são os 5 dias com maior movimento \(valor\) no banco.

~~~sql
USE MinhaCaixa_Beta;
GO
SELECT TOP 5
       DAY(Movimentos.MovimentoData) AS DIA,
       SUM(dbo.Movimentos.MovimentoValor * dbo.Movimentos.MovimentoTipo) AS VALOR
FROM dbo.Movimentos
GROUP BY DAY(Movimentos.MovimentoData)
ORDER BY 2 DESC;
~~~

19) Crie uma função que receba o código do estado civil e mostre ele por extenso.

20) Crie uma função que receba o código do sexo e mostre ele por extenso.

21) Crie um procedure que receba o número da conta e cadastre um cartão de crédito com limite de R$ 500 para o cliente caso ele não tenha \(MinhaCaixa\).

22) Use o script abaixo para criar uma procedure que receba a matricula, disciplina, ano e calcule o total de pontos e a média do aluno.

~~~sql
CREATE TABLE Notas
(
    Matricula INT,
    Materia CHAR(3),
    Ano INT,
    Nota1 FLOAT,
    Nota2 FLOAT,
    Nota3 FLOAT,
    Nota4 FLOAT,
    TotalPontos FLOAT,
    MediaFinal FLOAT
);
INSERT Notas
(
    Matricula,
    Materia,
    Ano,
    Nota1,
    Nota2,
    Nota3,
    Nota4
)
VALUES
(1, 'BDA', 2016, 7, 7, 7, 7);
~~~

23) Use o script abaixo para criar duas procedures: Uma procedure para cadastrar os alunos em duas matérias \(BDA e PRG\). Exemplo: exec procedure @matricula, @materia, @ano \(matricular 6 alunos\) Uma procedure que receba a matricula, disciplina, ano, bimestre, aulas dadas, notas e faltas. Quando a condição dentro da procedure identificar que é o quarto bimestre calcule o total de pontos, total de faltas, percentual de frequencia,a média do aluno e calcule o resultado final, A, E ou R. Exemplo: exec procedure @matricula, @materia, @ano, 1, 32, 7, 0

~~~sql
CREATE TABLE Notas
(
    Matricula INT,
    Materia CHAR(3),
    Ano INT,
    Aulas1 INT,
    Aulas2 INT,
    Aulas3 INT,
    Aulas4 INT,
    Nota1 FLOAT,
    Nota2 FLOAT,
    Nota3 FLOAT,
    Nota4 FLOAT,
    Falta1 INT,
    Falta2 INT,
    Falta3 INT,
    Falta4 INT,
    TotalPontos FLOAT,
    TotalFaltas INT,
    TotalAulas INT,
    MediaFinal FLOAT,
    PercentualFrequencia FLOAT,
    Resultado CHAR(1)
);
~~~



