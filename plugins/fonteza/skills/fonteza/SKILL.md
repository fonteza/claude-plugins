---
name: fonteza
description: Consulta dados públicos oficiais do Brasil pelo MCP da Fonteza — cadastro de empresas da Receita Federal (CNPJ, sócios, CNAE), e números de mercado por setor e estado. Use quando a tarefa pedir dados de empresas brasileiras, CNPJs, CNAEs ou o tamanho de um setor.
---

# Fonteza

A Fonteza republica dados públicos oficiais por um MCP remoto (servidor `fonteza`). Toda resposta de dados vem em `{"data": ..., "meta": ...}`; `meta.fonte` diz de onde veio o dado.

Se alguma resposta trouxer `meta.aviso`, como manutenção ou carga em andamento, repasse o aviso ao usuário.

## Ferramentas

| Ferramenta | Para quê | Argumentos principais |
| --- | --- | --- |
| `consultar_empresas` | Encontrar empresas por nome, CNPJ, atividade ou códigos CNAE. | `q` e/ou `cnae` (até 10 códigos de 7 dígitos); filtros `uf`, `municipio` (envie também `uf`), `somente_matriz`, `porte` (00/01/03/05), `capital_min`, `capital_max`, `data_inicio_de`, `data_inicio_ate` (YYYY-MM-DD), `tem_email`, `tem_telefone`, `simples` (S/N), `mei` (S/N); `ordenar_por`, `ordem`; `limit` (até 50), `offset` |
| `consultar_cnpj` | Cadastro, sócios e despesas federais de um CNPJ. | `cnpj` |
| `consultar_mercado` | Quantas empresas de um setor existem, abrem e encerram num estado. | `uf` e exatamente um de `cnae`, `categoria` ou `grupo`; `municipio`, `perfil`, `faixa_capital`, `agrupar_por` (municipio, cnae ou faixa_capital) |

A listagem do próprio MCP traz a descrição completa de cada argumento; leia-a quando houver dúvida.

## Cite a procedência

- Toda resposta com dados da Receita Federal traz `meta.competencia` e `meta.snapshot_id`. Ao apresentar o resultado, cite a competência (e o `snapshot_id` quando o usuário for guardar ou reutilizar o dado), junto com a fonte de `meta.fonte`.

## Receita Federal: só a publicação mais recente

A Fonteza usa exclusivamente a publicação mais recente da Receita Federal. Não há consulta a versões anteriores, comparação entre publicações nem reconstrução do cadastro em outra data. `competencia` e `snapshot_id` identificam a publicação corrente; não são parâmetros.

Se a tarefa pedir histórico ("qual era o endereço em 2022", "quando o sócio saiu"), explique a limitação e responda só com o que a publicação atual contém: datas de abertura, encerramento e entrada de sócios continuam disponíveis. A série mensal de `consultar_mercado` agrupa datas dessa publicação; não é uma série de cadastros mensais.

## CPFs ficam mascarados

Os CPFs de sócios chegam mascarados, como a Receita os publica. Nunca tente desmascarar, completar ou inferir um CPF, nem cruze fontes para identificar uma pessoa física. Telefone e e-mail são os cadastrais da empresa; trate-os como dado de contato, não como dado para perfilar pessoas.

## Buscas amplas: confirme antes

Uma busca que exigiria varrer boa parte da base volta com o erro `busca_ampla_requer_confirmacao`. Pelo MCP não há como forçar a varredura: estreite a busca.

- Antes de rodar uma busca vaga ("todas as empresas de tecnologia do Brasil"), confirme com o usuário o recorte: estado, município, um CNAE mais específico.
- Ao receber o erro, use `meta.cnaes_sugeridos`, quando vier, para propor códigos CNAE ao usuário e refaça com `uf`, `municipio` ou um código mais específico. Não repita a mesma chamada.
- Para contagens, prefira `consultar_mercado` a paginar `consultar_empresas`: `consultar_empresas` não traz total (`total` vem `null`); pagine com `offset` enquanto `has_more` for verdadeiro.

## Cota

Cada chamada bem-sucedida a uma ferramenta de dados consome uma unidade da cota diária da conta, mesmo com resultado vazio. A listagem de ferramentas, argumentos inválidos, falhas e timeouts não consomem.

- Não pagine nem repita chamadas sem necessidade; peça ao usuário antes de percorrer muitas páginas.
- Ao atingir a cota, o MCP responde que a cota diária foi atingida. Pare e avise o usuário; a contagem recomeça à meia-noite de Brasília.

## Leitura correta dos números de mercado

`consultar_mercado` conta estabelecimentos de empresas (natureza jurídica 2xxx), só pelo CNAE principal. Encerramento é baixada ou inapta. MEI nunca é somado a ME. `demais` é o porte acima de EPP e não separa média de grande empresa. Capital social é o declarado, não faturamento, e cada filial leva o capital da empresa. Para comparar estados, faça uma chamada por estado. Meses sem aberturas ou encerramentos não aparecem: preencha zeros.

## Erros

Erros voltam como JSON com `codigo` e `mensagem` (ou `erro`). Leia o código antes de tentar de novo: `parametros_invalidos` pede corrigir argumentos; `*_indisponiveis` e `base_indisponivel` pedem avisar o usuário, não repetir em laço.
