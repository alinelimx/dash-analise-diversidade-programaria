# Análise de Diversidade na Área de Dados - Dashboard em Power BI

Dashboard criado como projeto final do curso de Análise de Dados do [Programaria](https://www.programaria.org/cursos-programaria/) sobre representatividade de gênero e raça entre profissionais de dados no Brasil, construído a partir de uma pesquisa com 4271 respondentes.

O projeto inclui o tratamento de dados, o modelo de dados em estrela e as medidas DAX, com destaque para três problemas encontrados nos dados originais que, se ignorados, produziriam números errados.

---

## Prévia
<img width="1192" height="672" alt="pagina-1-visao-geral" src="https://github.com/user-attachments/assets/3c178084-b548-4570-8ee7-8fd50f3e557d" />

<img width="1202" height="672" alt="pagina-2-carreira" src="https://github.com/user-attachments/assets/adfc8401-40b0-4e8f-ad55-144c3f981b6b" />

<img width="1197" height="675" alt="pagina-3-barreiras" src="https://github.com/user-attachments/assets/5f8a47f4-b1f5-4f8f-8206-310a573be649" />


---

## Sobre os dados

- **Fonte:** pesquisa State of Data Brazil, edição 2022
- **Respondentes:** 4271
- **Colunas originais:** 46
- **Natureza:** participação voluntária, divulgada em canais da comunidade de dados

A amostra é autosselecionada. Os resultados descrevem **quem respondeu**, não o mercado brasileiro de dados como um todo. Logo, pessoas engajadas com a comunidade tendem a estar sobre-representadas.

---

## Principais achados

**A área é majoritariamente masculina e a desigualdade aumenta com a senioridade.**

| Indicador | Valor |
|---|---|
| Mulheres | 24.7% |
| Pessoas pretas e pardas | 31.5% |
| Pessoas com deficiência | 1.3% |
| Mulheres em cargos de gestão | 18.5% |

**O funil de senioridade.** A participação feminina se mantém estável nos níveis iniciais e cai a partir do sênior:

| Senioridade | % mulheres | n |
|---|---|---|
| Júnior | 26.6% | 1.023 |
| Pleno | 27.8% | 1.060 |
| Sênior | 20.8% | 898 |
| Pessoa gestora | 18.5% | 713 |

**O gap salarial se concentra na gestão.** Comparando a mediana salarial dentro de cada nível e não a média geral, que mediria apenas a composição da amostra:

| Senioridade | Mulheres | Homens | Diferença |
|---|---|---|---|
| Júnior | R$ 3.284 | R$ 3.401 | -3.4% |
| Pleno | R$ 7.042 | R$ 7.175 | -1.9% |
| Sênior | R$ 11.327 | R$ 11.798 | -4% |
| Pessoa gestora | R$ 13.805 | R$ 16.448 | **-16.1%** |

**Não se explica por escolaridade.** A proporção de mulheres vai de 17.1% entre estudantes de graduação a 29.8% no doutorado, ou seja, elas são proporcionalmente mais presentes nos níveis mais altos de formação e ainda assim menos presentes na gestão.

**Metade das mulheres relata prejuízo na carreira.** Entre as 2190 pessoas a quem a pergunta foi feita, 53.3% das mulheres afirmam que sua experiência profissional foi prejudicada, contra 25.4% dos homens. O aspecto com maior diferença é a atenção dada às próprias ideias: **31.9% das mulheres contra 8.8% dos homens**.

**A composição racial varia muito por região.** De 51.1% de pessoas pretas e pardas no Nordeste a 17.3% no Sul. Na base como um todo, 31.5%, o que é abaixo da proporção na população brasileira.

---

## Tratamento dos dados

A parte mais importante do projeto. A base original tinha três problemas que produziriam conclusões erradas se usada como veio.

### 1. Salários preenchidos artificialmente

577 registros tinham `SALARIO` igual a exatamente **R$ 7.625,50**, e outros 19 com R$ 53.127,85. Não era coincidência: os valores faltantes haviam sido imputados com a média antes da distribuição do arquivo.

A confirmação veio do cruzamento: todos os 577 tinham `FAIXA SALARIAL` vazia e correspondiam a pessoas sem vínculo de trabalho (350 desempregadas, 126 apenas estudantes, 86 na academia).

**Tratamento:** os 596 valores foram anulados. Mantê-los achataria qualquer média salarial em torno de R$ 7.625 e comprometeria justamente a análise de gap.

### 2. Denominador diferente na pergunta sobre prejuízo na carreira

A coluna `EXPERIENCIA_PROFISSIONAL_PREJUDICADA` tinha 2081 valores vazios. Não eram não-respostas: a pergunta simplesmente **não foi feita** a essas pessoas.

Cruzando com gênero e raça: os vazios eram 2079 homens e 2065 pessoas brancas, sem nenhuma pessoa com deficiência. O formulário exibia a pergunta apenas a quem pertencia a pelo menos um grupo sub-representado.

**Tratamento:** criada a coluna `Respondeu_Prejuizo` e todas as medidas relacionadas usam 2190 como denominador. Calculando sobre a base inteira, o indicador sairia 20.1% em vez dos 39.2% reais.

### 3. Coluna com valores invertidos

`MUDOU DE ESTADO?` indicava 3363 pessoas com valor `1` e 808 com `0`, o que sugeriria que 81% haviam mudado de estado.

O cruzamento com `REGIAO DE ORIGEM` (preenchida apenas para quem migrou) mostrou que os 772 registros com origem declarada estavam todos no grupo `0`. A coluna provavelmente nasceu de uma pergunta como "mora no estado de origem?" e foi renomeada de forma invertida.

**Tratamento:** recodificada como `Mudou_de_Estado`, com rótulos textuais explícitos. 808 pessoas (19%) mudaram de estado.

### Outros ajustes

- **74 idades corrompidas:** todas as pessoas da faixa 55+ tinham `IDADE` preenchida com o texto `31.153.517.220.250.300`. Valores anulados; a faixa etária segue utilizável.
- **590 células com lixo de exportação** (`0` e `1`) na coluna de linguagens, convertidas em nulo.
- **Colunas duplicadas removidas:** duas perguntas de experiência apareciam em versão maiúscula e minúscula; `NIVEL_Júnior/Pleno/Sênior` eram redundantes e incompletas (pessoas gestoras apareciam como `FALSE` nas três).
- **Colunas de múltipla resposta** (aspectos prejudicados, critérios de escolha, linguagens, motivos de insatisfação) separadas em tabelas auxiliares, uma linha por pessoa-e-resposta. Duas opções de `ASPECTOS_PREJUDICADOS` contêm vírgula no próprio texto e precisaram ser protegidas antes da divisão.
- **Colunas de ordenação** criadas para senioridade, faixa salarial, escolaridade, faixa etária e porte da empresa.
- **"Prefiro não informar" mantido** em gênero, raça e PCD. Remover essas categorias reduziria o total e distorceria todos os percentuais.

---

## Modelo de dados

Modelo em estrela, com `fFonte` no centro e quatro tabelas auxiliares para as perguntas de múltipla resposta. Relacionamentos 1:* com direção de filtro única, a partir de `fFonte[Indice]`.

| Tabela | Linhas | Grão |
|---|---|---|
| `fFonte` | 4271 | uma linha por pessoa |
| `fAspectos` | 2777 | pessoa × aspecto prejudicado |
| `fCriterios` | 10.272 | pessoa × critério de escolha |
| `fLinguagens` | 4377 | pessoa × linguagem |
| `fMotivos` | 1920 | pessoa × motivo de insatisfação |

Nas tabelas auxiliares, a soma dos percentuais ultrapassa 100%, já que cada pessoa podia marcar várias opções. As medidas dividem por `DISTINCTCOUNT(fFonte[Indice])`, não pelo total de linhas.

---

## Medidas DAX

```dax
Total Pessoas = COUNTROWS(fFonte)
```

```dax
% Mulheres =
DIVIDE(
    CALCULATE([Total Pessoas], fFonte[Genero] = "Feminino"),
    [Total Pessoas]
)
```

```dax
% Mulheres na Gestão =
DIVIDE(
    CALCULATE([Total Pessoas],
        fFonte[Genero] = "Feminino",
        fFonte[Senioridade] = "Pessoa Gestora"),
    CALCULATE([Total Pessoas], fFonte[Senioridade] = "Pessoa Gestora")
)
```

```dax
Mediana Salarial = MEDIAN(fFonte[Salario_Limpo])
```

```dax
-- Denominador restrito a quem recebeu a pergunta
% Prejudicados =
DIVIDE(
    CALCULATE([Total Pessoas], fFonte[Prejudicado_SN] = "Sim"),
    CALCULATE([Total Pessoas], fFonte[Respondeu_Prejuizo] = "Respondeu")
)
```

```dax
-- Percentual de pessoas que citaram cada item, nas tabelas auxiliares
% Que Citou =
DIVIDE(
    DISTINCTCOUNT(fAspectos[Indice]),
    CALCULATE(DISTINCTCOUNT(fFonte[Indice]), fFonte[Respondeu_Prejuizo] = "Respondeu")
)
```

---

## Decisões de visualização

- **Mediana em vez de média** para salário, pela cauda longa à direita.
- **Comparação dentro de cada nível de senioridade** e não entre gêneros no agregado, para não medir composição da amostra em vez de desigualdade.
- **Barras 100% empilhadas** onde os grupos comparados têm tamanhos muito diferentes.
- **Tamanho da amostra nas dicas de ferramenta** de todo gráfico de percentual ou mediana.
- **Paleta verificada para daltonismo**, com separação mínima entre cores adjacentes nos três tipos principais e contraste de pelo menos 3:1 contra o fundo.
- **Títulos curtos com subtítulo interpretativo**, para o leitor obter o achado sem precisar decifrar os eixos.

---

## Limitações

- Amostra autosselecionada, não probabilística.
- Os salários são valores estimados dentro da faixa declarada pela pessoa, não valores informados diretamente. Servem para mediana e comparação entre grupos, não para precisão absoluta.
- Grupos pequenos como "Prefiro não informar" em gênero (21 pessoas), Indígena (11), pessoas com deficiência (54) aparecem nos gráficos, mas não sustentam conclusões estatísticas.
- Comparações entre grupos raciais na pergunta sobre prejuízo exigem cuidado: entre as pessoas brancas que responderam, 96% são mulheres, o que torna a comparação direta com o grupo negro (majoritariamente masculino) um artefato de composição.
- Dados de 2022.

---

## Ferramentas

Power BI Desktop · Power Query (M) · DAX

