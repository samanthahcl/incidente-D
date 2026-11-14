# Comandos executados — Incidente D

**Ambiente:** Windows CMD · PostgreSQL 16.14 · laboratório em `C:\Users\Samsung\lab1`

> Os horários exatos dos primeiros comandos não foram coletados. Não foram inventados. Para as próximas etapas, registrar data/hora antes de cada bloco de testes com `powershell -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss.fff'"`.

## Preparação do ambiente (etapas anteriores)
```cmd
psql --version
pg_ctl -D primary status
pg_ctl -D replica status
psql -h 127.0.0.1 -p 5442 -U postgres -c "SELECT client_addr, state, sync_state FROM pg_stat_replication;"
```
**Resultados registrados:** PostgreSQL 16.14; instâncias ligadas em 5442 e 5443; replicação `streaming`, `async`.

## Linha de base (horário não registrado)
```cmd
psql -h 127.0.0.1 -p 5442 -U postgres -c "SHOW max_connections;"
psql -h 127.0.0.1 -p 5442 -U postgres -c "SELECT count(*) AS conexoes_ativas FROM pg_stat_activity;"
psql -h 127.0.0.1 -p 5442 -U postgres -c "SHOW superuser_reserved_connections;"
```
**Saídas observadas:** 100, 7 e 3, respectivamente.

## Preparação da simulação exploratória (horário não registrado)
```cmd
python --version
psql -h 127.0.0.1 -p 5442 -U postgres -c "CREATE ROLE teste_incidente WITH LOGIN PASSWORD '[REDACTED]';"
psql -h 127.0.0.1 -p 5442 -U postgres -c "GRANT CONNECT ON DATABASE postgres TO teste_incidente;"
```
**Saídas observadas:** Python 3.13.14, `CREATE ROLE` e `GRANT`. A senha foi omitida do histórico público.

## Testes exploratórios (horário não registrado)
```cmd
python simular_incidente.py
psql -h 127.0.0.1 -p 5442 -U postgres -c "SELECT count(*) AS conexoes_ativas FROM pg_stat_activity;"
findstr /C:"06_erros_incidente.txt" simular_incidente.py
type C:\Users\Samsung\lab1\simular_incidente.py
findstr /C:"06_erros_incidente.txt" C:\Users\Samsung\lab1\simular_incidente_v2.py
python simular_incidente_v2.py
```
**Resultados observados:** depois do primeiro script, 7 conexões. A versão v2 registrou `110 tentativas realizadas`, `Conexoes ativas: 97`, `Tentativas com erro: 13` e `Evidencias salvas em 06_erros_incidente.txt`.

## Procedimento formal do cenário D
**Ainda não executado.** Será adicionado aqui com horários reais, comandos e saídas, seguindo o PDF da atividade.

## Observações
- Não publicar senhas, arquivos `pw.txt`, dados internos `primary/`, `replica/` nem configurações com credenciais.
- Capturas de tela e arquivos de saída reais serão adicionados à pasta `evidencias/` conforme disponibilizados.
