# Evidência 03 — Recuperação do primário e restabelecimento da replicação

**Data local:** 14/11/2026. Horários exatos não registrados para todos os comandos.

## Diagnóstico

O primário não respondia em `127.0.0.1:5442` e `pg_ctl status` indicava que não havia servidor executando. A primeira tentativa de inicialização com `primary.log` retornou `Permission denied`. Uma nova tentativa usando `primary_recuperacao.log` retornou:

```text
2026-11-14 11:07:37.891 -03 [3572] FATAL: bloco de memória compartilhada pré-existente ainda está em uso
2026-11-14 11:07:37.891 -03 [3572] DICA: Verifique se ainda há processos servidor antigos sendo executados, e termine-os.
2026-11-14 11:07:37.892 -03 [3572] LOG: sistema de banco de dados está desligado
```

## Identificação e recuperação do processo antigo

A consulta a processos Windows mostrou um processo `postgres.exe` PID 22984, com `ParentProcessId: 14940`; o PID 14940 havia sido identificado anteriormente como o primário do laboratório e não estava mais em execução.

```cmd
tasklist /FI "IMAGENAME eq postgres.exe"
powershell -NoProfile -Command "Get-CimInstance Win32_Process -Filter \"name='postgres.exe'\" | Select-Object ProcessId,ParentProcessId,CommandLine | Format-List"
tasklist /FI "PID eq 14940"
tasklist /FI "PID eq 22984"
taskkill /PID 22984 /F
```

Saída relevante:

```text
INFORMAÇÕES: nenhuma tarefa em execução correspondente aos critérios especificados.
postgres.exe 22984 Console 1 3.924 K
ÊXITO: o processo com PID 22984 foi finalizado.
```

Depois desse encerramento controlado, a estudante informou que o primário voltou a funcionar na porta 5442. A consulta administrativa ao primário passou a funcionar, mas inicialmente mostrou zero linhas em `pg_stat_replication`.

## Restabelecimento da réplica

```cmd
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\replica" status
"C:\Program Files\PostgreSQL\16\bin\pg_ctl.exe" -D "C:\Users\Samsung\lab1\replica" -l "C:\Users\Samsung\lab1\replica_recuperacao.log" -o "-p 5443" -w start
psql -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT client_addr, state, sync_state FROM pg_stat_replication;"
```

Saídas relevantes:

```text
pg_ctl: nenhum servidor está executando
pg_ctl: outro servidor pode estar executando; tentando iniciar o servidor assim mesmo
esperando o servidor iniciar....feito
servidor iniciado

 client_addr |   state   | sync_state
-------------+-----------+------------
 127.0.0.1   | streaming | async
(1 linha)
```

## Conclusão

O primário voltou a aceitar conexões administrativas e a replicação em streaming assíncrono foi restabelecida. **Ainda falta verificar diretamente `pg_is_in_recovery()` na réplica e o conjunto de dados.** Não foi registrada uma medição de RTO/RPO da parada inesperada, e ela não deve ser confundida com a medição formal do cenário D.

A causa da terminação original do processo de replicação não foi definitivamente determinada. O processo remanescente e a memória compartilhada ocupada explicam a dificuldade na tentativa de reinicialização, mas não permitem atribuir a parada ao limite de conexões do usuário `app`.
