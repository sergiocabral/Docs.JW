# Extração de PDFs JW para agentes de inteligência artificial

## Finalidade

Este arquivo é a instrução permanente para interpretar publicações JW em PDF,
especialmente os arquivos oficiais cuja aparência visual está correta, mas cuja
camada de texto usa fontes personalizadas sem mapeamento Unicode confiável.

O objetivo é produzir uma transcrição auditável em TXT e Markdown, preservar a
estrutura editorial e, quando existirem, extrair também as imagens com a melhor
qualidade disponível.

Leia este documento inteiro antes de converter um PDF desse tipo. Não presuma
que a saída de pdftotext, de uma biblioteca de PDF ou de um OCR representa
sozinha o conteúdo da página.

## Resultado esperado

Uma conversão completa pode produzir:

~~~text
build/
  nome-da-publicacao.pdf
  nome-da-publicacao.txt
  nome-da-publicacao.md

tmp/pdfs/nome-da-publicacao/
  diagnostics/
  render-preview/
  render-ocr/
  vector/
  ocr-text/
  ocr-tsv/
  images-native/
  images-rendered/
  review/
~~~

Os arquivos em tmp são evidências intermediárias e não devem ser versionados.
Os arquivos finais só podem ser incorporados aos módulos AsciiDoc depois da
revisão visual.

## Regras de fidelidade

1. O PDF examinado é a fonte de verdade para aquela edição.
2. Preserve redação, ortografia, pontuação, maiúsculas, abreviações, números,
   referências, ênfases e ordem editorial.
3. Não atualize silenciosamente o texto usando a WOL ou uma edição posterior.
4. Outra fonte pode sugerir onde há um erro, mas a correção deve ser confirmada
   na página renderizada da mesma edição.
5. Registre a página física do PDF e a página impressa.
6. Não descarte conteúdo repetido sem confirmação visual. Destaques laterais
   frequentemente repetem parte de um parágrafo de propósito.
7. Preserve imagens, legendas e a posição lógica de cada figura.
8. Não invente descrição, legenda, título, referência ou número ausente.
9. Trate números e pontuação como itens de alto risco mesmo quando o OCR
   informar confiança alta.
10. Normalize a saída final em UTF-8, Unicode NFC e finais de linha LF.

## Modelo de evidências

Use fontes independentes e resolva as divergências:

| Camada | Melhor uso | Limitação |
| --- | --- | --- |
| Página renderizada | Autoridade visual | Exige inspeção |
| OCR em português | Caracteres e prosa | Perde estilos e pode errar números |
| OCR TSV ou hOCR | Palavras, caixas e confiança | Ordem de blocos pode estar errada |
| Camada vetorial | Fontes, coordenadas e números pequenos | Texto pode estar decomposto |
| Objetos de imagem | Imagem original incorporada | Não inclui desenhos vetoriais |
| Sumário e índice | Estrutura e completude | Não substituem o corpo |

Quando houver conflito, a página renderizada decide.

## Reconhecer o problema

### Sintomas comuns

- acentos aparecem separados da vogal;
- cedilha aparece antes ou depois do c;
- i acentuado vira i sem ponto mais acento separado;
- palavras visualmente corretas saem sem acento;
- a ordem das palavras muda;
- números sobrescritos desaparecem ou viram sinais;
- colunas, quadros e destaques são misturados;
- títulos aparecem no fim da página extraída;
- pdftotext produz muitos caracteres estranhos apesar de o PDF estar nítido.

### Diagnóstico objetivo

Execute pdfinfo, pdffonts e pdftotext numa amostra. Em pdffonts, observe:

- fontes Type 1, Type 1C ou subconjuntos incorporados;
- encoding Custom ou WinAnsi;
- coluna uni igual a no;
- famílias editoriais personalizadas;
- várias fontes semelhantes para regular, medium, demi, bold e italic.

Se a maioria das fontes relevantes não tiver ToUnicode e a extração direta
decompuser acentos, adote o fluxo híbrido deste documento.

Se o PDF for uma digitalização sem texto vetorial, use OCR como fonte primária
e geometria da imagem para a estrutura. Se o PDF tiver ToUnicode correto, use a
camada textual como primária, mas mantenha a verificação visual.

## Escolher Windows ou WSL

As duas rotas são válidas. Prefira o ambiente que já tenha as ferramentas e
que seja mais reproduzível na máquina atual.

### Quando preferir WSL

- uma distribuição Ubuntu ou Debian já está configurada;
- deseja-se instalar pacotes por versão do sistema;
- scripts Bash facilitam o processamento em lote;
- as ferramentas do Windows não estão no PATH;
- é importante evitar variações entre instaladores de terceiros.

O repositório em D:\git normalmente aparece como /mnt/d/git no WSL. Confirme
com wslpath; não suponha o ponto de montagem.

### Quando preferir Windows

