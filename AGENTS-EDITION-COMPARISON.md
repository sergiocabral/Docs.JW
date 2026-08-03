# Comparação entre edições de uma publicação para agentes de inteligência artificial

## Finalidade

Este arquivo é a instrução permanente para criar uma análise longitudinal de
duas ou mais edições da mesma publicação. O resultado deve explicar, em
português claro e com evidências verificáveis, o que permaneceu, o que mudou,
quando mudou e onde o assunto aparece em cada edição.

A comparação não é um `diff` técnico apresentado ao leitor. Ela é uma nova
publicação editorial, derivada das edições já preservadas no repositório, com:

- uma visão geral das edições e das mudanças mais importantes;
- um mapa de correspondência entre temas e locais de cada edição;
- um mapa de expressões com as redações exatas ao longo do tempo;
- uma linha do tempo editorial;
- uma página para cada tema principal, mesmo que o número ou a divisão dos
  capítulos tenha mudado;
- análise das transições entre edições consecutivas;
- referências diretas às páginas AsciiDoc usadas como evidência;
- geração independente em HTML, PDF e EPUB quando o repositório oferece esses
  formatos.

Leia este documento inteiro antes de iniciar uma comparação. Leia também
`AGENTS.md` e as demais instruções especializadas aplicáveis ao repositório.

## Regra de ativação

Não crie nem atualize a análise comparativa apenas porque há várias edições no
repositório. Execute este procedimento somente quando o usuário solicitar
explicitamente uma comparação, uma análise entre edições ou a atualização de
uma análise existente.

Uma execução completa exige:

1. pelo menos duas edições da mesma publicação;
2. identidade editorial confirmada, incluindo mnemônico, idioma e data;
3. módulos AsciiDoc suficientemente completos e revisados;
4. ordem cronológica confiável;
5. autorização separada para commits, tags, releases ou pushes.

Se uma edição estiver incompleta, declare a limitação. Não preencha lacunas com
outra edição nem trate ausência de transcrição como remoção editorial.

## Princípios obrigatórios

1. Cada edição é uma fonte histórica independente.
2. Compare primeiro edições consecutivas e só depois sintetize a evolução
   completa.
3. Não associe capítulos apenas pelo número, caminho ou posição no sumário.
4. O tema é a unidade lógica principal; capítulos são locais onde o tema
   aparece.
5. Uma correspondência pode ser de um para um, um para muitos, muitos para um
   ou muitos para muitos.
6. Não force correspondência quando as evidências forem insuficientes.
7. Separe observação textual de interpretação editorial.
8. Não atribua intenção, motivo, aprovação ou reprovação ao autor ou à
   organização sem uma declaração explícita da fonte.
9. Preserve a terminologia histórica de cada edição, especialmente em
   publicações das Testemunhas de Jeová.
10. Toda afirmação sobre mudança deve apontar para evidências nas duas edições
    envolvidas.
11. Mudanças de marcação AsciiDoc, OCR, links, âncoras ou paginação não são
    automaticamente mudanças da publicação.
12. O texto comparativo deve ser útil sem exigir que o leitor examine um diff,
    mas deve permitir que ele confira cada conclusão.
13. A comparação também funciona como uma revisão secundária da importação.
    Erros de português, grafia ou estrutura não podem ser ignorados nem
    presumidos como diferenças entre edições: valide-os na fonte e sinalize-os
    ao humano.
14. Registre toda alteração lexical confirmada, inclusive troca, acréscimo ou
    remoção de uma palavra, mudança de ordem e reformulação sintática, mesmo
    quando a relevância for apenas editorial ou menor.
15. Mostre a redação exata anterior e posterior. Não esconda mudanças reais em
    descrições genéricas como “ajustes de estilo”, “modernização da linguagem”
    ou “pequenas revisões”.
16. Diferenças exclusivamente tipográficas, de pontuação ou de apresentação
    podem ser agrupadas somente depois de confirmar que nenhuma palavra, ordem
    ou construção mudou.

## Produto editorial padrão

Use `edition-comparison` como nome técnico padrão do módulo. O título visível em
português deve ser **Análise entre edições**. Não crie um nível adicional de
idioma quando toda a análise estiver num único idioma.

### Páginas canônicas

As cinco primeiras páginas são canônicas e devem ser reproduzidas com estes
nomes, títulos e esta ordem em qualquer repositório, salvo instrução explícita
do usuário em contrário:

| Arquivo | Título visível | Função |
|---|---|---|
| `0000-index.adoc` | Análise entre edições | visão geral e principais resultados |
| `0010-metodologia.adoc` | Metodologia e limites | método explicado para o leitor e limites da análise |
| `0020-mapa-de-correspondencia.adoc` | Mapa de correspondência | localização dos temas em cada edição |
| `0025-mapa-de-expressoes.adoc` | Mapa de expressões | rastreamento exato de palavras e construções ao longo das edições |
| `0030-linha-do-tempo.adoc` | Linha do tempo editorial | síntese cronológica das transições |

Esses nomes formam o vocabulário editorial estável da análise. O conteúdo se
adapta às edições existentes, mas a função das páginas não muda. Se uma obra
exigir uma página própria para mudanças transversais, acrescente
`0040-mudancas-transversais.adoc` depois da linha do tempo; não substitua uma
das cinco páginas canônicas.

Estrutura padrão:

~~~text
modules/
  edition-comparison/
    nav.adoc
    pages/
      0000-index.adoc
      0010-metodologia.adoc
      0020-mapa-de-correspondencia.adoc
      0025-mapa-de-expressoes.adoc
      0030-linha-do-tempo.adoc
      0040-mudancas-transversais.adoc  # somente quando necessário
      0100-tema-estavel-um.adoc
      0110-tema-estavel-dois.adoc
      ...
      0800-materiais-preliminares.adoc
      0810-apendices-e-materiais-finais.adoc
      0900-indice-de-mudancas.adoc

asciidoctor/
  edition-comparison-contents.adoc

tmp/edition-comparison/
  inventory/
  normalized/
  candidates/
  alignments/
  review/
~~~

Os arquivos em `tmp/edition-comparison/` são evidências de trabalho e não devem
ser versionados. O conteúdo final pertence ao módulo AsciiDoc. Adapte a
quantidade de páginas ao livro; não crie páginas vazias apenas para reproduzir
o exemplo.

O módulo comparativo é uma publicação derivada, não uma nova edição do livro.
Não o registre como se fosse outra data editorial da publicação original.

### Linguagem voltada ao leitor

Todo texto dentro de `modules/edition-comparison/pages/` é conteúdo publicado e
deve ser escrito para a pessoa que lê pelo site, PDF ou EPUB. Não exponha nessa
camada detalhes de manutenção ou implementação, como:

- hashes de commit, branch, estado do Git ou alterações locais;
- caminhos de workspace, `build/`, `tmp/` ou diretórios internos;
- nomes de módulos, arquivos intermediários, scripts ou comandos;
- AsciiDoc, Antora, atributos, macros ou decisões do pipeline de build;
- instruções de commit, revisão técnica ou correção destinadas ao mantenedor.

Traduza a proveniência técnica para linguagem editorial. Por exemplo, escreva
“o trecho foi conferido no PDF da edição” em vez de mencionar a pasta onde o
PDF está armazenado; use “transcrição digital” em vez de “módulo importado”; e
identifique evidências por edição, capítulo, parágrafo ou página impressa, não
por caminho de arquivo. Informações técnicas continuam permitidas em
`tmp/edition-comparison/`, nos arquivos `AGENTS*.md`, no README de manutenção e
em relatórios que não façam parte da publicação.

## Integração com a navegação e os formatos

Primeiro identifique a topologia de navegação realmente usada pelo componente.
O módulo comparativo deve aparecer exatamente uma vez e como o último bloco de
nível superior, depois das publicações-fonte, dos adendos e das instruções
complementares:

- se `antora.yml` registrar um `nav.adoc` por módulo, acrescente
  `modules/edition-comparison/nav.adoc` por último na lista `nav`;
- se `antora.yml` registrar vários arquivos agregados
  `modules/ROOT/nav*.adoc`, crie ou atualize um arquivo exclusivo para a análise,
  como `modules/ROOT/nav-analise.adoc`, mantenha nele uma única raiz visível e
  registre-o por último; nesse caso, não registre também
  `modules/edition-comparison/nav.adoc`;
- se `antora.yml` registrar apenas `modules/ROOT/nav.adoc` e esse arquivo
  agregar toda a navegação, acrescente nele o bloco comparativo ao final e não
  registre outro `nav.adoc` em `antora.yml`;
- não duplique a mesma árvore entre uma navegação agregada e uma navegação de
  módulo registrada separadamente.

Como em todo repositório JW, as edições-fonte ficam da mais recente para a mais
antiga. Num componente que registra um arquivo por módulo, por exemplo:

~~~yaml
nav:
  - modules/edicao-recente/nav.adoc
  - modules/edicao-antiga/nav.adoc
  - modules/edition-comparison/nav.adoc
~~~

Essa ordem externa não altera a direção da análise. Dentro das páginas
comparativas, listas de evidências, transições, linhas do tempo e cadeias de
expressões seguem da edição mais antiga para a mais recente.

O modelo de navegação deve começar diretamente pelo título visível:

~~~adoc
* Análise entre edições
** xref:0000-index.adoc[Análise entre edições]
** xref:0010-metodologia.adoc[Metodologia e limites]
** xref:0020-mapa-de-correspondencia.adoc[Mapa de correspondência]
** xref:0025-mapa-de-expressoes.adoc[Mapa de expressões]
** xref:0030-linha-do-tempo.adoc[Linha do tempo editorial]
** xref:0040-mudancas-transversais.adoc[Mudanças transversais]
** Evolução por tema
*** xref:0100-tema-estavel-um.adoc[Tema estável um]
*** xref:0110-tema-estavel-dois.adoc[Tema estável dois]
** xref:0800-materiais-preliminares.adoc[Materiais preliminares]
** xref:0810-apendices-e-materiais-finais.adoc[Apêndices e materiais finais]
** xref:0900-indice-de-mudancas.adoc[Índice de mudanças]
~~~

Crie `asciidoctor/edition-comparison-contents.adoc` com a mesma ordem editorial
do `nav.adoc`, excluindo a página de índice apenas quando ela repetir uma capa
ou uma página de título gerada pelo conversor.

Quando o repositório já suporta várias publicações independentes, registre a
análise como outra publicação, com `contents`, título, slug, idioma e recursos
de apresentação próprios: uma entrada de metadados por publicação e um arquivo
de conteúdo para cada produto.

Quando `publication.yml` e `Rakefile` suportarem apenas uma publicação:

