# Extração de publicações JWPUB para agentes de inteligência artificial

## Finalidade

Este documento orienta a obtenção de uma estrutura auditável a partir de um
arquivo nativo `.jwpub` do JW Library. Ele não ensina a reconstruir o conversor
nem substitui a documentação técnica da aplicação. Seu objetivo é fazer o
agente:

1. localizar uma versão confiável do **JWPUB Converter**;
2. executar a conversão sem alterar a entrada;
3. reconhecer o que existe dentro de um JWPUB;
4. validar integralmente a saída antes de usá-la como fonte editorial;
5. entregar a estrutura aprovada ao procedimento de importação em
   `AGENTS-JWPUB-IMPORT.md`.

Leia este arquivo por completo antes de processar um JWPUB. Para transformar a
saída em páginas deste repositório, leia depois, também por completo,
`AGENTS-JWPUB-IMPORT.md`.

## Regra de ativação

Use este procedimento quando a fonte recebida for um arquivo `.jwpub` ou quando
o usuário pedir para trazer uma publicação diretamente do JW Library.

Não aplique estas instruções a:

- PDFs, que seguem `AGENTS-JW-PDF-EXTRACTION.md`;
- EPUBs isolados;
- backups, notas ou dados pessoais do JW Library;
- uma pasta HTML sem o contêiner JWPUB original;
- conteúdo obtido de procedência desconhecida.

O agente só deve iniciar a importação editorial depois que a conversão terminar
com código zero e todas as provas obrigatórias forem aprovadas.

## Aplicação que é a fonte de verdade

O conversor pronto, seu código, testes e documentação estão em:

- repositório: <https://github.com/sergiocabral/App.JwPubConverter>;
- releases: <https://github.com/sergiocabral/App.JwPubConverter/releases>;
- formato JWPUB:
  <https://github.com/sergiocabral/App.JwPubConverter/blob/main/docs/02-formato-jwpub.md>;
- criptografia e descompactação:
  <https://github.com/sergiocabral/App.JwPubConverter/blob/main/docs/03-criptografia-e-descompactacao.md>;
- documentos e mídias:
  <https://github.com/sergiocabral/App.JwPubConverter/blob/main/docs/04-extracao-texto-e-midias.md>;
- validações e segurança:
  <https://github.com/sergiocabral/App.JwPubConverter/blob/main/docs/05-validacoes-e-seguranca.md>;
- operação e diagnóstico:
  <https://github.com/sergiocabral/App.JwPubConverter/blob/main/docs/10-operacao-e-solucao-de-problemas.md>.

O projeto histórico que demonstrou o formato e serviu de prova de conceito pode
existir em `D:\jw\app-jw-library`, mas esse caminho é apenas contexto local. Ele
não é uma dependência e não deve ser incorporado a scripts, páginas ou
configurações deste repositório.

Não copie para cá a implementação criptográfica. Não crie outro extrator por
conveniência. Se o comportamento do formato mudar, a correção pertence primeiro
ao `App.JwPubConverter`, com testes e nova versão; este repositório deve apenas
consumir uma saída validada.

## Como obter o executável

### 1. Release publicada

Em uma máquina conectada, prefira a release mais recente compatível com o
ambiente. Baixe o executável, o arquivo de checksums e o manifesto do mesmo
release. Não misture arquivos de versões diferentes.

Antes de executar:

```powershell
& $converter --version
Get-FileHash -LiteralPath $converter -Algorithm SHA256
```

Registre a versão e o SHA-256 no relatório de trabalho ou na mensagem do commit
da futura importação. Não exponha caminhos locais ou dados operacionais nas
páginas destinadas ao leitor.

### 2. Snapshot de contingência do clone

Um clone comum do repositório contém este snapshot Windows x64:

```text
dist/win-x64/jwpub-converter.exe
```

Ele existe para uso emergencial sem SDK e não acompanha automaticamente cada
nova release. Confirme sua versão e compare seu SHA-256 com:

```text
dist/win-x64/SHA256SUMS.txt
```

Exemplo:

```powershell
$converterRoot = Resolve-Path '..\App.JwPubConverter'
$converter = Join-Path $converterRoot 'dist\win-x64\jwpub-converter.exe'
$checksumFile = Join-Path $converterRoot 'dist\win-x64\SHA256SUMS.txt'

$expected = ((Get-Content -LiteralPath $checksumFile -Raw) -split '\s+')[0]
$actual = (Get-FileHash -LiteralPath $converter -Algorithm SHA256).Hash
if ($actual -ne $expected) {
    throw "SHA-256 inesperado para o conversor: $actual"
}

& $converter --version
```

