# Importação de uma estrutura JWPUB para AsciiDoc

## Finalidade

Este documento orienta a transformação da estrutura produzida pelo **JWPUB
Converter** em uma publicação deste modelo Antora/AsciiDoctor. O objetivo não é
copiar HTML para dentro de AsciiDoc nem achatar tudo em texto, mas reconstruir a
semântica editorial com fidelidade:

- ordem e divisão dos documentos;
- títulos, subtítulos, parágrafos e perguntas;
- listas, quadros, tabelas e notas;
- imagens, textos alternativos, legendas e créditos;
- vínculos entre documentos e referências externas;
- metadados, navegação, capas e formatos de saída.

Leia antes:

1. `AGENTS.md` deste repositório;
2. `AGENTS-JWPUB-EXTRACTION.md` por completo;
3. o `README.md` e os relatórios da estrutura extraída;
4. este arquivo por completo.

Execute a importação somente quando o usuário pedir para trazer a publicação.
A existência de um JWPUB ou de uma pasta extraída não autoriza alterações no
conteúdo versionado.

## Pré-condição bloqueante

Não comece a gerar páginas se qualquer condição abaixo for falsa:

- o conversor retornou código zero;
- `validation-report.json` contém `status = "passed"`;
- manifesto e tamanho expandido conferem;
- `PRAGMA quick_check` retornou `ok`;
- total de documentos = descriptografados = sequências semânticas exatas;
- entrada permaneceu inalterada;
- referências de mídia ausentes e arquivos inválidos estão vazios;
- não há links locais quebrados;
- todos os avisos foram revisados;
- `html-original/`, `documents.json`, `media/` e `checksums.sha256` existem.

Se a saída falhar nesse contrato, volte ao repositório do conversor. Não tente
compensar uma extração incompleta durante a escrita do AsciiDoc.

## Fontes de verdade em ordem de prioridade

Use as representações de modo complementar:

1. **`html-original/`** — autoridade para estrutura, texto, ênfase, notas,
   links, imagens, legendas, perguntas e ordem interna;
2. **`documents.json`** — autoridade para a lista ordenada de documentos,
   identificadores, títulos preferidos, hashes e caminhos;
3. **`container/manifest.pretty.json`** — identidade da publicação, edição,
   símbolos e imagens de capa/navegação;
4. **`media/` + `inner-inventory.json` + `media-audit.json`** — bytes, nomes,
   hashes, dimensões, referências e integridade das mídias;
5. **SQLite em `database/`** — metadados relacionais ou de navegação que não
   estejam expostos de modo suficiente nos arquivos acima;
6. **`documents/*.txt` e TXT integral** — leitura, busca e conferência textual,
   nunca fonte exclusiva de estrutura;
7. **`html-browser/`** — conveniência para inspeção visual, não evidência exata.

O TXT derivado pode incluir “Sua resposta” ou “Suas respostas”, colar a letra de
uma nota à palavra anterior, omitir informação gerada por CSS e transformar
caixas, títulos e figuras em parágrafos comuns. Não converta o TXT linha por
linha para AsciiDoc.

O SQLite deve ser usado como evidência relacional: ele permite confrontar a
quantidade e a associação de documentos, parágrafos, perguntas, notas,
citações, links e mídias. Comece por `database/schema.sql` e
`database/tables.txt`. Não importe índices de busca como conteúdo e não tente
descriptografar colunas `Content` secundárias; o contrato atual do conversor
cobre `Document.Content`. Se um fragmento necessário existir apenas em outro
BLOB, trate isso como ampliação do conversor, não como correção improvisada da
importação.

## Princípios obrigatórios

1. **Preserve antes de interpretar.** Mantenha a estrutura extraída intacta em
   uma área ignorada pelo Git durante toda a importação.
2. **Use semântica, não aparência.** Converta cabeçalho em cabeçalho, lista em
   lista, figura em figura e nota em nota. Não reproduza classes de CSS do JW
   Library sem necessidade.
3. **Não invente conteúdo.** Texto alternativo, legenda, crédito, página,
   título ou URL ausente devem ser investigados, não preenchidos por palpite.