1. não insira `edition-comparison` como edição fictícia;
2. adapte o metadado para várias publicações de forma genérica;
3. faça o build percorrer publicações e edições sem nomes hardcoded;
4. preserve nomes, slugs e diretórios de saída já existentes;
5. mantenha a infraestrutura reutilizável por outros repositórios.

A saída independente deve ficar em:

~~~text
build/publications/edition-comparison/
~~~

e conter HTML, PDF e EPUB quando esses formatos fizerem parte do repositório.

Para a análise entre edições, não crie nem declare uma imagem de capa completa.
No PDF, a primeira página deve ser a folha de rosto gerada pelo tema, usando o
fundo compartilhado da publicação e título, período, autoria ou identificação
em texto digital. Não insira uma imagem antes dessa folha de rosto. Mantenha a
capa completa como recurso opcional na infraestrutura genérica, para que as
edições do livro continuem a usá-la sem obrigar o produto comparativo a fazer o
mesmo. Não crie `cover-complete`, banner ou outro asset exclusivo que só serviria
à análise. Valide visualmente a primeira página do PDF.

## Fontes de verdade e instantâneo da análise

Use como fontes primárias:

- `asciidoctor/publication.yml` para identificar as edições e seus idiomas;
- a página editora de cada módulo para confirmar a data exata;
- `modules/<edição>/pages/` para o texto efetivamente preservado;
- `modules/<edição>/nav.adoc` e o sumário para a estrutura declarada;
- `source-citation`, âncoras, imagens e referências para localizar a evidência;
- o PDF ou a fonte original somente para resolver uma dúvida de transcrição.

O nome do diretório, o número do capítulo e a data do arquivo no sistema não
substituem a página editora.

Antes de ordenar as fontes, separe as linhagens editoriais. Um manual mundial,
um adendo territorial e uma instrução independente podem tratar dos mesmos
temas sem serem edições consecutivas da mesma publicação. Nesses casos:

1. use a sucessão cronológica direta somente dentro de cada linhagem com
   identidade própria;
2. analise adendos, formulários e instruções como materiais relacionados, em
   páginas próprias ou nas páginas temáticas pertinentes;
3. não invente uma edição intermediária nem incorpore o material relacionado ao
   livro para preencher uma lacuna;
4. só afirme incorporação, antecipação ou continuidade quando o assunto e a
   redação fornecerem evidência verificável;
5. trate os anos históricos mencionados num rótulo como histórico editorial,
   não como exemplares completos disponíveis para comparação. Compare o texto
   integral apenas das edições efetivamente catalogadas.

Uma instrução independente pode funcionar como *ponte operacional* sem se
tornar uma edição do livro. Use essa classificação quando houver evidência
documental explícita, por exemplo, quando a própria instrução determinar
aplicação imediata e anunciar uma revisão futura da publicação principal.
Nesse caso:

1. preserve o mnemônico, a data e a identidade independente da instrução;
2. mostre a sequência `redação anterior do livro → ajuste imediato → redação
   posterior do livro` por assunto e por expressão;
3. diferencie o que a edição posterior *incorpora*, o que ela *amplia ou
   atualiza* e o que ela *não repete expressamente*;
4. não interprete a ausência de uma frase na edição posterior como revogação,
   a menos que outra evidência sustente essa conclusão;
5. explique qualquer aparente conflito entre datas usando o conteúdo e as
   declarações editoriais dos documentos, não apenas a ordem dos rótulos.

No início da execução, registre em `tmp/edition-comparison/inventory/`:

- commit Git ou estado exato usado como base;
- módulos incluídos e excluídos;
- data editorial e rótulo de cada edição;
- idioma;
- total de páginas AsciiDoc;
- capítulos, materiais preliminares, apêndices e índices;
- páginas ou trechos incompletos;
- método e data da análise.

Esse é um registro técnico de reprodutibilidade e não deve ser copiado
literalmente para a publicação. Na página canônica **Metodologia e limites**,
apresente apenas o que ajuda o leitor: edições comparadas, natureza das fontes,
unidades examinadas, critérios de correspondência, normalizações relevantes e
limitações. Não mostre commit, estado do Git, caminhos, nomes de arquivo ou
ferramentas internas. Se o Git estiver sujo, registre em `tmp` quais alterações
locais foram consideradas; não atribua automaticamente essas alterações a uma
edição publicada.

## Fase 1 — Inventário estrutural

Antes de comparar frases, monte um inventário de cada edição. Para cada página,
capture:

- caminho e módulo;
- título principal;
- número de capítulo declarado, se houver;
- título exibido no sumário e no `nav.adoc`;
- subtítulos em ordem;
- âncoras e números de parágrafo;
- listas, tabelas, quadros, citações e destaques;
- referências bíblicas e referências a outras publicações;
- imagens e legendas;
- citação de fonte e páginas impressas;
- quantidade aproximada de palavras e parágrafos.

Inclua materiais que não são capítulos:

- capa, página de rosto e página editora;
- carta, prefácio ou introdução;
- sumário;
- apêndices;
- perguntas, formulários ou roteiros;
- glossários;
- índices;
- notas e páginas finais.

Um índice não deve ser usado como substituto do corpo, mas pode revelar que um
tema foi introduzido, renomeado ou removido.

## Fase 2 — Duas representações do texto

Mantenha duas representações separadas para cada unidade.

### Representação exata

Preserve:

- redação;
- pontuação;
- maiúsculas;
- aspas;
- ênfase;
- números de parágrafo;
- referências;
- termos históricos.

