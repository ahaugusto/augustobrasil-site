# Experimento: mesa de escritório 3D interativa (protótipo, não integrado)

**Status: pausado.** Não está em produção nem planejado para entrar no curto
prazo. Este diretório existe só para preservar o código e o histórico de
decisão, caso a ideia seja retomada mais adiante.

## A ideia original

Substituir a home do portfólio pessoal por uma cena 3D estilizada de uma
mesa de escritório, onde cada objeto físico dá acesso a uma parte do
conteúdo:

- **Monitor** → trabalho já entregue, organizado em pastas por contexto
  (Fóton/Caixa, Plataforma Brasil, Digibee Academy, ferramentas próprias).
- **Caderno** → conceitos ainda em esboço (Mercado Mútuo, Tempus Meum,
  SardinhIA).
- **Telefone** → contato (e-mail, WhatsApp, LinkedIn, Behance, Medium,
  Figma).
- **Caneca de café** → apresentação pessoal / soft skills.
- **Post-its** → reflexões e to-dos.

Interação: câmera fixa em visão geral, clique num objeto faz a câmera
aproximar (dolly) e um painel 2D com o conteúdo aparece por cima da cena.
Micro-interações por objeto: telefone "treme" no hover, caneca solta
fumaça, post-its balançam sozinhos.

## O que foi tentado, em ordem

### v1 — geometria estilizada (o arquivo preservado aqui)

Objetos construídos com primitivas do Three.js (`BoxGeometry`,
`CylinderGeometry`, `TorusGeometry` etc.), materiais `MeshStandardMaterial`
com `flatShading`, cores sólidas sem textura. **Esta é a versão que o
usuário aprovou como base** ("gostei, mas...") antes de pedir uma direção
visual mais refinada.

### v2 — tentativa de acabamento "Pixar" (formas orgânicas)

Substituição de `BoxGeometry` por uma função própria de "caixa arredondada"
(via `THREE.Shape` + `ExtrudeGeometry` com bevel, já que o addon
`RoundedBoxGeometry` dependia de um CDN bloqueado — ver seção de riscos
técnicos abaixo), desligamento de `flatShading`, redesenho do celular,
composição mais compacta. **Rejeitada** — feedback do usuário: "Esse já
ficou bem ruim! [...] os elementos ficaram muito espalhados [...] o celular
mesmo nem parece um celular". A tentativa de suavizar a geometria à mão,
sem um artista 3D ou ferramenta de modelagem, não chegou a um resultado
visualmente convincente.

### v3 — tentativa de realismo com assets prontos (CC0)

Pesquisa e download de modelos 3D reais e gratuitos (licença CC0, sem
exigência de atribuição) via a API pública do
[Poly Haven](https://polyhaven.com) para mesa, caderno, post-its e planta,
embutidos no HTML como glTF com texturas em base64 (arquivo final de
~10.7MB). Monitor, celular e caneca permaneceram como geometria por código,
porque **não existe modelo CC0 real e utilizável de celular ou monitor
moderno nos bancos gratuitos pesquisados** — são objetos de design
registrado, e o Sketchfab (que teria mais opções) exige login/token de API
para download até em modelos CC0, o que não foi possível automatizar neste
ambiente. **Rejeitada** — feedback do usuário: "Credo! precisamos melhorar
muito isso [...] Essas duas últimas ficaram bem ruins". O resultado
híbrido (parte com textura fotográfica real, parte geometria simples)
não ficou coeso.

## Decisão

O usuário pediu para pausar a ideia da mesa 3D e voltar ao formato atual do
site (institucional convencional). Este diretório arquiva o código da v1
(única versão aprovada) para referência futura, mas **não deve ser
publicado nem linkado do site em produção**.

## Riscos e limitações técnicas reais, para quem retomar isso depois

Estes são fatos verificados durante a tentativa, não suposições — vale
economizar tempo relendo isto antes de tentar de novo:

1. **CDNs externos de JS não funcionam dentro do sandbox de Artifacts.**
   O CSP do ambiente bloqueia scripts de `unpkg.com`/`jsdelivr.net`, então
   qualquer `<script type="importmap">` apontando para um CDN de Three.js
   trava a cena para sempre em "carregando", sem erro visível no fluxo
   normal. A correção que funcionou: embutir o código-fonte do Three.js
   (e de qualquer addon usado) como texto inerte dentro de um
   `<script type="application/octet-stream">`, e no boot da página
   transformá-lo em `Blob` + `URL.createObjectURL()`, resolvendo os
   imports do módulo principal para essa blob URL antes de executá-lo.
   Isso funciona porque blob URLs são same-origin, sem requisição de rede.

2. **Nem todo addon do Three.js pode ser embutido sem esforço.** O
   `OutlinePass` (usado para o contorno de hover) depende de uma cadeia de
   outros arquivos do pacote `examples/jsm/postprocessing/` e
   `examples/jsm/shaders/` — um deles (`ConvolutionShader.js`) não existia
   no caminho esperado da versão testada (r160), quebrando a árvore de
   dependência. Solução adotada: abandonar o outline shader e usar
   feedback de hover mais simples (bounce de escala), que só depende do
   core do Three.js.

3. **Assets 3D gratuitos e sem login são mais raros do que parecem.** Poly
   Haven tem bom catálogo de mobília/objetos genéricos (mesa, caderno,
   post-its, luminária, planta) e é 100% aberto (nenhuma autenticação para
   baixar). Mas não tem monitores nem celulares. O Sketchfab tem catálogo
   maior e uma boa busca por licença CC0, mas **exige autenticação até
   para baixar modelos CC0** — o endpoint de download retorna
   `401 Authentication credentials were not provided` sem login, o que
   inviabiliza automação sem um fluxo OAuth interativo.

4. **Orçamento de tamanho do Artifact (16MB) aperta rápido com assets
   reais.** Cada modelo glTF em resolução 1k (a menor disponível) com
   texturas difusas + normal + AO/roughness já fica entre 1.3MB e 2.7MB
   depois de inlineado em base64 (que adiciona ~33-37% sobre o tamanho
   binário original). Quatro modelos + o Three.js core já consomem
   ~10-11MB, deixando pouca margem para mais objetos ou para versões de
   textura maiores.

5. **"Baixo-poli estilizado, mas reconhecível" é genuinamente difícil sem
   um 3D artist ou ferramenta de modelagem (Blender etc.).** Ajustar
   proporções manualmente via código (constantes de largura/altura/raio de
   bevel) até algo parecer "bonito" e não "genérico" é um processo de
   tentativa e erro visual que não convergiu nas duas rodadas tentadas
   aqui — vale considerar isso como um custo real antes de tentar de novo
   por essa via.