4. **Não modernize o jargão.** Preserve a terminologia da edição, conforme a
   seção de terminologia de `AGENTS.md`.
5. **Não confunda interface com publicação.** Campos de resposta e controles do
   aplicativo não são conteúdo editorial impresso.
6. **Não confunda identificadores.** `DocumentId`, `MepsDocumentId`, símbolo
   criptográfico, mnemônico editorial e código de idioma têm funções distintas.
7. **Não deixe esquemas privados como links públicos.** Nenhum `jwpub://` deve
   permanecer num site, PDF ou EPUB final.
8. **Mantenha rastreabilidade.** Toda omissão, fusão, correção ou link não
   resolvido precisa aparecer numa matriz temporária de revisão.
9. **Sinalize anomalias.** Grafia, pontuação ou estrutura estranha pode ser erro
   da fonte, do conversor ou da importação; não corrija nem aceite em silêncio.
10. **Valide os três destinos.** Antora, HTML/PDF/EPUB e o site agregador podem
    resolver recursos de maneiras diferentes.

## Fase 1 — Inventário da publicação

### Identidade técnica da execução

Leia `run-metadata.json` e registre temporariamente:

- versão do conversor;
- SHA-256 da entrada;
- título e título curto;
- símbolo, idioma numérico e ano;
- `issueTagNumber`, `issueId`, `issueNumber` e fonte da identidade;
- quantidade de documentos, palavras e mídias;
- avisos.

Esses dados comprovam a importação, mas não devem aparecer automaticamente nas
páginas do leitor.

### Matriz de documentos

Crie sob `build/` uma tabela de trabalho com pelo menos:

| Campo | Uso |
| --- | --- |
| `ordinal` | posição entregue pelo conversor; |
| `documentId` | ordem primária dentro do SQLite; |
| `mepsDocumentId` | identidade editorial e alvo de vínculos; |
| `class` | indício de tipo, nunca regra única; |
| `chapterNumber` | número declarado, quando houver; |
| `title`, `tocTitle`, `preferredTitle` | candidatos a título e navegação; |
| `originalHtml` | fonte estrutural; |
| página AsciiDoc planejada | destino futuro; |
| papel editorial | capa, sumário, artigo, capítulo, apêndice etc.; |
| decisão | importar, fundir, dividir ou omitir, com justificativa. |

Comece pela ordem crescente de `DocumentId`, pois é a ordem usada pelo
conversor no texto integral. Depois confronte essa ordem com:

- documento de sumário e seus links;
- `PublicationView*` no SQLite, quando necessário;
- números de página presentes no HTML;
- `ChapterNumber`, títulos e datas;
- a edição oficial exibida no JW Library ou em jw.org, quando disponível.

Não mapeie um tipo apenas pelo valor numérico de `class`. Classes variam entre
publicações e esquemas. Na amostra `w_T_202605`, valores diferentes distinguem
capa, sumário, artigos e uma matéria curta, mas isso não constitui uma tabela
universal.

### Inventário estrutural por documento

Para cada HTML original, conte e catalogue:

- `h1` a `h6`;
- parágrafos e números editoriais;
- perguntas e relações `data-rel-pid`;
- listas, tabelas, descrições e citações;
- `aside` e outros quadros;
- figuras, `src`, `alt`, legenda e crédito;
- notas e seus `data-fnid`;
- links por esquema e destino;
- `data-pid` que recebem links;
- elementos ocultos, campos de interface e parágrafos vazios.

Esse inventário se torna a lista de controle de completude. Uma página só está
pronta quando cada unidade tem destino ou uma justificativa explícita.

Confronte as contagens do HTML com as relações correspondentes do SQLite, quando
existirem. Uma diferença não prova automaticamente perda de conteúdo: uma
pergunta ou nota pode estar incorporada ao HTML e também catalogada numa tabela,
enquanto um índice de busca é apenas uma representação derivada. Investigue a
relação e registre a decisão, em vez de duplicar unidades para fazer os números
coincidirem.

