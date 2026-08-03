# Instruções para agentes de inteligência artificial

## Objetivo

Este é um modelo de publicação em AsciiDoc com três usos:

- componente importável por outro site Antora;
- site Antora executável localmente;
- geração de HTML, PDF e EPUB.

Priorize conteúdo, caminhos previsíveis e pouca infraestrutura. Leia `README.adoc` antes de agir e preserve alterações preexistentes do usuário.

## Fontes de verdade

- `modules/`: conteúdo reconhecido pelo Antora.
- `modules/ROOT/pages/index.adoc`: apresentação da publicação.
- `modules/ROOT/assets/`: recursos compartilhados e capas.
- `modules/<módulo>/nav.adoc`: navegação Antora da edição.
- `modules/ROOT/nav*.adoc`: grupos de navegação agregada, quando essa topologia
  for necessária.
- `antora.yml`: componente, página inicial e módulos de navegação.
- `antora-playbook.yml`: execução local do site.
- `asciidoctor/publication.yml`: autor, identificador, títulos, slugs, idiomas e edições.
- `asciidoctor/contents.adoc` e os arquivos de conteúdo declarados em
  `publication.yml`: ordem do HTML, PDF e EPUB de cada produto.
- `asciidoctor/support/attributes/common.adoc`: atributos compartilhados.
- `asciidoctor/support/attributes/<idioma>.adoc`: traduções dos rótulos do Asciidoctor.
- `asciidoctor/support/publication.adoc` e `asciidoctor/support/themes/`: infraestrutura comum das publicações.

Não coloque identidade editorial no `Rakefile`, em `package.json` ou em `asciidoctor/support`.

## Terminologia das Testemunhas de Jeová

Todo conteúdo deste repositório está no contexto editorial das Testemunhas de
Jeová. Trate `jw.org` e a Biblioteca On-line da Torre de Vigia como referências
primárias para nomes, expressões e jargões oficiais no idioma de destino.

- Preserve os termos usados pela própria publicação; não os substitua por
  sinônimos religiosos genéricos.
- Ao traduzir, procure o equivalente oficial usado pelas Testemunhas de Jeová
  no idioma de destino, em vez de fazer uma tradução apenas literal.
- Considere sempre a data da edição: a terminologia atual não substitui
  automaticamente a terminologia histórica. Quando possível, consulte fontes
  JW do mesmo período.
- Não modernize silenciosamente um texto antigo. Por exemplo, preserve _Torre
  de Vigia_ numa edição que usa esse nome e _A Sentinela_ numa edição que usa o
  nome posterior; não presuma uma data de mudança sem confirmá-la.

## Ordem das publicações e variações

Em repositórios JW, apresente publicações, edições, idiomas e outras variações
da mais recente para a mais antiga. A pessoa deve encontrar primeiro o conteúdo
mais atual. Ao acrescentar uma variação, insira-a na posição cronológica
correta; não a acrescente automaticamente ao fim.

Aplique a ordem decrescente de forma consistente em:

- blocos de navegação de `antora.yml`;
- listas e tabelas do `README.adoc`;
- página inicial em `modules/ROOT/pages/index.adoc`;
- entradas de metadados em `asciidoctor/publication.yml`;
- qualquer seletor ou catálogo visível de variações.

Produtos derivados que abrangem várias edições, como **Análise entre edições**,
vêm depois das publicações-fonte. Dentro de uma análise longitudinal, porém,
transições, linhas do tempo e cadeias de expressões continuam na ordem antiga
→ recente, pois ali a cronologia faz parte do raciocínio.

## Navegação em aplicativos agregadores

O Antora entrega cada arquivo listado em `antora.yml` como uma árvore de
navegação independente. Aplicativos agregadores podem acrescentar acima dela um
nó próprio para identificar o componente. Se um único arquivo registrado tiver
várias raízes visíveis, o agregador poderá criar também um agrupamento com o
título do componente, produzindo um nível redundante como
`componente → componente → publicações`.

Para que a árvore possa ser incorporada diretamente pelo agregador:

