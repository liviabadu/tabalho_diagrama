sequenceDiagram
    %% Atores
    actor Vendedor
    actor Gerente

    %% Objetos / Serviços
    participant UI           as "UI de Pedido"
    participant PedidoSvc    as "Serviço de Pedido"
    participant EstoqueSvc   as "Serviço de Estoque"
    participant RelatorioSvc as "Serviço de Relatórios"
    database   BD            as "Banco de Dados"

    %% Fluxo principal de pedido
    Vendedor   ->> UI           : registrarPedido()
    UI         ->> PedidoSvc    : criarPedido(dados)
    PedidoSvc  ->> EstoqueSvc   : verificarEstoque(itens)
    EstoqueSvc ->> BD           : consultarEstoque()
    BD         -->> EstoqueSvc  : níveis
    EstoqueSvc -->> PedidoSvc   : estoqueOK
    PedidoSvc  ->> EstoqueSvc   : reservarEstoque()
    EstoqueSvc ->> BD           : atualizarEstoque()
    PedidoSvc  -->> UI          : confirmarPedido()

    %% Alerta de reposição
    EstoqueSvc -->> Gerente     : alertaReposicao(produto)

    %% Relatório diário
    Gerente    ->> RelatorioSvc : solicitarRelatorioDiario()
    RelatorioSvc ->> BD         : agregarVendas()
    BD         -->> RelatorioSvc: dados
    RelatorioSvc -->> Gerente   : exibirRelatorio()
