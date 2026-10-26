# Entregas e Retiradas

Protótipo navegável de um módulo de logística: quem precisa levar ou buscar material (obras, manutenção, administrativo) cadastra uma **viagem** com as paradas, e o motorista executa a corrida pelo celular, com roteiro do dia e mapa.

**Demonstração:** https://erickea888.github.io/entregas-e-retiradas/

> Todos os dados exibidos são **fictícios**, criados só para demonstrar o fluxo. Não há backend: o estado vive na página e some ao recarregar.

## O problema

Pedidos de entrega e retirada costumam chegar ao motorista por mensagem, ligação ou papel: falta o endereço completo, ninguém sabe a ordem das paradas, o que deve ser feito em cada local nem em que pé está cada corrida. Este módulo concentra tudo em um lugar: a viagem tem endereços completos, instruções e anexos por parada, status claro e um roteiro do dia que mostra ao motorista a sequência e o caminho.

## O que o protótipo faz

- **Nova viagem em etapas:**
  1. **Dados da viagem:** solicitante, setor (Obras, Manutenção, Administrativo), data, nº do projeto (opcional) e responsável pela viagem. O botão "Continuar" só avança com os campos obrigatórios preenchidos.
  2. **Retirada:** endereço completo. Ao digitar o **CEP**, endereço, bairro, cidade e estado são preenchidos sozinhos; só o número fica manual.
  3. **Destino:** os mesmos campos, com os botões **Adicionar outra parada** e **Concluir**.
- **Várias paradas por viagem:** a primeira é a retirada, a última é o destino final e as do meio são paradas intermediárias, para o motorista passar por vários locais na mesma corrida.
- **Instruções e anexos em cada parada** (o que fazer, ligar antes, pedido de compra, fotos), que o motorista vê ao abrir a corrida. Anexos múltiplos, até 5 MB por arquivo.
- **Roteiro do Dia:** escolhendo data e motorista, monta a sequência de retiradas e entregas com horário estimado a partir das 08:00, desenha a rota no mapa e avisa quando uma parada passa do expediente, com opção de mover a viagem para o dia seguinte.
- **Fluxo de status:** Pendente → Em rota → Concluída, com histórico de quem mudou e quando.
- **Duas visões:** tabela e Kanban com arrastar e soltar entre os status.
- **Detalhe da viagem** em gaveta lateral, com o percurso completo, instruções, anexos e histórico.
- **Editar, duplicar e excluir** viagens, com confirmação, pelo menu "⋮" de cada linha ou card.
- **Filtros** por status, setor e responsável, busca livre (solicitante, projeto, origem, destino) e tema claro/escuro.
- **Pensado para o celular:** o motorista é o usuário principal, então os botões têm toque grande e os campos não provocam zoom no iPhone.

## O que ainda é protótipo

| Hoje | Para produção |
| --- | --- |
| Sem login: o usuário se identifica por um campo de nome | Login por e-mail e senha (Firebase Authentication) |
| Dados fixos de exemplo na própria página | Banco compartilhado (Cloud Firestore) |
| Anexos em memória, somem ao recarregar | Armazenamento de arquivos (Cloud Storage) |
| Mapa, rota e CEP por serviços públicos e gratuitos de demonstração | Serviço contratado de mapa e roteirização, com chave de API |
| Rota sem trânsito em tempo real e base fixa do motorista | Rota com trânsito e localização real do motorista |

## Decisões técnicas

- **Arquivo único em HTML, CSS e JavaScript puros**, sem build. Abre com duplo clique e publica em qualquer hospedagem estática. A única biblioteca externa é o **Leaflet**, para o mapa.
- **CEP pelo ViaCEP**, com a AwesomeAPI como alternativa quando o primeiro falha; o **Nominatim** (OpenStreetMap) completa as coordenadas quando o CEP não as traz.
- **Rota pelo OSRM**, serviço público de roteirização pelas ruas. Se ele estiver fora do ar, a página cai sozinha para uma estimativa em linha reta, e a demonstração continua funcionando.
- **Dados de exemplo determinísticos** (sem sorteio), para a demonstração ser idêntica a cada carregamento.
- **Mesmo visual da [Requisição de Compras](https://github.com/erickea888/requisicao-de-compras)** (cores, cartões, gaveta de detalhe), para os módulos parecerem um só sistema.

## Como rodar

Baixe o `index.html` e abra no navegador. Não há instalação. É preciso internet para o mapa, a rota e a busca de CEP.

Teste rápido: crie uma viagem (use um CEP real, por exemplo 01310-100), adicione uma parada intermediária, conclua e abra o Roteiro do Dia.

## Próximos passos

- Tela **"Minhas corridas"** para o motorista, com a próxima parada em destaque e botões grandes: abrir no Waze ou Google Maps, ligar, marcar chegada e concluída.
- Comprovação da entrega por foto ou assinatura.
- Notificação ao solicitante quando a viagem mudar de status.
- Revisão de segurança, no mesmo padrão da Requisição de Compras.
- Login e banco de dados reais (Firebase).

## Autor

Erick Rodrigues · [github.com/erickea888](https://github.com/erickea888)