Use essa representação para confirmar a diferença e citar “antes” e “depois”.

### Representação de comparação

Crie uma cópia apenas em `tmp`, sem alterar os módulos, na qual seja permitido:

- normalizar Unicode para NFC;
- normalizar finais de linha e espaços;
- retirar atributos, roles, âncoras e marcação AsciiDoc;
- substituir links pelo texto visível;
- separar números de parágrafo do corpo;
- remover `source-citation` do cálculo textual;
- distinguir cabeçalhos de conteúdo.

Não elimine automaticamente acentos, números, referências, negações ou termos
teológicos. Eles podem ser a própria mudança analisada.

Calcule, quando útil:

- hash exato;
- hash do texto de comparação;
- contagem de palavras e parágrafos;
- conjunto de subtítulos;
- conjunto de referências;
- frases adicionadas, removidas e reescritas.

Os hashes servem para detectar igualdade, não para explicar diferença.

## Fase 3 — Segmentar em unidades de evidência

Não compare o capítulo inteiro como um bloco único. Separe:

1. título;
2. subtítulos;
3. parágrafos;
4. itens de lista;
5. células ou linhas de tabela;
6. citações e destaques;
7. notas;
8. imagens e legendas.

Cada unidade deve ter um localizador auditável, nesta ordem de preferência:

1. `xref` para uma âncora de parágrafo ou seção;
2. `xref` para a página AsciiDoc e número do parágrafo;
3. `xref` para a página e intervalo impresso de `source-citation`;
4. caminho do módulo mais um pequeno trecho identificador.

Não use somente “capítulo 4, parágrafo 7” se a numeração mudar entre edições.

## Fase 4 — Criar perfis temáticos

Para cada capítulo ou seção relevante, escreva internamente uma descrição
neutra de uma ou duas frases respondendo:

- qual é o assunto principal;
- quais assuntos secundários são desenvolvidos;
- quais pessoas, funções, práticas ou doutrinas são centrais;
- quais subtítulos estruturam a exposição;
- quais referências são características;
- qual é a relação com o capítulo anterior e o seguinte.

Crie um identificador temático estável em ASCII, como:

~~~text
congregation-leadership
public-ministry
meeting-organization
~~~

O identificador não deve conter o número de um capítulo nem depender do título
da edição mais recente. O título português da página pode ser mais natural e
descritivo.

Se um capítulo tratar de dois temas que mais tarde foram separados, crie dois
perfis. Se dois capítulos forem reunidos numa edição posterior, permita que
ambos apontem para o mesmo perfil.

## Fase 5 — Gerar candidatos de correspondência

Use vários sinais. Nenhum deles decide sozinho:

| Sinal | Uso | Risco |
| --- | --- | --- |
| título | identifica renomeações pequenas | títulos podem ser genéricos ou reutilizados |
| subtítulos | revela a estrutura argumentativa | podem ser reescritos sem mudança de tema |
| termos e entidades centrais | aproxima assuntos equivalentes | vocabulário comum gera falsos positivos |
| referências bíblicas e bibliográficas | funciona como impressão digital | referências comuns têm pouco poder distintivo |
| frases e parágrafos semelhantes | confirma continuidade textual | reescritas profundas reduzem a similaridade |
| posição e capítulos vizinhos | ajuda a desempatar | capítulos podem ser movidos |
| caminho e número | fornece uma pista barata | não prova identidade temática |
| imagens, tabelas e listas | confirma conteúdo especializado | elementos podem ter sido removidos por layout |

Uma pontuação auxiliar pode usar, como ponto de partida:

~~~text
20% título
20% subtítulos
25% conteúdo semântico das unidades
15% referências e entidades distintivas
10% estrutura e capítulos vizinhos
5% imagens, tabelas e listas
5% caminho, número e posição
~~~

Registre a fórmula efetivamente usada. Esses pesos ordenam candidatos; não
autorizam uma correspondência automática.

Não use similaridade do corpo inteiro como decisão principal. Dentro de um
mesmo livro, capítulos diferentes compartilham vocabulário, nomes e textos
bíblicos suficientes para produzir correspondências lexicalmente altas e
semanticamente erradas.

Se houver embeddings ou outro modelo semântico disponível, use-os apenas para
propor candidatos. Não envie o conteúdo a serviço externo sem autorização e
não substitua a leitura das evidências pelo resultado do modelo.

## Fase 6 — Julgar a correspondência

Leia os melhores candidatos e classifique a relação:

- `same-topic`: mesmo tema e continuidade reconhecível;
- `renamed`: mesmo tema, título diferente;
- `moved`: mesmo tema, posição diferente;
- `split`: um local antigo corresponde a vários locais posteriores;
- `merged`: vários locais antigos correspondem a um local posterior;
- `reframed`: tema central preservado com estrutura ou ênfase substancialmente
  diferente;
- `introduced`: tema sem antecedente suficiente;
- `removed`: tema sem continuação suficiente;
- `partial`: apenas parte do conteúdo tem correspondência;
- `uncertain`: evidência insuficiente ou conflitante.

Use confiança qualitativa:

- **alta**: título ou estrutura compatível e várias unidades textuais ou
  referências distintivas confirmam a relação;
- **média**: o tema é compatível, mas houve reescrita, divisão ou fusão;
- **baixa**: a relação depende de poucos indícios e precisa de revisão.

Não esconda itens de baixa confiança. Coloque-os numa fila de revisão e numa
seção de limitações.

## Mapa de correspondência

