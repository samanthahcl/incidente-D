# Evidência 06 — Reprodução formal do cenário D

**Data:** 14/11/2026, horário local UTC−03:00. **Origem:** saída real enviada pela estudante.

## Preparação

O papel `app` foi criado com limite de 3 conexões:

```sql
CREATE ROLE app LOGIN PASSWORD '[REDACTED]' CONNECTION LIMIT 3;
```

A configuração foi conferida:

```text
 rolname | rolconnlimit
---------+--------------
 app     |            3
(1 linha)
```

Após confirmar que não havia sessões anteriores do papel, foram iniciadas três sessões com `SELECT pg_sleep(300);` no primário `127.0.0.1:5442`.

## Evidência das três sessões ocupadas

Comando executado pelo administrador:

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT pid, usename, state, now() - query_start AS tempo FROM pg_stat_activity WHERE usename = 'app';"
```

Saída:

```text
  pid  | usename | state  |      tempo
-------+---------+--------+-----------------
 24300 | app     | active | 00:00:31.396946
  4196 | app     | active | 00:00:31.384635
 15244 | app     | active | 00:00:31.403581
(3 linhas)
```

## Quarta conexão recusada — T0

Comando executado:

```cmd
powershell -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss.fff'; & 'C:\Program Files\PostgreSQL\16\bin\psql.exe' -h 127.0.0.1 -p 5442 -U app -d postgres -c 'SELECT 1;'"
```

Saída literal:

```text
2026-11-14 11:23:54.715
psql: erro: a conexão com o servidor em "127.0.0.1", porta 5442 falhou: FATAL:  excesso de conexões para a função de banco de dados (role) "app"
```

**T0 operacional registrado:** 2026-11-14 11:23:54.715 (-03). É o horário imediatamente anterior à tentativa recusada; o instante exato de rejeição pode ser alguns milissegundos posterior.

## Interpretação

O teste reproduziu a indisponibilidade do acesso por limite de conexões **do papel `app`** (e não por queda do primário ou limite global). As três sessões abertas e a recusa da quarta conexão satisfazem a etapa de reprodução formal do cenário D.

**Ainda pendente:** encerramento das sessões via `pg_terminate_backend`, confirmação de conexão restabelecida, T1, cálculo de RTO, avaliação do RPO, configuração e teste de timeouts. As sessões `pg_sleep(300)` podem acabar automaticamente antes da correção; caso isso ocorra, registrar como limitação da medição em vez de atribuir a recuperação a uma ação não realizada.