Não suponha que o snapshot de `dist` seja a versão mais recente. Se ele não
suportar a publicação, consulte releases e a documentação da aplicação.
Ajuste `converterRoot` quando o clone não estiver ao lado deste repositório.

### 3. Download direto de contingência

A cópia versionada pode ser consultada em:

<https://github.com/sergiocabral/App.JwPubConverter/blob/main/dist/win-x64/jwpub-converter.exe>

O endereço bruto é:

<https://raw.githubusercontent.com/sergiocabral/App.JwPubConverter/main/dist/win-x64/jwpub-converter.exe>

Se usar esse caminho móvel de emergência, baixe também
`dist/win-x64/SHA256SUMS.txt` da mesma revisão e confira o hash antes de executar.
Para uma execução reproduzível, prefira uma release ou fixe o commit do URL;
`main` pode mudar entre dois downloads.

### Compatibilidade do ambiente

O snapshot conhecido é autocontido para Windows x64. Ele não exige instalação
de .NET, SQLite, Node.js, Python, Visual Studio nem do JW Library. Consulte a
release para saber se outras plataformas passaram a ser oferecidas.

Em Windows com WSL, prefira executar o `.exe` pelo PowerShell do host e use
caminhos Windows. A interoperabilidade do WSL pode chamar executáveis Windows,
mas a mistura de caminhos `/mnt/...` e `C:\...` ou `D:\...` aumenta o risco de
gravar a saída no lugar errado.

## Teoria mínima do formato

### Contêiner em duas camadas

Na variante observada, um JWPUB é um ZIP externo sem senha no nível do
contêiner:

```text
publicacao.jwpub
├── manifest.json
└── contents
    └── ZIP interno
        ├── <publicação>.db
        └── mídias e recursos
```

`manifest.json` informa, entre outros dados:

- hash SHA-256 dos bytes exatos de `contents`;
- tamanho expandido esperado;
- variante `contentFormat`;
- nome do banco SQLite;
- título, símbolo, idioma numérico, ano e identificadores da edição;
- imagens editoriais e suas dimensões declaradas.

O membro `contents` é outro ZIP, mesmo sem a extensão `.zip`. O banco declarado
é SQLite legível e não SQLCipher. Texto, metadados relacionais, referências e
mídias fazem parte da publicação; o TXT é somente uma representação derivada.

### O que é criptografado

O conteúdo principal de cada unidade editorial está no BLOB
`Document.Content`. Para `contentFormat = "z-a"`, o pipeline observado é:

```text
HTML/XML UTF-8
  -> zlib
  -> AES-128-CBC com PKCS#7
  -> Document.Content
```

A leitura executa o inverso:

```text
Document.Content
  -> AES-128-CBC/PKCS#7
  -> zlib
  -> HTML/XML UTF-8
```

O material criptográfico é derivado deterministicamente de uma identidade com
esta forma:

```text
<language>_<symbol>_<year>[_<issueIdentifier>]
```

A precedência do identificador da edição é:

1. `publication.issueTagNumber`, quando diferente de zero;
2. senão `publication.issueId`, quando diferente de zero;
3. senão nenhum sufixo.

`publication.issueNumber` é ordinal editorial e **não** participa da derivação.
A aplicação calcula SHA-256 da identidade e combina o resultado com a constante
da variante para obter chave e IV. Os detalhes e a constante pertencem ao código
e à documentação do conversor; não os duplique aqui.

### O SQLite é um modelo relacional da publicação

O banco não é apenas um recipiente para o HTML criptografado. Ele relaciona
documentos, unidades editoriais, navegação, mídia e índices usados pelo JW
Library. Os nomes e a presença das tabelas variam conforme a versão do esquema,
mas estas famílias ajudam a orientar a inspeção:

| Família | Exemplos observados | O que pode comprovar |
| --- | --- | --- |
| publicação e catálogo | `Publication`, `PublicationIssue*`, `PublicationYear`, `RefPublication` | título, símbolos, idioma, edição e publicações relacionadas; |
| documentos | `Document`, `DocumentParagraph`, `DocumentMetadata`, `TextUnit` | ordem, títulos, capítulos, páginas e associação de parágrafos; |
| estrutura editorial | `Question`, `Footnote`, `Endnote`, `Extract`, `BibleCitation` | perguntas, notas, extratos e citações; |
| navegação | `Hyperlink`, `InternalLink`, `DocumentHyperlink`, `PublicationView*` | vínculos, destinos e visualizações da publicação; |
| mídia | `Multimedia`, `DocumentMultimedia`, `ExtractMultimedia` | MIME, dimensões, legendas, créditos e relação com documentos; |
| pesquisa | `Word`, `SearchIndex*`, `SearchTextRange*` | vocabulário e posições pesquisáveis; |
| recursos opcionais | `Topic*`, `*Commentary*`, `VideoMarker*`, `*Verse*Map` | recursos usados somente por algumas publicações. |

