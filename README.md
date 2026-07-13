# Manual do Usuário — WGPT

Objetivo desse repositório é explicar **o que cada tela faz**, **como a IA decide quem responde** e **como escrever pedidos** para obter as melhores respostas.

---

## 1. Visão geral

Depois do login, você chega na tela **Início**, onde escolhe um dos módulos disponíveis:

```mermaid
flowchart TD
    A[Login] --> B[Início — escolha do módulo]
    B --> C["Análise Jurídica<br/>(chat multiagentes)"]
    B --> D["Revisão Textual<br/>(revisão de um documento)"]
    B --> E["Análise de Legislação<br/>(estudo de leis e normativos)"]
    B --> F["Consulta Jurisprudencial<br/>(em breve)"]
```

| Módulo | Para que serve |
|---|---|
| **Análise Jurídica** | Conversa (chat) para dúvidas, teses, estratégias, riscos e resumos. Por trás, vários agentes especializados dividem o trabalho — veja a seção 2. |
| **Revisão Textual** | Envie um texto ou documento e receba uma avaliação de redação (ortografia, clareza, estrutura). Revisão única, sem follow-up de conversa. |
| **Análise de Legislação** | Envie uma lei, portaria ou normativo, e receba uma análise de adequação, riscos e oportunidades, com apoio de pesquisa quando necessário. Análise única, sem follow-up de conversa. |
| **Consulta Jurisprudencial** | Ainda não disponível — buscará decisões e precedentes diretamente no Banco Nacional de Precedentes (BNP/CNJ). |

---

## 2. Como o Orquestrador escolhe o agente certo (Análise Jurídica)

No módulo **Análise Jurídica**, sua mensagem nunca é respondida "no escuro" por um único robô genérico. Ela primeiro passa pelo **Orquestrador**, que lê o pedido (e a instrução que você eventualmente selecionou — ver seção 3) e decide **qual especialista** deve assumir a resposta, com base no tipo de tarefa identificada. O Orquestrador nunca elabora a resposta jurídica em si — só direciona.

O Orquestrador também sabe lidar com duas situações antes de rotear:

- **Perguntas sobre a própria plataforma** — se você perguntar o que ele ou os demais agentes fazem (ex.: "quais análises você consegue fazer?"), ele responde brevemente explicando as funcionalidades e especialidades disponíveis, em vez de tentar encaminhar isso como se fosse uma tarefa jurídica.
- **Pedidos genuinamente ambíguos** — se a tarefa não puder ser identificada mesmo pelo contexto (ex.: "preciso de uma análise", sem dizer análise do quê), o Orquestrador pede um esclarecimento objetivo antes de escolher o especialista, em vez de arriscar um encaminhamento errado. Se o pedido já tiver informação suficiente para inferir a natureza da tarefa, ele segue direto para o especialista, sem perguntas desnecessárias.

```mermaid
flowchart TD
    U["Sua mensagem"] --> O{"Orquestrador<br/>identifica o tipo de tarefa"}
    O -->|"dúvida, orientação, explicação"| CO["Consultor"]
    O -->|"tese, estratégia, parecer, riscos"| ES["Estrategista"]
    O -->|"resumo de petição/documento"| SU["Sumarizador"]
    CO -->|"precisa de pesquisa"| PE["Pesquisador"]
    ES -->|"precisa de pesquisa"| PE
    PE -->|"devolve o material pesquisado"| CO
    PE -->|"devolve o material pesquisado"| ES
    CO --> R["Resposta final na conversa"]
    ES --> R
    SU --> R
```

### Os agentes e suas especialidades

| Agente | Especialidade | Aparece na conversa? |
|---|---|---|
| **Orquestrador** | Só direciona — identifica a natureza do pedido e passa a tarefa ao especialista certo. Explica as funcionalidades da plataforma quando perguntado e pede esclarecimento em pedidos genuinamente ambíguos. Não elabora a resposta final e **não pesquisa nada por conta própria**. | Não responde diretamente |
| **Consultor** | Orientações jurídicas objetivas e dúvidas em geral (conceitos, dúvidas processuais). Pode acionar o Pesquisador. | Sim |
| **Estrategista** | Teses jurídicas, estratégias de defesa, pareceres técnicos, análise de viabilidade e riscos, contraposição de argumentos. Pode acionar o Pesquisador. | Sim |
| **Sumarizador** | Resumo e extração de informações de documentos jurídicos anexados. **Não tem acesso ao Pesquisador** — trabalha só com o que foi anexado/enviado na conversa. | Sim |
| **Pesquisador** | Busca jurisprudência, legislação e conteúdo na internet. **Nunca responde você diretamente** — sempre entrega o resultado da pesquisa de volta para quem o acionou concluir a resposta. | Não responde diretamente |