- cada `nav*.adoc` listado em `antora.yml` deve ter exatamente uma raiz visível;
- quando houver vários grupos de primeiro nível, distribua-os entre arquivos
  como `modules/ROOT/nav-<grupo>.adoc` e registre cada arquivo separadamente, na
  ordem desejada;
- não crie uma raiz guarda-chuva que apenas repita o título já exibido pelo
  catálogo ou pelo nó da fonte no aplicativo agregador;
- não registre ao mesmo tempo a navegação local de um módulo e uma navegação
  agregada que reproduza a mesma árvore;
- quando o repositório tiver apenas uma raiz, mantenha um único `nav.adoc`; não
  crie arquivos adicionais sem necessidade.

Antes de reorganizar a navegação, confirme a forma como o agregador monta o nó
da fonte. O conteúdo concreto dos grupos e a quantidade de arquivos continuam
pertencendo a cada repositório; a regra estrutural é uma raiz por arquivo
registrado.

## Títulos das edições

Em `asciidoctor/publication.yml`, o campo `title` de cada edição JW deve ser
autoexplicativo e terminar com o identificador editorial entre parênteses. Esse
identificador combina o mnemônico, o código de idioma e a data mais exata da
edição:

```yaml
od_T-2021-12:
  title: Organizados para Fazer a Vontade de Jeová (od-T 2021-12)
  label: 'od-T 2005, 2015, 2020, 2021-12'
  mnemonic: od
  language_code: T
  edition_date: 2021-12
```

Use no título apenas `edition_date`, isto é, a data da própria edição. Não
copie para o título a sequência histórica de datas da página editora; preserve
essa sequência em `label` e no atributo `edition-label`. A regra se aplica às
edições da publicação, não automaticamente a produtos derivados, como uma
análise entre edições.

## PDFs JW como fonte

Antes de interpretar ou converter um PDF JW, leia integralmente
[AGENTS-JW-PDF-EXTRACTION.md](AGENTS-JW-PDF-EXTRACTION.md). Esse arquivo é a
fonte de verdade para diagnosticar fontes sem mapeamento Unicode, escolher
entre Windows e WSL, reconstruir títulos e ordem de leitura, extrair imagens e
validar a transcrição.

Não aceite a extração textual de uma única ferramenta como conteúdo final.
Mantenha as evidências temporárias em tmp/pdfs, compare texto e imagens com as
páginas renderizadas e só depois leve os arquivos revisados para modules e
modules/ROOT/assets.

## Arquivos JWPUB como fonte

Quando a fonte for um arquivo nativo `.jwpub` do JW Library, leia primeiro e
por completo
[AGENTS-JWPUB-EXTRACTION.md](AGENTS-JWPUB-EXTRACTION.md). Essa instrução
explica como obter e verificar o **JWPUB Converter** mantido em
`App.JwPubConverter`, executar a extração sem alterar a entrada e aprovar os
relatórios de integridade. Use a aplicação existente; não replique neste
repositório a lógica de contêiner, criptografia ou descompactação.

Depois de validar a estrutura extraída, leia integralmente
[AGENTS-JWPUB-IMPORT.md](AGENTS-JWPUB-IMPORT.md) antes de montar uma publicação.
Esse arquivo define como reconstruir documentos, títulos, parágrafos, notas,
links e mídias no modelo Antora/AsciiDoctor deste repositório. Trate
`html-original/` como fonte semântica principal e os arquivos TXT como apoio de
leitura e conferência, nunca como substitutos da estrutura editorial.

Incorpore as imagens editoriais com seus textos alternativos, legendas,
créditos e notas. Nenhum esquema privado `jwpub://` pode permanecer no conteúdo
público. Execute esse fluxo somente quando o usuário pedir a importação;
analisar uma estrutura JWPUB não autoriza criar o módulo correspondente.

## Comparação entre edições

Antes de criar ou atualizar uma análise longitudinal da mesma publicação, leia
integralmente
[AGENTS-EDITION-COMPARISON.md](AGENTS-EDITION-COMPARISON.md). Esse arquivo é a
fonte de verdade para inventariar as edições, associar capítulos por tema,
tratar renomeações, deslocamentos, divisões e fusões, alinhar mudanças por
transição e gerar uma publicação comparativa independente.

