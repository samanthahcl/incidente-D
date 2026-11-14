# Índice de evidências — Cenário D

As evidências foram obtidas no laboratório local com PostgreSQL 16.14 (primário: 5442; réplica: 5443), em 14/11/2026. Este índice diferencia **arquivos originais enviados**, **transcrições fornecidas no chat** e **interpretações**.

## Arquivos originais enviados pela estudante

### Capturas de tela da preparação e replicação

- [01_servidor_principal.png..png](01_servidor_principal.png..png) — inicialização e verificação do primário.
- [02_servidor_replica.png](02_servidor_replica.png) — verificação do papel de réplica.
- [03_replicacao_ativa.png..png](03_replicacao_ativa.png..png) — estado do streaming assíncrono.
- [04_teste_dados.png](04_teste_dados.png) — teste de dados replicados (tabela `teste`).

**Nota:** os nomes de dois PNGs preservam a extensão duplicada do upload; isso não altera seu conteúdo.

### Arquivos de texto originais

- [05_estado_normal.txt.txt](05_estado_normal.txt.txt) — limite global, conexões observadas e reservas administrativas antes do incidente.
- [validacao_ambiente.txt.txt](validacao_ambiente.txt.txt) — configuração, replicação e teste inicial.
- [06_erros_incidente.txt](06_erros_incidente.txt) — mensagens de recusa de conexões no teste exploratório de 110 tentativas.

Os dois nomes terminados em `.txt.txt` foram mantidos conforme enviados. **O arquivo `06_erros_incidente.txt` pertence ao teste exploratório de limite global, e não à reprodução formal por papel.**

## Evidências documentais do laboratório

- [01_esgotamento_global.md](01_esgotamento_global.md) — interpretação do teste exploratório de limite global.
- [02_diagnostico_parada_inesperada.md](02_diagnostico_parada_inesperada.md) — logs e investigação.
- [03_recuperacao_primario_replica.md](03_recuperacao_primario_replica.md) — recuperação do ambiente.
- [04_desligamento_rapido_1115.md](04_desligamento_rapido_1115.md) e [05_desligamento_rapido_1118.md](05_desligamento_rapido_1118.md) — paradas não planejadas e seus logs.
- [06_reproducao_formal_role_app.md](06_reproducao_formal_role_app.md) — **evidência central**: três sessões `app` e quarta conexão recusada às 11:23:54.715.
- [07_correcao_rto.md](07_correcao_rto.md) — três backends encerrados e conexão de `app` restabelecida às 11:24:59.771; RTO observado 65,056 s.
- [08_timeouts_preventivos.md](08_timeouts_preventivos.md) — configuração de timeouts e cancelamento real de `pg_sleep(30)`.
- [09_clientes_replicados.md](09_clientes_replicados.md) — 1.000 clientes inseridos e replicados **depois** do incidente.
- [10_conferencia_dados_rpo.md](10_conferencia_dados_rpo.md) — contagens atuais e limites da estimativa do RPO.

## Limites das evidências

Não foram recebidas capturas de tela específicas da quarta conexão recusada, da terminação dos backends nem do timeout. Seus resultados foram transcritos a partir das **saídas reais copiadas do terminal**, sem simular capturas.

A prova quantitativa pré/pós de dados confirmados para o episódio formal também não foi coletada. Por isso o RPO é apresentado como estimativa conceitual, não medição retrospectiva comprovada.

**Não publicar** senhas, dados internos de `primary/` e `replica/` ou arquivos com credenciais.
