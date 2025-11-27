<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema Financeiro — Hyago</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #111;
    color: #fff;
    margin: 0;
    padding: 0;
}

header {
    background: #222;
    padding: 15px;
    text-align: center;
    font-size: 22px;
    font-weight: bold;
}

nav {
    display: flex;
    justify-content: center;
    gap: 10px;
    background: #1a1a1a;
    padding: 10px;
}

nav button {
    padding: 10px 15px;
    border: none;
    background: #444;
    color: white;
    border-radius: 6px;
    cursor: pointer;
}

nav button:hover {
    background: #666;
}

section {
    display: none;
    padding: 20px;
}

input, select {
    width: 100%;
    padding: 10px;
    margin-top: 5px;
    border-radius: 5px;
    border: none;
}

button {
    padding: 10px;
    width: 100%;
    margin-top: 10px;
    background: #0a84ff;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
}

button:hover {
    opacity: 0.8;
}

.table {
    width: 100%;
    margin-top: 20px;
    border-collapse: collapse;
}

.table th, .table td {
    padding: 8px;
    border-bottom: 1px solid #333;
}

.blue { color: #4aa3ff; font-weight: bold; }
.red { color: #ff4d4d; font-weight: bold; }
.green { color: #4dff4d; font-weight: bold; }
</style>

</head>
<body>

<header>Sistema Financeiro — Hyago</header>

<nav>
    <button onclick="showPage('cadastro')">Cadastro</button>
    <button onclick="showPage('clientes')">Clientes Registrados</button>
    <button onclick="showPage('resumo')">Resumo</button>
    <button onclick="showPage('caixa')">Caixa</button>
</nav>

<!-- CADASTRO -->
<section id="cadastro">
    <h2>Cadastro de Empréstimo</h2>

    <label>Nome do Cliente:</label>
    <input id="nomeCliente">

    <label>Valor Emprestado:</label>
    <input id="valorEmp" type="number">

    <label>Data do Empréstimo:</label>
    <input id="dataEmp" type="date">

    <button onclick="cadastrarEmprestimo()">Cadastrar</button>
</section>

<!-- CLIENTES REGISTRADOS -->
<section id="clientes">
    <h2>Clientes Registrados</h2>
    <table class="table" id="tabelaClientes"></table>
</section>

<!-- RESUMO -->
<section id="resumo">
    <h2>Resumo Geral</h2>
    <p id="resumoConteudo"></p>
</section>

<!-- CAIXA -->
<section id="caixa">
    <h2>Controle de Caixa</h2>

    <label>Tipo:</label>
    <select id="tipoLancamento">
        <option value="entrada">Entrada</option>
        <option value="saida">Saída</option>
    </select>

    <label>Valor:</label>
    <input id="valorLancamento" type="number">

    <label>Motivo:</label>
    <input id="motivoLancamento">

    <button onclick="adicionarLancamento()">Lançar no Caixa</button>

    <h3>Resumo do Caixa</h3>
    <p class="blue" id="entradasTotais"></p>
    <p class="red" id="saidasTotais"></p>
    <p class="green" id="saldoTotal"></p>

    <h3>Histórico Completo</h3>
    <table class="table" id="historicoCaixa"></table>
</section>

<script>
// ============================
// SISTEMA DE NAVEGAÇÃO
// ============================
function showPage(page) {
    document.querySelectorAll("section").forEach(sec => sec.style.display = "none");
    document.getElementById(page).style.display = "block";

    if (page === "clientes") atualizarTabelaClientes();
    if (page === "resumo") atualizarResumo();
    if (page === "caixa") atualizarCaixa();
}

// BANCO LOCAL
let emprestimos = [];
let caixa = [];

// ============================
// CADASTRAR EMPRÉSTIMO
// ============================
function cadastrarEmprestimo() {
    let nome = document.getElementById("nomeCliente").value;
    let valor = Number(document.getElementById("valorEmp").value);
    let data = document.getElementById("dataEmp").value;

    if (!nome || !valor) {
        alert("Preencha todos os campos!");
        return;
    }

    emprestimos.push({
        nome,
        valor,
        data,
        pago: false
    });

    alert("Empréstimo cadastrado!");
    document.getElementById("nomeCliente").value = "";
    document.getElementById("valorEmp").value = "";
    document.getElementById("dataEmp").value = "";
}

// ============================
// TABELA DE CLIENTES
// ============================
function atualizarTabelaClientes() {
    let tabela = document.getElementById("tabelaClientes");
    tabela.innerHTML = `
        <tr><th>Cliente</th><th>Valor</th><th>Status</th><th>Ação</th></tr>
    `;

    emprestimos.forEach((e, i) => {
        tabela.innerHTML += `
            <tr>
                <td>${e.nome}</td>
                <td>R$ ${e.valor.toFixed(2)}</td>
                <td>${e.pago ? "Pago" : "Em aberto"}</td>
                <td>
                    ${!e.pago ? `<button onclick="marcarPago(${i})">Pagar</button>` : ""}
                </td>
            </tr>
        `;
    });
}

function marcarPago(i) {
    emprestimos[i].pago = true;
    alert("Pagamento registrado! (Sem lançamento automático no caixa)");
    atualizarTabelaClientes();
}

// ============================
// RESUMO
// ============================
function atualizarResumo() {
    let totalEmp = emprestimos.reduce((acc, e) => acc + e.valor, 0);
    let totalReceb = emprestimos.filter(e => e.pago).reduce((acc, e) => acc + e.valor, 0);

    document.getElementById("resumoConteudo").innerHTML = `
        <p>Total Emprestado: <b>R$ ${totalEmp.toFixed(2)}</b></p>
        <p>Total Recebido: <b>R$ ${totalReceb.toFixed(2)}</b></p>
        <p>Total Em Aberto: <b>R$ ${(totalEmp - totalReceb).toFixed(2)}</b></p>
    `;
}

// ============================
// CAIXA MANUAL
// ============================
function adicionarLancamento() {
    let tipo = document.getElementById("tipoLancamento").value;
    let valor = Number(document.getElementById("valorLancamento").value);
    let motivo = document.getElementById("motivoLancamento").value;
    let data = new Date().toLocaleString();

    if (!valor || !motivo) {
        alert("Preencha todos os campos!");
        return;
    }

    caixa.push({ tipo, valor, motivo, data });

    document.getElementById("valorLancamento").value = "";
    document.getElementById("motivoLancamento").value = "";

    atualizarCaixa();
}

function atualizarCaixa() {
    let entradas = caixa.filter(c => c.tipo === "entrada").reduce((acc, c) => acc + c.valor, 0);
    let saidas = caixa.filter(c => c.tipo === "saida").reduce((acc, c) => acc + c.valor, 0);
    let saldo = entradas - saidas;

    document.getElementById("entradasTotais").innerHTML = `Entradas Totais: R$ ${entradas.toFixed(2)}`;
    document.getElementById("saidasTotais").innerHTML = `Retorno Investido: R$ ${saidas.toFixed(2)}`;
    document.getElementById("saldoTotal").innerHTML = `Lucro: R$ ${saldo.toFixed(2)}`;

    let tabela = document.getElementById("historicoCaixa");
    tabela.innerHTML = `
        <tr><th>Tipo</th><th>Valor</th><th>Motivo</th><th>Data</th></tr>
    `;

    caixa.forEach(c => {
        tabela.innerHTML += `
            <tr>
                <td>${c.tipo}</td>
                <td>R$ ${c.valor.toFixed(2)}</td>
                <td>${c.motivo}</td>
                <td>${c.data}</td>
            </tr>
        `;
    });
}

showPage('cadastro');
</script>

</body>
</html>


