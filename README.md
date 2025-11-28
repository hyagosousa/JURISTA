<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Sistema de Empréstimos</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f5f5f5;
            padding: 20px;
        }

        .container {
            background: white;
            padding: 20px;
            margin-bottom: 25px;
            border-radius: 10px;
            box-shadow: 0 0 10px #ddd;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        table th, table td {
            border: 1px solid #ccc;
            padding: 8px;
        }

        button {
            padding: 8px 12px;
            border: none;
            background: #007bff;
            color: white;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover { opacity: 0.8; }

        .pago {
            background: #28a745 !important;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>Cadastro de Empréstimo</h2>

        <label>Cliente:</label>
        <input type="text" id="nome"><br><br>

        <label>Valor:</label>
        <input type="number" id="valor"><br><br>

        <label>Juros (%):</label>
        <input type="number" id="juros"><br><br>

        <button onclick="adicionarCliente()">Adicionar</button>
    </div>

    <div class="container">
        <h2>Clientes</h2>

        <table>
            <thead>
                <tr>
                    <th>Cliente</th>
                    <th>Valor</th>
                    <th>Juros</th>
                    <th>Total</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody id="listaClientes"></tbody>
        </table>
    </div>

    <!-- ===================== CONTROLE DE CAIXA ===================== -->

    <div class="container">
        <h2>Controle de Caixa</h2>

        <p><strong>Total Recebido:</strong> <span id="totalCaixa">R$ 0,00</span></p>

        <table>
            <thead>
                <tr>
                    <th>Data</th>
                    <th>Cliente</th>
                    <th>Valor Recebido</th>
                </tr>
            </thead>
            <tbody id="tabelaCaixa"></tbody>
        </table>
    </div>

    <script>
        let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
        let caixa = JSON.parse(localStorage.getItem("caixa") || "[]");

        renderClientes();
        renderCaixa();

        function formatarMoeda(v) {
            return v.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
        }

        function adicionarCliente() {
            let nome = document.getElementById("nome").value;
            let valor = Number(document.getElementById("valor").value);
            let juros = Number(document.getElementById("juros").value);

            if (!nome || !valor || !juros) {
                alert("Preencha todos os campos!");
                return;
            }

            let valorFinal = valor + (valor * (juros / 100));

            clientes.push({
                nome,
                valor,
                juros,
                valorFinal,
                pago: false
            });

            salvarClientes();
            renderClientes();
        }

        function salvarClientes() {
            localStorage.setItem("clientes", JSON.stringify(clientes));
        }

        function renderClientes() {
            let lista = document.getElementById("listaClientes");
            lista.innerHTML = "";

            clientes.forEach((c, i) => {
                lista.innerHTML += `
                    <tr>
                        <td>${c.nome}</td>
                        <td>${formatarMoeda(c.valor)}</td>
                        <td>${c.juros}%</td>
                        <td>${formatarMoeda(c.valorFinal)}</td>
                        <td>
                            <button class="${c.pago ? "pago" : ""}" onclick="marcarPago(${i})">
                                ${c.pago ? "Pago" : "Pagar"}
                            </button>
                        </td>
                    </tr>
                `;
            });
        }

        // ==================== CONTROLE DE PAGAMENTO ====================

        function marcarPago(i) {
            if (!clientes[i].pago) {
                registrarCaixa(clientes[i].nome, clientes[i].valorFinal);
            }

            clientes[i].pago = true;
            salvarClientes();
            renderClientes();
        }

        // ==================== CONTROLE DE CAIXA ====================

        function registrarCaixa(nome, valor) {
            caixa.push({
                data: new Date().toLocaleString(),
                nome,
                valor
            });

            localStorage.setItem("caixa", JSON.stringify(caixa));
            renderCaixa();
        }

        function renderCaixa() {
            let tabela = document.getElementById("tabelaCaixa");
            let total = 0;

            tabela.innerHTML = "";

            caixa.forEach(item => {
                total += item.valor;

                tabela.innerHTML += `
                    <tr>
                        <td>${item.data}</td>
                        <td>${item.nome}</td>
                        <td>${formatarMoeda(item.valor)}</td>
                    </tr>
                `;
            });

            document.getElementById("totalCaixa").innerText = formatarMoeda(total);
        }
    </script>
</body>
</html>

