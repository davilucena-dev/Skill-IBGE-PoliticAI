---
name: ibge-politicai
description: >
  Use esta skill sempre que precisar buscar dados oficiais do IBGE,
  como PIB, desemprego, inflação, pobreza, educação, saúde população e etc.
  Ativa quando o usuário pede dados econômicos ou sociais do Brasil
  por período ou estado. Nunca invente dados. Se não achar, pense mais e se não achar de jeito nenhum, diga que não achou.
---
# Skill: IBGE — Dados Econômicos e Sociais

## Propósito
Buscar dados oficiais do IBGE via API pública para embasar
análises de governos na PoliticAI. Nunca invente dados —
use sempre a API ou informe que o dado não está disponível.

## APIs Disponíveis

### SIDRA — Sistema IBGE de Recuperação Automática
Base principal. Acessa tabelas de PIB, desemprego, inflação, pobreza, etc.
URL base: `https://apisidra.ibge.gov.br/values`

### IBGE Localidades
Busca estados e municípios por código.
URL base: `https://servicodados.ibge.gov.br/api/v1/localidades`
## Como Buscar Dados

### Passo a Passo
1. Identifique o indicador pedido pelo usuário (PIB, desemprego, etc.)
2. Consulte a tabela correspondente na seção "Tabelas Disponíveis" abaixo
3. Monte a URL com o código da tabela e o período
4. Faça a requisição e extraia os valores
5. Sempre informe a fonte: "Fonte: IBGE/SIDRA, Tabela [código]"

### Código Python para Requisição
```python
import requests

def buscar_ibge(tabela, periodo, variavel="allxp"):
    url = f"https://apisidra.ibge.gov.br/values/t/{tabela}/n1/all/v/{variavel}/p/{periodo}"
    response = requests.get(url)
    dados = response.json()
    # O primeiro item é o cabeçalho, os demais são os dados
    cabecalho = dados[0]
    valores = dados[1:]
    return valores

# Exemplo: buscar PIB de 2010 a 2022
resultado = buscar_ibge(tabela="1621", periodo="2010-2022")
```

### Formato da URL SIDRA
```
https://apisidra.ibge.gov.br/values/t/{tabela}/n1/all/v/{variavel}/p/{periodo}

Onde:
- {tabela}   → código da tabela (veja seção abaixo)
- {variavel} → código da variável ou "allxp" para todas
- {periodo}  → ano (ex: 2022) ou intervalo (ex: 2010-2022)
```
## Tabelas Disponíveis

| Indicador | Tabela SIDRA | Variável | Observação |
|---|---|---|---|
| PIB — crescimento anual | 1621 | 583 | % de variação real |
| PIB — valor em R$ | 1620 | 582 | Em milhões de reais |
| Desemprego (PNAD) | 4099 | 4099 | Taxa % trimestral |
| Inflação (IPCA) | 1737 | 2266 | Variação % mensal |
| Inflação acumulada (IPCA) | 1737 | 63 | Variação % anual |
| Pobreza extrema | 6691 | 10835 | % da população |
| Desigualdade (Gini) | 6691 | 10836 | Índice 0 a 1 |
| Taxa de analfabetismo | 9543 | 10688 | % acima de 15 anos |
| Mortalidade infantil | 2612 | 218 | Por mil nascidos vivos |
| População total | 6579 | 9324 | Estimativa anual |

## Exemplo Real — Buscando o PIB do governo Lula (2003-2010)

```python
import requests

def buscar_pib(ano_inicio, ano_fim):
    url = (
        f"https://apisidra.ibge.gov.br/values/"
        f"t/1621/n1/all/v/583/p/{ano_inicio}-{ano_fim}"
    )
    response = requests.get(url)
    dados = response.json()[1:]  # pula o cabeçalho
    
    for item in dados:
        print(f"Ano: {item['D3N']} | PIB: {item['V']}%")

buscar_pib(2003, 2010)
```

