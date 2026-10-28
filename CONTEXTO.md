# CONTEXTO: Entregas e Retiradas

Leia este arquivo primeiro. Ele resume o estado do módulo e as decisões já tomadas, para continuar o trabalho sem recomeçar.

Autor: Erick Rodrigues (github.com/erickea888). Idioma do produto e da conversa: português do Brasil.

## O que é

Módulo de logística de um sistema de suprimentos: quem precisa de uma entrega ou retirada de material (obras, manutenção, administrativo) cadastra uma **viagem**, e o motorista executa. O motorista usa o **celular**, então o módulo deve ser pensado primeiro para tela pequena.

Este repositório é um protótipo para portfólio e para mostrar a um engenheiro de software. Ele é **autônomo**: um único `index.html` (HTML, CSS e JavaScript puros, sem build), com dados fictícios. Na implantação real vai ganhar login e banco de dados (Firebase foi a sugestão).

> **Repositório privado por enquanto.** A marca da empresa já foi removida, mas os dados de exemplo ainda precisam de revisão (veja "Pendências"). Só deixe público depois disso e da revisão de segurança.

## Estado atual (`index.html`, cerca de 1.550 linhas)

- **Nova viagem em uma tela só** (antes era um assistente em etapas): dados da viagem + cartão "Endereço de origem (retirada)" + "Endereço de entrega (destino)", com "Adicionar parada no caminho" entre os dois. O solicitante escreve os endereços; o CEP é **opcional** e só ajuda a preencher (ViaCEP, coordenadas pela AwesomeAPI). Obrigatórios: endereço, número, bairro, cidade e UF de cada parada; os que faltam ficam em vermelho ao clicar em "Criar viagem". Cada bloco (Dados, origem, paradas, entrega) recolhe pela seta ao lado do título e mostra um resumo de uma linha; um bloco recolhido com campo faltando abre sozinho ao criar (`definirRecolhidaNv`, estado em `wiz.recolhidas` e `wiz.dadosRecolhido`). A rolagem da página de trás fica travada com o formulário aberto. Estado: `wiz = {dados, paradas}`; funções `renderParadasForm`, `htmlParadaForm`, `buscarCepParada`, `validarFormularioEr`, `travarRolagemFundoEr`.
- **Painel de quem pede** (aba "Viagens"; o Roteiro do Dia virou a segunda aba): quadros de resumo clicáveis, status único `etapaSolicitante` (aguardando → aceita → em_rota → concluida) com barra de progresso, coluna Motorista, rota curta (bairros ou cidades), grupos por data (concluídas antigas recolhidas), botão "Minhas viagens" (compara com o nome em "Definir meu usuário") e alerta de responsável não cadastrado. No celular a lista vira cartões e o formulário ocupa a tela inteira.
- **Paradas:** a primeira é a retirada; a última endereço informado é o **destino final**; as do meio são **intermediárias** (o motorista passa por vários locais até chegar ao destino).
- **Roteiro do Dia:** mapa (Leaflet) com a rota entre as paradas (OSRM, serviço público de demonstração, com plano B quando falha) e os tiles do OpenStreetMap France. Geocodificação de apoio pelo Nominatim quando o CEP não traz coordenadas.
- **Minhas corridas (visão do motorista, celular primeiro):** "Entrar como" com os motoristas cadastrados (`MOTORISTAS_SEED`, login simulado). Depois de entrar, no celular some o topo do sistema e fica só uma barra (nome, Trocar, tema, Painel). No topo vem a **corrida atual** (em rota primeiro; `corridasSeparadasMc`) com bolinhas de progresso, endereço grande (`htmlEnderecoGrande`) e um botão **Navegar** que lembra o app escolhido (Google Maps ou Waze, `MAPA_APP_KEY`). Embaixo: Novas para aceitar, Próximas aceitas e Concluídas hoje. **Ordem das corridas = ordem do Roteiro do Dia** (dias mais antigos primeiro; no mesmo dia, a posição da corrida na rota de "vizinho mais próximo"), com o selo "Nª no roteiro"; a corrida em rota continua sendo a atual até terminar. Lógica compartilhada com o mapa: `viagensDoRoteiro`, `ordenarEtapasRoteiro`, `posicoesNoRoteiro`. A **ação principal fica fixa no rodapé** (`renderAcaoFixaMc`): Aceitar → Iniciar rota → Cheguei na retirada/parada/entrega → Concluir. "Cheguei" abre uma janela (`abrirConfirmacaoChegadaMc`) para confirmar **Material coletado / Parada feita / Entregue**; na entrega o **nome de quem recebeu** é obrigatório, com foto (reduzida para 1280 px) e observação opcionais. Isso fica em `parada.comprovante` e aparece no detalhe da viagem e no histórico. O aceite é um marcador (`v.aceitaEm`), não um status novo.
- Solicitante é texto livre obrigatório (sem lista de sugestões). O mapa do Roteiro fica numa camada isolada para não passar por cima das janelas.
- Tema claro/escuro, perfil do usuário (nome) em localStorage, dados de exemplo fixos (sem sorteio), com viagens de exemplo, inclusive uma com parada intermediária.
- Visual igual ao da Requisição de Compras (mesmas cores, cartões, gaveta de detalhe).

