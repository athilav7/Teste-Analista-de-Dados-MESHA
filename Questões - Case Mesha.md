#Resolução de questões - Via Jupyter - sqalchemy


```python
!pip install sqlalchemy
```

    Requirement already satisfied: sqlalchemy in c:\users\renat\anaconda3\lib\site-packages (1.4.39)
    Requirement already satisfied: greenlet!=0.4.17 in c:\users\renat\anaconda3\lib\site-packages (from sqlalchemy) (2.0.1)
    


```python
from sqlalchemy import create_engine, text

# Credenciais e detalhes do banco de dados
connection_url = f"mssql+pyodbc://B3NJ4M/importCSV?trusted_connection=yes&driver=ODBC+Driver+17+for+SQL+Server"
engine = create_engine(connection_url)

```


```python
from sqlalchemy import create_engine

# Detalhes do SQL Server e banco de dados
server_name = 'B3NJ4M'
database_name = 'importCSV'

# Configuração da conexão usando autenticação do Windows
connection_url = f"mssql+pyodbc://{server_name}/{database_name}?trusted_connection=yes&driver=ODBC+Driver+17+for+SQL+Server"

# Criação da engine
engine = create_engine(connection_url)

```


```python
#Questão 1 - Qual a escola com a maior média de notas?
result = engine.execute("""SELECT TOP 1
    NO_MUNICIPIO_PROVA,
    (CAST(NU_NOTA_CN AS FLOAT) + CAST(NU_NOTA_CH AS FLOAT) + 
	CAST(NU_NOTA_LC AS FLOAT) + CAST(NU_NOTA_MT AS FLOAT) + 
	CAST(NU_NOTA_REDACAO AS FLOAT)) / 5 AS media
FROM dbo.data
ORDER BY media DESC;
""").fetchone()
print(result)

#Como não tem código correspondente a escola, e não havia município registrado, peguei o código corresponte ao nome 
#do local que o aluno fez a prova.
```

    ('Piracicaba', 858.5799999999999)
    


```python
#Questão 2 - Qual o aluno com a maior média de notas e o valor dessa média?

result = engine.execute("""SELECT TOP 1
    NU_INSCRICAO,
    (CAST(NU_NOTA_CN AS FLOAT) + CAST(NU_NOTA_CH AS FLOAT) + 
	CAST(NU_NOTA_LC AS FLOAT) + CAST(NU_NOTA_MT AS FLOAT) + 
	CAST(NU_NOTA_REDACAO AS FLOAT)) / 5 AS media
FROM dbo.data
ORDER BY media DESC;
""").fetchone()
print(result)
```

    ('200005996961', 858.5799999999999)
    


```python
#Questão 3 - Qual a média geral?
#Diferente do POWER BI, Calculei a média geral apenas dos alunos que foram nos dois dias de prova.

result = engine.execute("""SELECT 
    AVG(
        CASE WHEN TP_PRESENCA_CH <> 0 AND TP_PRESENCA_CN <> 0 AND TP_PRESENCA_MT <> 0 AND TP_PRESENCA_LC <> 0
             THEN (CAST(NU_NOTA_CN AS FLOAT) + CAST(NU_NOTA_CH AS FLOAT) + CAST(NU_NOTA_LC AS FLOAT) + CAST(NU_NOTA_MT AS FLOAT) + CAST(NU_NOTA_REDACAO AS FLOAT)) / 5
             ELSE NULL
        END
    ) AS media_geral
FROM dbo.data;
""").fetchone()
print(result)

```

    (526.4255358448479,)
    


```python
#Questão 4 - Qual o % de Ausentes?

result = engine.execute("""SELECT
    (COUNT(*) * 100.0) / (SELECT COUNT(*) FROM dbo.data) AS PorcentagemAusentes
FROM dbo.data
WHERE
    TP_PRESENCA_CN = 0
    OR TP_PRESENCA_CH = 0
    OR TP_PRESENCA_MT = 0
    OR TP_PRESENCA_LC = 0

    
""").fetchone()
print(result)
```

    (Decimal('55.208210670073'),)
    


```python
#Questão 5 - Qual o número total de Inscritos?
result = engine.execute("""SELECT COUNT(*)
    FROM dbo.data;
""").fetchall()
print(result)
```

    [(5783109,)]
    