Examine primeiro `database/schema.sql` e `database/tables.txt`; eles permitem
entender a forma do banco sem depender de uma ferramenta SQLite. Consulte o
banco em modo somente leitura quando precisar contar relações ou resolver uma
ambiguidade que não esteja exposta nos inventários.

Na amostra de referência `w_T_202605`, 33 das 52 tabelas tinham registros. A
reconciliação encontrou 8 documentos, 460 associações de parágrafos, 93
perguntas, 277 links, 245 citações bíblicas, 30 registros de multimídia,
2.214 palavras distintas no índice e 9 publicações referenciadas. Esses valores
são evidência de uma execução conhecida, não requisitos para outro JWPUB.

### Limite atual da descriptografia

O contrato do conversor descriptografa e descomprime cada `Document.Content`.
O SQLite também pode declarar uma coluna `Content` em tabelas como `Question`,
`Footnote`, `Endnote`, `Extract`, `DatedText` e tabelas de comentários. Não
suponha que esses BLOBs sejam plaintext, usem necessariamente o mesmo pipeline
ou precisem ser convertidos separadamente; parte dessa informação já pode estar
representada no HTML do documento.

Da mesma forma, BLOBs de índices de busca podem ser estruturas binárias de
posições e identificadores, e não conteúdo editorial criptografado. O banco
também contém metadados legíveis, como títulos, palavras, legendas, créditos e
links. Portanto, nem “todo o banco é criptografado” nem “todo BLOB é texto” são
descrições corretas.

Se uma importação depender de um `Content` secundário que não aparece no HTML,
interrompa e leve o caso ao `App.JwPubConverter`. A aplicação deve ganhar suporte
determinístico e validações antes que este repositório consuma o novo dado.

### Limites dessa explicação

JWPUB é um formato de implementação e pode evoluir. A teoria acima descreve a
variante comprovada, não uma promessa de compatibilidade futura. O conversor
deve rejeitar outra variante explicitamente. Um plaintext aparentemente legível
de um único documento não prova que a publicação foi recuperada corretamente.

## Preparar a conversão

Trate a entrada como somente leitura e mantenha a saída fora das pastas
versionadas de conteúdo. `build/jwpub/` é um destino adequado quando já está
ignorado pelo Git. Outra pasta temporária específica também pode ser usada.

Antes de executar:

1. confirme o caminho absoluto do JWPUB;
2. calcule e registre seu SHA-256;
3. confirme que a pasta de saída não existe ou está vazia;
4. não reutilize uma saída parcial;
5. confira espaço livre suficiente;
6. preserve qualquer arquivo preexistente do usuário.

Exemplo seguro:

```powershell
$converter = 'D:\ferramentas\jwpub-converter.exe'
$inputFile = 'D:\fontes\publicacao.jwpub'
$outputDirectory = 'D:\trabalho\publicacao-estrutura'

if (-not (Test-Path -LiteralPath $converter -PathType Leaf)) {
    throw "Conversor não encontrado: $converter"
}
if (-not (Test-Path -LiteralPath $inputFile -PathType Leaf)) {
    throw "JWPUB não encontrado: $inputFile"
}
if (Test-Path -LiteralPath $outputDirectory) {
    $items = @(Get-ChildItem -LiteralPath $outputDirectory -Force)
    if ($items.Count -ne 0) {
        throw "A saída precisa estar vazia: $outputDirectory"
    }
}

$inputHash = (Get-FileHash -LiteralPath $inputFile -Algorithm SHA256).Hash
& $converter $inputFile -o $outputDirectory
if ($LASTEXITCODE -ne 0) {
    throw "Conversão JWPUB falhou com código $LASTEXITCODE"
}
```

Não automatize a exclusão da saída para conseguir uma segunda tentativa. Uma
falha deixa dados parciais úteis para diagnóstico; use outra pasta e apague a
anterior somente depois de resolver e conferir o caminho exato.

## Contrato da saída

A saída não é apenas uma conversão para texto. Ela é uma cópia de auditoria do
pacote: preserva contêiner, banco e mídias, acrescenta representações legíveis e
registra provas de integridade. O módulo AsciiDoc futuro é um produto editorial
derivado dessa estrutura, não um substituto para suas evidências de auditoria.