### Inventário de mídias

Separe quatro grupos:

1. mídias referenciadas por `jwpub-media://` no HTML;
2. mídias relacionadas pela tabela `Multimedia`;
3. imagens declaradas em `publication.images` no manifesto;
4. arquivos preservados, mas sem uso editorial identificado.

Registre nome original, SHA-256, dimensões, documento de origem, texto
alternativo, legenda e função planejada. Não copie todas as variantes de
miniatura por padrão.

## Fase 2 — Definir a identidade no repositório

### Mnemônico, idioma e data

O nome do arquivo pode ajudar, mas não é autoridade isolada. Confronte:

- nome do JWPUB;
- `publication.symbol`, `rootSymbol`, `undatedSymbol` e propriedades da edição;
- código de idioma presente no nome e nos vínculos;
- título e data editoriais;
- catálogo oficial em jw.org ou WOL.

Não cometa estes erros:

- transformar o índice numérico `language` diretamente em código como `T` sem
  evidência;
- usar `issueNumber` como mês ou identificador criptográfico;
- assumir que `issueId` é sempre uma data sem verificar;
- usar um símbolo anual como `w26` quando o mnemônico editorial desejado é `w`;
- usar apenas `year` quando a edição tem mês ou dia conhecido.

Na amostra `w_T_202605`, por exemplo, o manifesto apresenta símbolo
criptográfico `w26`, `issueId = 20260500` e a propriedade editorial “A
Sentinela, maio de 2026”. Um módulo coerente seria planejado como
`w_T-2026-05`, depois da confirmação editorial; não como `w26_5` nem apenas
`w_T-2026`.

### Título da edição

Siga a regra geral de `AGENTS.md`: o título em
`asciidoctor/publication.yml` termina com o identificador formado por
mnemônico, idioma e data exata:

```yaml
w_T-2026-05:
  title: A Sentinela Anunciando o Reino de Jeová — Edição de Estudo (w-T 2026-05)
  mnemonic: w
  language_code: T
  edition_date: 2026-05
  lang: pt-BR
```

O exemplo demonstra a forma, não fixa o título de outra publicação. Preserve o
título oficial da fonte e valide pontuação, variante e capitalização.

### Uma edição por módulo

Cada edição ou número catalogado recebe módulo próprio. Insira variações da
mais recente para a mais antiga nas listas externas, conforme `AGENTS.md`.

O módulo deve ser estável mesmo que o título de um artigo mude. Use o padrão de
nomes já adotado pelo repositório e slugs ASCII.

## Fase 3 — Projetar páginas e ordem de leitura

### Divisão inicial

Em geral, um registro `Document` com conteúdo editorial corresponde a uma
página AsciiDoc. Essa é uma hipótese inicial, não uma obrigação. É aceitável:

- manter capa/editorial e sumário em páginas próprias;
- fundir documentos técnicos que formem uma única unidade visível;
- dividir um documento excepcionalmente grande quando a navegação oficial já
  o divide em partes claras;
- omitir somente controles ou documentos sem conteúdo editorial, registrando a
  justificativa.

Nunca una artigos distintos apenas para reduzir a quantidade de arquivos.

### Nomes de página

Use a convenção geral:

```text
NNNN-slug-descritivo.adoc
```

- quatro dígitos;
- intervalos de dez;
- slug ASCII minúsculo;
- mesma ordem da publicação;
- título AsciiDoc sem número artificial;
- número, data ou tipo editorial no `nav.adoc` quando necessário.

Mantenha na matriz a relação:

```text
DocumentId -> MepsDocumentId -> arquivo AsciiDoc -> título de navegação
```

### Ordem

Atualize em conjunto:

- `modules/<módulo>/nav.adoc`;
- o arquivo `asciidoctor/*contents.adoc` usado pelo produto;
- `antora.yml`;
- `asciidoctor/publication.yml`;
- página inicial e README quando a edição for nova.

