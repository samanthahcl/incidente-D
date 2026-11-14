# Evidência 07 — Correção e medição de RTO do cenário D

**Data:** 14/11/2026; horários locais UTC−03.  
**Cenário:** recusa da quarta conexão do papel `app`, com três sessões simultâneas ocupadas.

## T0 — Indisponibilidade observada

```text
2026-11-14 11:23:54.715
psql: erro: a conexão com o servidor em "127.0.0.1", porta 5442 falhou: FATAL:  excesso de conexões para a função de banco de dados (role) "app"
```

## Correção executada

O administrador executou:

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U postgres -d postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE usename = 'app';"
```

Resposta recebida:

```text
 pg_terminate_backend
----------------------
 t
 t
 t
```

Os três resultados verdadeiros confirmam que os três backends foram sinalizados para terminação. O horário exato da execução do comando não foi registrado.

## T1 — Teste de conexão bem-sucedido após a correção

```cmd
powershell -NoProfile -Command "Get-Date -Format 'yyyy-MM-dd HH:mm:ss.fff'; & 'C:\Program Files\PostgreSQL\16\bin\psql.exe' -h 127.0.0.1 -p 5442 -U app -d postgres -c 'SELECT 1;'"
```

Saída:

```text
2026-11-14 11:24:59.771
 ?column?
----------
        1
(1 linha)
```

## Cálculo do RTO

- T0: **11:23:54.715** (instante da tentativa que falhou, timestamp impresso imediatamente antes da conexão).
- T1: **11:24:59.771** (timestamp imediatamente antes da tentativa que teve sucesso).
- Diferença: **65,056 segundos**, ou **1 minuto e 5,056 segundos**.

```text
RTO observado = (11:24:59.771) − (11:23:54.715) = 65,056 s
```

**Limitação metodológica:** T0 e T1 são horários registrados imediatamente antes de cada tentativa de conexão; não são timestamps exatos das respostas do servidor. Portanto, 65,056 s é o intervalo entre os testes observados e uma aproximação operacional do RTO. O acesso pode ter sido restabelecido antes de T1, após a terminação dos backends. Inclui tempo de atuação manual.

## Pendente

- Avaliar RPO com dados persistidos; não afirmar perda zero apenas porque a conexão voltou.
- Aplicar `ALTER ROLE app SET statement_timeout = '3s'` e `ALTER ROLE app SET idle_session_timeout = '5s'`.
- Confirmar em nova sessão `app` que `SELECT pg_sleep(30)` é cancelado aproximadamente aos 3 segundos.
- Completar comparação com serviços gerenciados e respostas do cenário.