Saída esperada:
```
Ano: 2003 | PIB: 1.1%
Ano: 2004 | PIB: 5.8%
Ano: 2005 | PIB: 3.2%
...
```
## Regras de Uso

### O que SEMPRE fazer
- Sempre instalar requests antes de usar: `pip install requests`
- Sempre informar a fonte ao apresentar um dado:
  `Fonte: IBGE/SIDRA, Tabela [código], acessado em [data]`
- Sempre mostrar o período do dado (ano ou trimestre)
- Se o dado vier em milhões de reais, deixar claro na apresentação

### O que NUNCA fazer
- Nunca inventar valores — se a API não retornar, informe o usuário
- Nunca usar dados de fontes que não sejam o IBGE para esta skill
- Nunca apresentar dado sem informar o ano de referência
- Nunca comparar indicadores de períodos diferentes sem avisar

### Quando a API falhar
```python
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    dados = response.json()
except Exception as e:
    print(f"⚠️ Não foi possível buscar o dado no IBGE: {e}")
    print("Verifique sua conexão ou tente novamente mais tarde.")
```

### Mensagem padrão ao apresentar dados
📊 Dado oficial — IBGE

Indicador : [nome]

Período   : [ano ou intervalo]

Valor     : [resultado]

Fonte     : IBGE/SIDRA, Tabela [código]

## Parte 6 — Encontrar Qualquer Pesquisa do IBGE

### Pesquisar o catálogo completo do IBGE
Quando o usuário pedir um dado que não está na tabela acima,
use a API de metadados para encontrar a tabela correta.

```python
import requests

def pesquisar_catalogo(termo):
    url = f"https://servicodados.ibge.gov.br/api/v2/pesquisas"
    response = requests.get(url)
    pesquisas = response.json()
    
    # filtra pelo termo buscado
    encontradas = [
        p for p in pesquisas
        if termo.lower() in p['nome'].lower()
    ]
    for p in encontradas:
        print(f"ID: {p['id']} | Nome: {p['nome']}")
    return encontradas

# Exemplo
pesquisar_catalogo("agricultura")
pesquisar_catalogo("saúde")
pesquisar_catalogo("educação")
```

### Buscar tabelas dentro de uma pesquisa
```python
def buscar_tabelas_da_pesquisa(id_pesquisa):
    url = f"https://servicodados.ibge.gov.br/api/v2/pesquisas/{id_pesquisa}/indicadores"
    response = requests.get(url)
    indicadores = response.json()
    
    for ind in indicadores:
        print(f"Indicador: {ind['id']} | {ind['nome']}")
    return indicadores

# Exemplo: PAM tem id 24 no IBGE
buscar_tabelas_da_pesquisa(24)
```

### Buscar o resultado de qualquer indicador
```python
def buscar_indicador(pesquisa_id, indicador_id, periodo="2010|2011|2012|2013|2014|2015|2016|2017|2018|2019|2020|2021|2022"):
    url = (
        f"https://servicodados.ibge.gov.br/api/v3/agregados/"
        f"{pesquisa_id}/periodos/{periodo}/variaveis/{indicador_id}"
        f"?localidades=N1[all]"
    )
    response = requests.get(url)
    return response.json()
```

### IDs das principais pesquisas do IBGE

| Pesquisa | ID | O que contém |
|---|---|---|
| PNAD Contínua | 6381 | Desemprego, renda, informalidade trimestral |
| PNAD (antiga) | 4 | Séries históricas até 2015 |
| PAM | 1618 | Produção agrícola por município |
| Censo Demográfico | 3 | População, renda, educação, moradia |
| PIB dos Municípios | 5938 | PIB por município |
| PNS | 5267 | Pesquisa Nacional de Saúde |
| MUNIC | 48 | Gestão e serviços municipais |
| RAIS | 57 | Mercado de trabalho formal |
| Pecuária (PPM) | 74 | Produção pecuária municipal |
| Aquicultura | 3940 | Produção de pesca e aquicultura |

