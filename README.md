# Fonteza para Claude Code

Plugin oficial da [Fonteza](https://fonteza.com.br) para o [Claude Code](https://claude.com/claude-code). Ele conecta o Claude Code ao MCP remoto da Fonteza e instala uma skill que ensina o Claude a consultar os dados corretamente.

Com ele, o Claude consulta dados públicos oficiais do Brasil:

- cadastro de empresas da Receita Federal: CNPJ, endereço, CNAEs, porte, situação e sócios (CPFs mascarados);
- números de mercado de um setor num estado: empresas ativas, aberturas e encerramentos;
- oportunidades de licitação do PNCP;
- empenhos, liquidações e pagamentos federais a uma empresa.

## Instalação

No Claude Code:

```
/plugin marketplace add <dono>/<repositório>
/plugin install fonteza@fonteza
```

Depois, rode `/mcp`, escolha `fonteza` e entre com a sua conta Fonteza no navegador. Não é preciso copiar chave.

Para conferir, peça ao Claude: "Quais fontes a Fonteza cobre?". Ele chama `descobrir_cobertura`, que não consome cota.

## Alternativa: chave de API

Se preferir não usar o login pelo navegador, ou se o seu ambiente não abre um navegador, crie uma chave no [painel](https://fonteza.com.br/app/login), em **Chaves de API**, e adicione o MCP direto:

```bash
claude mcp add --transport http fonteza https://fonteza.com.br/mcp --header "Authorization: Bearer $FONTEZA_API_KEY"
```

Guarde a chave numa variável de ambiente ou num gerenciador de segredos; nunca em arquivos versionados ou prompts. Neste caso a skill não vem junto: instale o plugin também se quiser as instruções de uso, ou leia o [guia para agentes](https://fonteza.com.br/docs/agentes).

## Ferramentas

| Ferramenta | O que faz |
| --- | --- |
| `descobrir_cobertura` | Fontes publicadas, período, cobertura, limitações e aviso operacional vigente. Não consome cota. |
| `buscar_empresas` | Busca empresas por nome, CNPJ, atividade ou códigos CNAE, com filtros. |
| `obter_empresa` | Cadastro completo e sócios de um CNPJ. |
| `mercado_por_cnae` | Números de mercado de um setor num estado. |
| `iniciar_agente_mercado` | Pede uma análise escrita de um setor para um objetivo (contas liberadas). |
| `consultar_execucao` | Acompanha a análise pedida. |
| `buscar_oportunidades` | Oportunidades de licitação do PNCP. |
| `despesas_federais_da_empresa` | Despesas federais de um CNPJ. |

## Cota

Cada chamada bem-sucedida a uma ferramenta de dados consome uma unidade da cota diária da sua conta, mesmo com resultado vazio. `descobrir_cobertura`, argumentos inválidos, falhas e timeouts não consomem. A contagem recomeça à meia-noite de Brasília.

## Sobre os dados

- A Fonteza usa só a publicação mais recente da Receita Federal. Não há consulta a versões anteriores do cadastro.
- Toda resposta traz a fonte e, para dados da Receita, a competência e o `snapshot_id` da publicação.
- CPFs de sócios chegam mascarados, como a Receita os publica.

Documentação completa: [fonteza.com.br/docs/agentes](https://fonteza.com.br/docs/agentes).

## Suporte

contato@fonteza.com.br