- o PDF e as ferramentas já estão acessíveis nativamente;
- a inspeção visual usa aplicações do Windows;
- não há distribuição WSL instalada;
- a movimentação de milhares de imagens entre sistemas de arquivos seria mais
  custosa do que o processamento nativo.

Não processe metade de uma página num ambiente e metade em outro sem registrar
as versões. Resultados de OCR podem variar entre versões.

## Dependências

### WSL Ubuntu ou Debian

~~~bash
sudo apt-get update
sudo apt-get install -y poppler-utils tesseract-ocr tesseract-ocr-por qpdf mupdf-tools python3-venv
~~~

Ferramentas opcionais para inspeção programática:

~~~bash
python3 -m venv tmp/pdfs/pdf-tools-venv
. tmp/pdfs/pdf-tools-venv/bin/activate
python3 -m pip install pdfplumber pypdf pillow
~~~

### Windows

~~~powershell
winget install --id oschwartz10612.Poppler --exact --accept-package-agreements --accept-source-agreements
winget install --id tesseract-ocr.tesseract --exact --accept-package-agreements --accept-source-agreements
~~~

O pacote do Tesseract para Windows pode trazer apenas eng e osd. Verifique
antes de usar. Se por não estiver disponível, baixe por.traineddata do
tessdata_best para uma pasta tessdata dentro do diretório temporário do
trabalho e passe essa pasta com --tessdata-dir.

Exemplo, depois de definir a pasta de trabalho:

~~~powershell
$tessdata = Join-Path $work 'tessdata'
New-Item -ItemType Directory -Path $tessdata -Force | Out-Null
$porModel = Join-Path $tessdata 'por.traineddata'
Invoke-WebRequest -Uri 'https://github.com/tesseract-ocr/tessdata_best/raw/refs/heads/main/por.traineddata' -OutFile $porModel
~~~

### Descoberta, não suposição

No WSL:

~~~bash
command -v pdfinfo
command -v pdffonts
command -v pdftoppm
command -v pdftotext
command -v pdfimages
command -v tesseract
tesseract --list-langs
~~~

No PowerShell:

~~~powershell
Get-Command pdfinfo,pdffonts,pdftoppm,pdftotext,pdfimages,tesseract -ErrorAction SilentlyContinue
& 'C:\Program Files\Tesseract-OCR\tesseract.exe' --list-langs
~~~

Se o Poppler instalado por winget não entrar no PATH da sessão atual,
localize pdftoppm.exe dentro de
Microsoft\WinGet\Packages e derive os outros executáveis a partir da mesma
pasta. Não codifique esse caminho como se fosse universal.

### Variáveis da sessão do Windows

Quando o Poppler estiver no PATH:

~~~powershell
$pdftoppm = (Get-Command pdftoppm -ErrorAction Stop).Source
$popplerBin = Split-Path -Parent $pdftoppm
$pdfinfo = Join-Path $popplerBin 'pdfinfo.exe'
$pdffonts = Join-Path $popplerBin 'pdffonts.exe'
$pdftotext = Join-Path $popplerBin 'pdftotext.exe'
$pdftohtml = Join-Path $popplerBin 'pdftohtml.exe'
$pdfimages = Join-Path $popplerBin 'pdfimages.exe'
$tesseract = (Get-Command tesseract -ErrorAction Stop).Source
~~~

Quando ele não estiver:

~~~powershell
$wingetPackages = Join-Path $env:LOCALAPPDATA 'Microsoft\WinGet\Packages'
$pdftoppm = Get-ChildItem -LiteralPath $wingetPackages -Recurse -Filter 'pdftoppm.exe' | Select-Object -First 1 -ExpandProperty FullName
if (-not $pdftoppm) { throw 'Poppler não encontrado.' }

$popplerBin = Split-Path -Parent $pdftoppm
$pdfinfo = Join-Path $popplerBin 'pdfinfo.exe'
$pdffonts = Join-Path $popplerBin 'pdffonts.exe'
$pdftotext = Join-Path $popplerBin 'pdftotext.exe'
$pdftohtml = Join-Path $popplerBin 'pdftohtml.exe'
$pdfimages = Join-Path $popplerBin 'pdfimages.exe'

$tesseract = 'C:\Program Files\Tesseract-OCR\tesseract.exe'
if (-not (Test-Path -LiteralPath $tesseract)) { throw 'Tesseract não encontrado.' }
~~~

Não reutilize um caminho descoberto em outra máquina sem validá-lo.

## Preparar o trabalho

### WSL

~~~bash
REPO="$(pwd)"
PDF="$REPO/build/nome-da-publicacao.pdf"
WORK="$REPO/tmp/pdfs/nome-da-publicacao"
mkdir -p "$WORK/diagnostics" "$WORK/render-preview" "$WORK/render-ocr"
mkdir -p "$WORK/vector" "$WORK/ocr-text" "$WORK/ocr-tsv"
mkdir -p "$WORK/images-native" "$WORK/images-rendered" "$WORK/review"
~~~