### Fluxo completo para qualquer dado
1. Usuário pede um dado → identifique a pesquisa na tabela acima
2. Se não souber o ID → use `pesquisar_catalogo(termo)`
3. Com o ID da pesquisa → use `buscar_tabelas_da_pesquisa(id)`
4. Com o ID do indicador → use `buscar_indicador(pesquisa_id, indicador_id)`
5. Apresente o resultado com fonte e período

## Parte 7 — Ler o Dicionário Antes de Buscar

### Por que isso é importante
Cada tabela do IBGE tem variáveis com códigos que mudam ou
se acumulam com o tempo. Nunca assuma o código da variável
— sempre consulte o dicionário da tabela primeiro.

### Passo 1 — Consultar o dicionário de variáveis da tabela
```python
import requests

def ler_dicionario(tabela_id):
    url = f"https://servicodados.ibge.gov.br/api/v3/agregados/{tabela_id}/variaveis"
    response = requests.get(url)
    variaveis = response.json()
    
    print(f"Variáveis disponíveis na tabela {tabela_id}:")
    for v in variaveis:
        print(f"  ID: {v['id']} | Nome: {v['nome']} | Unidade: {v.get('unidade', '—')}")
    return variaveis

# Sempre faça isso antes de buscar qualquer dado
ler_dicionario(1621)  # exemplo: tabela do PIB
```

### Passo 2 — Consultar os períodos disponíveis
```python
def ler_periodos(tabela_id):
    url = f"https://servicodados.ibge.gov.br/api/v3/agregados/{tabela_id}/periodos"
    response = requests.get(url)
    periodos = response.json()
    
    anos = [p['id'] for p in periodos]
    print(f"Períodos disponíveis: {anos[0]} até {anos[-1]}")
    return anos

ler_periodos(1621)
```

### Passo 3 — Montar a requisição com os códigos corretos
```python
def buscar_dado_correto(tabela_id, nome_variavel_buscada, periodo):
    # 1. lê o dicionário
    variaveis = ler_dicionario(tabela_id)
    
    # 2. encontra a variável pelo nome
    variavel = next(
        (v for v in variaveis if nome_variavel_buscada.lower() in v['nome'].lower()),
        None
    )
    
    if not variavel:
        print(f"⚠️ Variável '{nome_variavel_buscada}' não encontrada na tabela {tabela_id}.")
        print("Variáveis disponíveis:")
        for v in variaveis:
            print(f"  → {v['nome']}")
        return None
    
    # 3. monta a URL com o código correto
    variavel_id = variavel['id']
    url = (
        f"https://servicodados.ibge.gov.br/api/v3/agregados/"
        f"{tabela_id}/periodos/{periodo}/variaveis/{variavel_id}"
        f"?localidades=N1[all]"
    )
    
    response = requests.get(url)
    dados = response.json()
    
    print(f"📊 {variavel['nome']} — Tabela {tabela_id}")
    for serie in dados[0]['resultados'][0]['series']:
        for ano, valor in serie['serie'].items():
            print(f"  {ano}: {valor} {variavel.get('unidade', '')}")
    
    return dados

# Exemplo de uso correto
buscar_dado_correto(
    tabela_id=1621,
    nome_variavel_buscada="variação",
    periodo="2003|2004|2005|2006|2007|2008|2009|2010"
)
```

### Fluxo obrigatório — sempre nessa ordem
1. `ler_dicionario(tabela_id)` → descobre as variáveis disponíveis
2. `ler_periodos(tabela_id)` → descobre os anos disponíveis
3. `buscar_dado_correto(tabela_id, nome, periodo)` → busca com código certo
4. Apresenta resultado com fonte e unidade correta
---
*Skill criada para a PoliticAI — análise baseada exclusivamente em dados oficiais.*
