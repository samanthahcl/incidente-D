# Evidência 04 — Desligamento rápido durante tentativa formal do cenário D

**Fonte:** saída real enviada da consulta `Get-Content 'primary_recuperacao.log' -Tail 50` em 14/11/2026.

## Linha do tempo confirmada pelo log (fuso -03)

- **11:09:41.640:** servidor primário pronto para aceitar conexões na porta 5442 após recuperação automática.
- **11:15:05.229:** cancelamento solicitado para `SELECT pg_sleep(300);` e para `START_REPLICATION SLOT "replica1_slot"`.
- **11:15:05.236:** registro explícito de **pedido de desligamento rápido**.
- **11:15:05.355:** sistema de banco de dados desligado.
- **11:15:49.102:** horário T0 anotado pela estudante na quarta janela.
- Após T0, `psql -U app -c "SELECT 1;"` retornou `Connection refused (0x0000274D/10061)`.
- `pg_ctl -D primary status` retornou `nenhum servidor está executando`.

## Trecho literal relevante

```text
2026-11-14 11:15:05.229 -03 [23284] ERRO:  cancelando a instrução devido a pedido do usuário
2026-11-14 11:15:05.229 -03 [23284] INSTRUÇÂO:  START_REPLICATION SLOT "replica1_slot" 0/3000000 TIMELINE 1
2026-11-14 11:15:05.229 -03 [12688] ERRO:  cancelando a instrução devido a pedido do usuário
2026-11-14 11:15:05.229 -03 [12688] INSTRUÇÂO:  SELECT pg_sleep(300);
2026-11-14 11:15:05.236 -03 [18492] LOG:  pedido de desligamento rápido foi recebido
2026-11-14 11:15:05.244 -03 [18492] LOG:  interrompendo quaisquer transações ativas
2026-11-14 11:15:05.279 -03 [12688] FATAL:  terminando a conexão devido a um comando do administrador
2026-11-14 11:15:05.324 -03 [23168] LOG:  desligando
2026-11-14 11:15:05.355 -03 [18492] LOG:  sistema de banco de dados está desligado
```

## Interpretação

O log comprova um pedido de desligamento rápido, sem demonstrar que houve esgotamento do limite de conexões por papel. Uma interrupção pelo terminal é hipótese, não causa comprovada. A quarta tentativa recebeu `Connection refused` porque foi feita após o desligamento; **não pode ser usada como prova de `too many connections for role "app"`** nem como T0 válido para medir o RTO específico do cenário D.

## Próxima ação

Reiniciar o primário com segurança, validar a réplica e repetir o teste formal evitando sinais de interrupção no terminal. Registrar os horários reais de uma quarta conexão recusada **pelo limite do papel**, encerramento de sessões e restabelecimento.