Se o caminho vier do Windows:

~~~powershell
wsl -e wslpath -a 'D:\git\Docs.JW.NomeDaPublicacao'
~~~

### Windows

~~~powershell
$repo = (Resolve-Path -LiteralPath '.').Path
$pdf = Join-Path $repo 'build\nome-da-publicacao.pdf'
$work = Join-Path $repo 'tmp\pdfs\nome-da-publicacao'
@('diagnostics','render-preview','render-ocr','vector','ocr-text','ocr-tsv','images-native','images-rendered','review') | ForEach-Object {
  New-Item -ItemType Directory -Path (Join-Path $work $_) -Force | Out-Null
}
~~~

Use nomes estáveis. Nunca sobrescreva o PDF de origem.

## Inventário inicial

Registre antes de extrair:

- nome e hash SHA-256;
- tamanho;
- título, autor, produtor e datas declaradas;
- quantidade e dimensões das páginas;
- criptografia e restrições;
- fontes e mapeamento Unicode;
- objetos de imagem;
- páginas aparentemente vazias;
- primeira e última página impressa;
- relação entre página física e página impressa.

### WSL

~~~bash
sha256sum "$PDF" > "$WORK/diagnostics/sha256.txt"
pdfinfo "$PDF" > "$WORK/diagnostics/pdfinfo.txt"
pdffonts "$PDF" > "$WORK/diagnostics/pdffonts.txt"
pdfimages -list "$PDF" > "$WORK/diagnostics/pdfimages.txt"
qpdf --check "$PDF" > "$WORK/diagnostics/qpdf-check.txt" 2>&1
~~~

### Windows

~~~powershell
Get-FileHash -Algorithm SHA256 -LiteralPath $pdf | Format-List | Out-String | Set-Content -Encoding utf8 (Join-Path $work 'diagnostics\sha256.txt')
& $pdfinfo $pdf | Set-Content -Encoding utf8 (Join-Path $work 'diagnostics\pdfinfo.txt')
& $pdffonts $pdf | Set-Content -Encoding utf8 (Join-Path $work 'diagnostics\pdffonts.txt')
& $pdfimages -list $pdf | Set-Content -Encoding utf8 (Join-Path $work 'diagnostics\pdfimages.txt')
~~~

Se qpdf relatar dano estrutural, repare uma cópia temporária e preserve o
original. Documente a transformação e compare as páginas antes de continuar.

Avisos de linearização ou de hint tables não significam necessariamente que o
conteúdo esteja danificado. O arquivo od_T 2015-09.pdf faz qpdf encerrar com
avisos, mas informa que a operação foi bem-sucedida e suas 228 páginas
renderizam corretamente. Não regrave um PDF apenas para eliminar avisos; faça
uma cópia reparada somente quando houver erro real que afete leitura,
renderização ou extração.

## Catálogo visual e mapa de páginas

Renderize uma versão leve de todas as páginas para reconhecer a publicação:

### WSL

~~~bash
pdftoppm -png -r 160 "$PDF" "$WORK/render-preview/page"
~~~

### Windows

~~~powershell
& $pdftoppm -png -r 160 $pdf (Join-Path $work 'render-preview\page')
~~~

Examine:

- capa, página de rosto e página editora;
- sumário;
- carta, prefácio ou introdução;
- todas as aberturas de capítulo;
- primeira ocorrência de cada layout;
- listas, tabelas, quadros, perguntas e formulários;
- imagens e respectivas legendas;
- apêndices;
- início e continuação do índice;
- notas e última página.

Crie uma tabela com página física, página impressa, função, título corrente,
tipo de layout e presença de imagens. Confirme qualquer deslocamento entre
página física e impressa com pelo menos três âncoras distantes.

Não classifique como página vazia apenas porque pdftotext não retornou texto.
Ela pode conter imagem, desenho vetorial ou texto sem mapeamento.

## Extração do texto

### Manter três leituras

Gere pelo menos:

1. texto vetorial com layout;
2. coordenadas vetoriais em bbox ou XML;
3. OCR da página renderizada, em TXT e TSV.

A combinação permite usar caracteres do OCR e estrutura do PDF.

### Camada vetorial

No WSL:

~~~bash
pdftotext -layout "$PDF" "$WORK/vector/layout.txt"
pdftotext -raw "$PDF" "$WORK/vector/raw.txt"
pdftotext -bbox-layout "$PDF" "$WORK/vector/bbox.html"
pdftohtml -xml -hidden -i "$PDF" "$WORK/vector/text-layout.xml"
~~~

No Windows:

~~~powershell
& $pdftotext -layout $pdf (Join-Path $work 'vector\layout.txt')
& $pdftotext -raw $pdf (Join-Path $work 'vector\raw.txt')
& $pdftotext -bbox-layout $pdf (Join-Path $work 'vector\bbox.html')
& $pdftohtml -xml -hidden -i $pdf (Join-Path $work 'vector\text-layout.xml')
~~~

Use bbox para coordenadas. Use o XML para associar trechos às famílias de
fonte. O sinal -i ignora imagens nessa cópia do XML; uma extração separada,
sem -i, será feita na seção de imagens.

Opcionalmente, pdfplumber pode expor caracteres, linhas, retângulos e imagens:

~~~python
import pdfplumber

with pdfplumber.open("publicacao.pdf") as pdf:
    page = pdf.pages[0]
    print(page.width, page.height)
    print(page.chars[:5])
    print(page.images)
~~~

Não grave uma transcrição final diretamente dessa camada quando os caracteres
estiverem decompostos.

### Renderização para OCR

Para livros com tipografia nítida, 300 dpi em escala de cinza costuma ser o
melhor ponto inicial:

No WSL:

~~~bash
pdftoppm -gray -png -r 300 "$PDF" "$WORK/render-ocr/page"
~~~

No Windows:

~~~powershell
& $pdftoppm -gray -png -r 300 $pdf (Join-Path $work 'render-ocr\page')
~~~

Use 400 dpi apenas em recortes pequenos, sobrescritos muito finos ou páginas
que falharam a 300 dpi. Aumentar o livro inteiro encarece o processo e nem
sempre melhora o reconhecimento.

### OCR principal

Para prosa comum, comece com Tesseract PSM 3 e idioma da publicação.

No WSL:

~~~bash
for image in "$WORK"/render-ocr/*.png; do
  stem="$(basename "$image" .png)"
  tesseract "$image" "$WORK/ocr-text/$stem" -l por --dpi 300 --psm 3 -c preserve_interword_spaces=1
  tesseract "$image" "$WORK/ocr-tsv/$stem" -l por --dpi 300 --psm 3 -c preserve_interword_spaces=1 tsv
done
~~~

No Windows, quando o modelo por está numa pasta própria:

~~~powershell
Get-ChildItem -LiteralPath (Join-Path $work 'render-ocr') -Filter '*.png' | Sort-Object Name | ForEach-Object {
  $stem = $_.BaseName
  & $tesseract $_.FullName (Join-Path $work ('ocr-text\' + $stem)) -l por --tessdata-dir $tessdata --dpi 300 --psm 3 -c preserve_interword_spaces=1
  & $tesseract $_.FullName (Join-Path $work ('ocr-tsv\' + $stem)) -l por --tessdata-dir $tessdata --dpi 300 --psm 3 -c preserve_interword_spaces=1 -c tessedit_create_tsv=1
}
~~~

Ao usar uma pasta tessdata mínima no Windows, tessedit_create_tsv=1 é mais
confiável do que depender do arquivo de configuração tsv.

### Escolher o modo de segmentação

| Layout | Estratégia inicial |
| --- | --- |
| Prosa de uma coluna | PSM 3 |
| Sumário em linhas | PSM 4 |
| Bloco isolado ou recortado | PSM 6 |
| Texto esparso | PSM 11 como leitura alternativa |
| Índice | PSM 3 ou 4 mais ordenação por coordenadas |
| Duas colunas reais | Separar colunas ou usar TSV |

Faça testes numa página representativa antes de processar exceções em lote.
Não escolha o modo apenas pela maior quantidade de texto: o critério é ordem e
fidelidade.

### Reconstrução pelo TSV

O TSV contém nível, página, bloco, parágrafo, linha, palavra, left, top, width,
height, confiança e texto.

Procedimento obrigatório:

1. mantenha registros de palavra, normalmente level 5;
2. descarte registros vazios;
3. agrupe por bloco, parágrafo e linha;
4. ordene as palavras da linha por left;
5. calcule top e left mínimos da linha;
6. não preserve a ordem de retorno dos grupos;
7. ordene todas as linhas por top e, no empate, por left;
8. reconstrua parágrafos usando recuo, distância vertical e números
   sobrescritos;
9. registre confiança mínima e média;
10. envie linhas de baixa confiança para revisão visual.

Tesseract pode devolver blocos do fim da página antes de blocos do meio. Isso
é comum no índice e em páginas com destaques laterais.

## Detectar a estrutura editorial

### Medir antes de classificar

Calcule a mediana do tamanho da fonte no corpo e use proporções, posição e
espaçamento. Valores absolutos variam entre publicações.

Sinais úteis:

- título de capítulo: fonte grande, espaço acima e abaixo, próximo ao início
  de uma seção;
- subtítulo: peso maior, frequentemente em maiúsculas, seguido de prosa;
- número de parágrafo: fonte pequena, sobrescrita, à esquerda da primeira
  linha;
- destaque lateral: fonte maior que o corpo numa coluna curta;
- cabeçalho ou rodapé: fonte pequena em faixa extrema da página;
- legenda: próxima a uma imagem e menor ou diferente do corpo;
- nota: corpo menor, linha divisória ou posição inferior;
- índice: termos principais em peso maior e subentradas recuadas;
- tabela: alinhamentos repetidos em x e y, não apenas espaços no texto.

Famílias com nomes Regular, Medium, Demi, Bold, Italic e Display ajudam a
confirmar o papel, mas não devem ser usadas sem a posição visual.

### Abertura de capítulo

Uma publicação JW desse período pode usar:

1. faixa ou retângulo cinza com CAPÍTULO e número;
2. título grande em caixa normal;
3. primeiro parágrafo sem número visível;
4. parágrafos seguintes com números pequenos sobrescritos;
5. subtítulos internos em maiúsculas;
6. cabeçalho corrido e número de página no rodapé.

No Markdown:

~~~markdown
## Capítulo 1 — Título do capítulo

<!-- source-page: 6; pdf-page: 10 -->

PRIMEIRAS palavras do parágrafo inicial...

<sup>2</sup> Segundo parágrafo...

### SUBTÍTULO INTERNO
~~~

Não invente o número 1 se ele não estiver impresso.

### Títulos e subtítulos

Classifique um trecho como título somente quando vários sinais concordarem:

- tamanho relativo;
- peso da fonte;
- caixa;
- posição;
- espaço em branco;
- alinhamento;
- relação com o sumário;
- início de uma nova sequência de parágrafos.

Uma frase em maiúsculas no começo do primeiro parágrafo não é necessariamente
um título. Um cabeçalho corrido repetido também não é título.

### Números de parágrafo

O OCR pode ler o sobrescrito 2, 5 ou 7 como interrogação, maior que ou outro
sinal. Recupere-o por:

- objeto vetorial pequeno próximo ao início do parágrafo;
- sequência dos números anteriores e posteriores;
- inspeção ampliada da imagem;
- posição e tamanho, nunca apenas pela expectativa da sequência.

Preserve o número inline no TXT e como sup no Markdown. Se o número realmente
não estiver visível, não o crie.

### Destaques laterais

Destaques laterais interrompem a largura normal da prosa e podem ser
posicionados antes, ao lado ou depois do parágrafo correspondente. Eles podem
repetir literalmente uma sentença do corpo.

Preserve as duas ocorrências. No Markdown, use uma citação em bloco precedida
de comentário estrutural:

~~~markdown
<!-- pull-quote -->
> Texto do destaque lateral.
~~~

Coloque-o no ponto lógico indicado pela página, normalmente antes do parágrafo
do qual foi extraído. Não misture suas linhas às da coluna vizinha.

### Colunas

Detecte colunas por agrupamentos de coordenadas x e por grandes intervalos
horizontais. Determine a ordem:

1. de cima para baixo dentro da coluna;
2. da coluna esquerda para a direita, salvo indicação contrária;
3. com exceções para títulos que atravessam colunas;
4. com tratamento separado para destaques, imagens e legendas.

Não confie na ordem dos objetos do PDF. Compare o resultado com a página.

### Listas, tabelas e formulários

- lista: preserve marcador, número, recuo e continuidade;
- tabela: reconstrua linhas e colunas pela geometria;
- formulário: preserve campos, linhas e rótulos;
- perguntas de batismo: preserve partes, perguntas, referências e numeração;
- sumário: use tabela Markdown;
- índice: use termos principais e subentradas, mantendo remissões Veja.

No índice, termos principais podem ser confirmados por negrito ou fonte Demi,
e subentradas por recuo. Margens alternam entre páginas pares e ímpares; use
recuo relativo à margem da própria página.

### Cabeçalhos, rodapés e páginas

Remova do fluxo do corpo:

- título corrente repetido;
- número de página;
- marcas de impressão sem conteúdo editorial.

Antes de remover, registre a página. Insira marcadores no ponto exato:

~~~markdown
<!-- source-page: 29; pdf-page: 33 -->
~~~

Se a página começar no meio de um parágrafo, o marcador deve ficar naquele
ponto, não antes ou depois do parágrafo completo.

### Hifenização

Una quebras tipográficas:

~~~text
po- + líticas = políticas
conhecê- + lo = conhecê-lo
recém-con- + vertido = recém-convertido
~~~

A regra de próxima linha em minúscula não basta. Confirme compostos reais,
pronomes enclíticos, intervalos, travessões e a imagem. Não remova hífen
lexical.

### Acentos decompostos

Em fontes JW antigas, o acento pode ser um objeto estreito colocado de 4 a 7
pontos acima da base. A cedilha e o i sem ponto também podem ser separados.

Recomposição típica:

| Componentes | Resultado |
| --- | --- |
| agudo mais a, e, i, o, u | á, é, í, ó, ú |
| circunflexo mais a, e, o | â, ê, ô |
| til mais a ou o | ã, õ |
| grave mais a | à |
| c mais cedilha | ç |
| i sem ponto mais agudo | í |

Use a posição horizontal, a linha de base e a largura média dos caracteres
para escolher a vogal. Normalize depois em NFC. Não aplique substituição global
sem geometria e inspeção visual.

### Itálico, negrito e versaletes

OCR não preserva estilo. Recupere por:

- XML do pdftohtml;
- nome e peso da fonte;
- inspeção visual;
- coordenadas do trecho;
- padrão editorial.

No Markdown, use ênfase somente quando ela estiver realmente impressa. Títulos
de publicações costumam estar em itálico, mas isso deve ser confirmado.

## Detectar e extrair imagens

### Tipos de conteúdo visual

Uma página pode conter:

1. imagem raster incorporada, como JPEG, JPEG 2000 ou bitmap;
2. imagem raster com máscara ou canal alfa separado;
3. ilustração dividida em vários objetos;
4. desenho vetorial, linhas e formas;
5. página inteira digitalizada como uma imagem;
6. texto convertido em curvas;
7. combinação de imagem, vetores e texto.

pdfimages encontra objetos raster, mas não encontra desenhos puramente
vetoriais. A ausência de objetos em pdfimages não prova que a página não tem
ilustração.

### Inventário de objetos raster

No WSL:

~~~bash
pdfimages -list "$PDF" > "$WORK/diagnostics/pdfimages.txt"
~~~

No Windows:

~~~powershell
& $pdfimages -list $pdf | Set-Content -Encoding utf8 (Join-Path $work 'diagnostics\pdfimages.txt')
~~~

Registre para cada objeto:

- página;
- número do objeto;
- tipo;
- largura e altura;
- espaço de cor;
- quantidade de componentes;
- bits por componente;
- codificação;
- máscara ou soft mask;
- resolução x e y;
- tamanho;
- object ID.

Um objeto quase do tamanho da página pode ser uma página digitalizada, não uma
figura editorial independente.

### Extração nativa

Prefira a imagem incorporada, pois ela evita nova compressão.

No WSL:

~~~bash
pdfimages -all -p "$PDF" "$WORK/images-native/image"
~~~

No Windows:

~~~powershell
& $pdfimages -all -p $pdf (Join-Path $work 'images-native\image')
~~~

A opção -all preserva formatos suportados, como JPEG e JPEG 2000, quando
possível. A opção -p inclui a página no nome. Não converta JPEG para PNG apenas
para uniformizar extensões.

Depois:

1. calcule SHA-256 de cada arquivo;
2. associe-o à página e ao object ID;
3. identifique máscara e imagem principal;
4. abra cada imagem;
5. compare cores, orientação e recorte com a página;
6. registre duplicatas sem perder as ocorrências.

### Posição da imagem

pdfimages -list não fornece toda a posição editorial. Obtenha caixas por:

- XML do pdftohtml executado sem -i;
- page.images do pdfplumber;
- saída estruturada do MuPDF;
- inspeção da página renderizada.

Execute a versão do XML que conserva imagens dentro do diretório temporário,
pois ela também criará arquivos auxiliares:

~~~bash
cd "$WORK/vector"
pdftohtml -xml -hidden "$PDF" image-layout.xml
~~~

No PowerShell:

~~~powershell
Push-Location (Join-Path $work 'vector')
try {
  & $pdftohtml -xml -hidden $pdf 'image-layout.xml'
} finally {
  Pop-Location
}
~~~

Procure elementos image com top, left, width, height e src. Compare com a lista
de pdfimages; os números podem não coincidir diretamente.

Com pdfplumber:

~~~python
import pdfplumber

with pdfplumber.open("publicacao.pdf") as pdf:
    for page_number, page in enumerate(pdf.pages, start=1):
        for index, image in enumerate(page.images, start=1):
            print(page_number, index, image)
~~~

### Máscaras, mosaicos e objetos repetidos

- Soft mask pode ser o canal de transparência de outra imagem.
- Imagem monocromática pode ser uma máscara, não uma figura visível sozinha.
- Uma ilustração pode estar dividida em blocos ou faixas.
- Logotipos e ornamentos podem ser reutilizados em muitas páginas.
- O mesmo hash pode ter várias posições legítimas.

Mantenha os objetos nativos e, se necessário, produza também uma composição
renderizada. O manifesto deve explicar a relação.

### Conteúdo vetorial

Se uma ilustração aparece na renderização, mas não em pdfimages:

1. confirme que ela é vetorial;
2. preserve uma versão da página em SVG quando isso for útil;
3. produza um recorte raster em resolução suficiente;
4. compare o recorte com a página;
5. não confunda texto do corpo com parte da ilustração.

No WSL, MuPDF pode gerar SVG por página:

~~~bash
mutool draw -F svg -o "$WORK/images-rendered/page-%03d.svg" "$PDF"
~~~

O SVG pode representar a página completa. Para uso editorial, um recorte PNG
da figura costuma ser mais simples, desde que tenha resolução adequada.

### Recortes renderizados

Quando não for possível recuperar um objeto único, renderize a página e
recorte a região. Em pdftoppm, x, y, W e H são pixels na resolução escolhida:

~~~bash
pdftoppm -f 33 -l 33 -singlefile -png -r 300 -x 120 -y 260 -W 900 -H 700 "$PDF" "$WORK/images-rendered/p033-figure-01"
~~~

No PowerShell, use os mesmos parâmetros:

~~~powershell
& $pdftoppm -f 33 -l 33 -singlefile -png -r 300 -x 120 -y 260 -W 900 -H 700 $pdf (Join-Path $work 'images-rendered\p033-figure-01')
~~~

Conversão entre pontos do PDF e pixels:

~~~text
pixels = pontos × dpi ÷ 72
~~~

Use a caixa visual efetiva, sem cabeçalho, rodapé ou texto vizinho. Se a figura
tiver legenda, extraia a legenda como texto e registre a associação; não a
recorte junto à imagem a menos que o projeto exija uma reprodução fac-símile.

### Texto dentro de imagem

Se uma figura contém texto:

- preserve a imagem;
- execute OCR do recorte como leitura auxiliar;
- confira todas as palavras visualmente;
- mantenha o texto da imagem separado do fluxo do corpo;
- não transforme automaticamente o OCR em legenda;
- registre que a origem é texto incorporado à imagem.

### Nomes e manifesto

Use nomes previsíveis:

~~~text
p033-object-004.jpg
p033-object-005-mask.pbm
p033-figure-01.png
p033-figure-01.svg
~~~

Crie manifest.csv ou manifest.md com:

| Campo | Conteúdo |
| --- | --- |
| arquivo | nome final |
| página física | página do PDF |
| página impressa | página editorial |
| object ID | quando disponível |
| tipo | nativa, máscara, composição, vetor ou recorte |
| dimensões | pixels ou caixa |
| hash | SHA-256 |
| legenda | texto literal ou vazio |
| posição lógica | antes ou depois do parágrafo |
| revisão | pendente ou conferida |

Não dê nome semântico a uma figura se seu assunto não estiver claro. Um nome
neutro e auditável é melhor do que uma interpretação inventada.

### Destino no repositório

Durante a conversão, mantenha imagens em tmp. Depois da revisão e somente
quando forem usadas pela publicação, copie-as para o diretório de assets
apropriado, com nomes slug em ASCII, e crie referências AsciiDoc. Não versione
imagens intermediárias, máscaras descartadas ou renderizações de diagnóstico.

## Normalizar a transcrição

### Saída bruta e revisada

Mantenha:

- OCR bruto por página;
- TSV ou hOCR;
- camada vetorial;
- log de correções;
- TXT final;
- Markdown final.

O TXT final deve ser legível, sem marcação visual, mas pode conservar
marcadores de página temporários:

~~~text
[[source-page:29; pdf-page:33]]
~~~

O Markdown deve preservar títulos, listas, tabelas, ênfases, destaques e
imagens:

~~~markdown
![Legenda literal, se houver](images/p033-figure-01.png)
~~~

Se não houver legenda ou texto alternativo verificável, não invente um. Deixe
a decisão editorial anotada para a incorporação em AsciiDoc.

### Unir linhas e parágrafos

1. una linhas do mesmo parágrafo com espaço;
2. remova apenas hifenização tipográfica confirmada;
3. preserve parágrafos;
4. preserve números e listas;
5. separe colunas e destaques;
6. remova cabeçalhos e rodapés do corpo;
7. mantenha o marcador no ponto da quebra de página;
8. associe figuras à posição lógica.

### Referências e pontuação

Revise visualmente:

- livros bíblicos;
- capítulo e versículo;
- páginas e intervalos;
- anos, ISBN e códigos;
- aspas simples e duplas;
- travessões, hífens e reticências;
- símbolos de copyright;
- números de parágrafo.

Confiança alta do OCR não elimina essa revisão.

## Controle de qualidade

### Verificações automáticas

Confirme:

- total de páginas renderizadas igual ao pdfinfo;
- OCR para toda página que contém texto;
- inventário de todas as imagens;
- sequência das páginas;
- capítulos e seções do sumário presentes;
- ausência de caractere de substituição;
- ausência de i sem ponto indevido;
- ausência de acentos isolados;
- ausência de hifenização quebrada;
- números de parágrafo crescentes dentro da seção;
- imagens referenciadas existentes;
- nenhum arquivo de diagnóstico referenciado como asset final.

Buscas úteis:

~~~bash
rg -n '[�ı]' build/nome-da-publicacao.md
rg -n '[´ˆ˜¸](\s|$)' build/nome-da-publicacao.md
rg -n '\w-$' build/nome-da-publicacao.md
rg -n '\?\s+[A-ZÁÉÍÓÚÂÊÔÃÕ]' build/nome-da-publicacao.md
~~~

Uma busca limpa não prova fidelidade; elimina apenas padrões conhecidos.

### Fila de revisão obrigatória

Revise:

1. linhas com confiança baixa;
2. números, datas e referências;
3. divergências entre OCR e camada vetorial;
4. títulos e subtítulos;
5. parágrafos com sobrescritos;
6. destaques laterais;
7. itálico e negrito;
8. sumário, tabelas e índice;
9. todas as imagens e legendas;
10. palavras corrigidas manualmente.

### Conferência visual integral

Para declarar a conversão concluída:

1. abra texto e página renderizada lado a lado;
2. percorra todas as páginas em ordem;
3. confira início e fim de página;
4. confira ordem de leitura;
5. confira cada parágrafo e número;
6. confira títulos, estilos, referências e pontuação;
7. confira imagem, recorte, cor, máscara, legenda e posição;
8. marque a página como revisada;
9. faça uma segunda passagem pelos itens alterados.

### Critério de aceite

A extração está pronta quando:

- todas as páginas foram classificadas;
- nenhuma seção ou figura foi omitida;
- TXT e Markdown têm a mesma redação;
- a hierarquia corresponde ao PDF;
- imagens nativas foram preferidas quando disponíveis;
- composições e recortes foram documentados;
- todos os itens de risco foram vistos na página;
- os arquivos finais estão em UTF-8, NFC e LF;
- não há dúvida não registrada.

## Perfil conhecido: PDFs JW oficiais de 2015

A análise de od_T 2015-09.pdf encontrou:

- 228 páginas físicas e 224 impressas;
- fontes WtKnoll Type 1C incorporadas;
- quase todas sem ToUnicode;
- acentos e cedilha como objetos separados;
- números de parágrafo em fonte próxima de 6 pt;
- corpo próximo de 10 pt;
- títulos de capítulo próximos de 18 pt;
- destaques laterais próximos de 12 pt;
- apêndices com títulos próximos de 15 pt;
- cabeçalhos corridos próximos de 6 pt.

Nesse arquivo, pdfimages encontrou seis ocorrências raster apenas nas páginas
físicas 1 a 4: dois JPEGs de página inteira reutilizados e dois pequenos
stencils. Não encontrou imagem raster no corpo das páginas 5 a 228. Assim, os
painéis cinza, linhas e demais elementos visuais do miolo devem ser tratados
como vetores, não como figuras omitidas pela extração.

Falhas reais do OCR:

| Origem | OCR | Correto |
| --- | --- | --- |
| página editora | O 2005, 2015 | © 2005, 2015 |
| parágrafo sobrescrito | ? | 2 |
| nome bíblico | Eden | Éden |
| índice | pp.1/0-208 | pp.170-208 |
| título com aspas simples | aspas duplas ou apóstrofo | aspas simples curvas |

O sumário teve melhor ordem com PSM 4. O índice exigiu TSV e ordenação global
por top e left. Esses achados são pistas para PDFs semelhantes, não regras
universais.

## Limpeza e rastreabilidade

Conserve os intermediários enquanto houver revisão pendente. Ao concluir:

1. confirme que TXT, Markdown, imagens finais e manifesto estão salvos;
2. confirme que nenhuma página depende de arquivo temporário;
3. resolva o caminho absoluto de tmp/pdfs/nome-da-publicacao;
4. verifique que ele está dentro do repositório e de tmp/pdfs;
5. remova somente esse diretório temporário;
6. confira o estado do Git.

Nunca faça exclusão recursiva sobre caminho vazio, raiz do repositório, variável
não resolvida ou diretório compartilhado.

## Checklist curto para o agente

1. Leia AGENTS.md e este arquivo inteiro.
2. Identifique o PDF por hash.
3. Execute pdfinfo, pdffonts, pdfimages e qpdf.
4. Renderize todas as páginas para catálogo visual.
5. Mapeie páginas físicas e impressas.
6. Extraia texto vetorial, bbox e XML.
7. Execute OCR em português, TXT e TSV.
8. Reprocesse layouts especiais com PSM adequado.
9. Reconstrua títulos, parágrafos, colunas, listas e índice.
10. Extraia imagens nativas e identifique vetores.
11. Produza recortes apenas quando a extração nativa não for suficiente.
12. Gere TXT, Markdown e manifesto de imagens.
13. Faça validações automáticas.
14. Compare visualmente todas as páginas e imagens.
15. Só então incorpore o conteúdo aos módulos AsciiDoc.
