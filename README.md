<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos Avançado</title>
<style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 900px; margin: 20px auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    h2, h3 { text-align: center; }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input, select { width: 100%; padding: 12px; margin-top: 5px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; }
    button { width: 100%; padding: 12px; border: none; border-radius: 8px; margin-top: 10px; font-size: 16px; cursor: pointer; }
    button:hover { opacity: 0.9; }
    .resultado, .dashboard { background: #eee; padding: 15px; border-radius: 10px; margin-top: 20px; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; white-space: nowrap; }
    th { background: #ddd; cursor: pointer; }

    /* ---- ABAS ---- */
    .tabs { display: flex; justify-content: center; gap: 10px; margin-top: 20px; }
    .tab-btn {
        padding: 12px 20px;
        background: #007bff;
        color: white;
        border-radius: 6px;
        cursor: pointer;
        border: none;
    }
    .tab-btn.active { background: #0056b3; }

    .tab-content { display: none; }
    .tab-content.active { display: block; }

    .piscar { animation: piscarAnim 1s infinite; }
    @keyframes piscarAnim {
        0% { background-color: #ffb3b3; }
        50% { background-color: #fff; }
        100% { background-color: #ffb3b3; }
    }
</style>
</head>
<body>

<!-- LOGIN -->
<script>
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div style='max-width:400px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003;'>
        <h2 style='text-align:center;'>Login</h2>
        <input id='loginUser' placeholder='Login'>
        <input id='loginPass' type='password' placeholder='Senha'>
        <button id='btnEntrar' style="background:#007bff;color:white;">Entrar</button>
    </div>`;

    document.getElementById("btnEntrar").onclick = () => {
        let user = loginUser.value.trim();
        let pass = loginPass.value.trim();
        if (user === "H07y0321" && pass === "Helo2020@") {
            localStorage.setItem("logado", "true");
            location.reload();
        } else alert("Login ou senha incorretos!");
    };
}
</script>

<!-- ABAS -->
<div class="tabs">
    <button class="tab-btn active" onclick="trocarAba('cadastro')">Cadastro de Clientes</button>
    <button class="tab-btn" onclick="trocarAba('resumo')">Resumo Geral</button>
    <button class="tab-btn" onclick="trocarAba('clientes')">Clientes Cadastrados</button>
</div>

<!-- ABA 1 - CADASTRO -->
<div id="cadastro" class="tab-content active">
    <div class="container">
        <h2>Sistema de Empréstimos Avançado</h2>

        <label>Nome do Cliente:</label>
        <input type="text" id="nome">

        <label>CPF:</label>
        <input type="text" id="cpf">

        <label>Telefone:</label>
        <input type="text" id="telefone">

        <label>Endereço:</label>
        <input type="text" id="endereco">

        <label>Valor Emprestado:</label>
        <input type="number" id="valor" oninput="atualizarCalculo()">

        <label>Porcentagem de Juros (%):</label>
        <input type="number" id="juros" oninput="atualizarCalculo()">

        <label>Data do Empréstimo:</label>
        <input type="date" id="dataEmp">

        <label>Data de Vencimento:</label>
        <input type="date" id="dataVenc">

        <div class="resultado">
            <p>Valor: <span id="prevValor">R$ 0,00</span></p>
            <p>Juros: <span id="prevJuros">R$ 0,00</span></p>
            <p>Total a Receber: <span id="prevFinal">R$ 0,00</span></p>
        </div>

        <button style="background:#28a745;color:white;" onclick="salvarCliente()">Salvar Cliente</button>
    </div>
</div>

<!-- ABA 2 - RESUMO -->
<div id="resumo" class="tab-content">
    <div class="container dashboard">
        <h3>Resumo Geral</h3>

        <p>Total de Clientes: <span id="totalClientes">0</span></p>
        <p>Clientes Pagos: <span id="totalPagos">0</span></p>
        <p>Clientes Pendentes: <span id="totalPendentes">0</span></p>
        <p>Clientes Atrasados: <span id="totalAtrasados">0</span></p>

        <p>Valor Total Emprestado: <span id="totalEmpGeral">R$ 0,00</span></p>
        <p>Total em Juros: <span id="totalJurosGeral">R$ 0,00</span></p>
        <p>Total a Receber: <span id="totalReceberGeral">R$ 0,00</span></p>

        <label>Buscar Cliente:</label>
        <input type="text" id="buscar" oninput="atualizarTabela()">

        <label>Filtrar por Status:</label>
        <select id="filtroStatus" onchange="atualizarTabela()">
            <option value="todos">Todos</option>
            <option value="pendente">Pendentes</option>
            <option value="pago">Pagos</option>
            <option value="atrasado">Atrasados</option>
        </select>

        <button style="background:#ffc107;" onclick="exportarPDF()">Exportar PDF</button>
        <button style="background:#17a2b8;color:white;" onclick="enviarAlertaAtrasados()">📩 Enviar alerta</button>
    </div>
</div>

<!-- ABA 3 - CLIENTES -->
<div id="clientes" class="tab-content">
    <div class="container historico">
        <h3>Clientes Registrados</h3>
        <table id="tabelaClientes">
            <thead>
                <tr>
                    <th onclick="ordenarTabela('nome')">Nome</th>
                    <th onclick="ordenarTabela('valor')">Valor</th>
                    <th onclick="ordenarTabela('valorJuros')">Juros</th>
                    <th onclick="ordenarTabela('valorFinal')">Total</th>
                    <th onclick="ordenarTabela('dataVenc')">Vencimento</th>
                    <th>Ações</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>
</div>

<script>
/* ---- FUNÇÃO DAS ABAS ---- */
function trocarAba(aba) {
    document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(div => div.classList.remove('active'));

    event.target.classList.add('active');
    document.getElementById(aba).classList.add('active');
}

/* ---- RESTANTE DO SEU CÓDIGO ORIGINAL ---- */
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let ordemAtual = '';

function formatarMoeda(v){ return v.toLocaleString('pt-BR',{style:'currency',currency:'BRL'}); }

function atualizarCalculo(){
    let valor = parseFloat(valorInput.value) || 0;
    let juros = parseFloat(jurosInput.value) || 0;
}

function salvarCliente(){ /* (...) seu código aqui */ }
function calcularTotais(){ /* (...) */ }
function atualizarTabela(){ /* (...) */ }
function ordenarTabela(campo){ /* (...) */ }
function excluirCliente(i){ /* (...) */ }
function marcarPago(i){ /* (...) */ }
function cobrar(){ /* (...) */ }
function enviarAlertaAtrasados(){ /* (...) */ }
function exportarPDF(){ /* (...) */ }

/* Inicializa */
atualizarTabela();
calcularTotais();
</script>

</body>
</html>

