```mermaid
sequenceDiagram
    %% Atores
    actor Vendedor as "Vendedor"
    actor Gerente  as "Gerente"

    %% Objetos de interface
    participant UIPedido   as "UI de Pedido"
    participant UIGerente  as "UI Gerencial"

    %% Serviços de controle
    participant ServicoPedido     as "Serviço de Pedido"
    participant ServicoEstoque    as "Serviço de Estoque"
    participant ServicoRelatorio  as "Serviço de Relatórios"

    %% Repositório de dados
    database BD as "Banco de Dados"

    %% 1 – Vendedor registra um novo pedido
    Vendedor ->> UIPedido        : registrarPedido()
    UIPedido ->> ServicoPedido   : criarPedido(dadosPedido)

    %% Verificação de estoque (iteração para cada item)
    loop para cada Item
        ServicoPedido  ->> ServicoEstoque : verificarEstoque(item)
        ServicoEstoque ->> BD             : SELECT estoque WHERE produto = item
        BD             -->> ServicoEstoque: nivelEstoque
        ServicoEstoque -->> ServicoPedido : estoqueOK / estoqueBaixo
    end

    alt Todos os itens em estoque
        ServicoPedido  ->> ServicoEstoque : reservarEstoque(itens)
        ServicoEstoque ->> BD             : UPDATE estoque – reservado
        ServicoPedido  ->> BD             : INSERT INTO pedidos
        ServicoPedido  -->> UIPedido      : confirmarPedido(idPedido)
        ServicoPedido  -->> UIGerente     : notificarNovoPedido(idPedido)  ++
    else Algum item em falta
        ServicoPedido  -->> UIPedido      : exibirFaltaEstoque()
    end

    %% 2 – Alerta automático de baixo estoque (assíncrono)
    ServicoEstoque -->> UIGerente : alertaReposicao(produto)  ##

    %% 3 – Gerente solicita relatório diário
    Gerente       ->> UIGerente       : abrirPainel()
    UIGerente     ->> ServicoRelatorio: solicitarRelatorioDiario()
    ServicoRelatorio ->> BD           : agregarVendas()
    BD            -->> ServicoRelatorio: dadosRelatorio
    ServicoRelatorio -->> UIGerente   : exibirRelatorio()

    %% 4 – Ciclo de vida de objetos
    note over ServicoPedido,BD
        «create» Entidade Pedido persistida no BD  
        «destroy» Objeto Pedido após commit
    end
```
