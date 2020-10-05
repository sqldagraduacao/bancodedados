# EXERCÍCIOS Procedure

1. Neste exercício vamos criar um banco de dados para armazenar os dados dos alunos de uma universidade. Além de desenhar o diagrama, criar o banco de dados e seus objetos, você deverá criar os scripts de população básica. Em seguida deverá criar as procedures que irão executar as operações de manipulação das notas e faltas. 

2. O que falta complementar nesse exercício

2.1 Falta criar uma procedure que realize a matricula do Aluno com base no curso escolhido. Isso significa que se o aluno escolher um determinado curso, devem ser lançadas para ele as disciplinas daquele curso na tabela de matricula.

2.2 Falta criar também dentro da procedure sp_Cadastra_Notas a lógica de apuração dos resultados, ou seja, se o aluno será aprovado, reprovado ou ficará em exame. Se existe a possiblidade de exame, então teremos que ter uma procedure ou lógica para receber a nota do exame do aluno e processar o resultado.

Abaixo uma sugestão de parte da solução:

~~~~sql
    USE master;
    GO
    CREATE DATABASE Universidade;
    GO
    USE Universidade;
    GO
    CREATE TABLE ALUNOS
    (
        MATRICULA INT NOT NULL IDENTITY
            CONSTRAINT PK_ALUNO PRIMARY KEY,
        NOME VARCHAR(50) NOT NULL
    );
	INSERT ALUNOS
    (
        NOME
    )
    VALUES
    ('Pedro');
    GO
	SELECT * FROM ALUNOS;
	GO
    CREATE TABLE CURSOS
    (
        CURSO CHAR(3) NOT NULL
            CONSTRAINT PK_CURSO PRIMARY KEY,
        NOME VARCHAR(50) NOT NULL
    );
	INSERT CURSOS
    (
        CURSO,
        NOME
    )
    VALUES
    ('SIS', 'SISTEMAS'),
    ('ENG', 'ENGENHARIA');
    GO
	SELECT * FROM CURSOS;
    GO
    CREATE TABLE PROFESSOR
    (
        PROFESSOR INT IDENTITY NOT NULL
            CONSTRAINT PK_PROFESSOR PRIMARY KEY,
        NOME VARCHAR(50) NOT NULL
    );
    GO
	INSERT PROFESSOR
    (
        NOME
    )
    VALUES
    ('DORNEL'),
    ('WALTER');
    GO
	SELECT * FROM PROFESSOR;
	GO
    CREATE TABLE MATERIAS
    (
        SIGLA CHAR(3) NOT NULL,
        NOME VARCHAR(50) NOT NULL,
        CARGAHORARIA INT NOT NULL,
        CURSO CHAR(3) NOT NULL,
        PROFESSOR INT
            CONSTRAINT PK_MATERIA
            PRIMARY KEY (
                            SIGLA,
                            CURSO,
                            PROFESSOR
                        )
            CONSTRAINT FK_CURSO
            FOREIGN KEY (CURSO) REFERENCES CURSOS (CURSO),
        CONSTRAINT FK_PROFESSOR
            FOREIGN KEY (PROFESSOR)
            REFERENCES PROFESSOR (PROFESSOR)
    );
    GO
    INSERT MATERIAS
    (
        SIGLA,
        NOME,
        CARGAHORARIA,
        CURSO,
        PROFESSOR
    )
    VALUES
    ('BDA', 'BANCO DE DADOS', 144, 'SIS', 1),
    ('PRG', 'PROGRAMAÇÃO', 144, 'SIS', 2);
    GO
    INSERT MATERIAS
    (
        SIGLA,
        NOME,
        CARGAHORARIA,
        CURSO,
        PROFESSOR
    )
    VALUES
    ('BDA', 'BANCO DE DADOS', 144, 'ENG', 1),
    ('PRG', 'PROGRAMAÇÃO', 144, 'ENG', 2);
    GO
	SELECT * FROM MATERIAS;
	GO
    CREATE TABLE MATRICULA
    (
        MATRICULA INT,
        CURSO CHAR(3),
        MATERIA CHAR(3),
        PROFESSOR INT,
        PERLETIVO INT,
        N1 FLOAT,
        N2 FLOAT,
        N3 FLOAT,
        N4 FLOAT,
        TOTALPONTOS FLOAT,
        MEDIA FLOAT,
        F1 INT,
        F2 INT,
        F3 INT,
        F4 INT,
        TOTALFALTAS INT,
        PERCFREQ FLOAT,
        RESULTADO VARCHAR(20)
            CONSTRAINT PK_MATRICULA
            PRIMARY KEY (
                            MATRICULA,
                            CURSO,
                            MATERIA,
                            PROFESSOR,
                            PERLETIVO
                        )
        CONSTRAINT FK_ALUNOS_MATRICULA
            FOREIGN KEY (MATRICULA)
            REFERENCES ALUNOS (MATRICULA),
        CONSTRAINT FK_CURSOS_MATRICULA
            FOREIGN KEY (CURSO)
            REFERENCES CURSOS (CURSO),
       CONSTRAINT FK_PROFESSOR_MATRICULA
            FOREIGN KEY (PROFESSOR)
            REFERENCES PROFESSOR (PROFESSOR),
		CONSTRAINT FK_MATERIAS_MATRICULA
			FOREIGN KEY (MATERIA,CURSO,PROFESSOR)
			REFERENCES  MATERIAS (SIGLA,CURSO,PROFESSOR)
    );

