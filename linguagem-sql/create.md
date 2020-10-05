# CREATE

* Comando utilizado para criar os principais objetos em um banco de dados.

Neste tópico vamos trabalhar com as diversas variações do comando `CREATE` relacionados ao início dos trabalhos com criação das entidades no banco de dados.

O Primeiro comando é o `CREATE DATABASE`, que cria o Banco de dados e suas dependências, como arquivos e metadados dentro do sistema. Vale lembrar que alguns sistemas gerenciadores de bancos de dados podem implementar maneiras diferentes de tratar os bancos de dados ou espaços de trabalho de cada usuário ou sistema. Sugiro a leitura do link abaixo, que explica como o Oracle trabalha, ao contrário do SQL Server que vemos em sala de aula.

[http://www.oracle.com/technetwork/pt/articles/database-performance/introducao-conceito-de-tablespaces-495850-ptb.html](http://www.oracle.com/technetwork/pt/articles/database-performance/introducao-conceito-de-tablespaces-495850-ptb.html)

No nosso banco de dados de Exemplo temos a criação básica de um banco de dados e a criação de uma tabela chamada `Clientes`. Depois usamos o comando use para posicionar a execução dos comandos no banco de dados `MinhaCaixa`.

```text
CREATE DATABASE MinhaCaixa;

use MinhaCaixa;

CREATE TABLE Clientes (
  ClienteCodigo int,
  ClienteNome varchar(20)
);
```

 Podemos ter variações do comando `CREATE TABLE` de acordo com a necessidade. Abaixo temos diversas implementações do comando `CREATE` e suas CONSTRAINT´s.

### CONSTRAINT PRIMARY KEY & IDENTITY

 Nesse exemplo adicionamos uma chave primária ao campo `ClienteCodigo` e configuramos a propriedade `IDENTITY` que vai gerar um número com incremento de \(um\) a cada inserção na tabela `Clientes`. Você pode personalizar o incremento de acordo com sua necessidade, neste exemplo temos \(1,1\) iniciando em um e incrementando um.

```text
CREATE TABLE Clientes (
  ClienteCodigo int IDENTITY (1,1) CONSTRAINT PK_Cliente PRIMARY KEY,
  ...
);
```

 Nesse exemplo adicionamos uma chave primária composta.

```text
CREATE TABLE Clientes (
  ClienteCodigo int IDENTITY (1,1) ,
  ClienteCPF(11)
      CONSTRAINT PK_Cliente PRIMARY KEY (ClienteCodigo,ClienteCPF)
);
```

### CONSTRAINT FOREIGN KEY

 Neste exemplo temos a criação da `FOREIGN KEY` dentro do bloco de comando `CREATE`. Se tratando de uma chave estrangeira temos que tomar o cuidado de referenciar tabelas que já existem para evitar erros. Repare que no comando abaixo estamos criando uma tabela nova chamada `Contas` e especificando que o código de cliente deverá estar cadastrado na tabela de Cliente, portanto deve existir antes uma tabela `Cliente` que será referenciada nessa chave estrangeira `FOREIGN KEY`. Repare que sempre damos um nome para a `CONSTRAINT`, isso é uma boa prática, para evitar que o sistema dê nomes automáticos.

```text
CREATE TABLE Contas
  (
  AgenciaCodigo int,
  ContaNumero VARCHAR (10) CONSTRAINT PK_CONTA PRIMARY KEY,
  ClienteCodigo int,
  ContaSaldo MONEY,
  ContaAbertura datetime
  CONSTRAINT FK_CLIENTES_CONTAS FOREIGN KEY (ClienteCodigo) REFERENCES Clientes(ClienteCodigo)
  );
```

### ALTER TABLE ADD COLUMN

```text
ALTER TABLE Pessoas ADD PessoaSexo CHAR(2);
```

### ALTER TABLE ADD CONSTRAINT

 Também podemos adicionar CONSTRAINT´s através do comando `ALTER TABLE ... ADD CONSTRAINT`. Geralmente após criar todas as entidades podemos então criar as restrições entre elas.

```text
ALTER TABLE Contas ADD CONSTRAINT FK_CLIENTES_CONTAS FOREIGN KEY (ClienteCodigo)
REFERENCES Clientes(ClienteCodigo);
```

### CONSTRAINT´s de domínio

```text
ALTER TABLE Clientes ADD CONSTRAINT chk_cliente_saldo CHECK ([ClienteNascimento] < GETDATE() AND ClienteNome <> 'Sara');
```

 Abaixo a mensagem de tentativa de violação da `CONSTRAINT` acima.

```text
The INSERT statement conflicted with the CHECK constraint "chk_cliente_saldo". The conflict occurred in database "MinhaCaixa", table "dbo.Clientes".
```

 Apenas checando uma condição, data de nascimento menor que data atual. No SQL Server para pegarmos a data atual usamos `GETDATE()`:

```text
ALTER TABLE Clientes ADD CONSTRAINT TESTE CHECK ([ClienteNascimento] < GETDATE());
```



