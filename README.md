<!-- PARTE 1/3 - INÍCIO DO ARQUIVO -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos Avançado + Caixa</title>
<style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 1000px; margin: 20px auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    h2, h3 { text-align: center; }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input, select, textarea { width: 100%; padding: 12px; margin-top: 5px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; box-sizing: border-box; }
    button { width: 100%; padding: 12px; border: none; border-radius: 8px; margin-top: 10px; font-size: 16px; cursor: pointer; }
    button:hover { opacity: 0.95; }
    .resultado, .dashboard, .caixa-panel { background: #eee; padding: 15px; border-radius: 10px; margin-top: 20px; }
    .resultado p, .dashboard p, .caixa-panel p { margin: 5px 0; font-weight: bold; }
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

    /* MENU */
    .menu { display: flex; gap:8px; justify-content: space-between; background: #222; padding: 10px; flex-wrap:wrap; }
    .menu button {
        flex: 1 1 22%;
        margin: 5px;
        padding: 12px;
        border: none;
        border-radius: 6px;
        background: #444;
        color: white;
        font-size: 15px;
        cursor: pointer;
    }
    .menu button.ativo { background: #007bff; }

    .aba { display: none; }
    .aba.ativa { display: block; }

    @media(max-width:800px){
      .menu button { flex:1 1 45%; font-size:14px; padding:10px; }
      .container { margin:10px; padding:12px; }
    }
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

<!-- MENU -->
<div class="menu">
    <button onclick="abrirAba('cadastro')" class="ativo" id="btnCadastro">Cadastro</button>
    <button onclick="abrirAba('resumo')" id="btnResumo">Resumo Geral</button>
    <button onclick="abrirAba('clientes')" id="btnClientes">Clientes Registrados</button>
    <button onclick="abrirAba('caixa')" id="btnCaixa">Controle de Caixa</button>
</div>

<script>
function abrirAba(aba) {
    document.querySelectorAll(".aba").forEach(a => a.classList.remove("ativa"));
    document.querySelectorAll(".menu button").forEach(b => b.classList.remove("ativo"));

    let btnId = "btn" + aba.charAt(0).toUpperCase() + aba.slice(1);
    let abaId = "aba_" + aba;
    let btn = document.getElementById(btnId);
    let el = document.getElementById(abaId);
    if(btn) btn.classList.add("ativo");
    if(el) el.classList.add("ativa");
}
</script>

<!-- ABA 1 - CADASTRO -->
<div class="container aba ativa" id="aba_cadastro">
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

<!-- ABA 2 - RESUMO GERAL -->
<div class="container aba" id="aba_resumo">
    <h3>Resumo Geral</h3>

    <div class="dashboard">
      <p>Total de Clientes: <span id="totalClientes">0</span></p>
      <p>Clientes Pagos: <span id="totalPagos">0</span></p>
      <p>Clientes Pendentes: <span id="totalPendentes">0</span></p>
      <p>Clientes Atrasados: <span id="totalAtrasados">0</span></p>
      <p>Valor Total Emprestado: <span id="totalEmpGeral" class="valor-emp">R$ 0,00</span></p>
      <p>Total em Juros: <span id="totalJurosGeral" class="valor-juros">R$ 0,00</span></p>
      <p>Total a Receber: <span id="totalReceberGeral" class="valor-total">R$ 0,00</span></p>
    </div>

    <label>Buscar Cliente (Nome ou CPF):</label>
    <input type="text" id="buscar" oninput="atualizarTabela()">

    <label>Filtrar por Status:</label>
    <select id="filtroStatus" onchange="atualizarTabela()">
        <option value="todos">Todos</option>
        <option value="pendente">Pendentes</option>
        <option value="pago">Pagos</option>
        <option value="atrasado">Atrasados</option>
    </select>

    <div style="display:flex;gap:10px;margin-top:10px;">
      <button style="background:#ffc107;color:black;flex:1;" onclick="exportarPDF()">Exportar PDF</button>
      <button style="background:#17a2b8;color:white;flex:1;" onclick="enviarAlertaAtrasados()">📩 Enviar alerta para atrasados</button>
    </div>
</div>
<!-- FIM PARTE 1/3 -->
<!-- PARTE 2/3 - continuação -->
<!-- ABA 3 – CLIENTES REGISTRADOS -->
<div class="container historico aba" id="aba_clientes">
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

<!-- ABA 4 – CONTROLE DE CAIXA -->
<div class="container aba" id="aba_caixa">
    <h3>Controle de Caixa</h3>

    <div class="caixa-panel">
        <p>Total de entradas: <span id="totalEntradas">R$ 0,00</span></p>
        <p>Total de saídas: <span id="totalSaidas">R$ 0,00</span></p>
        <p>Saldo atual (Lucro): <span id="saldoAtual" class="valor-total">R$ 0,00</span></p>
    </div>

    <h4>Lançar saída manual</h4>
    <label>Descrição da saída:</label>
    <input type="text" id="saidaDesc" placeholder="Ex: Compra de material">
    <label>Valor da saída:</label>
    <input type="number" id="saidaValor" step="0.01" placeholder="0.00">
    <button style="background:#dc3545;color:white;" onclick="registrarSaidaManual()">Registrar Saída</button>

    <h4 style="margin-top:20px;">Histórico do Caixa</h4>
    <table id="tabelaCaixa">
        <thead>
            <tr>
                <th>Data / Hora</th>
                <th>Tipo</th>
                <th>Cliente / Descrição</th>
                <th>Valor</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <div style="display:flex;gap:10px;margin-top:10px;">
      <button style="background:#ffc107;color:black;flex:1;" onclick="exportarCaixaPDF()">Exportar Caixa (PDF)</button>
      <button style="background:#6c757d;color:white;flex:1;" onclick="limparCaixa()">Limpar Caixa</button>
    </div>
</div>

<script>
// Recupera dados existentes
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let caixa = JSON.parse(localStorage.getItem("caixa") || "[]");
let ordemAtual = '';
atualizarTabela();
calcularTotais();
atualizarTabelaCaixa();
calcularTotaisCaixa();

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
    let valorFinal = parseFloat((valor + valorJuros).toFixed(2));
    clientes.push({nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc, pago:false});
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela(); calcularTotais(); alert("Cliente salvo com sucesso!");
}

// Excluir cliente (não afeta caixa)
function excluirCliente(i){ 
    if(!confirm("Excluir cliente? Esta ação não pode ser desfeita.")) return;
    clientes.splice(i,1); 
    localStorage.setItem("clientes", JSON.stringify(clientes)); 
    atualizarTabela(); calcularTotais(); 
}

// MARCAR PAGO -> agora registra ENTRADA no caixa automaticamente
function marcarPago(i){
    if(!clientes[i]) return;
    if(clientes[i].pago){ alert("Cliente já marcado como pago."); return; }
    clientes[i].pago = true; 
    localStorage.setItem("clientes", JSON.stringify(clientes)); 
    // registra entrada no caixa
    let registro = {
        cliente: clientes[i].nome,
        descricao: `Recebimento - Empréstimo vencido (${clientes[i].dataVenc || '-'})`,
        valor: parseFloat(clientes[i].valorFinal) || 0,
        tipo: 'entrada',
        data: new Date().toISOString()
    };
    caixa.push(registro);
    localStorage.setItem("caixa", JSON.stringify(caixa));
    atualizarTabela(); calcularTotais(); atualizarTabelaCaixa(); calcularTotaisCaixa();
    alert("Cliente marcado como pago e entrada registrada no caixa.");
}

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
        else { 
            pendentes++; 
            if(c.dataVenc && new Date(c.dataVenc)<hoje) atrasados++; 
            totalEmp+=c.valor; 
            totalJuros+=c.valorJuros; 
            totalFinal+=c.valorFinal; 
        }
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
    if(!tbody) return;
    tbody.innerHTML="";
    let busca = document.getElementById('buscar')?.value.toLowerCase() || "";
    let filtro = document.getElementById('filtroStatus')?.value || "todos";
    clientes.forEach((c,i)=>{
        let atrasado=false;
        let hoje=new Date();
        if(c.dataVenc && new Date(c.dataVenc)<hoje && !c.pago) atrasado=true;
        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        if(busca && !(c.nome.toLowerCase().includes(busca) || (c.cpf||'').toLowerCase().includes(busca))) return;
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
            <td style="min-width:160px;">
                <button onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${c.nome}')">💰 Cobrar</button>
                <button onclick="excluirCliente(${i})" style="background:red;color:white; margin-top:5px;">❌ Excluir</button>
                ${!c.pago ? `<button onclick="marcarPago(${i})" style="background:green;color:white;margin-top:5px;">✅ Pago</button>` : `<span style="display:inline-block;margin-top:6px;color:green;font-weight:bold;">Pago</span>`}
            </td>
        `;
        tbody.appendChild(tr);
    });
}
<!-- FIM PARTE 2/3 -->
<!-- PARTE 3/3 - FINAL -->
<script>
function ordenarTabela(campo){
    if(ordemAtual===campo){ clientes.reverse(); }
    else { clientes.sort((a,b)=>{ 
        if(a[campo] < b[campo]) return -1; 
        if(a[campo] > b[campo]) return 1; 
        return 0; 
    }); ordemAtual=campo; }
    atualizarTabela();
}

// EXPORTAR CLIENTES (PDF)
function exportarPDF(){
    let conteudo = document.querySelector('#aba_clientes').innerHTML;
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório de Clientes</title></head><body>'+conteudo+'</body></html>');
    win.document.close();
    win.print();
}

/* -------------------  FUNÇÕES DO CAIXA  ------------------- */
// registra entrada (usada quando marcarPago e pode ser usada manualmente se quiser)
function registrarEntradaManual(cliente, descricao, valor){
    let v = parseFloat(valor) || 0;
    if(v <= 0){ alert('Valor inválido para entrada'); return; }
    let registro = {
        cliente: cliente || 'Entrada manual',
        descricao: descricao || 'Entrada',
        valor: v,
        tipo: 'entrada',
        data: new Date().toISOString()
    };
    caixa.push(registro);
    localStorage.setItem("caixa", JSON.stringify(caixa));
    atualizarTabelaCaixa(); calcularTotaisCaixa();
    alert('Entrada registrada no caixa.');
}

function registrarSaidaManual(){
    let desc = document.getElementById('saidaDesc').value.trim();
    let valor = parseFloat(document.getElementById('saidaValor').value) || 0;
    if(!desc){ alert('Descreva a saída'); return; }
    if(valor <= 0){ alert('Informe um valor válido'); return; }
    let registro = {
        cliente: null,
        descricao: desc,
        valor: valor,
        tipo: 'saida',
        data: new Date().toISOString()
    };
    caixa.push(registro);
    localStorage.setItem("caixa", JSON.stringify(caixa));
    document.getElementById('saidaDesc').value = '';
    document.getElementById('saidaValor').value = '';
    atualizarTabelaCaixa(); calcularTotaisCaixa();
    alert('Saída registrada no caixa.');
}

function calcularTotaisCaixa(){
    let totalEntradas = 0, totalSaidas = 0;
    caixa.forEach(r=>{
        if(r.tipo === 'entrada') totalEntradas += parseFloat(r.valor) || 0;
        if(r.tipo === 'saida') totalSaidas += parseFloat(r.valor) || 0;
    });
    let saldo = totalEntradas - totalSaidas;
    document.getElementById('totalEntradas').innerText = formatarMoeda(totalEntradas);
    document.getElementById('totalSaidas').innerText = formatarMoeda(totalSaidas);
    document.getElementById('saldoAtual').innerText = formatarMoeda(saldo);
}

function atualizarTabelaCaixa(){
    let tbody = document.querySelector('#tabelaCaixa tbody');
    if(!tbody) return;
    tbody.innerHTML = '';
    // exibimos do mais recente para o mais antigo
    let lista = [...caixa].reverse();
    lista.forEach(r=>{
        let tr = document.createElement('tr');
        let data = new Date(r.data);
        let dataStr = data.toLocaleString('pt-BR');
        tr.innerHTML = `
            <td>${dataStr}</td>
            <td>${r.tipo === 'entrada' ? 'Entrada' : 'Saída'}</td>
            <td>${r.tipo === 'entrada' ? (r.cliente || r.descricao) : r.descricao}</td>
            <td>${formatarMoeda(r.valor)}</td>
        `;
        tbody.appendChild(tr);
    });
}

// Exportar caixa para impressão (PDF via print)
function exportarCaixaPDF(){
    let conteudo = document.querySelector('#aba_caixa').innerHTML;
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório do Caixa</title></head><body>'+conteudo+'</body></html>');
    win.document.close();
    win.print();
}

// Limpar caixa (apaga histórico)
function limparCaixa(){
    if(!confirm('Tem certeza que deseja limpar todo o histórico do caixa? Esta ação não pode ser desfeita.')) return;
    caixa = [];
    localStorage.setItem("caixa", JSON.stringify(caixa));
    atualizarTabelaCaixa(); calcularTotaisCaixa();
    alert('Caixa limpo.');
}

/* Inicialização (caso já tenha dados) */
atualizarTabela();
calcularTotais();
atualizarTabelaCaixa();
calcularTotaisCaixa();

/* Foco de usabilidade: quando marcarPago registra a entrada já vai para a aba Caixa (opcional)
   Não alterei o fluxo automático, apenas deixei a aba Caixa disponível para visualização.
*/

</script>

</body>
</html>
<!-- FIM PARTE 3/3 -->