~~~~

 Exemplo de execução da PROCEDURE para inserir \(atualizar\) as notas

~~~~sql
    EXEC sp_CadastraNotas @MATRICULA = 4,      -- int
						  @CURSO = 'ENG',      -- char(3)
						  @MATERIA = 'BDA',    -- char(3)
						  @PERLETIVO = '2018', -- char(4)
						  @NOTA = 7.0,         -- float
						  @FALTA = 2,
						  @PARAMETRO = 4;      -- int
~~~~

Exemplo de INSERT com SELECT (Esse trecho de código é usado para que; quando o aluno for efetuar a matricula dele no curso, automaticamente sejam cadastradas as disciplinas na tabela matricula para esse aluno)

~~~~sql

    INSERT MATRICULA
    (
            MATRICULA,
            CURSO,
            MATERIA,
            PROFESSOR,
            PERLETIVO

    )
    SELECT 1 AS MATRICULA, CURSO, SIGLA,PROFESSOR, 
        YEAR(GETDATE()) FROM MATERIAS WHERE CURSO ='ENG'

~~~~
	
Exemplo de PROCEDURE para lançar as notas dos alunos

~~~~sql

    CREATE PROCEDURE sp_CadastraNotas
    (
        @MATRICULA INT,
        @CURSO CHAR(3),
        @MATERIA CHAR(3),
        @PERLETIVO CHAR(4),
        @NOTA FLOAT,
        @FALTA INT,
        @PARAMETRO INT
    )
    AS
    BEGIN

        IF @PARAMETRO = 1
        BEGIN

            UPDATE MATRICULA
            SET N1 = @NOTA,
                F1 = @FALTA,
                TOTALPONTOS = @NOTA,
                TOTALFALTAS = @FALTA,
                MEDIA = @NOTA
            WHERE MATRICULA = @MATRICULA
                  AND CURSO = @CURSO
                  AND MATERIA = @MATERIA
                  AND PERLETIVO = @PERLETIVO;
        END;

        ELSE IF @PARAMETRO = 2
        BEGIN

            UPDATE MATRICULA
            SET N2 = @NOTA,
                F2 = @FALTA,
                TOTALPONTOS = @NOTA + N1,
                TOTALFALTAS = @FALTA + F1,
                MEDIA = (@NOTA + N1) / 2
            WHERE MATRICULA = @MATRICULA
                  AND CURSO = @CURSO
                  AND MATERIA = @MATERIA
                  AND PERLETIVO = @PERLETIVO;
        END;

        ELSE IF @PARAMETRO = 3
        BEGIN

            UPDATE MATRICULA
            SET N3 = @NOTA,
                F3 = @FALTA,
                TOTALPONTOS = @NOTA + N1 + N2,
                TOTALFALTAS = @FALTA + F1 + F2,
                MEDIA = (@NOTA + N1 + N2) / 3
            WHERE MATRICULA = @MATRICULA
                  AND CURSO = @CURSO
                  AND MATERIA = @MATERIA
                  AND PERLETIVO = @PERLETIVO;
        END;

        ELSE IF @PARAMETRO = 4
        BEGIN

            DECLARE @RESULTADO VARCHAR(50),
                    @FREQUENCIA FLOAT,
                    @MEDIAFINAL FLOAT;

            DECLARE @CARGAHORA INT 
            SET @CARGAHORA = (SELECT CARGAHORA 
                                FROM MATERIAS WHERE COD = @MATERIA)

            UPDATE MATRICULA
            SET N4 = @NOTA,
                F4 = @FALTA,
                TOTALPONTOS = @NOTA + N1 + N2 + N3,
                TOTALFALTAS = @FALTA + F1 + F2 + F3,
                MEDIA = (@NOTA + N1 + N2 + N3) / 4,
                MEDIAFINAL = (@NOTA + N1 + N2 + N3) / 4,
                PERCFREQ = 100 -( ((@FALTA + F1 + F2 + F3)*144 )/100)
                       WHERE MATRICULA = @MATRICULA
                  AND CURSO = @CURSO
                  AND MATERIA = @MATERIA
                  AND PERLETIVO = @PERLETIVO;


        END;

        SELECT *
        FROM MATRICULA
        WHERE MATRICULA = @MATRICULA;
    END;

~~~~

Exemplo de execução da PROCEDURE para inserir (atualizar) as notas

~~~~sql

    EXEC sp_CadastraNotas @MATRICULA = 4,      -- int
                          @CURSO = 'ENG',      -- char(3)
                          @MATERIA = 'BDA',    -- char(3)
                          @PERLETIVO = '2018', -- char(4)
                          @NOTA = 7.0,         -- float
                          @FALTA = 2,
                          @PARAMETRO = 4;      -- int
			
~~~~
