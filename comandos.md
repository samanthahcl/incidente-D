# Histórico de comandos — Cenário D

Ambiente: Windows CMD, PostgreSQL 16.14 instalado, diretório local `C:\Users\Samsung\lab1`. Comandos apresentados na ordem geral das etapas. **Horários não anotados não foram inventados**. Comandos com senhas em texto foram redigidos para não expor credenciais no repositório.

## 1. Preparação do laboratório (horário não coletado)

```cmd
psql --version
pg_ctl -D primary status
pg_ctl -D replica status
psql -h 127.0.0.1 -p 5442 -U postgres -c "SELECT client_addr, state, sync_state FROM pg_stat_replication;"
psql -h 127.0.0.1 -p 5442 -U postgres -c "SHOW max_connections;"
psql -h 127.0.0.1 -p 5442 -U postgres -c "SELECT count(*) AS conexoes_ativas FROM pg_stat_activity;"
psql -h 127.0.0.1 -p 5442 -U postgres -c "SHOW superuser_reserved_connections;"
python --version
```

Resultados: PostgreSQL 16.14; replicação `streaming / async`; limites 100 globais e 3 reservadas; 7 conexões iniciais; Python 3.13.14.

## 2. Simulação exploratória — limite global (horário não coletado)

```sql
CREATE ROLE teste_incidente WITH LOGIN PASSWORD '[REDACTED]';
GRANT CONNECT ON DATABASE postgres TO teste_incidente;
```

```cmd
python simular_incidente.py
python simular_incidente_v2.py
type C:\Users\Samsung\lab1\06_erros_incidente.txt
```

Resultados: 110 tentativas, 97 processos ainda ativos e 13 tentativas com erro. Os erros incluíram conexões reservadas para SUPERUSER e muitos clientes conectados.

## 3. Papel da aplicação e primeiras tentativas (horário não coletado)

```sql
CREATE ROLE app LOGIN PASSWORD '[REDACTED]' CONNECTION LIMIT 3;
GRANT CONNECT ON DATABASE postgres TO teste_incidente;
```

Foram iniciadas sessões `app` com `SELECT pg_sleep(300);`, mas houve erros de senha e episódios não planejados de desligamento do servidor, documentados separadamente. A saída de `pg_ctl` e os logs foram inspecionados; o processo órfão PID 22984 foi encerrado para permitir reinício do primário.

## 4. Diagnóstico e recuperação do ambiente (horários nos próprios logs)

```cmd
pg_isready -h 127.0.0.1 -p 5442 -t 5
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\primary" status
dir /O:-D *.log
powershell -NoProfile -Command "Get-Content 'primary.log' -Tail 40"
powershell -NoProfile -Command "Get-Content 'primary_recuperacao.log' -Tail 40"
tasklist /FI "IMAGENAME eq postgres.exe"
powershell -NoProfile -Command "Get-CimInstance Win32_Process -Filter \"name='postgres.exe'\" | Select-Object ProcessId,ParentProcessId,CommandLine | Format-List"
tasklist /FI "PID eq 14940"
tasklist /FI "PID eq 22984"
taskkill /PID 22984 /F
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\primary" -l "C:\Users\Samsung\lab1\primary_recuperacao.log" -o "-p 5442" -w start
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\replica" -l "C:\Users\Samsung\lab1\replica_recuperacao.log" -o "-p 5443" -w start
powershell -NoProfile -Command "Get-Content 'primary_recuperacao_2.log' -Tail 50"
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\primary" -l "C:\Users\Samsung\lab1\primary_recuperacao_3.log" -o "-p 5442" -w start
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\replica" -l "C:\Users\Samsung\lab1\replica_recuperacao_2.log" -o "-p 5443" -w start
```

O ambiente voltou a mostrar `streaming / async`; na réplica `pg_is_in_recovery()` retornou `t`. **Nem todos os comandos acima foram executados consecutivamente; houve verificações e novas tentativas entre eles.** Logs de 11:15:05 e 11:18:40 indicaram pedidos de desligamento rápido; causa emissora não determinada.

## 5. Reprodução formal — cenário D

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT rolname, rolconnlimit FROM pg_roles WHERE rolname = 'app';"
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT pid, usename, state, now() - query_start AS tempo FROM pg_stat_activity WHERE usename = 'app';"
```

Depois de garantir 0 sessões `app`, três processos `psql` executaram `SELECT pg_sleep(300);`. A consulta de `pg_stat_activity` retornou 3 PIDs `active`: 24300, 4196, 15244.

```cmd
powershell -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss.fff'; & 'C:\Program Files\PostgreSQL\16\bin\psql.exe' -h 127.0.0.1 -p 5442 -U app -d postgres -c 'SELECT 1;'"
```

**T0 (horário impresso antes do teste): 2026-11-14 11:23:54.715 (-03).** Retorno: `FATAL: excesso de conexões ... role "app"`.

## 6. Mitigação e retorno à operação

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = 'app';
```

Retorno do comando: `t`, `t`, `t`.

```cmd
powershell -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss.fff'; & 'C:\Program Files\PostgreSQL\16\bin\psql.exe' -h 127.0.0.1 -p 5442 -U app -d postgres -c 'SELECT 1;'"
```

**T1 (horário impresso antes do teste): 2026-11-14 11:24:59.771 (-03).** Retorno: `1` (uma linha). **RTO observado = 65,056 s**.

## 7. Medidas preventivas (horário não coletado)

```sql
ALTER ROLE app SET statement_timeout = '3s';
ALTER ROLE app SET idle_session_timeout = '5s';
```

Teste em nova sessão:

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U app -d postgres -c "SELECT pg_sleep(30);"
```

Resposta: `ERRO: cancelando a instrução porque o tempo de espera da instrução se esgotou (timeout)`. O `idle_session_timeout` foi configurado, mas não testado diretamente.

## 8. Dados de preparação complementar (depois do incidente, horários não coletados)

```sql
CREATE TABLE clientes (id serial PRIMARY KEY, nome text, email text);
INSERT INTO clientes (nome, email)
SELECT 'cliente ' || g, 'c' || g || '@loja.com' FROM generate_series(1, 1000) g;
CREATE TABLE eventos (id bigserial PRIMARY KEY, ts timestamptz DEFAULT now());
```

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5443 -U postgres -d postgres -c "SELECT count(*) AS total_clientes FROM clientes;"
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5443 -U postgres -d postgres -c "SELECT count(*) AS total_clientes FROM clientes; SELECT count(*) AS total_eventos FROM eventos;"
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT count(*) AS total_clientes FROM clientes; SELECT count(*) AS total_eventos FROM eventos;"
```

Resultados confirmados: `INSERT 0 1000`, 1.000 clientes na réplica, e 1.000 clientes / 0 eventos no primário. A confirmação do segundo comando da réplica foi verbal, sem transcrição numérica. Essas tabelas não existiam antes da simulação formal, portanto não são a base de cálculo retrospectivo do RPO.