Importante: quem decide acionar o Pesquisador é o **agente especializado que está com a tarefa** (Consultor ou Estrategista) no meio da própria resposta, sempre que perceber que precisa de uma fonte (lei, jurisprudência ou informação atual) antes de concluir — o Orquestrador não pesquisa e não aciona o Pesquisador diretamente, e o Sumarizador não tem essa opção.

No painel **"Ver raciocínio"** abaixo de cada resposta, você acompanha em tempo real qual agente está ativo, quando ocorre um repasse (handoff) para outro agente e quais ferramentas de pesquisa foram consultadas — útil para conferir a base da resposta antes de confiar nela.

## 3. Boas práticas de prompt

Quanto mais contexto e mais claro o objetivo, melhor o agente identifica a tarefa e melhor é a resposta. Evite pedidos genéricos, especialmente quando há um documento anexado.

### Use as sugestões de instrução (pills)

![pills](pills.png)

Na Análise Jurídica, acima do campo de mensagem existem **pills de instrução** organizadas por categoria — clicar em uma delas já direciona o tom e o tipo de tarefa para o Orquestrador:

| Categoria | Instruções disponíveis |
|---|---|
| **Orientação** | Explicar conceitualmente • Tirar dúvida processual |
| **Estratégia** | Desenvolver tese • Revisar fundamentações • Analisar viabilidade e riscos • Elaborar parecer técnico • Contrapor argumentos |
| **Documentos** | Resumir petição |


A pill selecionada é somada à sua mensagem antes do envio e se desmarca sozinha depois — você pode digitar livremente além dela.

### Exemplos — do genérico ao eficaz

| ❌ Evite | ✅ Prefira | 💡 Motivo |
|---|---|---|
| "Me ajude com essa petição" (com anexo) | "Analise a viabilidade de recurso neste acórdão anexo, focando em prequestionamento e violação ao art. 927 do CPC. Traga um parecer estruturado com riscos e prazo recursal. | Explicar o documento e sua estrutura, poupa tokens que seriam utilizados para o agente identificar o tipo de tarefa que irá executar, e comprender o documento anexado." |
| "O que diz a lei sobre isso?" | "Explique conceitualmente os requisitos de estabilidade do servidor público estatutário segundo a Lei 8.112, citando os artigos aplicáveis." | Define o objeto central da análise sem que o agente precise adivinhar. |
| "Resume esse documento" | "Resuma a petição anexa em até 10 tópicos, destacando pedido principal, fundamentos jurídicos e valor da causa." | Define o foco da busca, melhorando o resultado para o que se pretende. |
| "Essa tese está boa?" | "Revise as seguintes fundamentações e aponte contradições ou lacunas argumentais, sugerindo ajustes." | Prioriza elementos específicos na revisão para que o agente analise os elementos mais importantes.  |

Dicas gerais:
- Diga **o que você quer receber de volta** (parecer, lista de riscos, tópicos, comparação) — não só o tema.
- Informe **contexto mínimo**: partes, datas, área do direito, instância processual.
- Se anexar um documento, **diga o que fazer com ele** — o agente não adivinha o objetivo só por receber o arquivo.
- Para casos com múltiplas perguntas, prefira dividir em mensagens separadas (lembre-se do limite de 5 mensagens por conversa — seção 6).

---

### 

### Análise de Legislação — fluxo dedicado

O módulo **Análise de Legislação** usa uma dupla própria, sem passar pelo Orquestrador:

```mermaid
flowchart LR
    U2["Lei/normativo/minuta enviada"] --> AN["Analista"]
    AN -->|"precisa de pesquisa complementar"| PE2["Pesquisador"]
    PE2 -->|"devolve o material pesquisado"| AN
    AN --> R2["Análise final"]
```