Materiais sem numeração automática, como capa, expediente e sumário, devem
ficar em trechos com `:sectnums!:` no arquivo de conteúdo. Restabeleça
`:sectnums:` apenas onde a publicação realmente usa capítulos numerados. Não
numere artigos de revista como capítulos por efeito colateral do Asciidoctor.

## Fase 4 — Converter HTML semântico em AsciiDoc

### Tabela de mapeamento

| Origem JWPUB | Destino recomendado |
| --- | --- |
| `header > h1` | título `= ...` da página; |
| `h2` a `h6` | seções `==` em diante, respeitando hierarquia; |
| parágrafo comum | parágrafo AsciiDoc; |
| `span.parNum[data-pnum]` | número editorial do parágrafo, mesmo quando o texto do `span` estiver vazio; |
| `p.qu` | pergunta de estudo em bloco próprio, preservando número e partes `(a)`, `(b)`; |
| `strong` | negrito; |
| `em` | itálico; |
| `ul`, `ol`, `li` | listas AsciiDoc reais; |
| `dl`, `dt`, `dd` | lista de descrição; |
| `table`, `tr`, `th`, `td` | tabela AsciiDoc, preservando cabeçalhos e células; |
| `aside`, `.blockTeach`, `.boxContent` | sidebar ou bloco delimitado nativo; |
| `figure` | imagem em bloco, legenda, alt e crédito; |
| marcador `data-fnid` + `.fn-ref` | nota AsciiDoc no ponto da chamada; |
| `blockquote` | bloco de citação; |
| `br` | quebra apenas quando semanticamente necessária; |
| `hr` decorativo | omitir, salvo quando for separador editorial real; |
| `.pageNum[aria-hidden=true]` | omitir do corpo; usar apenas para derivar páginas/citações; |
| `.gen-field`, `textarea`, rótulo de resposta | omitir como controle interativo; |
| elemento `hidden` ou `aria-hidden=true` | omitir depois de confirmar sua função; |
| parágrafo realmente vazio | omitir; |
| `data-pid` referenciado | âncora AsciiDoc estável e namespaced. |

Prefira recursos nativos do AsciiDoc. Não use HTML passthrough para evitar a
conversão. Roles específicas só devem ser adicionadas quando já houver suporte
na aplicação agregadora ou quando carregarem semântica útil mesmo sem CSS.

### Cabeçalho editorial do artigo

Datas de estudo, cântico, texto temático e objetivo não são parágrafos comuns
intercambiáveis. Preserve a ordem e a função visível. Um padrão possível é:

```adoc
= Título oficial do artigo

[.article-period]
6-12 DE JULHO DE 2026

*CÂNTICO 98* — A Bíblia, um presente de Deus

_“Texto temático.”_ — Rom. 12:1.

*OBJETIVO*

Descrição do objetivo.
```

Use o texto real da edição. O exemplo não autoriza criar classes, travessões ou
pontuação que a fonte não possua.

### Números de parágrafo

Não dependa apenas do `InnerText`. O JW Library pode representar um número em
`data-pnum` ou por CSS. Compare:

- `data-pnum`;
- conteúdo do `span.parNum`;
- `DocumentParagraph` no SQLite;
- apresentação oficial, quando disponível.

Preserve a numeração editorial sem ativar `:sectnums:` para parágrafos. Não
confunda o número da pergunta com o número do parágrafo relacionado.

### Perguntas e campos de resposta

Preserve a pergunta `p.qu`, incluindo intervalo como `1-2`, partes `(a)` e
`(b)`, referências e instruções “Veja também”. Remova:

- `div.gen-field`;
- `textarea`;
- `label` “Sua resposta” ou “Suas respostas” quando for somente interface;
- classes visuais do aplicativo.

Não remova texto apenas porque possui classe para leitor de tela. Confirme que
é rótulo de um controle; outras descrições acessíveis podem ser conteúdo útil.

### Quadros e materiais de revisão

Converta `aside`, quadros de ensino e perguntas finais em sidebar, seção ou
bloco delimitado. Preserve:

- título do quadro;
- introdução;
- lista e ordem dos itens;
- referências;
- notas e imagens internas.

