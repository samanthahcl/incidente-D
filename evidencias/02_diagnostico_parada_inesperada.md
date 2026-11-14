# Diagnóstico de parada inesperada do primário — 14/11/2026

## Contexto
Durante a preparação do teste formal do cenário D, o cliente não recebeu resposta do primário em `127.0.0.1:5442`. O comando `pg_isready -h 127.0.0.1 -p 5442 -t 5` retornou `nenhuma resposta`. O comando `pg_ctl -D C:\Users\Samsung\lab1\primary status` retornou `nenhum servidor está executando`.

## Logs consultados
Comandos reais:
```cmd
cd C:\Users\Samsung\lab1
dir /O:-D *.log
powershell -NoProfile -Command "Get-Content 'primary.log' -Tail 40"
```

Trecho relevante do `primary.log` (saída fornecida pela estudante):
```text
2026-11-14 10:51:30.895 -03 [14940] LOG:  processo servidor (PID 24552) foi terminado pela exceção 0xC000013A
2026-11-14 10:51:30.895 -03 [14940] DETALHE:  Processo que falhou estava executando: START_REPLICATION SLOT "replica1_slot" 0/3000000 TIMELINE 1
2026-11-14 10:51:30.895 -03 [14940] DICA:  Veja o arquivo de cabeçalho C "ntstatus.h" para obter uma descrição do valor hexadecimal.
2026-11-14 10:51:30.895 -03 [14940] LOG:  terminando quaisquer outros processos servidor ativos
2026-11-14 10:51:30.924 -03 [14940] LOG:  todos os processos servidor foram terminados; reinicializando
```

Também foram registrados vários erros de `conexão com cliente foi perdida` às 10:22:52 e uma falha de autenticação do usuário `app` às 10:38:56.

## Interpretação provisória
O código Windows `0xC000013A` é compatível com interrupção do processo; a origem exata ainda não foi identificada. O log registra a terminação de um processo de replicação e reinicialização dos processos do servidor. A parada **não pode ser atribuída com segurança** ao limite de três conexões do usuário `app`; o evento registrado ocorreu antes da tentativa de quarta conexão (marcada pela estudante às 11:02:20.144).

Este é um evento inesperado de laboratório, separado da prova do cenário D. As medições de RTO/RPO do teste formal ainda estão pendentes.
