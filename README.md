# Incidente D — Esgotamento de conexões (negação de serviço)

**Disciplina:** DBaaS — Alta disponibilidade e réplicas  
**Cenário:** D — Esgotamento de conexões  
**Ambiente:** PostgreSQL 16.14 instalado no Windows, primário em `127.0.0.1:5442` e réplica em `127.0.0.1:5443`  
**Status:** laboratório em andamento — resultados parciais.

## 1. Resumo do incidente
O cenário representa um cliente automatizado que ocupa conexões do banco por tempo prolongado, impedindo sessões legítimas de acessar o serviço. O banco pode permanecer em execução enquanto a aplicação sofre indisponibilidade. A mitigação planejada inclui limitar conexões por papel, encerrar sessões abusivas e impor limites de duração.

## 2. Ambiente e verificação inicial
- Primário e réplica iniciados pelo `pg_ctl`, com replicação `streaming` assíncrona verificada.
- Porta primária: `5442`; réplica: `5443`.
- `max_connections = 100`; `superuser_reserved_connections = 3`.
- Antes da simulação: 7 conexões registradas em `pg_stat_activity`.
- Após a primeira simulação: 7 conexões registradas, com o acesso administrativo funcionando.

## 3. Teste exploratório já executado
Foi criado o usuário `teste_incidente` com permissão `CONNECT` ao banco `postgres`. Um script local iniciou 110 tentativas de conexão usando `pg_sleep`.

Na versão que registra resultados, o terminal reportou:
- **110** tentativas;
- **97** processos ainda ativos após 5 segundos;
- **13** processos encerrados com mensagens de erro;
- Arquivo local de evidências: `06_erros_incidente.txt`.

**Importante:** os 13 erros ainda precisam ser inspecionados; esse número, isoladamente, não prova que todos se devem ao esgotamento. Esse teste exploratório não substitui a reprodução prescrita pelo professor para o cenário D.

## 4. Reprodução exigida para o cenário D — pendente
1. Criar papel `app` com `CONNECTION LIMIT 3`.
2. Abrir três sessões simultâneas como `app` executando `SELECT pg_sleep(300);`.
3. Registrar **T0** e tentar uma quarta conexão; capturar o erro `too many connections for role "app"`.
4. Consultar `pg_stat_activity` para esse papel.
5. Encerrar sessões com `pg_terminate_backend`, registrar **T1** quando uma conexão legítima voltar a funcionar e calcular `RTO = T1 - T0`.
6. Confirmar o **RPO**, distinguindo linhas persistidas de operações não concluídas durante a indisponibilidade.
7. Configurar `statement_timeout = '3s'` e `idle_session_timeout = '5s'` para novas sessões do papel `app`; testar o cancelamento de `pg_sleep(30)`.

## 5. Linha do tempo
| Momento | Ação | Resultado |
|---|---|---|
| Antes da simulação (horário não registrado) | Verificação de limites e sessões | 100 conexões máximas; 3 reservadas; 7 ativas |
| Teste exploratório (horário não registrado) | 110 tentativas de conexão | 97 processos ativos e 13 com erro |
| Após primeiro teste (horário não registrado) | Contagem de sessões | 7 sessões |
| A registrar | T0: quarta conexão do papel `app` recusada | Pendente |
| A registrar | T1: conexão legítima do papel `app` restabelecida | Pendente |

## 6. Medições — a concluir
**RTO:** ainda não foi medido no procedimento formal. Registrar o horário da primeira recusa (T0) e o horário da primeira conexão bem-sucedida após a correção (T1); calcular a diferença.

**RPO:** verificar se houve perda de dados persistidos. Sessões recusadas podem resultar em transações não executadas ou não confirmadas, que precisam ser distinguidas de dados já confirmados e perdidos.

**Failover:** não previsto para este cenário; o incidente é de conexões, não de falha do nó primário. Não promover a réplica.

## 7. Causa raiz e correção — a comprovar
Causa simulada: ocupação excessiva de sessões por um cliente, com impacto sobre conexões legítimas.

Correção a demonstrar: finalizar sessões problemáticas; prevenir recorrência com limite por usuário, timeout de instrução e timeout para sessão ociosa.

## 8. Serviço gerenciado, prevenção e respostas às perguntas
**A complementar ao final dos testes.** Comparar o papel de um proxy/pool de conexões (por exemplo, RDS Proxy ou PgBouncer), limites por aplicação e monitoramento. Responder:
- Por que limitar conexões por usuário em vez de depender apenas do limite global?
- Que papel teria um pool/proxy?

## 9. Evidências
As saídas e capturas obtidas localmente serão adicionadas à pasta `evidencias/`. Resultados ainda não enviados não são apresentados como comprovantes.

> Documento incremental. O relatório final será atualizado com comandos, horários reais, provas antes/depois, medições e conclusões.