Não misture o quadro ao parágrafo anterior nem o transforme numa imagem.

### Notas

Resolva notas por `data-fnid`, não pela proximidade visual. Para cada marcador:

1. encontre a definição correspondente em `.fn-ref`;
2. preserve ênfase, links e texto integral;
3. insira `footnote:` ou `footnote:<id>[]` no ponto exato da chamada;
4. remova o bloco de definições duplicado da sequência principal;
5. confirme que nenhuma definição ficou órfã e nenhum marcador ficou sem texto.

Use identificadores namespaced, por exemplo:

```text
fn-<MepsDocumentId>-<data-fnid>
```

Uma letra colada à legenda ou à palavra anterior no TXT pode ser apenas o
marcador de nota. Na amostra analisada, uma legenda terminava visualmente em
`e`; o HTML demonstrou que `e` apontava para “DESCRIÇÃO DA IMAGEM”. Corrigir
isso como suposto erro de digitação teria apagado conteúdo real.

### Âncoras

Links JWPUB podem apontar para intervalos de parágrafos. Para evitar colisão
entre páginas no PDF/EPUB combinado, não use somente `p73`. Prefira:

```text
meps-<MepsDocumentId>-p-<data-pid-ou-ordinal>
```

Crie âncoras para:

- começo de cada documento;
- destinos de vínculos internos;
- quadros e seções citados por faixa;
- parágrafos que precisam permanecer endereçáveis.

Mantenha um mapa temporário entre URI JWPUB, destino original e âncora AsciiDoc.

## Links

### Inventário antes da conversão

Classifique todos os `href`:

| Esquema | Significado comum |
| --- | --- |
| `jwpub://p/...` | documento ou publicação; |
| `jwpub://b/...` | passagem bíblica; |
| `jwpub://c/...` | comentário ou nota de estudo; |
| `#...` | nota ou alvo dentro do documento; |
| `http://` ou `https://` | recurso Web explícito; |
| relativo | recurso local que precisa ser resolvido. |

### Documentos dentro da mesma publicação

Quando o `MepsDocumentId` do URI existir em `documents.json`, converta para
`xref` da página correspondente. Se houver faixa, direcione à âncora
namespaced apropriada.

Não associe por título se o identificador exato estiver disponível.

### Publicações externas, Bíblia e comentários

Um navegador não entende `jwpub://`. Para destinos fora do módulo:

1. preserve sempre o texto visível da referência;
2. procure um destino oficial verificável em jw.org ou WOL;
3. use o código de idioma e o `MepsDocumentId` somente como pistas;
4. teste o URL final;
5. se não houver conversão confiável, mantenha a referência como texto e
   registre o vínculo na fila de revisão.

Não fabrique uma URL apenas substituindo o esquema. Rotas WOL variam por idioma
e contexto. Um resolvedor automático só pode ser usado quando estiver
documentado e testado para o padrão em questão.

Links `https://` já presentes podem ser preservados depois de conferir destino,
idioma e finalidade. Não transforme automaticamente todo vínculo em link da
fonte da página; referência interna e citação editorial são conceitos
diferentes.

### Verificação final de links

Antes do commit:

```powershell
rg -n 'jwpub://' modules asciidoctor
```

O resultado deve estar vazio no conteúdo público, salvo um exemplo documental
deliberado fora das páginas da publicação. Execute também Antora e verifique os
`xref` gerados.

## Imagens e outros recursos

### Fonte e cópia

Para cada `<img src="jwpub-media://...">`:

1. normalize o caminho sem permitir `..`;
2. localize o arquivo exato em `media/`;
3. confronte SHA-256 com `inner-inventory.json`;
4. preserve os bytes originais quando o formato for aceito;
5. copie para uma pasta namespaced pelo módulo;
6. registre origem -> destino na matriz;
7. preserve `alt`, legenda, crédito e nota associada.

Destino recomendado:

```text
modules/ROOT/assets/images/<módulo-da-edição>/
```

O namespace evita colisões entre edições que reutilizam nomes como
`image_1.jpg`. Mantenha o nome original quando ele já for ASCII, seguro e
estável. Se precisar renomear, use slug e conserve o mapa.

