# Evidências do incidente D

## Evidências documentadas no repositório
- `01_esgotamento_global.md`: limite global exploratório e erros.
- `02_diagnostico_parada_inesperada.md` a `05_desligamento_rapido_1118.md`: investigação das paradas inesperadas.
- `06_reproducao_formal_role_app.md`: três sessões `app` e recusa da quarta conexão.
- `07_correcao_rto.md`: término de três sessões, conexão recuperada e RTO observado.
- `08_timeouts_preventivos.md`: timeout de instrução comprovado.
- `09_clientes_replicados.md` e `10_conferencia_dados_rpo.md`: replicação de dados criada posteriormente e limitação de RPO.

## Capturas de tela recebidas na conversa — aguardando envio dos arquivos de imagem a esta pasta

1. `01_servidor_principal.png`: status inicial do primário (5442) e da réplica (5443), com `streaming / async` e `pg_is_in_recovery() = f` no primário.
2. `02_servidor_replica.png`: `pg_is_in_recovery() = t` na porta 5443, `f` na 5442, e replicação ativa.
3. `03_replicacao_ativa.png`: repetição das verificações de papéis e do estado `streaming / async`.
4. `04_teste_dados.png`: consulta na réplica à tabela `teste`, exibindo a linha `1 | ola`.

**Essas imagens comprovam a configuração inicial e o funcionamento da replicação, não a recusa de conexões do incidente D.** Os resultados formais do incidente estão transcritos nos arquivos `06`, `07` e `08`; prints dessas operações não foram fornecidos.

Ao adicionar os arquivos de imagem, preserve as imagens originais. Não publique credenciais ou arquivos internos das pastas `primary/` e `replica/`.
