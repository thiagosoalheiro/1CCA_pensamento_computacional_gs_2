# Bridge AI (Citizen Assist AI)

**Global Solution 2026.2 · CitizenTech · Pensamento Computacional e Automação com Python**
Assistente de orientação ao cidadão executado no terminal, com base de conhecimento em JSON, busca por pontuação e módulo **"Vale a viagem?"**.

> Os trechos marcados com **[PREENCHER]** dependem de dados que só a equipe tem (RM, resultado do teste, link do vídeo).

## Integrantes

| Nome | RM |
|---|---|
| Thiago **[PREENCHER sobrenome]** | **[PREENCHER]** |
| Kauan Damasceno de Lima | **[PREENCHER]** |

## Público e problema

**Público:** idosos que vivem no campo (população rural que envelhece, muitas vezes longe da cidade e com internet instável).

**Problema:** para resolver um serviço público (prova de vida, aposentadoria rural, BPC, Cartão SUS...), o idoso do campo costuma gastar um dia inteiro de viagem sem saber se a ida era necessária, se ele tem os documentos certos ou se dava para resolver pela internet ou pelo telefone. A informação existe, mas está espalhada e em linguagem técnica.

**Solução:** o Bridge AI (a ponte entre o campo e a cidade) responde a dúvida com **fonte oficial e data de verificação** e, em seguida, responde a pergunta que mais importa: **"Vale a viagem?"**

## ODS relacionados

- **ODS 1** (Erradicação da pobreza): orienta sobre benefícios como o BPC e o CadÚnico.
- **ODS 3** (Saúde e bem-estar): Cartão SUS, vacinação e Farmácia Popular.
- **ODS 10** (Redução das desigualdades): reduz a distância de acesso entre campo e cidade.
- **ODS 16** (Instituições eficazes): informação clara, com fonte, e alerta contra golpes.
- **ODS 9** (Inovação e infraestrutura): funciona offline, com base local.

## Como executar

Requisitos: **Python 3.8 ou mais novo**. Não precisa instalar nenhuma biblioteca.

**PyCharm:** `File > Open` e escolha a pasta do projeto, abra o `main.py` e clique em **Run**.
**Terminal:** dentro da pasta do projeto, `python main.py` (ou `python3 main.py`).

Como a base é um arquivo local, o assistente **funciona sem internet**.

## Estrutura

```
citizen-assist-ai/
├── README.md
├── main.py                 # menu principal
├── orientacao.py           # classe Orientacao (um registro do JSON)
├── base_conhecimento.py    # classe BaseConhecimento (JSON, categorias, busca por pontuação)
├── assistente.py           # classe Assistente (atendimento, histórico, indicadores, CSV)
└── data/
    ├── base_conhecimento.json   # 12 orientações, 4 categorias
    ├── historico.json           # um dicionário por atendimento
    └── atendimentos.csv         # exportado pelo menu (opção 4)
```

As classes se encaixam por **composição**: `Assistente` tem uma `BaseConhecimento`, que tem uma lista de objetos `Orientacao`.

## Regra de pontuação

A pergunta e os textos da base são normalizados (minúsculas, sem acentos, sem pontuação) e cada orientação ganha pontos:

| Regra | Pontos |
|---|---|
| Cada **palavra-chave** da orientação que aparece na pergunta | +3 |
| Cada **palavra da pergunta** (4 letras ou mais) que aparece no **título** | +1 |

| Melhor orientação | Resultado |
|---|---|
| 6 pontos ou mais | Confiança **ALTA** |
| 3 a 5 pontos | Confiança **MÉDIA** |
| menos de 3 pontos | "Não encontrei": sugere outras palavras e as categorias |

`buscar()` devolve uma lista de tuplas `(pontos, orientacao)` da maior para a menor pontuação (empate: menor `id`). O assistente mostra a melhor e até duas alternativas com 3 pontos ou mais.

**Exemplo:** "Como faço o cartão do SUS?" → palavras-chave `cartao do sus` (+3) e `sus` (+3) e a palavra `cartao` no título (+1) = **7 pontos, confiança ALTA**.