`0020-mapa-de-correspondencia.adoc` deve ser a fonte de verdade humana para o
alinhamento temático. Quando houver deslocamentos, divisões, fusões ou confiança
variável, use uma linha por ocorrência de tema em cada edição:

~~~adoc
[cols="2,2,2,3,2,1",options="header"]
|===
| ID do tema
| Edição
| Local
| Título naquela edição
| Relação
| Confiança

| congregation-leadership
| edição-a
| xref:edicao-a:pagina.adoc[Capítulo 4]
| Título histórico
| same-topic
| alta

| congregation-leadership
| edição-b
| xref:edicao-b:outra-pagina.adoc[Capítulos 3 e 4]
| Títulos posteriores
| split
| média
|===
~~~

Para fusões e divisões, use tantas linhas quantas forem necessárias. Não tente
comprimir uma relação muitos para muitos numa célula ambígua.

Quando todos os temas tiverem correspondência de um para um, permanecerem no
mesmo local e tiverem a mesma confiança, prefira um mapa compacto voltado ao
leitor: uma linha por tema, colunas cronológicas agrupando edições com o mesmo
título e um `xref` do tema para a página detalhada. Declare uma única vez a
confiança comum. Não repita 119 linhas se uma tabela de 17 linhas preservar a
mesma informação sem ambiguidade.

## Fase 7 — Alinhar texto dentro do tema

Depois de confirmar o tema, compare as unidades em ordem. Comece por edições
consecutivas:

~~~text
E1 -> E2
E2 -> E3
E3 -> E4
~~~

Só depois produza a síntese `E1 -> edição mais recente`.

Use:

- igualdade exata para trechos preservados;
- LCS, Myers ou outro diff para pequenas edições;
- alinhamento por frase para parágrafos renumerados;
- semelhança semântica para localizar reescritas;
- leitura manual para confirmar substituições, divisões e fusões.

Classifique cada mudança observada:

- correção tipográfica ou ortográfica;
- atualização terminológica;
- atualização de tradução bíblica ou de citação;
- alteração de referência a outra publicação;
- adição;
- remoção;
- expansão;
- condensação;
- reescrita;
- mudança de procedimento ou instrução;
- mudança de explicação doutrinal;
- mudança de estrutura, ordem ou título;
- alteração de imagem, legenda, tabela ou lista;
- alteração de metadado editorial.

Use “mudança de explicação doutrinal” somente quando o conteúdo analisado
tratar de uma doutrina e o antes/depois sustentar essa descrição. Não transforme
qualquer reescrita religiosa em mudança doutrinal.

## Rastreamento lexical e mapa de expressões

O alinhamento temático não substitui o rastreamento das palavras. Depois de
alinhar cada unidade, examine também as alterações de vocabulário e construção
entre todas as edições consecutivas. Registre:

- substituição de uma palavra ou locução;
- palavra acrescentada ou removida;
- mudança de singular, plural, artigo, pronome, preposição ou verbo;
- inversão da ordem dos termos;
- reformulação da estrutura da frase, ainda que a ideia geral permaneça;
- mudança de título, subtítulo, designação institucional ou jargão histórico;
- forma que desapareceu e depois voltou a ser usada.

Crie sempre `0025-mapa-de-expressoes.adoc`. Essa página deve acompanhar
expressões claramente contínuas ao longo do tempo e complementar o mapa de
capítulos. Agrupe por assunto, não por ordem alfabética isolada. Para cada forma,
informe a edição ou intervalo de edições, reproduza o menor trecho exato que
demonstre a diferença e ofereça `xref` para o contexto.

Para uma transição com duas formas, prefira:

~~~adoc
* *Antes — edição A:* “redação exata anterior” — xref:edicao-a:pagina.adoc[contexto]
* *Depois — edição B:* “redação exata posterior” — xref:edicao-b:pagina.adoc[contexto]
~~~

Quando houver três ou mais etapas, liste todas em ordem cronológica; não reduza
a cadeia à primeira e à última. Se uma redação se mantiver por várias edições,
indique o intervalo com precisão, por exemplo “2015-09 e 2017-09” ou “desde
2020-04”. Uma tabela de duas colunas — assunto e formas ao longo do tempo — pode
organizar várias cadeias sem perder a leitura dos bullets.

Use citações curtas para respeitar a função analítica do relatório. A exigência
de exatidão não autoriza reproduzir páginas ou parágrafos inteiros. Não trate
uma diferença lexical confirmada como irrelevante apenas porque a ideia geral
parece igual; registre-a e classifique separadamente sua relevância. Agrupe
somente alterações puramente tipográficas ou técnicas, depois de conferi-las na
fonte.

A obrigação de mostrar cada palavra alterada vale para expressões reconhecíveis
e para toda mudança curta mencionada na análise. Quando um parágrafo tiver sido
integralmente reescrito, não transforme o relatório num diff de cada token:
resuma a reescrita e escolha trechos exatos representativos. Já uma frase que
permanece quase igual, mas troca poucas palavras, deve aparecer integralmente em
antes/depois; ela não pode ser absorvida pelo resumo da reescrita maior.

## Mudanças transversais

Algumas alterações aparecem em quase todo o livro e não devem ser repetidas
como descoberta independente em cada página temática. Identifique padrões como:

- atualização geral da tradução bíblica;
- reforma ortográfica;
- substituição sistemática de um termo institucional;
- mudança uniforme de abreviações, aspas ou capitalização;
- alteração global de estilo editorial;
- renumeração causada por inserção anterior;
- modernização de links ou referências bibliográficas.