```python
#Questão 6 - Qual a média por disciplina?
result = engine.execute("""SELECT 'NU_NOTA_CN' AS Disciplina, AVG(CASE WHEN ISNUMERIC(NU_NOTA_CN) = 1 THEN TRY_CAST(NU_NOTA_CN AS FLOAT) ELSE NULL END) AS Media
FROM dbo.data
UNION
SELECT 'NU_NOTA_CH' AS Disciplina, AVG(CASE WHEN ISNUMERIC(NU_NOTA_CH) = 1 THEN TRY_CAST(NU_NOTA_CH AS FLOAT) ELSE NULL END) AS Media
FROM dbo.data
UNION
SELECT 'NU_NOTA_LC' AS Disciplina, AVG(CASE WHEN ISNUMERIC(NU_NOTA_LC) = 1 THEN TRY_CAST(NU_NOTA_LC AS FLOAT) ELSE NULL END) AS Media
FROM dbo.data
UNION
SELECT 'NU_NOTA_MT' AS Disciplina, AVG(CASE WHEN ISNUMERIC(NU_NOTA_MT) = 1 THEN TRY_CAST(NU_NOTA_MT AS FLOAT) ELSE NULL END) AS Media
FROM dbo.data
UNION
SELECT 'NU_NOTA_REDACAO' AS Disciplina, AVG(CASE WHEN ISNUMERIC(NU_NOTA_REDACAO) = 1 THEN TRY_CAST(NU_NOTA_REDACAO AS FLOAT) ELSE NULL END) AS Media
FROM dbo.data;
""").fetchall()
print(result)

```

    [('NU_NOTA_CH', 511.15220163100145), ('NU_NOTA_REDACAO', 573.4127241171473), ('NU_NOTA_CN', 490.4097924879871), ('NU_NOTA_LC', 523.8009359364424), ('NU_NOTA_MT', 520.578334821981)]
    


```python
#Questão 7 - Qual a média por Sexo?

result = engine.execute("""WITH medias AS (
    SELECT
        TP_SEXO,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_CN) = 1 THEN TRY_CAST(NU_NOTA_CN AS FLOAT) ELSE NULL END) AS media_CN,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_CH) = 1 THEN TRY_CAST(NU_NOTA_CH AS FLOAT) ELSE NULL END) AS media_CH,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_LC) = 1 THEN TRY_CAST(NU_NOTA_LC AS FLOAT) ELSE NULL END) AS media_LC,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_MT) = 1 THEN TRY_CAST(NU_NOTA_MT AS FLOAT) ELSE NULL END) AS media_MT,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_REDACAO) = 1 THEN TRY_CAST(NU_NOTA_REDACAO AS FLOAT) ELSE NULL END) AS media_REDACAO
    FROM dbo.data
    GROUP BY TP_SEXO
)
SELECT
    TP_SEXO,
    AVG(media_CN + media_CH + media_LC + media_MT + media_REDACAO) / 5 AS media_total
FROM medias
GROUP BY TP_SEXO;

""").fetchall()
print(result)

```

    [('F', 518.7388234458074), ('M', 531.6950796901564)]
    


```python
#Questão 8 - Qual a média por Etnia?
result = engine.execute("""WITH medias AS (
    SELECT
        TP_COR_RACA,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_CN) = 1 THEN TRY_CAST(NU_NOTA_CN AS FLOAT) ELSE NULL END) AS media_CN,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_CH) = 1 THEN TRY_CAST(NU_NOTA_CH AS FLOAT) ELSE NULL END) AS media_CH,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_LC) = 1 THEN TRY_CAST(NU_NOTA_LC AS FLOAT) ELSE NULL END) AS media_LC,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_MT) = 1 THEN TRY_CAST(NU_NOTA_MT AS FLOAT) ELSE NULL END) AS media_MT,
        AVG(CASE WHEN ISNUMERIC(NU_NOTA_REDACAO) = 1 THEN TRY_CAST(NU_NOTA_REDACAO AS FLOAT) ELSE NULL END) AS media_REDACAO
    FROM dbo.data
    GROUP BY TP_COR_RACA
)
SELECT
    TP_COR_RACA,
    AVG(media_CN + media_CH + media_LC + media_MT + media_REDACAO) / 5 AS media_total
FROM medias
GROUP BY TP_COR_RACA;
""").fetchall()
print(result)

```

    [('2', 498.36797051183265), ('5', 470.6458236604224), ('1', 553.8428311659322), ('4', 522.1053778667113), ('0', 530.5554930393689), ('3', 506.06661319761315)]
    

 



```python

```
