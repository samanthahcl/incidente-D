# Evidência 08 — Timeouts preventivos

**Data:** 14/11/2026. **Origem:** comandos e saída enviados pela estudante do laboratório PostgreSQL 16.14 instalado no Windows.

## Configuração preventiva

Executados pelo administrador `postgres` no primário (porta 5442), com retorno `ALTER ROLE` informado pela estudante:

```sql
ALTER ROLE app SET statement_timeout = '3s';
ALTER ROLE app SET idle_session_timeout = '5s';
```

As duas opções se aplicam a **novas sessões** do papel `app`.

- `statement_timeout`: cancela instruções que ultrapassam 3 segundos.
- `idle_session_timeout`: encerra sessões que permanecem ociosas por mais de 5 segundos (não é o mesmo que `idle_in_transaction_session_timeout`).

## Teste real de statement_timeout

Comando executado:

```cmd
"C:\Program Files\PostgreSQL\16\bin\psql.exe" -W -h 127.0.0.1 -p 5442 -U app -d postgres -c "SELECT pg_sleep(30);"
```

Saída real após autenticação correta:

```text
Senha:

ERRO:  cancelando a instrução porque o tempo de espera da instrução se esgotou (timeout)
```

**Resultado:** comprovado que a consulta foi cancelada pelo timeout. A duração de aproximadamente 3 segundos é esperada pela configuração, mas **não houve medição cronometrada enviada**; portanto, não se declara um tempo exato observado.

Houve uma tentativa anterior com falha de senha, não relacionada ao timeout; a execução bem-sucedida acima ocorreu depois da correção da autenticação.

## Pendente

Confirmar, se necessário, as configurações em nova sessão com `SHOW statement_timeout;` e `SHOW idle_session_timeout;`. O efeito de `idle_session_timeout` ainda não foi testado diretamente. Verificar RPO e concluir o relatório final.