**Ajuste da equipe:** palavras muito comuns (como, para, fazer, sobre...) não ganham o ponto do título, para não puxar a orientação errada. A lista está em `PALAVRAS_COMUNS` no `base_conhecimento.py`.

## Módulo diferencial: "Vale a viagem?"

Depois de cada resposta, o assistente diz se vale gastar o dia indo à cidade. Cada registro do JSON ganhou dois campos extras:

- `exige_presenca`: `true` se o serviço exige ir pessoalmente, `false` se dá para resolver à distância;
- `alternativa_remota`: onde resolver sem ir (Meu INSS, telefone 135, aplicativo, SMS).

Se não exige presença, o assistente diz **"Não precisa ir"** e mostra a alternativa. Se exige, faz duas perguntas de sim/não e aplica a regra da lógica booleana:

> **S = P · (D + U)** — vale a viagem quando o serviço **exige presença (P)** E (você **tem os documentos (D)** OU o caso é **urgente (U)**).

| P | D | U | S |
|---|---|---|---|
| 0 | x | x | 0 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

O registro guarda `viagem_evitada`, o que gera o indicador extra **"Orientações que evitaram uma viagem"**.

## Privacidade e transparência

- O assistente **nunca pede** nome, CPF, telefone ou e-mail. Pergunta com **8 ou mais dígitos** ou com **@** é recusada e **não é registrada**.
- Toda resposta mostra **fonte, data de verificação** e o aviso de que **o assistente não é canal oficial**.
- As perguntas "Vale a viagem?" são de sim/não e não identificam ninguém.
- JSON ausente, corrompido ou com registro inválido **não trava** o programa: ele avisa e segue (ou encerra com mensagem clara se não houver nenhuma orientação válida).

## Histórico, indicadores e CSV

Cada atendimento é salvo em `data/historico.json` com: `data`, `origem` (pergunta ou categoria), `pergunta`, `id_orientacao`, `encontrou`, `ajudou`, `tentativas`, `tempo_segundos`, mais `viagem_evitada`, `sessao` e `nota_facilidade`.

Indicadores (menu, opção 3):

- **Resposta encontrada (%)**: atendimentos em que houve orientação / total.
- **Taxa de acerto (%)**: atendimentos em que a pessoa respondeu "s" a "respondeu sua dúvida?" / total.
- **Tempo médio para encontrar**: tempo desde o início do atendimento até a orientação aparecer (só os atendimentos com resposta).
- **Tentativas por atendimento**: média de perguntas digitadas até achar (ou desistir).
- **Facilidade de uso (1 a 5)**: média das notas pedidas ao sair (uma nota por execução).
- **Dúvidas sem solução**: perguntas não encontradas ou em que a pessoa disse que não ajudou.
- **Orientações que evitaram uma viagem (%)** (extra).

O CSV (`data/atendimentos.csv`) é gerado com `csv.DictWriter` na opção 4 do menu.

## Integrações com outras disciplinas

- **Computer Science (lógica booleana):** a regra `S = P · (D + U)` do "Vale a viagem?" vem da atividade de tabela-verdade, mintermos, simplificação e Karnaugh; está implementada em `regra_vale_a_viagem()`, no `assistente.py`.
- **[PREENCHER]** outras integrações, se houver.

A busca por pontuação é independente dessas integrações e é a parte avaliada em PCP.

## Fontes consultadas

Todas as orientações foram conferidas em páginas oficiais em **11/11/2026** (a data fica no campo `verificado_em` de cada registro).

