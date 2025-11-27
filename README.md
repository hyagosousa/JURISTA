<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos Avançado — SPA</title>
<style>
    :root{
      --bg:#f5f5f5;
      --card:#fff;
      --accent:#28a745;
      --muted:#6c757d;
      --tab-border:#ddd;
    }
    *{box-sizing:border-box}
    body { font-family: Arial, sans-serif; margin: 0; background: var(--bg); color:#111; }
    .wrap { max-width: 980px; margin: 18px auto; padding: 12px; }
    .card { background: var(--card); padding: 18px; border-radius: 10px; box-shadow: 0 4px 14px rgba(0,0,0,0.06); margin-bottom: 14px; }

    .tabs { display:flex; gap:0; border-bottom: 2px solid var(--tab-border); border-radius: 10px 10px 0 0; overflow:hidden; }
    .tab {
      flex:1;
      text-align:center;
      padding:12px 16px;
      cursor:pointer;
      background: #fff;
      font-weight:700;
      color:#333;
      user-select:none;
      border-right:1px solid var(--tab-border);
    }
    .tab:last-child{ border-right: none; }
    .tab.active { background: linear-gradient(180deg,#fff,#f7f7f7); border-bottom: 3px solid var(--accent); color: var(--accent); }

    section.page { display:none; padding:14px 0; }
    section.page.active { display:block; }

    label { font-weight: bold; margin-top: 10px; display:block; color:#222; }
    input, select, textarea { width:100%; padding:12px; margin-top:6px; border-radius:8px; border:1px solid #ccc; font-size:15px; }
    button { padding:10px 14px; border:none; border-radius:8px; cursor:pointer; font-weight:700; }
    button.small { padding:6px 8px; font-size:13px; border-radius:6px; }
    .btn-success { background:var(--accent); color:white; }
    .btn-warning { background:#ffc107; color:#000; }
    .btn-info { background:#17a2b8; color:#fff; }
    .btn-danger { background:#dc3545; color:#fff; }

    .resultado, .dashboard { background:#f8f9fa; padding:12px; border-radius:8px; margin-top:12px; }
    .resultado p, .dashboard p { margin:6px 0; font-weight:700; }
    .valor-emp { color: #c82333; font-weight:800; }
    .valor-juros { color: #0b5ed7; font-weight:800; }
    .valor-total { color: #198754; font-weight:800; }

    table { width:100%; border-collapse:collapse; margin-top:10px; display:block; overflow-x:auto; }
    th, td { border:1px solid #e6e6e6; padding:10px; text-align:center; white-space:nowrap; }
    th { background:#f1f1f1; cursor:pointer; position:relative; }
    tr:nth-child(even) td { background: #fcfcfc; }
    .historico-actions button{ display:block; margin:6px 0; width:100%; }

    .piscar { animation: piscarAnim 1s infinite; }
    @keyframes piscarAnim {
        0% { background-color: #ffb3b3; }
        50% { background-color: #fff; }
        100% { background-color: #ffb3b3; }
    }

    @media(max-width:700px){
      .tabs{ font-size:14px; }
      input, select { font-size:14px; padding:10px; }
      th, td { font-size:12px; padding:8px; }
    }

    .login-box { max-width:400px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003; }
    .login-box h2{margin:0 0 10px 0;text-align:center}
    .top-actions { display:flex; gap:10px; margin-top:10px; flex-wrap:wrap; }
    .muted { color:var(--muted); font-size:13px; }
</style>
</head>
<body>

<script>
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div class="login-box">
        <h2>Login</h2>
        <input id="loginUser" placeholder="Login" />
        <input id="loginPass" type="password" placeholder="Senha" style="margin-top:10px;" />
        <button id="btnEntrar" style="width:100%;padding:12px;margin-top:16px;background:#007bff;color:white;border:none;border-radius:6px;">Entrar</button>
    </div>`;
    document.getElementById("btnEntrar").onclick = () => {
        let user = document.getElementById("loginUser").value.trim();
        let pass = document.getElementById("loginPass").value.trim();
        if (user === "H07y0321" && pass === "Helo2020@") {
            localStorage.setItem("logado", "true");
            location.reload();
        } else { alert("Login ou senha incorretos!"); }
    };
} else {
    document.write(`
    <div class="wrap">
      <div class="card">
        <div class="tabs" role="tablist">
          <div class="tab active" data-target="paginaCadastro" onclick="mostrarPagina(event)">CADASTRO</div>
          <div class="tab" data-target="paginaResumo" onclick="mostrarPagina(event)">RESUMO</div>
          <div class="tab" data-target="paginaClientes" onclick="mostrarPagina(event)">CLIENTES</div>
          <div class="tab" data-target="paginaCaixa" onclick="mostrarPagina(event)">CAIXA</div>
        </div>

        <!-- PAGE: Cadastro -->
        <section id="paginaCadastro" class="page active">
            <h2>Sistema de Empréstimos Avançado — Cadastro</h2>
            <label>Nome do Cliente:</label><input type="text" id="nome">
            <label>CPF:</label><input type="text" id="cpf">
            <label>Telefone:</label><input type="text" id="telefone">
            <label>Endereço:</label><input type="text" id="endereco">
            <label>Valor Emprestado:</label><input type="number" id="valor" step="0.01" oninput="atualizarCalculo()">
            <label>Porcentagem de Juros (%):</label><input type="number" id="juros" step="0.1" oninput="atualizarCalculo()">
            <label>Data do Empréstimo:</label><input type="date" id="dataEmp">
            <label>Data de Vencimento:</label><input type="date" id="dataVenc">

            <div class="resultado">
                <p>Valor: <span id="prevValor" class="valor-emp">R$ 0,00</span></p>
                <p>Juros: <span id="prevJuros" class="valor-juros">R$ 0,00</span></p>
                <p>Total a Receber: <span id="prevFinal" class="valor-total">R$ 0,00</span></p>
            </div>

            <div class="top-actions">
              <button class="btn-success" onclick="salvarCliente()">Salvar Cliente</button>
              <button class="btn-warning" onclick="exportarPDF()">Exportar PDF</button>
              <button class="btn-info" onclick="document.getElementById('importFile').click()">Importar JSON</button>
              <button class="btn-danger" onclick="exportarJSON()">Backup (Exportar JSON)</button>
              <input type="file" id="importFile" accept=".json" style="display:none" onchange="importarJSON(event)">
            </div>
            <p class="muted">Dica: use o botão <strong>Backup</strong> antes de fazer grandes alterações.</p>
        </section>

        <!-- PAGE: Resumo -->
        <section id="paginaResumo" class="page">
            <h3>Resumo Geral</h3>
            <div class="dashboard">
                <p>Total de Clientes: <span id="totalClientes">0</span></p>
                <p>Clientes Pagos: <span id="totalPagos">0</span></p>
                <p>Clientes Pendentes: <span id="totalPendentes">0</span></p>
                <p>Clientes Atrasados: <span id="totalAtrasados">0</span></p>
                <p>Valor Total Emprestado: <span id="totalEmpGeral" class="valor-emp">R$ 0,00</span></p>
                <p>Total em Juros: <span id="totalJurosGeral" class="valor-juros">R$ 0,00</span></p>
                <p>Total a Receber: <span id="totalReceberGeral" class="valor-total">R$ 0,00</span></p>
                <p>Saldo Recebido (após clicar em Receber): <span id="totalRecebido" class="valor-total">R$ 0,00</span></p>
            </div>

            <label>Buscar Cliente (Nome ou CPF):</label><input type="text" id="buscar" oninput="atualizarTabela()">
            <label>Filtrar por Status:</label>
            <select id="filtroStatus" onchange="atualizarTabela()">
                <option value="todos">Todos</option>
                <option value="pendente">Pendentes</option>
                <option value="pago">Pagos</option>
                <option value="atrasado">Atrasados</option>
            </select>

            <div style="display:flex;gap:10px;margin-top:10px;flex-wrap:wrap;">
              <button class="btn-warning" onclick="exportarPDF()">Exportar PDF</button>
              <button class="btn-info" onclick="enviarAlertaAtrasados()">📩 Enviar alerta para atrasados</button>
            </div>
        </section>

        <!-- PAGE: Clientes -->
        <section id="paginaClientes" class="page">
            <h3>Clientes Registrados</h3>
            <div class="card" style="padding:8px;">
              <table id="tabelaClientes" aria-label="Tabela de clientes">
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
        </section>

        <!-- PAGE: Caixa -->
        <section id="paginaCaixa" class="page">
            <h3>Histórico do Caixa</h3>
            <div class="dashboard">
                <p>Saldo do Caixa: <span id="saldoCaixa" class="valor-total">R$ 0,00</span></p>
                <p>Total Entradas: <span id="totalEntradas" class="valor-emp">R$ 0,00</span></p>
                <p>Total Saídas: <span id="totalSaidas" class="valor-juros">R$ 0,00</span></p>
            </div>
            <table id="tabelaCaixa">
                <thead>
                    <tr>
                        <th>Tipo</th>
                        <th>Valor</th>
                        <th>Motivo</th>
                        <th>Data e Hora</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
        </section>
      </div>
    </div>
    `);
}
</script>

<script>
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let caixa = JSON.parse(localStorage.getItem("caixa") || "[]");
let ordemAtual = '';

function formatarMoeda(valor){ return Number(valor||0).toLocaleString('pt-BR',{style:'currency',currency:'BRL'}); }

function mostrarPagina(e){
    let target = (typeof e === 'string') ? e : e.currentTarget.getAttribute('data-target');
    if(typeof e === 'string') target = e;
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    let tab = Array.from(document.querySelectorAll('.tab')).find(t => t.getAttribute('data-target') === target);
    if(tab) tab.classList.add('active');
    document.querySelectorAll('section.page').forEach(s=>s.classList.remove('active'));
    let page = document.getElementById(target);
    if(page) page.classList.add('active');
    atualizarTabela(); calcularTotais(); atualizarCaixa();
}
window.mostrarPagina = mostrarPagina;

function atualizarCalculo(){
    if(!document.getElementById('valor')) return;
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor*juros)/100;
    let vFinal = valor+vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}
window.atualizarCalculo = atualizarCalculo;

function salvarCliente(){
    if(!document.getElementById('nome')) return alert('Erro: formulário não encontrado');
    let nome = document.getElementById("nome").value.trim();
    let cpf = document.getElementById("cpf").value.trim();
    let telefone = document.getElementById("telefone").value.trim();
    let endereco = document.getElementById("endereco").value.trim();
    let valor = parseFloat(document.getElementById("valor").value)||0;
    let juros = parseFloat(document.getElementById("juros").value)||0;
    let dataEmp = document.getElementById("dataEmp").value;
    let dataVenc = document.getElementById("dataVenc").value;
    if(!nome||!cpf||!telefone||valor<=0||!dataEmp||!dataVenc) return alert('Preencha todos os campos corretamente!');
    let valorJuros = (valor*juros)/100;
    let valorFinal = valor+valorJuros;
    clientes.push({nome,cpf,telefone,endereco,valor,juros,valorJuros,valorFinal,dataEmp,dataVenc,pago:false});
    localStorage.setItem("clientes",JSON.stringify(clientes));
    atualizarTabela(); calcularTotais(); alert("Cliente salvo com sucesso!");
    document.getElementById("nome").value=""; document.getElementById("cpf").value=""; document.getElementById("telefone").value=""; document.getElementById("endereco").value=""; document.getElementById("valor").value=""; document.getElementById("juros").value=""; document.getElementById("dataEmp").value=""; document.getElementById("dataVenc").value=""; atualizarCalculo();
    mostrarPagina('paginaClientes');
}
window.salvarCliente = salvarCliente;

function excluirCliente(i){
    if(!confirm('Confirma exclusão deste cliente?')) return;
    clientes.splice(i,1);
    localStorage.setItem("clientes",JSON.stringify(clientes));
    atualizarTabela(); calcularTotais(); atualizarCaixa();
}
window.excluirCliente = excluirCliente;

function marcarPago(i){
    clientes[i].pago = true;
    localStorage.setItem("clientes",JSON.stringify(clientes));
    // adicionar entrada no caixa
    caixa.push({tipo:"Entrada",valor:clientes[i].valorFinal,motivo:`Recebimento do cliente ${clientes[i].nome}`,data:new Date()});
    localStorage.setItem("caixa",JSON.stringify(caixa));
    atualizarTabela(); calcularTotais(); atualizarCaixa();
}
window.marcarPago = marcarPago;

function atualizarTabela(){
    let tbody = document.querySelector("#tabelaClientes tbody"); if(!tbody) return;
    let filtro = document.getElementById("filtroStatus") ? document.getElementById("filtroStatus").value : "todos";
    let busca = document.getElementById("buscar") ? document.getElementById("buscar").value.toLowerCase() : "";
    tbody.innerHTML = "";
    clientes.forEach((c,i)=>{
        let status = c.pago ? "Pago" : (new Date(c.dataVenc) < new Date() ? "Atrasado" : "Pendente");
        if(filtro!=="todos" && filtro!==status.toLowerCase()) return;
        if(busca && !c.nome.toLowerCase().includes(busca) && !c.cpf.toLowerCase().includes(busca)) return;
        let tr = document.createElement('tr');
        tr.innerHTML = `
        <td>${c.nome}</td>
        <td>${formatarMoeda(c.valor)}</td>
        <td>${formatarMoeda(c.valorJuros)}</td>
        <td>${formatarMoeda(c.valorFinal)}</td>
        <td>${c.dataVenc}</td>
        <td class="historico-actions">
          ${!c.pago ? `<button class="btn-success small" onclick="marcarPago(${i})">Receber</button>` : ""}
          <button class="btn-danger small" onclick="excluirCliente(${i})">Excluir</button>
        </td>`;
        if(status==="Atrasado") tr.classList.add("piscar");
        tbody.appendChild(tr);
    });
}
window.atualizarTabela = atualizarTabela;

function calcularTotais(){
    let totalClientes = clientes.length;
    let totalPagos = clientes.filter(c=>c.pago).length;
    let totalPendentes = clientes.filter(c=>!c.pago && new Date(c.dataVenc)>=new Date()).length;
    let totalAtrasados = clientes.filter(c=>!c.pago && new Date(c.dataVenc)<new Date()).length;
    let totalEmp = clientes.reduce((a,c)=>a+c.valor,0);
    let totalJuros = clientes.reduce((a,c)=>a+c.valorJuros,0);
    let totalReceber = clientes.reduce((a,c)=>a+c.valorFinal,0);
    let totalRecebido = clientes.filter(c=>c.pago).reduce((a,c)=>a+c.valorFinal,0);

    if(document.getElementById("totalClientes")) document.getElementById("totalClientes").innerText = totalClientes;
    if(document.getElementById("totalPagos")) document.getElementById("totalPagos").innerText = totalPagos;
    if(document.getElementById("totalPendentes")) document.getElementById("totalPendentes").innerText = totalPendentes;
    if(document.getElementById("totalAtrasados")) document.getElementById("totalAtrasados").innerText = totalAtrasados;
    if(document.getElementById("totalEmpGeral")) document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    if(document.getElementById("totalJurosGeral")) document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    if(document.getElementById("totalReceberGeral")) document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalReceber);
    if(document.getElementById("totalRecebido")) document.getElementById("totalRecebido").innerText = formatarMoeda(totalRecebido);
}
window.calcularTotais = calcularTotais;

function atualizarCaixa(){
    let tbody = document.querySelector("#tabelaCaixa tbody"); if(!tbody) return;
    tbody.innerHTML = "";
    let totalEntradas = 0, totalSaidas = 0;
    caixa.forEach(c=>{
        let tr = document.createElement('tr');
        tr.innerHTML = `<td>${c.tipo}</td><td>${formatarMoeda(c.valor)}</td><td>${c.motivo}</td><td>${new Date(c.data).toLocaleString()}</td>`;
        tbody.appendChild(tr);
        if(c.tipo==="Entrada") totalEntradas+=c.valor;
        else totalSaidas+=c.valor;
    });
    if(document.getElementById("totalEntradas")) document.getElementById("totalEntradas").innerText = formatarMoeda(totalEntradas);
    if(document.getElementById("totalSaidas")) document.getElementById("totalSaidas").innerText = formatarMoeda(totalSaidas);
    if(document.getElementById("saldoCaixa")) document.getElementById("saldoCaixa").innerText = formatarMoeda(totalEntradas-totalSaidas);
}
window.atualizarCaixa = atualizarCaixa;

// Inicialização
atualizarTabela(); calcularTotais(); atualizarCaixa();
</script>

</body>
</html>