Não recomprima JPEG nem rasterize SVG por rotina. Se um backend não aceitar o
recurso, gere um derivado explícito, preserve a relação com o original e valide
visualmente.

### Macro compatível com Antora e Asciidoctor

As imagens compartilhadas em `modules/ROOT` precisam funcionar tanto como
recurso Antora quanto no livro construído diretamente. Sem uma abstração já
existente no repositório, use condicionais explícitas:

```adoc
.Legenda editorial preservada
ifdef::env-antora[]
image::ROOT:w_T-2026-05/2026400_univ_cnt_1.jpg["Texto alternativo preservado"]
endif::[]
ifndef::env-antora[]
image::w_T-2026-05/2026400_univ_cnt_1.jpg["Texto alternativo preservado"]
endif::[]
```

No build direto, `imagesdir` aponta para a raiz compartilhada; no Antora,
`ROOT:` identifica o módulo de recursos. Se o repositório já fornecer um
atributo ou macro equivalente, reutilize-o e evite duplicação. Valide HTML,
PDF, EPUB e o site agregador antes de padronizar outra solução.

Textos alternativos com vírgulas devem ser colocados entre aspas. Escape
caracteres que tenham significado no AsciiDoc sem alterar a frase editorial.

### Imagem, legenda e descrição não são a mesma coisa

- `alt` descreve a imagem para acessibilidade;
- `figcaption` é legenda editorial visível;
- uma nota “DESCRIÇÃO DA IMAGEM” continua sendo nota quando a fonte a trata
  assim;
- crédito deve ser preservado quando existir;
- texto contido graficamente na imagem não substitui a legenda nem o alt.

Não use a legenda como alt automaticamente e não apague uma descrição por
parecer repetitiva.

### Recursos não referenciados no HTML

Arquivos relacionados apenas pelo SQLite ou manifesto podem ser:

- miniaturas de navegação;
- variações de capa;
- banners em várias proporções;
- mídia de um recurso secundário;
- arquivos não usados na apresentação escolhida.

Inspecione antes de copiar. A publicação final deve conter todas as mídias
editoriais usadas e apenas as variantes adicionais com função definida.

## Capas

O manifesto pode declarar várias imagens:

- `type = c`: candidata a capa completa;
- `type = t`: miniatura quadrada;
- `type = lsr`: variante horizontal;
- `type = pnr`: panorama.

Esses tipos são indícios observados. Confirme cada arquivo visualmente e pelas
dimensões reais; não selecione somente pelo nome ou pela dimensão declarada.

Ao adaptar ao modelo deste repositório:

- use a capa editorial integral como `cover-complete` quando apropriado;
- escolha um banner horizontal de resolução suficiente para o HTML;
- trate `cover-background` conforme a função da folha de rosto digital do
  repositório, sem fingir que uma miniatura é um fundo;
- use atributos por edição quando as capas variam;
- preserve a imagem original sempre que o backend aceitar JPEG/SVG;
- não crie três cópias idênticas apenas para satisfazer nomes convencionais.

Se nenhuma imagem atender a uma função, adapte a configuração de forma mínima
e reutilizável ou solicite decisão humana. Não distorça, recorte ou gere uma
capa sem autorização.

## Citação editorial e procedência

O leitor precisa ver a origem editorial, não a pasta de engenharia. Para cada
página:

- derive uma citação curta do mnemônico, edição e páginas ou unidade editorial;
- use números de página do HTML/SQLite somente depois de confirmar o intervalo;
- use WOL ou jw.org como link quando houver correspondência segura;
- preserve a estrutura `source-citation` definida em `AGENTS.md`;
- não exiba `DocumentId`, hash, caminho local, versão do conversor ou nome da
  pasta extraída na página pública.

`MepsDocumentId` pode ajudar a localizar a página oficial e a manter
rastreabilidade interna. Ele não precisa aparecer no rótulo editorial mostrado
ao leitor.

Se não houver URL pública confiável, use a citação sem link. Não omita a fonte
nem crie um link aproximado.