O **Analista** é especializado em estudo de legislações, portarias e normativos (adequação, riscos, compliance, oportunidades de atuação) e pode acionar o Pesquisador quando precisar de apoio.

---

### Revisão Textual — agente único, sem repasses

O módulo **Revisão Textual** usa apenas o **Revisor**, especializado em revisão de textos jurídicos (ortografia, clareza, coesão, estrutura argumentativa). Não há conversa nem histórico: cada revisão é um envio único e independente.

---


## 4. Formato e formatação de arquivos para anexar

| Formato | Suportado? | Observações |
|---|---|---|
| `.pdf` | ✅ Sim | Cada página é enviada como imagem para o modelo "ler" diretamente — inclusive digitalizações/scans, sem necessidade de OCR à parte. Limite de leitura: 15 páginas por arquivo. |
| `.docx` | ✅ Sim | Texto extraído normalmente, mais imagens relevantes incorporadas no corpo do documento (ícones e logos pequenos são ignorados automaticamente). |
| `.txt` | ✅ Sim | Apenas texto simples. |
| `.doc` (Word 97-2003) | ❌ Não suportado | Salve como `.docx` antes de anexar ("Salvar como" → Word Document `.docx`). |

**Limite de tamanho:** 10 MB por arquivo.

Boas práticas ao anexar:
- Prefira **um documento por envio**, com propósito único (a petição, o acórdão, a minuta) em vez de compilados de vários arquivos colados em um só.
- Em PDFs digitalizados, garanta **boa legibilidade** (escaneamento reto, sem cortes, resolução legível) — como o modelo "enxerga" a página como imagem, texto borrado ou torto prejudica a leitura.
- Documentos muito longos (acima de 15 páginas em PDF) podem ter conteúdo além do limite ignorado — para peças extensas, considere enviar apenas as seções relevantes.
- Combine o anexo com uma instrução clara no texto da mensagem (ver seção 3).

---

## 5. Seleção de fontes (tools) no modo Análise Jurídica

Ao lado do campo de mensagem, dois controles permitem restringir onde o Pesquisador vai buscar informação:
![Ícones](source_icons.png)

- **Ícone de biblioteca ("Consultar fontes")** — abre uma lista com sete bases jurídicas internas: Jurisprudências, Roteiro de Ações, Constituição Federal, Código de Processo Civil, Lei 8.112, Lei 8.213, Lei 9.717, Lei 9.784 e Lei 12.772. Marque apenas as bases relevantes ao caso para focar a pesquisa e acelerar a resposta.
- **Ícone de globo ("Buscar na web")** — habilita pesquisa na internet (útil para notícias jurídicas recentes, mudanças normativas muito novas ou temas fora das bases internas).


Se nenhuma fonte for marcada, o Pesquisador decide sozinho quais bases consultar, conforme identificar necessidade durante a resposta. Marcar fontes específicas é recomendado quando você já sabe qual legislação rege o caso — isso reduz buscas desnecessárias e deixa a resposta mais objetiva.

---

## 6. Por que exportar a conversa é importante

O histórico da Análise Jurídica **não fica salvo em nenhum servidor** — ele existe apenas na aba do navegador, durante a sessão atual. Isso significa que:

- Fechar a aba, limpar o cache ou trocar de navegador **apaga a conversa permanentemente**.
- Cada conversa tem um **limite de 5 mensagens enviadas pelo usuário**; ao atingir o limite, o envio é bloqueado e é preciso iniciar uma nova conversa (o conteúdo da anterior não é recuperado automaticamente).

Por isso, use o botão **"Exportar .docx"** (disponível a partir da primeira resposta do agente, no topo da tela) sempre que quiser preservar um raciocínio, parecer ou pesquisa importante — o arquivo gerado inclui a conversa formatada, pronta para arquivar ou compartilhar. Os módulos de Revisão Textual e Análise de Legislação também oferecem exportação em `.docx` do resultado.

**Recomendação:** exporte antes de encerrar o expediente ou sempre que a conversa chegar a um ponto de conclusão relevante — não deixe para o final, especialmente perto do limite de 5 mensagens.

---

## Changelog 

**29/06/2026:**
- Adicionada ferramenta de OCR para leitura de imagens no corpo de arquivos .pdf e .docx.
- Separado módulo de "Análise de Legislação" para revisão única.