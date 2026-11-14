# Incidente D — Esgotamento de conexões (negação de serviço)

**Disciplina:** DBaaS — Alta disponibilidade e réplicas  
**Data:** sábado, 14/11/2026  
**Ambiente:** PostgreSQL 16.14 instalado no Windows (caminho B)  
**Primário:** 127.0.0.1:5442 · **Réplica:** 127.0.0.1:5443 · **Replicação:** streaming assíncrona  
**Responsável:** identificação da estudante a preencher antes da entrega.

## 1. Resumo do incidente

O cenário representa um cliente automatizado que abre conexões e deixa sessões ocupadas, bloqueando usuários legítimos. No laboratório, o papel `app` foi limitado a três conexões e três sessões executaram `pg_sleep(300)` ao mesmo tempo. A tentativa de uma quarta conexão foi recusada, mesmo com o primário operacional. O administrador recuperou o acesso terminando os três backends, e foram configurados timeouts para prevenir ocupação excessiva.

## 2. Preparação e arquitetura

- Primário: porta 5442; réplica: porta 5443.
- `pg_stat_replication` confirmou `127.0.0.1 | streaming | async`.
- Na réplica, `pg_is_in_recovery()` retornou `t`.
- Configuração global observada: `max_connections=100`, `superuser_reserved_connections=3`.
- Verificação inicial: 7 sessões em `pg_stat_activity`.
- Um teste exploratório anterior fez 110 tentativas de conexão: 97 processos ainda ativos e 13 erros, com mensagens de limite global e reservas administrativas.
- O teste formal abaixo utiliza limite **por papel**, conforme o enunciado.

## 3. Reprodução formal: antes da correção

Foi criado o papel `app` com `CONNECTION LIMIT 3`, e a consulta em `pg_roles` confirmou `rolconnlimit=3`. Três sessões simultâneas executaram `SELECT pg_sleep(300);`, validadas pelo administrador:

| PID | Usuário | Estado | Tempo observado |
|---:|---|---|---|
| 24300 | app | active | 00:00:31.396946 |
| 4196 | app | active | 00:00:31.384635 |
| 15244 | app | active | 00:00:31.403581 |

A quarta conexão retornou:

```text
2026-11-14 11:23:54.715
FATAL: excesso de conexões para a função de banco de dados (role) "app"
```

O primário continuava disponível ao administrador. Esse é o incidente D previsto no enunciado, e não uma falha da instância.

## 4. Linha do tempo

| Horário local (-03) | Ação | Resultado |
|---|---|---|
| Antes de 11:23:54 (sem horário exato) | Três sessões `app` executando `pg_sleep(300)` | Três PIDs ativos confirmados |
| **11:23:54.715** | T0: tentativa da quarta conexão | Recusa pelo limite do papel `app` |
| Entre T0 e T1 (sem horário exato) | Administrador executou `pg_terminate_backend` | Três respostas `t` |
| **11:24:59.771** | T1: nova tentativa como `app` | `SELECT 1` retornou 1 |
| Depois (horário não coletado) | Aplicação de `statement_timeout` e `idle_session_timeout` | Comandos aceitos |
| Depois (horário não coletado) | Teste `SELECT pg_sleep(30)` como `app` | Cancelamento pelo timeout |
| Depois (horário não coletado) | Criação de `clientes` e `eventos` | 1.000 clientes na réplica; no primário, 1.000 clientes e 0 eventos |

Houve também desligamentos inesperados do primário **antes** da reprodução formal, detalhados em `evidencias/02...` a `05...`. Logs mostraram pedidos de desligamento rápido. A causa emissora não foi identificada. Esses eventos não são usados para calcular o RTO do cenário D.

## 5. RTO, RPO e failover

### RTO

Método do professor: `RTO = T1 - T0`, com T0 no primeiro teste que recusou uma conexão e T1 na conexão confirmada após a intervenção.

- T0: 11:23:54.715.
- T1: 11:24:59.771.
- **RTO observado = 65,056 segundos (1 minuto e 5,056 segundos).**

Os timestamps foram impressos **imediatamente antes** das tentativas, não na resposta do servidor. O valor é uma aproximação operacional entre verificações, incluindo diagnóstico e intervenção manual; a disponibilidade pode ter voltado antes da tentativa bem-sucedida de T1.

### RPO

**Estimativa conceitual: 0 registros confirmados perdidos durante o incidente D.** A simulação recusou novas conexões do papel `app`; não envolveu comandos que apagassem dados, restauração nem promoção de réplica. As operações recusadas sequer começaram e precisariam ser reenviadas; isso é diferente de perder transações já confirmadas.

**Limitação da comprovação:** não foi registrada contagem da mesma tabela antes e depois da indisponibilidade formal. As tabelas `clientes` e `eventos` exigidas pelo enunciado foram criadas **depois** do incidente: atualmente, o primário tem 1.000 clientes e 0 eventos, e 1.000 clientes foram comprovados na réplica. Essas contagens demonstram persistência/replicação posterior, mas **não medem retroativamente** o RPO do incidente. Não há evidência de perda; isso não equivale a prova quantitativa conclusiva de perda zero.

### Failover

**Não houve failover no cenário D:** o primário não precisou ser promovido nem substituído para liberar as conexões do papel `app`. O problema era de acesso por limitação de sessões.