Resuma a época em que esses padrões surgem em `0030-linha-do-tempo.adoc`. Se a
quantidade ou a complexidade justificar uma página própria, detalhe-os em
`0040-mudancas-transversais.adoc`. Nas páginas temáticas, mencione-os apenas
quando afetarem o sentido local.

Uma mudança transversal não autoriza normalizar silenciosamente as citações
de cada edição.

## Distinguir edição real de artefato técnico

Antes de relatar uma diferença, descarte ou classifique corretamente:

- roles, atributos e âncoras AsciiDoc;
- URLs adicionadas sem mudança no texto visível;
- `source-citation` diferente;
- nomes de arquivos e slugs;
- quebras de linha e hifenização tipográfica;
- conversão de aspas ou travessões feita pela transcrição;
- diferença de ênfase perdida pelo OCR;
- cabeçalho ou rodapé incorporado por engano;
- número de página;
- erro de OCR;
- correção feita no módulo, mas não confirmada na fonte histórica.

Quando uma diferença curta parecer erro tipográfico, OCR ou caractere
decomposto:

1. consulte a página renderizada da edição;
2. siga `AGENTS-JW-PDF-EXTRACTION.md` quando esse arquivo existir e o PDF for a
   fonte;
3. confirme as duas edições;
4. classifique como “correção tipográfica confirmada”, “variante impressa” ou
   “incerteza de transcrição”.

Não chame um erro de OCR de revisão editorial.

### Tratar anomalias como possíveis falhas da conversão

Durante a comparação, considere que a extração do PDF, a transcrição, a revisão
ou a estruturação em AsciiDoc da etapa anterior pode ter falhado. Se uma
diferença não fizer sentido linguístico, semântico ou estrutural, não a
classifique imediatamente como mudança da publicação.

Trate como sinal de possível falha anterior, entre outros casos:

- palavra inexistente, grafia incompatível com o português da época ou
  terminologia estranha ao contexto das Testemunhas de Jeová;
- frase truncada, sem verbo, com palavras repetidas ou com conexão lógica
  quebrada;
- parágrafo ausente, duplicado, fora de ordem ou com numeração descontínua;
- título transformado em parágrafo, subtítulo omitido ou hierarquia incoerente;
- legenda, nota, quadro, imagem ou referência incorporada no lugar errado;
- caracteres trocados, separação indevida de palavras ou resíduo de
  cabeçalho, rodapé e coluna;
- divergência inexplicável entre sumário, navegação e conteúdo;
- mudança isolada que contradiz o restante da própria edição.

Ao encontrar uma anomalia:

1. suspenda a classificação daquele trecho como diferença editorial;
2. confira o PDF, a página renderizada, o WOL ou outra fonte primária da edição;
3. determine se o problema já existe na publicação, foi introduzido na
   conversão ou continua inconclusivo;
4. registre em `tmp/edition-comparison/review/conversion-anomalies.adoc` a
   edição, página, capítulo, parágrafo, trecho suspeito, fonte conferida,
   diagnóstico e impacto na comparação;
5. sinalize o item ao humano de forma explícita, principalmente se ele afetar
   uma correspondência temática ou uma conclusão importante;
6. não corrija silenciosamente o módulo-fonte nem use a provável falha como
   evidência de evolução editorial.

Se a grafia ou a construção estiver presente na fonte histórica, preserve-a na
citação e classifique-a como variante da própria publicação. Se o erro existir
apenas no material importado, exclua-o do relatório de mudanças editoriais e
aguarde autorização ou escopo adequado para corrigir a transcrição. Quando não
for possível resolver a dúvida, marque-a como **pendência de conversão** e não
derive dela conclusões mais amplas.

## Evidência, citações e linguagem

Use `xref` diretamente para as edições:

~~~adoc
xref:edicao-a:pagina.adoc[edição A, capítulo e parágrafo]
~~~

Ao mostrar o antes e o depois, cite apenas o trecho necessário para demonstrar
a mudança. Prefira uma frase completa ou uma pequena unidade lógica. Use
reticências somente quando a omissão não alterar o sentido.

Para trocas curtas de palavras ou de construção, use bullets como formato
preferencial:

~~~adoc
* *Antes — edição A:* “trecho exato anterior” — xref:edicao-a:pagina.adoc#ancora[Conferir no contexto]
* *Depois — edição B:* “trecho exato posterior” — xref:edicao-b:pagina.adoc#ancora[Conferir no contexto]
~~~

Use a tabela abaixo quando o contexto maior ou a comparação lado a lado tornar
a diferença materialmente mais clara.

Modelo:

~~~adoc
[cols="1,1",options="header"]
|===
| Antes — edição A
| Depois — edição B

a|
> Trecho exato e suficiente da edição A.

xref:edicao-a:pagina.adoc#ancora[Conferir no contexto]

a|
> Trecho exato e suficiente da edição B.

xref:edicao-b:pagina.adoc#ancora[Conferir no contexto]
|===

**Mudança observada:** descrição neutra do que foi acrescentado, removido ou
reescrito.

**Efeito no tema:** explicação limitada ao que as evidências permitem afirmar.
~~~

Não reproduza páginas inteiras apenas para exibir pequenas diferenças. O
relatório deve direcionar o leitor ao texto integral nos módulos originais.

Use formulações como:

- “A edição de 2015 acrescenta...”;
- “O parágrafo passa a mencionar...”;
- “O tema foi transferido para...”;
- “Não foi localizada uma continuação equivalente...”;
- “A evidência permite afirmar...”;
- “Não é possível determinar apenas pelo texto...”

Evite:

- “eles mudaram porque...” sem fonte explícita;
- “a doutrina foi escondida”;
- “isso prova que...” quando a evidência mostra apenas correlação;
- adjetivos promocionais ou acusatórios;
- terminologia atual aplicada retroativamente a uma edição antiga.

## Escala de relevância

Classifique a relevância para ajudar o leitor, sem confundi-la com confiança:

- **editorial**: grafia, pontuação, metadado, título ou referência sem mudança
  relevante de conteúdo;
- **menor**: esclarecimento local ou pequena atualização prática;
- **moderada**: expansão, redução ou reformulação que altera a compreensão do
  tema;
- **maior**: introdução, remoção, divisão, fusão ou mudança substancial de
  orientação ou explicação.

Confiança responde “quão segura é a correspondência?”. Relevância responde
“quanto a mudança afeta o conteúdo?”. Não misture as duas escalas.

## Página inicial da análise

`0000-index.adoc` deve conter:

1. título e objetivo;
2. edições incluídas, em ordem cronológica;
3. edição mais recente incluída no escopo;
4. aviso de que a análise compara o texto preservado, sem inferir motivos;
5. resumo curto das mudanças mais importantes;
6. orientação de leitura com links para metodologia, mapas, linha do tempo,
   páginas temáticas e pendências;
7. identificação clara de que se trata de análise editorial independente.

Não transforme a visão geral numa lista de todas as frases alteradas. Ela deve
orientar o leitor para as páginas canônicas e temáticas, sem duplicar nelas a
linha do tempo, o mapa ou o índice de mudanças.

## Modelo de página temática

Use esta estrutura enxuta como padrão:

~~~adoc
= Nome neutro e estável do tema

== Edições alinhadas

* xref:edicao-a:pagina.adoc[Edição A — título histórico]
* xref:edicao-b:pagina.adoc[Edição B — título posterior]

Relação: *mesmo tema, renomeado e revisto*. Descrição neutra do escopo.

== Evolução

=== Edição A para edição B

Mudança observada, redação exata quando necessária, relevância e evidências.

=== Edição B para edição C

Mudança observada ou declaração explícita de estabilidade.

== Síntese

O que permaneceu e a direção geral das mudanças.
~~~

Use atributos como `page-description` e `topic-id`, tabelas de correspondência,
ou seções próprias de estrutura e limites somente quando acrescentarem valor.
Se várias edições consecutivas tiverem texto igual, elas podem ser reunidas
numa única transição, desde que todas sejam nomeadas e a estabilidade seja
declarada. Não invente uma seção de antes/depois quando não houver mudança.

## Materiais preliminares, apêndices e índices

Compare materiais preliminares e finais separadamente dos temas dos capítulos.

Para página editora e direitos autorais, registre apenas diferenças relevantes:

- data e identificação da edição;
- editora;
- ISBN;
- tradução bíblica declarada;
- instruções de distribuição ou donativos;
- títulos de publicações citadas.

Para cartas e prefácios, use o mesmo alinhamento temático dos capítulos.

Para apêndices, perguntas e formulários, preserve a numeração de cada edição,
mas mapeie o assunto da pergunta. Uma pergunta renumerada não é necessariamente
nova.

Para índices:

- não liste toda mudança de paginação;
- compare termos, remissões e categorias;
- use o índice como evidência auxiliar de introdução ou remoção de temas;
- confirme no corpo antes de concluir que o conteúdo mudou.

## Índice de mudanças

O arquivo final com slug `indice-de-mudancas` deve permitir filtrar mentalmente
a análise. Sua numeração acompanha a estrutura do repositório; por exemplo,
`0900-indice-de-mudancas.adoc` ou `0330-indice-de-mudancas.adoc`. Use uma linha
por mudança temática, estrutural ou procedimental selecionada para a síntese:

~~~adoc
[cols="2,2,2,1,3",options="header"]
|===
| Transição
| Tema
| Categoria
| Relevância
| Resumo
|===
~~~

Ordene primeiro pela transição cronológica e depois pelo tema. O índice é uma
síntese orientadora, não um inventário lexical exaustivo: as trocas menores e
suas formas exatas pertencem ao **Mapa de expressões** e às páginas temáticas.
Não use a tabela como substituto dessas páginas detalhadas.

## Atualização quando surgir nova edição

Ao adicionar uma nova edição:

1. confirme que o módulo novo está completo;
2. atualize o inventário e o escopo;
3. compare a edição imediatamente anterior com a nova;
4. gere candidatos para os temas existentes;
5. crie novos temas somente quando necessário;
6. atualize relações `split`, `merged`, `introduced` e `removed`;
7. atualize a visão geral, a linha do tempo e o índice de mudanças;
8. preserve as análises anteriores, corrigindo-as apenas quando nova evidência
   justificar;
9. registre um novo instantâneo técnico e atualize a data de revisão da análise,
   sem levar conteúdo editorial ao commit-base estrutural do repositório;
10. regenere e valide todos os formatos.

Não recompute silenciosamente todo o histórico e substitua conclusões revisadas
por saída automática.

## Automação permitida e revisão humana obrigatória

Automatize:

- inventário de arquivos e títulos;
- extração de subtítulos e âncoras;
- hashes e contagens;
- diffs exatos;
- candidatos de correspondência;
- listas de referências;
- detecção de páginas idênticas;
- validação de `xref` e assets.

