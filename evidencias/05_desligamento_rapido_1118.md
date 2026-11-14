# Evidência 05 — Desligamento rápido após reinício às 11:18

**Data:** 14/11/2026; fuso -03. **Fonte:** saída fornecida do arquivo `primary_recuperacao_2.log`.

## Verificação anterior

```text
pg_ctl: servidor está executando (PID: 22980)
C:/Program Files/PostgreSQL/16/bin/postgres.exe "-D" "C:/Users/Samsung/lab1/primary" "-p" "5442"
```

Após essa verificação, uma conexão administrativa na porta 5442 retornou `Connection refused (0x0000274D/10061)`. O comando `tasklist /FI "PID eq 22980"` confirmou que o processo não estava mais em execução.

## Log correspondente

```text
2026-11-14 11:18:10.300 -03 [22980] LOG:  iniciando PostgreSQL 16.14, compiled by Visual C++ build 1944, 64-bit
2026-11-14 11:18:10.307 -03 [22980] LOG:  escutando no endereço IPv6 "::1", porta 5442
2026-11-14 11:18:10.308 -03 [22980] LOG:  escutando no endereço IPv4 "127.0.0.1", porta 5442
2026-11-14 11:18:10.444 -03 [22980] LOG:  sistema de banco de dados está pronto para aceitar conexões
2026-11-14 11:18:40.619 -03 [22980] LOG:  pedido de desligamento rápido foi recebido
2026-11-14 11:18:40.639 -03 [22980] LOG:  interrompendo quaisquer transações ativas
2026-11-14 11:18:40.651 -03 [20556] LOG:  desligando
2026-11-14 11:18:40.681 -03 [22980] LOG:  sistema de banco de dados está desligado
```

## Interpretação

O servidor **não sofreu falha comprovada por esgotamento de conexões neste episódio**. Ele ficou pronto às 11:18:10 e recebeu solicitação de desligamento rápido às 11:18:40 (aproximadamente 30 segundos depois). O mecanismo emissor do pedido ainda não foi identificado.

Uma hipótese operacional a testar é o encerramento de uma janela CMD associada ao processo PostgreSQL. Para as próximas etapas, manter o terminal de inicialização aberto e usar terminais separados, sem fechar janelas nem enviar Ctrl+C enquanto consultas ou servidores estão ativos. A hipótese não deve ser registrada como causa raiz confirmada sem teste.
