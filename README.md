<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema — Hyago Sousa</title>

<style>
body {
    background: #111;
    color: #fff;
    font-family: Arial;
    margin: 0;
    padding: 0;
}

.container {
    background: #1a1a1a;
    padding: 20px;
    margin: 15px;
    border-radius: 10px;
}

button {
    background: #007bff;
    padding: 10px;
    border: none;
    color: white;
    border-radius: 8px;
    cursor: pointer;
    margin-top: 10px;
}

button.pago {
    background: #25d366;
}

table {
    width: 100%;
    border-collapse: collapse;
    background: #222;
}

table th, table td {
    padding: 10px;
    border: 1px solid #333;
}
</style>

</head>
<body>

<div class="container">
    <h2>Cadastro de Clientes</h2>

    <label>Nome:</label>
    <input id="nome" type="text">

    <label>Valor Total:</label>
    <input id="valor" type="number">

    <button onclick="adicionarCliente()">Salvar Cliente</button>
</div>

<div class="container">
    <h2>Clientes Cadastrados</h2>

    <table id="tabelaClientes">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Valor</th>
                <th>Status</th>
                <th>Pago</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
</div>

<!-- RESUMO GERAL -->
<div class="container">
    <h2>Resumo Geral</h2>

    <p>Entradas Totais (Caixa): <span id="totalEntradaCaixa">R$ 0,00</span></p>
    <p>Valor Investido (Saídas): <span id="totalSaidaCaixa">R$ 0,00</span></p>
    <p>Lucro Total: <span id="lucroCaixa">R$ 0,00</span></p>
</div>

<!-- CONTROLE DE CAIXA -->
<div class="container">
    <h2>Controle de Caixa</h2>

    <label>Tipo:</label>
    <select id="cxTipo">
        <option value="entrada">Entrada</option>
        <option value="saida">Saída</option>
    </select>

    <label>Valor:</label>
    <input id="cxValor" type="number">

    <label>Descrição:</label>
    <input id="cxDesc" type="text">

    <button onclick="salvarCaixa()">Registrar</button>

    <h3>Histórico</h3>

    <table id="tabelaCaixa">
        <thead>
            <tr>
                <th>Tipo</th>
                <th>Valor</th>
                <th>Descrição</th>
                <th>Data/Hora</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
</div>


<script>
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let caixa = JSON.parse(localStorage.getItem("caixa") || "[]");

function formatar(valor){
    return Number(valor).toLocaleString("pt-BR", { style:"currency", currency:"BRL" });
}

function adicionarCliente(){
    let nome = document.getElementById("nome").value;
    let valor = Number(document.getElementById("valor").value);

    if(!nome || valor <= 0){
        alert("Preencha todos os campos!");
        return;
    }

    clientes.push({
        id: Date.now(),
        nome: nome,
        valor: valor,
        pago: false
    });

    localStorage.setItem("clientes", JSON.stringify(clientes));

    carregarClientes();
}

function carregarClientes(){
    let tbody = document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML = "";

    clientes.forEach((c, i)=>{
        let tr = document.createElement("tr");

        tr.innerHTML = `
            <td>${c.nome}</td>
            <td>${formatar(c.valor)}</td>
            <td>${c.pago ? "Pago" : "A receber"}</td>
            <td><button class="pago" onclick="marcarPago(${i})">PAGO</button></td>
        `;

        tbody.appendChild(tr);
    });
}

function marcarPago(i){
    if(clientes[i].pago){
        alert("Esse cliente já está pago!");
        return;
    }

    clientes[i].pago = true;
    localStorage.setItem("clientes", JSON.stringify(clientes));

    // VALOR RECEBIDO PELO CLIENTE
    let valorRecebido = clientes[i].valor;

    // LANÇAMENTO AUTOMÁTICO NO CAIXA
    caixa.push({
        tipo: "entrada",
        valor: valorRecebido,
        descricao: "Pagamento do cliente: " + clientes[i].nome,
        data: new Date().toLocaleString()
    });

    localStorage.setItem("caixa", JSON.stringify(caixa));

    atualizarCaixaResumo();
    atualizarTabelaCaixa();
    carregarClientes();

    alert("Pagamento registrado e entrada lançada no caixa!");
}

function salvarCaixa(){
    let tipo = document.getElementById("cxTipo").value;
    let valor = Number(document.getElementById("cxValor").value);
    let desc = document.getElementById("cxDesc").value;

    if(valor <= 0 || !desc){
        alert("Preencha todos os campos!");
        return;
    }

    caixa.push({
        tipo: tipo,
        valor: valor,
        descricao: desc,
        data: new Date().toLocaleString()
    });

    localStorage.setItem("caixa", JSON.stringify(caixa));

    atualizarCaixaResumo();
    atualizarTabelaCaixa();

    alert("Lançamento registrado!");
}

function atualizarCaixaResumo(){
    let entradas = 0;
    let saidas = 0;

    caixa.forEach(c=>{
        if(c.tipo === "entrada") entradas += c.valor;
        if(c.tipo === "saida")   saidas += c.valor;
    });

    document.getElementById("totalEntradaCaixa").innerText = formatar(entradas);
    document.getElementById("totalSaidaCaixa").innerText = formatar(saidas);
    document.getElementById("lucroCaixa").innerText = formatar(entradas - saidas);
}

function atualizarTabelaCaixa(){
    let tbody = document.querySelector("#tabelaCaixa tbody");
    tbody.innerHTML = "";

    caixa.forEach(r=>{
        let tr = document.createElement("tr");
        tr.innerHTML = `
            <td>${r.tipo}</td>
            <td>${formatar(r.valor)}</td>
            <td>${r.descricao}</td>
            <td>${r.data}</td>
        `;
        tbody.appendChild(tr);
    });
}

carregarClientes();
atualizarCaixaResumo();
atualizarTabelaCaixa();
</script>

</body>
</html>