Revise semanticamente:

- toda correspondência de tema;
- toda divisão ou fusão;
- toda mudança classificada como moderada ou maior;
- toda afirmação doutrinal ou procedimental;
- toda citação antes/depois;
- toda possível correção de OCR;
- todo item de confiança média ou baixa;
- o resumo executivo.

Uma ferramenta pode dizer que dois textos são diferentes. Apenas a análise
revisada pode explicar de maneira confiável em que eles diferem.

## Controle de qualidade

### Verificações estruturais

Confirme:

- todas as edições do escopo aparecem na visão geral;
- linhagens principais, territoriais e complementares permanecem identificadas
  separadamente;
- as edições-fonte aparecem da mais recente para a mais antiga na navegação
  externa;
- transições e evidências aparecem da mais antiga para a mais recente dentro da
  análise;
- as cinco páginas canônicas existem, usam os títulos definidos e aparecem na
  ordem padrão;
- o módulo comparativo é o último bloco da navegação;
- não há nível de idioma desnecessário;
- todas as páginas do `nav.adoc` existem;
- todas as páginas temáticas estão no arquivo de conteúdo independente;
- todos os `xref` apontam para módulo, página e âncora existentes;
- nenhuma edição foi registrada como produto comparativo;
- HTML, PDF e EPUB da análise são independentes dos livros.

### Verificações analíticas

Confirme:

- todo capítulo ou seção substancial foi mapeado ou marcado como sem
  correspondência;
- números de capítulo não foram usados como prova de identidade;
- cada tema possui definição estável;
- cada mudança tem evidência nas edições envolvidas;
- transições cronológicas diretas ocorrem somente dentro da mesma linhagem;
- materiais relacionados não foram transformados em edições artificiais;
- transições consecutivas foram analisadas;
- toda troca confirmada de palavra, ordem ou construção foi registrada com a
  forma anterior e a posterior;
- cadeias de expressão com três ou mais estágios mostram todas as formas em
  ordem cronológica;
- o mapa de expressões contém vínculos para o contexto em cada edição;
- mudanças transversais não foram contadas repetidamente;
- ausência de transcrição não foi chamada de remoção;
- diferenças técnicas não foram chamadas de revisão editorial;
- erros linguísticos ou estruturais suspeitos foram conferidos na fonte e
  comunicados ao humano;
- pendências de conversão não foram usadas como evidência editorial;
- incertezas estão visíveis;
- observação e interpretação estão separadas;
- trechos citados conferem exatamente com os módulos;
- o conteúdo publicado fala com o leitor e não expõe commits, caminhos,
  módulos, arquivos intermediários ou detalhes de build;

### Verificações de build

Execute os comandos definidos pelo repositório, normalmente:

~~~shell
npm exec -- antora antora-playbook.yml
bundle exec rake
~~~

Inspecione visualmente:

- página inicial;
- mapa de correspondência;
- mapa de expressões;
- pelo menos uma página com renomeação;
- uma página com divisão ou fusão, se houver;
- tabelas antes/depois;
- colunas estreitas sem cabeçalhos ou termos-chave partidos no meio da palavra;
- sumário do PDF;
- navegação do EPUB;
- links entre a análise e as edições.

## Critério de aceite

A análise está pronta quando:

- o leitor entende quais edições foram comparadas;
- os temas podem ser acompanhados mesmo com renumeração, divisão ou fusão;
- as principais mudanças estão resumidas e detalhadas;
- as pequenas transições também foram verificadas;
- mudanças lexicais menores aparecem de modo exato, e não apenas resumidas;
- todo antes/depois pode ser conferido no texto original;
- não há conclusões baseadas apenas em similaridade automática;
- limitações e incertezas estão declaradas;
- a análise aparece por último na navegação;
- o produto comparativo é gerado separadamente;
- Antora, HTML, PDF e EPUB passam na validação;
- o estado do Git contém somente mudanças intencionais.

## Checklist curto para o agente

1. Leia `AGENTS.md` e este arquivo inteiro.
2. Confirme a solicitação explícita do usuário.
3. Identifique as linhagens e ordene suas edições pela página editora.
4. Registre o instantâneo e o escopo.
5. Inventarie páginas, capítulos, subtítulos, parágrafos e referências.
6. Crie representações exata e normalizada sem alterar as fontes.
7. Defina perfis e IDs temáticos estáveis.
8. Gere candidatos usando vários sinais.
9. Revise e classifique correspondências, inclusive divisões e fusões.
10. Compare primeiro cada par de edições consecutivas.
11. Separe mudanças transversais de mudanças temáticas.
12. Verifique na fonte erros de português, grafia, estrutura, OCR ou variantes
    impressas e sinalize anomalias ao humano.
13. Não converta uma pendência de conversão em diferença editorial.
14. Escreva as cinco páginas canônicas, as páginas temáticas e o índice em
    linguagem voltada ao leitor.
15. Registre no mapa de expressões todas as formas confirmadas, com bullets de
    antes/depois ou uma cadeia cronológica equivalente.
16. Ordene as edições-fonte da mais recente para a mais antiga e adicione o
    módulo comparativo por último na navegação.
17. Configure o build independente sem edição fictícia e sem capa completa para
    a análise.
18. Valide evidências, `xref`, HTML, PDF e EPUB.
19. Não faça commit, tag, release ou push sem autorização.
