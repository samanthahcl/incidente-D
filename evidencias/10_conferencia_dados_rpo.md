# Evidência 10 — Conferência posterior dos dados e análise do RPO

**Data:** 14/11/2026. **Banco:** `postgres`, primário na porta 5442.

## Verificação do primário

Comando:

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT count(*) AS total_clientes FROM clientes; SELECT count(*) AS total_eventos FROM eventos;"
```

Saída literal enviada pela estudante:

```text
 total_clientes
----------------
           1000
(1 linha)

 total_eventos
---------------
             0
(1 linha)
```

## Réplica

Na etapa anterior, foi solicitada a contagem das mesmas tabelas na porta 5443; a estudante confirmou o sucesso sem reenviar a saída integral da consulta. A consulta anterior com `SELECT count(*) FROM clientes` na réplica havia comprovado **1.000 registros**.

## Interpretação de RPO

No incidente formal, a quarta conexão do papel `app` foi recusada pelo limite de 3 sessões, e o acesso foi restabelecido após `pg_terminate_backend`. Não houve failover nem restauração. Não há indicação de comandos de exclusão de dados persistidos na simulação formal.

**Estimativa conceitual do RPO: 0 registros confirmados perdidos**, pois a indisponibilidade foi causada por recusa de novas conexões, não por perda ou reversão de dados já confirmados.

**Limitação importante:** as tabelas `clientes` e `eventos` foram criadas **depois** do incidente; portanto, suas contagens atuais não comprovam retroativamente RPO=0. Não houve medição quantitativa pré/pós de registros confirmados para esse episódio. Operações que não conseguiram conectar podem não ter sido executadas e exigiriam novas tentativas pela aplicação; não equivalem à perda de transações já confirmadas.