## 6. Causa raiz

**Causa simulada e comprovada:** as três sessões do papel `app` ocuparam integralmente seu limite `CONNECTION LIMIT 3`. A quarta tentativa foi recusada. Em uma aplicação real, a condição pode surgir por código que não devolve conexões ao pool, conexões indevidamente retidas ou excesso de concorrência.

Desligamentos anteriores do primário tiveram logs de parada rápida; não foram atribuídos ao limite `app`, pois não há evidência disso.

## 7. Correção e prova

**Mitigação imediata:**

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = 'app';
```

O comando retornou `t` três vezes. Em seguida, a nova conexão do papel `app` executou `SELECT 1` e retornou `1` às 11:24:59.771, comprovando acesso restabelecido.

**Prevenção aplicada:**

```sql
ALTER ROLE app SET statement_timeout = '3s';
ALTER ROLE app SET idle_session_timeout = '5s';
```

Em sessão nova do papel `app`, a consulta `SELECT pg_sleep(30);` retornou erro de cancelamento por timeout, comprovando a eficácia do `statement_timeout`. O teste específico do `idle_session_timeout` não foi realizado; a configuração foi aplicada.

Essas medidas reduzem o risco, mas não garantem proteção contra todos os casos de exaustão: sessões ativas podem continuar dentro dos limites de consulta e um pool de conexões exige configuração e monitoramento adicionais.

## 8. Comparação com serviço gerenciado

Em um **Amazon RDS for PostgreSQL** com **RDS Proxy**, o proxy poderia reaproveitar conexões abertas, controlar o número de conexões enviadas ao banco, manter pool de conexões e enfileirar ou recusar requisições quando excedido o limite, reduzindo a chance de sobrecarga por conexões. Isso não elimina a necessidade de limites adequados e pode introduzir espera quando a capacidade está saturada.

**Multi-AZ** é voltado principalmente à disponibilidade diante de falhas da infraestrutura/instância, e não resolveria automaticamente sessões de uma aplicação que ocuparam o limite de conexões do papel. **PITR** (recuperação para ponto no tempo) serve para restaurar dados a um ponto anterior e não é a correção principal desse incidente, pois não identificamos exclusão ou corrupção de dados.

Referências:
- AWS — [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)
- AWS — [RDS Proxy: conexões e limites](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy-connections.html)
- PostgreSQL 16 — [Client Connection Defaults](https://www.postgresql.org/docs/16/runtime-config-client.html)

## 9. Prevenção e lições

- Limitar conexões por papel, além do limite global do banco, para evitar que uma aplicação consuma toda a capacidade.
- Definir tamanho máximo de pool, timeouts e backoff/retry controlado na aplicação.
- Monitorar `pg_stat_activity`, rejeições de conexões, taxa de criação de sessões, sessões ociosas e pressão no pool.
- Criar alertas antes de atingir os limites; preservar conexões administrativas para investigação.
- Diferenciar indisponibilidade causada por limite de conexões da queda efetiva do primário; não promover uma réplica para resolver um problema de sessões.
- Registrar timestamps e evidências durante os testes; manter as janelas de console dos servidores abertas para evitar interrupções acidentais durante o laboratório.

## 10. Respostas às perguntas do cenário D

**Por que limitar conexões por usuário e não somente o limite global?** Porque o limite por papel restringe o impacto de um cliente abusivo. Se `app` atingir 3 conexões, outras contas com permissões adequadas ainda podem acessar o banco para serviços essenciais ou diagnóstico. O limite global protege a capacidade total, mas isoladamente não distribui essa capacidade entre aplicações.

**Qual o papel de um pool/proxy (RDS Proxy, PgBouncer)?** Reutilizar conexões já estabelecidas, limitar conexões simultâneas ao banco e amortecer picos de requisições. Reduz a pressão sobre o servidor e pode colocar requisições em espera, mas precisa de limites e timeouts; não substitui controle de consultas, segurança e monitoramento.

**O que aconteceu com transações tentadas durante o incidente?** As que não conseguiram estabelecer conexão não foram executadas pelo servidor; devem ser tratadas pela aplicação com erro e eventual repetição segura. Isso não representa perda de linhas previamente confirmadas.

## 11. Arquivos e evidências

- `comandos.md`: histórico de comandos executados e horários disponíveis.
- `evidencias/01_esgotamento_global.md`: teste exploratório de limite global.
- `evidencias/02_diagnostico_parada_inesperada.md` até `05_desligamento_rapido_1118.md`: eventos adicionais de instabilidade do laboratório.
- `evidencias/06_reproducao_formal_role_app.md`: três conexões e recusa da quarta.
- `evidencias/07_correcao_rto.md`: encerramento dos backends e medição do RTO.
- `evidencias/08_timeouts_preventivos.md`: teste de timeout.
- `evidencias/09_clientes_replicados.md`, `10_conferencia_dados_rpo.md`: dados inseridos posteriormente e análise limitada do RPO.

**Antes da entrega:** preencher nome(s) da equipe. Os quatro prints da configuração/replicação e três TXT originais estão em `evidencias/`; as provas da reprodução formal, correção e timeout constam como transcrições reais em Markdown, não como capturas de tela. Conferir `evidencias/README.md`. O hash enviado ao professor deve corresponder ao último commit, sem novos commits após a entrega.