Execute esse procedimento somente quando o usuário solicitar explicitamente a
análise. Não associe conteúdo apenas pelo número ou pelo caminho do capítulo e
não apresente um diff bruto como relatório editorial.

## Criar outra publicação

Ao substituir o conteúdo deste modelo:

1. Atualize a tabela e a seção Sobre do `README.adoc` e de `modules/ROOT/pages/index.adoc`.
2. Substitua `modules/main/pages/` e reconstrua `modules/main/nav.adoc`; renomeie o módulo se a publicação exigir uma identificação mais específica.
3. Atualize `antora.yml`, `antora-playbook.yml` e `asciidoctor/publication.yml`.
4. Reconstrua `asciidoctor/contents.adoc` na ordem editorial correta.
5. Substitua as capas em `modules/ROOT/assets/images/`.
6. Remova todas as referências, imagens, títulos e slugs da publicação anterior.
7. Execute Antora e Asciidoctor antes de considerar o trabalho concluído.

`main` é somente o módulo neutro inicial. Quando houver variedades da publicação, use nomes que expressem a dimensão organizada:

- idiomas: `en-us`, `pt-br`;
- edições: `primeira-edicao`, `segunda-edicao`;
- combinação necessária: `pt-br-primeira-edicao`.

Use slugs ASCII minúsculos e uma convenção consistente. Para cada módulo, a chave em `publication.yml` deve ser idêntica ao diretório em `modules/`. O valor `lang`, como `pt-BR`, seleciona o perfil `pt-br.adoc`; o nome do módulo não substitui esse campo.

Ao renomear um módulo, atualize o diretório, `antora.yml`, `publication.yml`, o `nav.adoc` e todos os `xref` que mencionem o nome anterior. Preserve os mesmos nomes de página quando os conteúdos forem correspondentes.

Edite arquivos de infraestrutura somente quando a solicitação tratar da forma de executar ou publicar o projeto.

## Conteúdo e nomenclatura

Antes de criar ou atualizar conteúdo preparado para busca ou conversa com
inteligência artificial, leia integralmente
[AGENTS-RAG.md](AGENTS-RAG.md). Esse arquivo define a pasta `rag/`, os corpora,
as coordenadas de origem, o chunking, os resumos e a geração determinística
compartilhada com as demais famílias `Docs.*`.

Use:

```text
NNNN-slug-descritivo.adoc
```

Regras:

- quatro dígitos, preferencialmente em intervalos de dez;
- slug ASCII minúsculo com números e hífens;
- sem espaços, acentos, sublinhados ou pontuação;
- ordenação pelo nome igual à ordem natural da publicação;
- mesmos caminhos para páginas correspondentes entre edições.

Exemplo:

```text
0010-prefacio.adoc
0100-capitulo-01-titulo.adoc
0110-capitulo-02-titulo.adoc
```

No título AsciiDoc de um capítulo, escreva somente o título. Deixe número e rótulo explícitos no `nav.adoc`:

```adoc
= Um capítulo de exemplo
```

```adoc
* xref:0100-capitulo-01-exemplo.adoc[Capítulo 1 - Um capítulo de exemplo]
```

Sempre atualize os `nav*.adoc` afetados e o arquivo de conteúdo Asciidoctor
correspondente ao produto ao adicionar, remover, renomear ou reordenar páginas.
Quando houver mais de uma variação, mantenha a mais recente no topo das listas
externas.

## Citação da fonte

Em páginas baseadas em publicações JW, declare a referência no atributo `source-citation` e apresente somente a citação editorial curta. Não exiba os rótulos “Fonte” ou “WOL” nem a URL bruta. Use `pág.` ou `págs.` conforme necessário e inclua a edição quando ela for indispensável para distinguir a origem, por exemplo `od págs. 213-222` e `od-T 2005 págs. 219-224`.

Quando houver uma página correspondente na Biblioteca On-line da Torre de
Vigia, use como destino a URL completa conferida para aquela publicação, edição
e língua. Os segmentos de localidade, biblioteca e idioma não são universais e
não devem ser copiados automaticamente de uma publicação em português para
outra língua. Exemplo de uma URL confirmada para português do Brasil:

```adoc
:wol-source-url: https://wol.jw.org/pt/wol/d/r5/lp-t/1102014931
:source-citation: od cap. 1 págs. 6-11

[.source-reference]
{wol-source-url}[{source-citation}]
```

Quando não houver uma URL confiável para a edição, apresente a mesma estrutura como rótulo sem link:

```adoc
:source-citation: od-T 2005 págs. 5-9

[.source-reference]
[.source-label]#{source-citation}#
```

Prefira o link sempre que a correspondência com a WOL for segura. Preserve
exatamente as roles `source-reference` e `source-label` esperadas pela interface
agregadora e não use HTML ou passthrough para reproduzir a apresentação.

## Recursos e capas

Compartilhe imagens entre edições por meio de `modules/ROOT/assets/images/`.
Use nomes slug em ASCII e valide as referências no Antora.

Quando o repositório tiver uma única família de capas, preserve os nomes padrão:

- `cover-complete.png`: capa pronta do PDF e do EPUB;
- `cover-background.png`: página de título do PDF;
- `cover-banner.png`: apresentação inicial do HTML.

Quando o mesmo repositório reunir publicações ou linhagens com capas diferentes,
acrescente o mnemônico a toda a família, mantendo as três funções alinhadas:

- `cover-complete-<mnemônico>.png`;
- `cover-background-<mnemônico>.png`;
- `cover-banner-<mnemônico>.png`.

Não imponha o sufixo a um repositório com uma única capa e não fixe no código o
mnemônico de uma publicação específica. Declare a família padrão em `covers` no
`publication.yml`; quando edições do mesmo produto precisarem de famílias
diferentes, substitua os atributos `cover-complete-image`,
`cover-background-image` e `cover-banner-image` na edição correspondente.

Para a publicação derivada **Análise entre edições**, siga também
`AGENTS-EDITION-COMPARISON.md`: não use uma imagem de capa completa. O PDF deve
começar pela folha de rosto gerada, com o fundo compartilhado e texto digital.
A infraestrutura deve aceitar capas opcionais sem alterar o uso normal das três
capas nas edições do livro.

Não inclua páginas artificiais de capa ou sumário em `contents.adoc`; os conversores geram esses elementos.

## Infraestrutura

- Preserve `:doctype: book`; é o tipo técnico do Asciidoctor.
- Mantenha `asciidoctor/support` reutilizável entre repositórios.
- Mantenha `package.json` genérico; versões editoriais são tags Git.
- Não versione `build/` nem `node_modules/`.
- Atualize lockfiles somente junto com mudanças de dependências.
- Não acrescente bibliotecas, serviços, temas ou automações sem necessidade explícita.
- Não crie commits, tags, releases ou pushes sem autorização explícita.

## Validação

Execute:

```shell
npm exec -- antora antora-playbook.yml
bundle exec rake
```

Confirme:

- ausência de referências ou inclusões quebradas;
- exatamente uma raiz visível em cada arquivo de navegação registrado no
  `antora.yml` quando o componente for consumido pelo agregador;
- HTML, PDF e EPUB para todas as edições;
- variações em ordem decrescente nas navegações e catálogos;
- títulos de edição identificados por mnemônico, código de idioma e data;
- ordem definida no arquivo de conteúdo de cada produto;
- rótulos no idioma selecionado;
- capas aplicadas aos destinos esperados, respeitando a exceção documentada da
  análise entre edições;
- quando a fonte for JWPUB, relatórios do conversor aprovados, avisos revisados,
  mídias reconciliadas e ausência de `jwpub://` no produto público;
- somente mudanças intencionais no `git status`.

## Releases

O workflow `.github/workflows/release.yml` responde a tags anotadas `vX.Y.Z` e publica os formatos gerados. Antes de sugerir uma versão, compare a última tag com o estado atual e proponha número e mensagem objetivos.

Não crie ou envie uma tag sem autorização. Depois de publicada, não mova, apague ou reutilize a tag e não reescreva os commits alcançados por ela.