## Grafia, jargão e anomalias

O HTML original passou por uma transformação criptográfica e descompactação
determinística; por isso, uma palavra estranha não é automaticamente “erro de
OCR”. Ainda assim, pode haver:

- erro editorial na própria fonte;
- marcador de nota colado no TXT;
- texto gerado por atributo/CSS que não aparece no TXT;
- elemento de interface incluído na leitura;
- caractere com função estrutural;
- falha introduzida na conversão para AsciiDoc.

Quando algo não fizer sentido:

1. confira `html-original/`;
2. identifique atributos e notas relacionadas;
3. confira o SQLite quando necessário;
4. compare com a apresentação no JW Library ou a página oficial da mesma
   edição;
5. sinalize ao humano se a origem continuar ambígua.

Não corrija silenciosamente a fonte oficial. Se o usuário autorizar uma
correção editorial, registre a divergência e mantenha evidência do texto de
origem. Preserve sempre o jargão histórico da edição.

## Fase 5 — Montar o módulo

Depois de aprovar a matriz:

1. crie `modules/<módulo>/pages/`;
2. copie somente os recursos selecionados para a pasta namespaced;
3. crie as páginas em ordem;
4. crie `modules/<módulo>/nav.adoc`;
5. adicione a entrada em `antora.yml` na posição cronológica correta;
6. adicione os metadados em `asciidoctor/publication.yml`;
7. crie ou atualize o arquivo de conteúdo usado pelo produto;
8. atualize README e página inicial;
9. configure capas por publicação ou edição;
10. verifique todos os `xref`, imagens e citações.

Não modifique `Rakefile`, temas ou suporte compartilhado apenas para contornar
uma página mal convertida. Uma mudança estrutural precisa ser genérica,
necessária para mais de uma publicação e separada do conteúdo no histórico.

## Fase 6 — Validação de completude

### Matriz de reconciliação

Para cada documento, compare origem e destino:

| Unidade | Conferência |
| --- | --- |
| título e subtítulos | mesma quantidade, ordem e texto; |
| parágrafos | texto, pontuação, número e ordem; |
| perguntas | texto, partes e relação com parágrafos; |
| listas/tabelas | itens/células completos e na mesma ordem; |
| quadros | título, corpo e posição; |
| notas | marcadores e definições um a um; |
| figuras | arquivo, hash, alt, legenda, crédito e posição; |
| links internos | destino por `MepsDocumentId` e âncora; |
| links externos | URL verificada ou referência preservada como texto; |
| ênfase | negrito e itálico semanticamente preservados. |

Uma diferença só é aceitável quando estiver na lista explícita de
transformações, como remover `textarea`, `pageNum` do corpo ou adaptar um link
privado. Não declare completude por comparação simples com o TXT achatado.

### Verificações automáticas

Execute ao menos:

```powershell
git diff --check
rg -n 'jwpub://' modules asciidoctor
rg -n '<(div|span|figure|textarea|aside)\b' modules --glob '*.adoc'
npm exec -- antora antora-playbook.yml
bundle exec rake
```

O segundo comando deve encontrar apenas exemplos intencionais fora do conteúdo,
e o terceiro não deve revelar HTML bruto usado como atalho.

Confirme ainda:

- todos os arquivos de imagem referenciados existem;
- nenhuma mídia usada ficou fora do Git;
- nenhum arquivo sem uso foi adicionado por engano;
- todos os módulos e conteúdos estão em ordem;
- títulos de edição seguem mnemônico, idioma e data;
- avisos do Antora/Asciidoctor foram entendidos;
- a árvore Git contém somente mudanças intencionais.

### Conferência visual

Renderize e inspecione, no mínimo:

1. capa;
2. expediente ou página editorial;
3. sumário;
4. artigo com títulos, perguntas e parágrafos numerados;
5. página com figura, legenda e nota;
6. página com quadro/lista;
7. última página;
8. sumário e folha de rosto do PDF/EPUB.