Uma conversão aprovada produz normalmente:

```text
<saída>/
├── container/
│   ├── manifest.json
│   ├── manifest.pretty.json
│   └── contents.zip
├── database/
│   ├── <publicação>.db
│   ├── schema.sql
│   └── tables.txt
├── media/
├── html-original/
├── html-browser/
├── documents/
├── <publicação>.txt
├── <publicação>-auditoria.txt
├── documents.json
├── documents.csv
├── inner-inventory.json
├── inner-inventory.csv
├── media-audit.json
├── validation-report.json
├── run-metadata.json
├── checksums.sha256
├── index.html
└── README.md
```

Use cada representação para a finalidade correta:

| Recurso | Finalidade |
| --- | --- |
| `html-original/` | Plaintext UTF-8 exato obtido depois de AES e zlib; fonte semântica principal. |
| `documents.json` | Ordem, `DocumentId`, `MepsDocumentId`, títulos, classe, capítulo, hashes e caminhos. |
| `documents/` | TXT legível por documento; útil para busca e comparação, mas sem toda a estrutura. |
| TXT integral | Leitura contínua na ordem de `DocumentId`; não deve ser convertido mecanicamente em AsciiDoc. |
| `html-browser/` | Inspeção visual com `jwpub-media://` adaptado; é derivado, não fonte exata. |
| `media/` | Mídias preservadas do ZIP interno. |
| `container/` | Manifesto e ZIP interno preservados para auditoria. |
| `database/` | SQLite original, esquema e inventário das tabelas; modelo relacional para conferências. |
| `validation-report.json` | Provas consolidadas e avisos. |
| `media-audit.json` | Mídias, referências, ausências, dimensões e arquivos inválidos. |
| `run-metadata.json` | Versão, identidade, caminhos locais, ambiente, métricas e hashes da execução. |
| `checksums.sha256` | Integridade de todos os arquivos da estrutura, exceto ele próprio. |

O TXT pode conter rótulos de interface para leitores de tela, fundir marcadores
de nota ao texto e achatar títulos, caixas e figuras. Nunca o trate como fonte
única quando `html-original/` está disponível.

## Portão de validação obrigatório

Depois do código zero, leia os relatórios. Exemplo:

```powershell
$validation = Get-Content -LiteralPath `
    (Join-Path $outputDirectory 'validation-report.json') -Raw |
    ConvertFrom-Json
$run = Get-Content -LiteralPath `
    (Join-Path $outputDirectory 'run-metadata.json') -Raw |
    ConvertFrom-Json

if ($validation.status -ne 'passed') {
    throw 'A validação final do conversor não foi aprovada.'
}
if ($validation.sqlite.quickCheck -ne 'ok') {
    throw 'PRAGMA quick_check não retornou ok.'
}
if (-not $validation.manifest.hashMatches -or
    -not $validation.manifest.expandedSizeMatches) {
    throw 'Manifesto, hash ou tamanho expandido divergente.'
}
if ($validation.documents.total -ne $validation.documents.decrypted -or
    $validation.documents.total -ne
        $validation.documents.exactSemanticTokenSequences) {
    throw 'Nem todos os documentos foram validados semanticamente.'
}
if (-not $validation.input.unchanged) {
    throw 'O arquivo de entrada mudou durante a conversão.'
}
```

Além disso, confirme:

- todos os JPEG e SVG reconhecidos são válidos;
- listas de mídia ausente estão vazias;
- `invalidFiles` está vazio;
- nenhum link local está quebrado;
- `checksums.sha256` existe;
- `documents.json` contém o mesmo total de documentos;
- os caminhos nele registrados existem;
- cada aviso foi lido e julgado, não apenas contado.

Uma divergência entre dimensão declarada e dimensão física pode ser um aviso
legítimo quando a mídia é válida e está presente. Isso não autoriza ignorar
outros avisos nem redimensionar a imagem silenciosamente.

## Quando a conversão falhar

Falhas em hash, ZIP interno, SQLite, padding AES, zlib, UTF-8, XML, mídia ou
links são bloqueantes. Não edite o manifesto, não desative validações e não
tente combinações de campos até alguma produzir texto plausível.

Registre para diagnóstico:

- versão e SHA-256 do conversor;
- SHA-256 da entrada;
- `contentFormat` e `schemaVersion`;
- `language`, `symbol`, `year`, `issueTagNumber`, `issueId` e `issueNumber`;
- mensagem completa e etapa da falha;
- relatórios da saída parcial que não exponham conteúdo desnecessário.

