# Evidência 09 — Preparação complementar e replicação de 1.000 clientes

**Data:** 14/11/2026. **Ambiente:** PostgreSQL 16.14 no Windows; primário 5442, réplica 5443.

## Contexto

Ao tentar contar a tabela `clientes` após o incidente D, a consulta retornou `ERRO: não existe a relação "clientes"`. A inspeção de `\dt` no banco `postgres` exibiu somente a tabela `public.teste`.

Portanto, a tabela `clientes` foi criada **após** a reprodução e recuperação do incidente D, como preparação complementar exigida na atividade. Esses registros não são uma prova retroativa do RPO da simulação anterior.

## Comandos executados no primário

```sql
CREATE TABLE clientes (id serial PRIMARY KEY, nome text, email text);

INSERT INTO clientes (nome, email)
SELECT 'cliente ' || g, 'c' || g || '@loja.com'
FROM generate_series(1, 1000) g;
```

Saída confirmada pela estudante:

```text
CREATE TABLE
INSERT 0 1000
```

## Consulta de confirmação na réplica

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5443 -U postgres -d postgres -c "SELECT count(*) AS total_clientes FROM clientes;"
```

Saída fornecida:

```text
 total_clientes
----------------
           1000
(1 linha)
```

## Resultado

A réplica retornou os 1.000 clientes após a inserção no primário, comprovando a replicação desses dados. A tabela `eventos` ainda será criada. Para concluir o relatório, distinguir replicação atual, perda de dados confirmados (RPO) e operações que falharam ao tentar abrir conexões durante o incidente.