Compare o HTML Antora, o HTML do livro, o PDF e o EPUB. Imagem que funciona no
site local pode falhar no EPUB se o caminho não for portátil.

## Evidência da amostra `w_T_202605`

A análise que fundamenta estas regras encontrou:

- 8 documentos em ordem de `DocumentId`;
- capa, sumário, cinco artigos de estudo e uma matéria curta;
- 14.002 tokens validados nos dois caminhos do conversor;
- 29 JPEG íntegros;
- 14 imagens únicas referenciadas pelo HTML;
- outras imagens de miniatura, capa e banner ligadas pelo SQLite/manifesto;
- 245 vínculos bíblicos `jwpub://b`;
- 30 vínculos de publicação `jwpub://p`;
- um vínculo `jwpub://c` e um HTTPS explícito;
- perguntas, campos de resposta, quadros, listas, notas e figuras;
- uma nota de descrição de imagem cujo marcador seria confundido com erro no
  TXT;
- uma divergência não fatal entre dimensão declarada e física da capa.

Esses números não são um esquema obrigatório para outra publicação. Eles
demonstram que importar apenas oito TXTs ou copiar as 29 imagens sem função
editorial perderia relações importantes.

## Fila de revisão humana

Interrompa e apresente ao usuário quando houver:

- data ou mnemônico ambíguo;
- ordem divergente entre `DocumentId`, sumário e visão da publicação;
- documento sem papel editorial claro;
- link privado sem destino oficial confiável;
- nota sem marcador ou marcador sem nota;
- mídia referenciada ausente;
- legenda, crédito ou alt contraditório;
- texto suspeito que continue estranho no HTML original;
- recurso vetorial ou multimídia que o pipeline atual não represente;
- necessidade de alterar infraestrutura compartilhada;
- decisão de corrigir a fonte em vez de reproduzi-la.

Não esconda essas pendências em uma página pública. Resolva-as antes do commit
ou registre a decisão humana no histórico de trabalho.

## Critério de aceite

A publicação importada está pronta quando:

1. todas as validações do conversor continuam aprovadas;
2. cada documento foi reconciliado com uma ou mais páginas;
3. hierarquia, texto, números, perguntas, quadros, listas e notas conferem;
4. cada figura preserva arquivo, alt, legenda, crédito e nota;
5. nenhum `jwpub://` chegou ao produto público;
6. vínculos internos usam `MepsDocumentId`, arquivo e âncora corretos;
7. metadados, título, módulo e data foram confirmados editorialmente;
8. navegação e conteúdos têm a mesma ordem;
9. Antora, HTML, PDF e EPUB foram gerados sem falhas;
10. páginas representativas foram comparadas visualmente;
11. anomalias foram resolvidas ou decididas pelo humano;
12. artefatos brutos e caminhos locais não foram expostos ao leitor;
13. o diff contém apenas o conteúdo e os recursos autorizados.

## Checklist curto para o agente

- [ ] Li `AGENTS.md`, a instrução de extração e este arquivo por completo.
- [ ] A estrutura JWPUB passou pelo portão de validação.
- [ ] Criei matrizes de documentos, estruturas, links e mídias.
- [ ] Confirmei mnemônico, código de idioma e data editorial.
- [ ] Planejei um módulo e páginas em ordem verificável.
- [ ] Usei `html-original/` como fonte semântica principal.
- [ ] Usei o SQLite para reconciliar relações, sem importar índices ou BLOBs por palpite.
- [ ] Removi somente controles de interface comprovados.
- [ ] Preservei parágrafos, perguntas, quadros, notas e ênfases.
- [ ] Mapeei vínculos internos por `MepsDocumentId`.
- [ ] Resolvi ou sinalizei todos os esquemas `jwpub://`.
- [ ] Copiei imagens por função, com hash, alt, legenda e crédito.
- [ ] Validei caminhos de imagem em Antora, HTML, PDF e EPUB.
- [ ] Reconciliou-se cada unidade da fonte com o destino.
- [ ] Não silenciei grafia ou estrutura suspeita.
- [ ] Não alterei infraestrutura nem publiquei sem autorização.