Funções e estados importantes: `LOCAIS_SEED`, `SETORES_ER`, `TIPO_PARADA_LABEL`, `ETAPA_UI`, `esc`, `tipoParada`, `rotuloParada`, `enderecoCompleto`, `paradasDaViagem`, `nIntermed`, `renderPercursoDrawer`, `geocodificarParada`; estado do wizard `wiz = {passo, dados, paradas}`; ids `wizVoltar`, `wizContinuar`, `wizAdicionar`, `wizConcluir`, `wizRemover`, `pCep`, `pNumero`, `pEndereco`, `pBairro`, `pCidade`, `pEstado`, `pInstrucoes`, `pAnexoInput`, `pAnexoLista`.

## Decisões já tomadas

- **Sem vínculo com requisição de compras.** Foi removido de propósito; a viagem não referencia uma requisição.
- **Regras da tela do motorista (confirmadas pelo autor em 26/10/2026):** nome em Responsável que bate com o cadastro é gravado como no cadastro; nome não cadastrado é aceito, mas não aparece para nenhum motorista. Trocar o Responsável ao editar apaga o aceite. Chegadas marcadas em ordem e só depois de Iniciar rota, com Desfazer na última. Concluir só depois de todas as paradas. Recusar corrida, botão Ligar e foto/assinatura ficaram para depois.
- **Permissões de status (definidas pelo autor em 26/10/2026):** pelo painel, só o **gestor geral da conta** muda o status, inclusive arrastando no Kanban (para os outros os cards não arrastam e aparece um aviso). Depois de **Em rota**, entende-se que o motorista está executando: só o **motorista da viagem** ou o gestor pode dar como **Concluída**. O gestor é marcado em "Definir meu usuário" (caixa "Sou gestor geral", chave `suprimentos-demo-entregas-usuario-gestor`); regras em `podeMudarStatusEr` e `motivoSemPermissaoEr`. **Editar:** quem pediu, enquanto pendente; o gestor sempre (`podeEditarEr`). **Cancelar:** quem pediu ou o gestor, enquanto pendente, com **motivo obrigatório**: status `cancelada`, dados em `v.cancelamento`, entra no histórico e o motorista vê o aviso em Minhas corridas (`podeCancelarEr`). **Excluir:** só o gestor (`podeExcluirEr`); quem pede usa Cancelar, que mantém o histórico. Os botões sem permissão nem aparecem.
- **Várias pessoas pedindo para o mesmo motorista (definido em 26/10/2026):** quem decide se leva junto é o **motorista** (carro cheio, peso, precisa de ajuda). (A) A Nova Viagem mostra a **agenda do motorista no dia** (corridas, paradas, km, término previsto, alerta se passar das 18:00; `estimarAgendaEr`). (B) Se ele já vai ao **mesmo endereço no mesmo dia**, o cartão do endereço avisa e diz que ele decide (`chaveEnderecoEr`). Campo opcional **Tamanho da carga** (`v.carga`). Na tela do motorista: selos de carga e "mesmo local"; três respostas para corrida nova: **Aceitar**, **Aceitar com ajudante** (`v.precisaAjudante`) e **Não consigo** com motivo pronto (`v.recusa`, histórico em `v.recusas`). A recusa não apaga nada: a viagem fica "Aguardando motorista" em vermelho no painel e no alerta "precisam de atenção"; qualquer edição (trocar motorista, data ou carga) devolve ao motorista. Recusadas saem do roteiro. (C) Corrida nova que chega com a tela aberta mostra aviso 🔔 no topo, sem trocar a corrida atual.
- Anexos funcionam como na Requisição de Compras (múltiplos, com limite de tamanho por arquivo).
- O motorista é o usuário final no celular: toque grande (44 px), campos de 16 px (evita zoom no iPhone), layout em cartões.
- Os sistemas de mapa e CEP são gratuitos e públicos, só para demonstração. Em produção, trocar por serviço contratado (rota e mapa com chave de API e contrato).

