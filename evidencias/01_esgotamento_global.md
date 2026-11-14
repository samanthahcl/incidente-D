# Evidência 01 — Esgotamento de conexões no PostgreSQL

**Origem:** saída de CMD fornecida pela estudante, executada no ambiente local `C:\Users\Samsung\lab1`.  
**Data:** 14/11/2026 (horário exato desta execução não registrado).  
**Primário:** `127.0.0.1:5442`.  
**Teste:** exploratório com `simular_incidente_v2.py` e usuário `teste_incidente`.

## Saída do script

```text
C:\Users\Samsung\lab1>python simular_incidente_v2.py
110 tentativas realizadas.
Conexoes ativas: 97
Tentativas com erro: 13
Evidencias salvas em 06_erros_incidente.txt
Simulacao encerrada.
```

## Saída de `type C:\Users\Samsung\lab1\06_erros_incidente.txt`

O CMD exibiu caracteres corrompidos por codificação, aqui normalizados apenas para leitura (sem alterar o significado):

```text
psql: erro: a conexão com o servidor em "127.0.0.1", porta 5442 falhou:
FATAL: os encaixes de conexão restantes estão reservados para funções de banco de dados (roles) com o atributo SUPERUSER

psql: erro: a conexão com o servidor em "127.0.0.1", porta 5442 falhou:
FATAL: desculpe, muitos clientes conectados
```

Ambas as mensagens se repetiram entre as 13 tentativas com erro.

Resumo ao final do arquivo:

```text
Conexoes ativas: 97
Tentativas com erro: 13
```

## Interpretação e limites

O teste comprova recusa de conexões pelo limite global e pelas reservas administrativas. O número de processos ainda ativos foi medido pelo script 5 segundos após iniciar as tentativas, e não equivale a uma consulta SQL instantânea ao total de sessões. A contagem por tipo de mensagem não foi medida separadamente.

**Importante:** esta é a simulação exploratória; a prova formal com `app CONNECTION LIMIT 3`, a quarta conexão, T0/T1 e correção ainda precisa ser executada. Não inventar RTO/RPO com base nesses resultados.