Leve a nova variante ao repositório do conversor. A correção precisa de uma
regra determinística, fixture ou teste de regressão e compatibilidade com os
formatos anteriores.

## Segurança, conteúdo e direitos

- Use apenas arquivos obtidos legitimamente.
- O conversor não prova autenticidade editorial por assinatura digital; ele
  prova consistência interna e transformação bem-sucedida.
- HTML original é conteúdo extraído e não é sanitizado para publicação pública.
- Abra entradas desconhecidas em ambiente controlado e não execute scripts do
  HTML.
- A pasta de saída contém texto e mídia em claro e deve receber a mesma proteção
  exigida pela publicação.
- Não versione o JWPUB, o SQLite, o ZIP interno nem toda a estrutura extraída por
  padrão.
- Não publique material apenas porque tecnicamente foi possível extraí-lo;
  respeite direitos autorais e o escopo autorizado pelo usuário.

Um `.jwpub` de publicação não deve ser confundido com um backup `.jwlibrary` ou
com o armazenamento do aplicativo, que podem conter notas e outros dados do
usuário. Na amostra `w_T_202605` não foram encontradas credenciais, tokens, dados
de conta nem informações pessoais do usuário, mas isso é uma observação da
amostra, não uma garantia para todo arquivo recebido. Inspecione uma origem
desconhecida antes de compartilhar sua extração.

`run-metadata.json` é criado pelo conversor e registra caminhos absolutos da
entrada, da saída e do executável, além de sistema operacional, runtime,
arquitetura, datas e hashes. Um caminho pode revelar nome de usuário ou
organização mesmo quando o JWPUB não contém dados pessoais. Não publique esse
arquivo sem revisão. Como `checksums.sha256` cobre a árvore validada, não edite a
cópia de auditoria para ocultar caminhos; mantenha o original protegido e gere,
quando necessário, uma cópia sanitizada claramente identificada como derivada.

Campos chamados `hash` ou `signature` podem ser impressões digitais editoriais
ou de mídia. Não os descreva como credenciais, chaves ou assinaturas de autoria
sem evidência do esquema. O hash do manifesto prova consistência interna dos
bytes declarados; sozinho, não prova autenticidade nem autoria do pacote.

## Rastreabilidade sem poluir o conteúdo

Para uma futura importação, conserve fora das páginas públicas:

- nome lógico da fonte;
- SHA-256 do JWPUB;
- versão e SHA-256 do conversor;
- data da execução;
- total de documentos, palavras e mídias;
- avisos revisados;
- confirmação de `validation-report.json` e dos checksums.

Essas informações podem aparecer no commit ou numa evidência temporária sob
`build/`. Não coloque caminhos da máquina, detalhes criptográficos ou relatórios
operacionais no texto que será lido pelo usuário do site.

## Critério de aceite

A extração está pronta para a etapa editorial somente quando:

1. a ferramenta veio de release, clone ou revisão identificável;
2. versão e hash foram conferidos;
3. a entrada permaneceu inalterada;
4. código de saída foi zero;
5. `validation-report.json` tem `status = "passed"`;
6. todos os documentos passaram por AES, zlib, UTF-8 e validação semântica;
7. SQLite, manifesto, mídias, links e checksums foram aprovados;
8. todos os avisos foram entendidos;
9. `html-original/`, `documents.json` e `media/` estão disponíveis;
10. o agente está pronto para seguir `AGENTS-JWPUB-IMPORT.md`.

## Checklist curto para o agente

- [ ] Li este arquivo por completo.
- [ ] Localizei o `App.JwPubConverter` ou uma release oficial do projeto.
- [ ] Conferi versão e SHA-256 do executável.
- [ ] Registrei o SHA-256 da entrada e usei uma saída nova ou vazia.
- [ ] A execução retornou código zero.
- [ ] `validation-report.json` foi aprovado integralmente.
- [ ] Todos os documentos, mídias, links e checksums passaram.
- [ ] Revisei cada aviso.
- [ ] Não confundi TXT derivado com a estrutura editorial completa.
- [ ] Entendi o SQLite como evidência relacional e não tratei todo BLOB como texto.
- [ ] Revisei `run-metadata.json` e preservei a cópia de auditoria antes de compartilhar.
- [ ] Não versione nem publique artefatos brutos sem autorização.
- [ ] Vou ler `AGENTS-JWPUB-IMPORT.md` antes de criar páginas AsciiDoc.