| Id | Orientação | Fonte |
|---|---|---|
| 1, 2 | Prova de vida e alerta de golpe | https://www.gov.br/inss/pt-br/noticias/prova-de-vida-nao-foi-suspensa-e-obrigatoria-e-por-cruzamento-de-dados |
| 3 | Aposentadoria por idade rural | https://www.gov.br/pt-br/servicos/solicitar-aposentadoria-por-idade-para-trabalhador-rural |
| 4, 5 | BPC/LOAS; Meu INSS e telefone 135 | https://www.gov.br/pt-br/servicos/solicitar-beneficio-assistencial-ao-idoso |
| 6 | Cartão SUS e CPF | https://agenciagov.ebc.com.br/noticias/202509/ministerio-da-saude-comeca-a-emitir-cartao-sus-com-base-no-cpf |
| 7 | Vacina contra a gripe | https://www.gov.br/saude/pt-br/campanhas-da-saude/2025/gripe |
| 8, 9 | Farmácia Popular; fraldas geriátricas | https://agenciagov.ebc.com.br/noticias/202503/saiba-como-retirar-medicamentos-e-insumos-pelo-farmacia-popular |
| 10 | CadÚnico | https://www.gov.br/pt-br/servicos/inscrever-se-no-cadastro-unico-para-programas-sociais-do-governo-federal |
| 11 | CAF | https://www.gov.br/mda/pt-br/acesso-a-informacao/acoes-e-programas/programas-projetos-acoes-obras-e-atividades/cadastro-nacional-da-agricultura-familiar/como-obter-o-caf |
| 12 | Alertas da Defesa Civil por SMS | https://www.parana.pr.gov.br/servicos/Seguranca/Defesa-Civil/Receber-alertas-da-Defesa-Civil-por-SMS-ERrZ4PN6 |

## Teste com usuários  **[PREENCHER depois do teste]**

Mínimo: 3 pessoas de fora da equipe e 10 atendimentos. Não registrar nomes nem dados das pessoas.

| Indicador | Resultado |
|---|---|
| Atendimentos | [PREENCHER] |
| Resposta encontrada (%) | [PREENCHER] |
| Taxa de acerto (%) | [PREENCHER] |
| Tempo médio para encontrar | [PREENCHER] |
| Tentativas por atendimento | [PREENCHER] |
| Facilidade de uso (1 a 5) | [PREENCHER] |
| Orientações que evitaram uma viagem (%) | [PREENCHER] |

**Dúvidas sem solução:** [PREENCHER]
**Melhoria feita a partir do teste:** [PREENCHER] (ex.: nova palavra-chave no JSON)

## Limitações

- A "inteligência" é **baseada em regras** (palavras-chave e pontuação), sem modelo de linguagem: ela só responde o que está na base e não entende frases muito diferentes das palavras-chave.
- A base tem 12 orientações. As regras dos serviços **mudam**: é preciso reconferir as fontes de tempos em tempos.
- Algumas fontes são notícias oficiais de 2025 (Farmácia Popular, Cartão SUS, gripe); o texto de cada orientação avisa quando é preciso confirmar a informação atual.
- A orientação sobre alertas da Defesa Civil usa a página do Paraná; o serviço pode variar por estado.
- A orientação sobre prova de vida se baseia na página do INSS conferida em 11/11/2026; vale reconferir no Meu INSS ou pelo 135.
- Este assistente **não é um canal oficial** e não confirma direitos ou benefícios.

## Vídeo técnico

Link (YouTube, não listado): **[PREENCHER]**

| Item do roteiro | Onde está no código |
|---|---|
| JSON, dicionários e tuplas | `data/base_conhecimento.json`; tuplas em `BaseConhecimento.buscar()` |
| Classes, atributos, métodos, `self`, composição | `orientacao.py`, `base_conhecimento.py`, `assistente.py` |
| `normalizar()` e `pontuar()` com cálculo real | `base_conhecimento.py` |
| Execução (achou, não achou, inválida, dado pessoal, categorias) | `main.py` → `Assistente.perguntar()` e `navegar()` |
| Alteração ao vivo | incluir a palavra-chave `"aposentar"` na orientação 3 do JSON e repetir "quero me aposentar" (antes: não encontrado; depois: encontrado, confiança MÉDIA) |
| `historico.json`, CSV e indicadores | `Assistente.registrar()`, `exportar_csv()`, `indicadores()` |
| Decisão técnica e limitação | seção **Limitações** e a regra `P · (D + U)` |
