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
    .resultado p, .dashboard p { margin: 5px 0; font-weight: bold; }
    .valor-emp { color: red; font-weight: bold; }
    .valor-juros { color: blue; font-weight: bold; }
    .valor-total { color: green; font-weight: bold; }
    table { width: 100%; border-collapse: collapse; display: block; overflow-x: auto; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; white-space: nowrap; }
    th { background: #ddd; cursor: pointer; }
    .historico { margin-top: 20px; }

    .piscar { animation: piscarAnim 1s infinite; }
    @keyframes piscarAnim {
        0% { background-color: #ffb3b3; }
        50% { background-color: #fff; }
        100% { background-color: #ffb3b3; }
    }

    @media(max-width: 600px){ input, select { font-size: 14px; padding: 10px; } button { font-size: 14px; padding: 10px; } td, th { font-size: 12px; padding: 6px; } }
</style>
</head>
<body>

<script>
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div style='max-width:400px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003;'>
        <h2 style='text-align:center;'>Login</h2>
        <input id='loginUser' placeholder='Login' style='width:100%;padding:12px;margin-top:10px;'>
        <input id='loginPass' type='password' placeholder='Senha' style='width:100%;padding:12px;margin-top:10px;'>
        <button id='btnEntrar' style='width:100%;padding:14px;margin-top:18px;background:#007bff;color:white;border:none;border-radius:6px;'>Entrar</button>
    </div>`;

    document.getElementById("btnEntrar").onclick = () => {
        let user = document.getElementById("loginUser").value.trim();
        let pass = document.getElementById("loginPass").value.trim();
        if (user === "H07y0321" && pass === "Helo2020@") {
            localStorage.setItem("logado", "true");
            location.reload();
        } else { alert("Login ou senha incorretos!"); }
    };
}
</script>

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
    <input type="number" id="valor" step="0.01" oninput="atualizarCalculo()">

    <label>Porcentagem de Juros (%):</label>
    <input type="number" id="juros" step="0.1" oninput="atualizarCalculo()">

    <label>Data do Empréstimo:</label>
    <input type="date" id="dataEmp">

    <label>Data de Vencimento:</label>
    <input type="date" id="dataVenc">

    <div class="resultado">
        <p>Valor: <span id="prevValor" class="valor-emp">R$ 0,00</span></p>
        <p>Juros: <span id="prevJuros" class="valor-juros">R$ 0,00</span></p>
        <p>Total a Receber: <span id="prevFinal" class="valor-total">R$ 0,00</span></p>
    </div>

    <button style="background:#28a745;color:white;" onclick="salvarCliente()">Salvar Cliente</button>
</div>

<div class="container dashboard">
    <h3>Resumo Geral</h3>
    <p>Total de Clientes: <span id="totalClientes">0</span></p>
    <p>Clientes Pagos: <span id="totalPagos">0</span></p>
    <p>Clientes Pendentes: <span id="totalPendentes">0</span></p>
    <p>Clientes Atrasados: <span id="totalAtrasados">0</span></p>
    <p>Valor Total Emprestado: <span id="totalEmpGeral" class="valor-emp">R$ 0,00</span></p>
    <p>Total em Juros: <span id="totalJurosGeral" class="valor-juros">R$ 0,00</span></p>
    <p>Total a Receber: <span id="totalReceberGeral" class="valor-total">R$ 0,00</span></p>

    <label>Buscar Cliente (Nome ou CPF):</label>
    <input type="text" id="buscar" oninput="atualizarTabela()">

    <label>Filtrar por Status:</label>
    <select id="filtroStatus" onchange="atualizarTabela()">
        <option value="todos">Todos</option>
        <option value="pendente">Pendentes</option>
        <option value="pago">Pagos</option>
        <option value="atrasado">Atrasados</option>
    </select>

    <button style="background:#ffc107;color:black;" onclick="exportarPDF()">Exportar PDF</button>
    <button style="background:#17a2b8;color:white;" onclick="enviarAlertaAtrasados()">📩 Enviar alerta para atrasados</button>
</div>

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

<script>
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let ordemAtual = '';
atualizarTabela();
calcularTotais();

function formatarMoeda(valor){ return valor.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' }); }
function atualizarCalculo(){
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor * juros)/100;
    let vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}

function salvarCliente(){
    let nome = document.getElementById("nome").value.trim();
    let cpf = document.getElementById("cpf").value.trim();
    let telefone = document.getElementById("telefone").value.trim();
    let endereco = document.getElementById("endereco").value.trim();
    let valor = parseFloat(document.getElementById("valor").value) || 0;
    let juros = parseFloat(document.getElementById("juros").value) || 0;
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;
    if(!nome || !cpf || !telefone || valor <= 0 || !dataEmp || !dataVenc){ alert('Preencha todos os campos corretamente!'); return; }
    let valorJuros = (valor * juros)/100;
    let valorFinal = valor + valorJuros;
    clientes.push({nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc, pago:false});
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela(); calcularTotais(); alert("Cliente salvo com sucesso!");
}

function excluirCliente(i){ clientes.splice(i,1); localStorage.setItem("clientes", JSON.stringify(clientes)); atualizarTabela(); calcularTotais(); }
function marcarPago(i){ clientes[i].pago = true; localStorage.setItem("clientes", JSON.stringify(clientes)); atualizarTabela(); calcularTotais(); }
function cobrar(telefone, valorFinal, dataVenc, nome){
    let valorFormatado = valorFinal.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
    let msg = `Opa ${nome}, sua dívida de ${valorFormatado} vence em ${dataVenc}. Por favor, realize o pagamento via PIX (chave: Hyagosousasous@gmail.com). Obrigado!`;
    window.open(`https://wa.me/55${telefone}?text=${encodeURIComponent(msg)}`, "_blank");
}

function enviarAlertaAtrasados(){
    let hoje = new Date();
    let atrasados = clientes.filter(c => !c.pago && c.dataVenc && new Date(c.dataVenc) < hoje);
    if(atrasados.length === 0){ alert("Não há clientes atrasados no momento!"); return; }
    atrasados.forEach(c => { cobrar(c.telefone, c.valorFinal, c.dataVenc, c.nome); });
}

function calcularTotais(){
    let totalEmp=0, totalJuros=0, totalFinal=0;
    let pagos=0, pendentes=0, atrasados=0;
    let hoje = new Date();
    clientes.forEach(c=>{
        if(c.pago) pagos++;
        else { pendentes++; if(c.dataVenc && new Date(c.dataVenc)<hoje) atrasados++; totalEmp+=c.valor; totalJuros+=c.valorJuros; totalFinal+=c.valorFinal; }
    });
    document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalFinal);
    document.getElementById("totalClientes").innerText = clientes.length;
    document.getElementById("totalPagos").innerText = pagos;
    document.getElementById("totalPendentes").innerText = pendentes;
    document.getElementById("totalAtrasados").innerText = atrasados;
}

function atualizarTabela(){
    let tbody=document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML="";
    let busca = document.getElementById('buscar').value.toLowerCase();
    let filtro = document.getElementById('filtroStatus').value;
    clientes.forEach((c,i)=>{
        let atrasado=false;
        let hoje=new Date();
        if(c.dataVenc && new Date(c.dataVenc)<hoje && !c.pago) atrasado=true;
        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        if(busca && !(c.nome.toLowerCase().includes(busca) || c.cpf.toLowerCase().includes(busca))) return;
        if(filtro==='pendente' && (c.pago || atrasado)) return;
        if(filtro==='pago' && !c.pago) return;
        if(filtro==='atrasado' && !atrasado) return;
        let tr = document.createElement('tr');
        tr.style.backgroundColor = corLinha;
        if(atrasado && !c.pago) tr.classList.add('piscar');
        tr.innerHTML = `
            <td>${c.nome}</td>
            <td class="valor-emp">${formatarMoeda(c.valor)}</td>
            <td class="valor-juros">${formatarMoeda(c.valorJuros)}</td>
            <td class="valor-total">${formatarMoeda(c.valorFinal)}</td>
            <td>${c.dataVenc||'-'}</td>
            <td>
                <button onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${c.nome}')">💰 Cobrar</button>
                <button onclick="excluirCliente(${i})" style="background:red;color:white; margin-top:5px;">❌ Excluir</button>
                ${!c.pago ? `<button onclick="marcarPago(${i})" style="background:green;color:white;margin-top:5px;">✅ Pago</button>` : ''}
            </td>
        `;
        tbody.appendChild(tr);
    });
}

function ordenarTabela(campo){
    if(ordemAtual===campo){ clientes.reverse(); }
    else { clientes.sort((a,b)=>{ if(a[campo]<b[campo]) return -1; if(a[campo]>b[campo]) return 1; return 0; }); ordemAtual=campo; }
    atualizarTabela();
}

function exportarPDF(){
    let conteudo = document.querySelector('.historico').innerHTML;
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório de Clientes</title></head><body>'+conteudo+'</body></html>');
    win.document.close();
    win.print();
}
</script>

</body>
</html>