## Pendências (em ordem de prioridade)

1. **Confirmar se o mapa do Roteiro do Dia aparece** no navegador e no celular. Esse ponto ficou sem confirmação do usuário. Se os tiles bloquearem ("Access blocked"), usar outro provedor ou mapa estático.
2. **Revisão de segurança.** O código herdou os mesmos padrões da Requisição de Compras antes da correção: aplicar o mesmo tratamento (ver "Aprendizados" abaixo). Ponto de atenção: textos de usuário (solicitante, responsável, instruções, nome de arquivo) entram na página via `innerHTML`; escapar tudo na exibição, validar tipo de arquivo por extensão, limitar tamanho de texto, e adicionar CSP (liberando só `cdnjs.cloudflare.com`, `router.project-osrm.org`, `viacep.com.br`, `cep.awesomeapi.com.br`, `nominatim.openstreetmap.org` e os tiles).
3. **Mobile em primeiro lugar.** Revisar tudo no celular (iPhone e Android): wizard, lista de viagens, detalhe, mapa. Considerar uma tela própria "Minhas corridas" para o motorista, com a próxima parada em destaque e botões grandes (abrir no Waze/Google Maps, ligar, marcar chegada e concluída).
4. ~~Neutralizar a marca~~ **Feito:** o nome da empresa foi removido (marca "ER / Entregas / Protótipo de portfólio", chaves de localStorage `suprimentos-demo-entregas-...`, "Escritório central", menu lateral só com este módulo). Falta revisar se os endereços e nomes de exemplo (`LOCAIS_SEED`, viagens de exemplo) são todos fictícios.
5. **README.md** em português, no mesmo molde da Requisição de Compras: problema, o que faz, o que ainda é protótipo, decisões técnicas, como rodar, próximos passos, autor.
6. **SEGURANCA.md** com achados e checklist de implantação.
7. Idéias futuras: status da viagem pelo motorista (a caminho, coletado, entregue) com comprovação por foto/assinatura, histórico por viagem, notificação ao solicitante.

## Aprendizados do outro módulo (reaproveitar)

O repositório irmão `requisicao-de-compras` (github.com/erickea888/requisicao-de-compras) já passou por esses ajustes; copie as soluções:

- **Layout responsivo em arquivo único** com `@media (max-width:700px)`: tabela vira cartões em CSS grid, filtros recolhíveis, detalhe em tela cheia, alvos de toque de 44 px, campos de 16 px.
- **iPhone:** usar `dvh` em vez de `vh` (a barra do Safari cobria o botão do rodapé); travar a rolagem da página de trás enquanto um detalhe/janela está aberto (`body` com `position:fixed` e guardar a posição); `overscroll-behavior:contain` no corpo rolável; barra fixa de ação com `position:sticky; bottom:0`.
- **Segurança:** função `escapeHtml` (escapa `& < > " '`) usada na exibição de todo texto de usuário; `urlSegura` que só aceita `blob:`; lista de extensões permitidas conferida em JavaScript; `maxLength` nos campos; CSP por `<meta>`; sem escape na gravação (escapar só ao exibir, para não duplicar).
- **Testes sem navegador:** extrair o `<script>`, `node --check`, e rodar no jsdom com ataques de exemplo (`<img src=x onerror=...>` em todos os campos) conferindo que nada executa.
- **Geração de planilha .xlsx sem biblioteca** (zip em JavaScript puro) existe lá e pode servir se este módulo precisar exportar viagens.
- Dados de exemplo **determinísticos** (sem `Math.random`), para a demonstração ser igual a cada carregamento.

## Como rodar e testar

Abra o `index.html` no navegador (precisa de internet para o mapa, o CEP e a rota). Para publicar de graça: GitHub Pages (Settings → Pages → Deploy from a branch → main / root). Atenção: em repositório privado o Pages pode exigir plano pago; para mostrar ao engenheiro, enquanto estiver privado, basta ele clonar ou abrir o arquivo.

Teste básico: criar viagem pelo wizard (testar CEP real, p. ex. 01310-100), adicionar uma parada intermediária, concluir e abrir o Roteiro do Dia.

## Regras de trabalho com o autor

- Responder em português, com linguagem simples; o autor não é programador de formação.
- Antes de mudar visual, preservar as cores atuais (ele pediu isso explicitamente no outro módulo).
- Explicar escolhas em poucas linhas e mostrar o resultado; confirmar com ele pontos de produto (fluxo, regras) em vez de decidir sozinho.
