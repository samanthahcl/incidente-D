# Evidências do incidente D

Esta pasta receberá as capturas de tela e saídas reais dos comandos, sem fabricação de resultados.

## Evidências a inserir
- Estado inicial do primário e da réplica.
- Limites de conexão e sessões existentes.
- Conteúdo de `06_erros_incidente.txt` (teste exploratório), após conferência.
- Três sessões `app` ocupadas e erro da quarta tentativa.
- Consulta de `pg_stat_activity` durante o incidente.
- Encerramento das sessões e nova conexão funcionando.
- Cancelamento de `pg_sleep(30)` pelo `statement_timeout`.
- Horários T0/T1 e conferência dos dados para RTO/RPO.

**Nota:** os dados dos testes ainda não enviados não foram inseridos como se fossem evidências verificadas.
