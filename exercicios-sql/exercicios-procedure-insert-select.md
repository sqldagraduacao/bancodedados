# EXERCÍCIOS Procedure

1. Neste exercício vamos criar um banco de dados para armazenar os dados dos alunos de uma universidade. Além de desenhar o diagrama, criar o banco de dados e seus objetos, você deverá criar os scripts de população básica. Em seguida deverá criar as procedures que irão executar as operações de manipulação das notas e faltas. Abaixo uma sugestão de parte da solução:

~~~~sql
        CREATE TABLE pedidos
    (
    idpedido INT,
    idproduto INT,
    valorpedido float
    )

    CREATE TABLE itenspedido
    (
    idpedido INT,
    iditem int,
    idproduto int
    )

    CREATE TABLE itens
    (
    iditem INT,
    nome varchar(50)
    )
    INSERT itens
    (
        iditem,
        nome
    )
    VALUES
    (   1, -- iditem - int
        'AR CONDICIONADO' -- nome - varchar(50)
        )


    CREATE TABLE subitens
    (
    idsubitem INT,
    iditem INT,
    nomesubitem VARCHAR(50)
    )
    INSERT subitens
    (
        idsubitem,
        iditem,
        nomesubitem
    )
    VALUES
    (   2, -- idsubitem - int
        1, -- iditem - int
        'MOTOR' -- nomesubitem - varchar(50)
        )



        SELECT * FROM itens
        SELECT * FROM subitens

        SELECT * FROM PEDIDOS


        INSERT pedidos
        (
            idpedido,
            idproduto,
            valorpedido
        )
        VALUES
        (   1,  -- idpedido - int
            1,  -- idproduto - int
            1000.00 -- valorpedido - float
            )

            DECLARE @produto INT
            SET @produto = (SELECT idproduto FROM pedidos WHERE idpedido =1)

            SELECT @produto AS 'AR COND'

            INSERT itenspedido
            (
                idpedido,
                iditem,
                idproduto
            )
    SELECT IDPEDIDO=1, idsubitem, iditem 
    FROM subitens WHERE iditem = 1--@CURSO

            --VALUES
            --(   0, -- idpedido - int
            --    0, -- iditem - int
            --    0  -- idproduto - int
            --    )
~~~~

