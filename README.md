<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Sistema de Empréstimos + Caixa — SPA</title>
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
/* Simple login (keeps your previous credentials) */
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
    /* Build SPA UI (tabs) */
    document.write(`
    <div class="wrap">
      <div class="card">
        <div class="tabs" role="tablist">
          <div class="tab active" data-target="paginaCadastro" onclick="mostrarPagina(event)">CADASTRO</div>
          <div class="tab" data-target="paginaResumo" onclick="mostrarPagina(event)">RESUMO</div>
          <div class="tab" data-target="paginaClientes" onclick="mostrarPagina(event)">CLIENTES</div>
          <div class="tab" data-target="paginaCaixa" onclick="mostrarPagina(event)">CAIXA</div>
        </div>

        <!-- CADASTRO -->
        <section id="paginaCadastro" class="page active">
            <h2>Sistema de Empréstimos — Cadastro</h2>
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
            </div>
            <p class="muted">Dica: ao salvar um empréstimo, o sistema registra automaticamente uma saída no caixa.</p>
        </section>

        <!-- RESUMO -->
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
                <hr>
                <p>Entradas Totais (acumulado): <span id="entradasAcumulado" class="valor-emp">R$ 0,00</span></p>
                <p>Saídas Totais (acumulado): <span id="saidasAcumulado" class="valor-juros">R$ 0,00</span></p>
                <p>Saldo Total Acumulado: <span id="saldoTotalAcumulado" class="valor-total">R$ 0,00</span></p>
                <p>Saldo do Dia (hoje): <span id="saldoDoDia" class="valor-total">R$ 0,00</span></p>
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

        <!-- CLIENTES -->
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

        <!-- CAIXA -->
        <section id="paginaCaixa" class="page">
            <h3>Controle de Caixa</h3>
            <div class="dashboard">
                <p>Entradas Totais: <span id="totalEntradas" class="valor-emp">R$ 0,00</span></p>
                <p>Saídas Totais: <span id="totalSaidas" class="valor-juros">R$ 0,00</span></p>
                <p>Saldo Total (Entradas - Saídas): <span id="saldoCaixaTotal" class="valor-total">R$ 0,00</span></p>
            </div>

            <div class="card" style="margin-top:12px;">
                <h4>Registrar Movimento Manual</h4>
                <label>Tipo:</label>
                <select id="movTipo">
                    <option value="Entrada">Entrada</option>
                    <option value="Saída">Saída</option>
                </select>
                <label>Valor:</label>
                <input type="number" id="movValor" step="0.01">
                <label>Cliente (opcional):</label>
                <input type="text" id="movCliente" placeholder="Nome do cliente (opcional)">
                <label>Motivo:</label>
                <input type="text" id="movMotivo" placeholder="Ex: Juros recebidos, Depósito, Pagamento de fornecedor...">
                <div style="display:flex;gap:10px;margin-top:10px;">
                  <button class="btn-success" style="flex:1" onclick="adicionarMovimentoManual()">Adicionar</button>
                  <button class="btn-danger" style="flex:1" onclick="limparMovimentoManual()">Limpar</button>
                </div>
            </div>

            <h4 style="margin-top:12px;">Histórico Completo do Caixa</h4>
            <table id="tabelaCaixa">
                <thead>
                    <tr>
                        <th>Tipo</th>
                        <th>Valor</th>
                        <th>Cliente</th>
                        <th>Motivo</th>
                        <th>Data/Hora</th>
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
/* ---------- Dados e utilitários ---------- */
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let caixa = JSON.parse(localStorage.getItem("caixa") || "[]");
let ordemAtual = '';

function salvarStorage(){
    localStorage.setItem("clientes", JSON.stringify(clientes));
    localStorage.setItem("caixa", JSON.stringify(caixa));
}

function formatarMoeda(valor){
    return Number(valor || 0).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
}

/* ---------- Navegação de abas ---------- */
function mostrarPagina(e){
    let target = (typeof e === 'string') ? e : e.currentTarget.getAttribute('data-target');
    if(typeof e === 'string') target = e;
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    let tab = Array.from(document.querySelectorAll('.tab')).find(t => t.getAttribute('data-target') === target);
    if(tab) tab.classList.add('active');
    document.querySelectorAll('section.page').forEach(s => s.classList.remove('active'));
    let page = document.getElementById(target);
    if(page) page.classList.add('active');
    // atualiza dados ao mudar aba
    atualizarTabela();
    calcularTotais();
    atualizarCaixa();
}
window.mostrarPagina = mostrarPagina;

/* ---------- Formulário / Clientes ---------- */
function atualizarCalculo(){
    if(!document.getElementById('valor')) return;
    let valor = parseFloat(document.getElementById('valor').value) || 0;
    let juros = parseFloat(document.getElementById('juros').value) || 0;
    let vJuros = (valor * juros)/100;
    let vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}
window.atualizarCalculo = atualizarCalculo;

function salvarCliente(){
    if(!document.getElementById('nome')){ alert('Erro: formulário não encontrado.'); return; }
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
    // adiciona cliente
    let novo = {nome, cpf, telefone, endereco, valor, juros, valorJuros, valorFinal, dataEmp, dataVenc, pago:false};
    clientes.push(novo);
    // registrar saída automática: empréstimo feito
    registrarMovimento('Saída', valor, nome, `Empréstimo para ${nome}`);
    salvarStorage();
    atualizarTabela();
    calcularTotais();
    alert("Cliente salvo com sucesso! (Saída registrada no caixa: Empréstimo)");
    // opcional: limpar campos (se desejar)
    document.getElementById("nome").value=""; document.getElementById("cpf").value=""; document.getElementById("telefone").value=""; document.getElementById("endereco").value="";
    document.getElementById("valor").value=""; document.getElementById("juros").value=""; document.getElementById("dataEmp").value=""; document.getElementById("dataVenc").value="";
    atualizarCalculo();
}
window.salvarCliente = salvarCliente;

function excluirCliente(i){
    if(!confirm('Confirma exclusão deste cliente?')) return;
    // NÃO altera o caixa ao excluir cliente — operação apenas remove registro de cliente
    clientes.splice(i,1);
    salvarStorage();
    atualizarTabela(); calcularTotais(); atualizarCaixa();
}
window.excluirCliente = excluirCliente;

function marcarPago(i){
    if(clientes[i].pago) return;
    clientes[i].pago = true;
    // registrar entrada automática no caixa
    registrarMovimento('Entrada', clientes[i].valorFinal, clientes[i].nome, `Recebimento do cliente ${clientes[i].nome}`);
    salvarStorage();
    atualizarTabela();
    calcularTotais();
    atualizarCaixa();
}
window.marcarPago = marcarPago;

/* ---------- Cobrança WhatsApp (mantida) ---------- */
function cobrar(telefone, valorFinal, dataVenc, nome){
    let tel = String(telefone || '').replace(/\D/g,'');
    if(!tel){ alert('Telefone inválido para envio via WhatsApp'); return; }
    let valorFormatado = Number(valorFinal || 0).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
    let msg = `Opa ${nome}, sua dívida de ${valorFormatado} vence em ${dataVenc}. Por favor, realize o pagamento via PIX (chave: Hyagosousasous@gmail.com). Obrigado!`;
    window.open(`https://wa.me/55${tel}?text=${encodeURIComponent(msg)}`, "_blank");
}
window.cobrar = cobrar;

/* ---------- Alertas para atrasados (mantida) ---------- */
function enviarAlertaAtrasados(){
    let hoje = new Date();
    let atrasados = clientes.filter(c => !c.pago && c.dataVenc && new Date(c.dataVenc) < hoje);
    if(atrasados.length === 0){ alert("Não há clientes atrasados no momento!"); return; }
    if(!confirm(`Encontrados ${atrasados.length} cliente(s) atrasado(s). Deseja abrir as cobranças no WhatsApp?`)) return;
    atrasados.forEach(c => { cobrar(c.telefone, c.valorFinal, c.dataVenc, c.nome); });
}
window.enviarAlertaAtrasados = enviarAlertaAtrasados;

/* ---------- Movimentos do Caixa (unificado) ---------- */
function registrarMovimento(tipo, valor, cliente, motivo){
    // tipo: 'Entrada' ou 'Saída'
    const mv = {
        tipo: tipo,
        valor: Number(valor || 0),
        cliente: cliente || '',
        motivo: motivo || '',
        data: new Date().toISOString()
    };
    caixa.push(mv);
    salvarStorage();
}

/* Usado no formulário manual do Caixa */
function adicionarMovimentoManual(){
    let tipo = document.getElementById('movTipo').value;
    let valor = parseFloat(document.getElementById('movValor').value) || 0;
    let cliente = document.getElementById('movCliente').value.trim();
    let motivo = document.getElementById('movMotivo').value.trim();
    if(valor <= 0 || !motivo){ alert('Preencha valor e motivo'); return; }
    registrarMovimento(tipo, valor, cliente, motivo);
    atualizarCaixa();
    // limpar
    limparMovimentoManual();
}
window.adicionarMovimentoManual = adicionarMovimentoManual;
function limparMovimentoManual(){
    document.getElementById('movValor').value = '';
    document.getElementById('movCliente').value = '';
    document.getElementById('movMotivo').value = '';
}
window.limparMovimentoManual = limparMovimentoManual;

/* ---------- Atualizar Caixa (histórico e saldos) ---------- */
function atualizarCaixa(){
    let tbody = document.querySelector("#tabelaCaixa tbody");
    if(!tbody) return;
    tbody.innerHTML = "";
    let totalEntradas = 0, totalSaidas = 0;
    let hojeStr = new Date().toISOString().slice(0,10); // YYYY-MM-DD
    let entradasHoje = 0, saidasHoje = 0;
    // mostrar itens do caixa em ordem cronológica (mais recente primeiro)
    let copia = [...caixa].sort((a,b)=> new Date(b.data) - new Date(a.data));
    copia.forEach(item=>{
        let tr = document.createElement('tr');
        tr.innerHTML = `<td>${item.tipo}</td>
                        <td>${formatarMoeda(item.valor)}</td>
                        <td>${item.cliente || '-'}</td>
                        <td>${item.motivo || '-'}</td>
                        <td>${(new Date(item.data)).toLocaleString()}</td>`;
        tbody.appendChild(tr);
        if(item.tipo === 'Entrada') totalEntradas += Number(item.valor || 0);
        else totalSaidas += Number(item.valor || 0);

        // saldo do dia
        if((item.data || '').slice(0,10) === hojeStr){
            if(item.tipo === 'Entrada') entradasHoje += Number(item.valor || 0);
            else saidasHoje += Number(item.valor || 0);
        }
    });
    // atualizar DOM
    if(document.getElementById('totalEntradas')) document.getElementById('totalEntradas').innerText = formatarMoeda(totalEntradas);
    if(document.getElementById('totalSaidas')) document.getElementById('totalSaidas').innerText = formatarMoeda(totalSaidas);
    if(document.getElementById('saldoCaixaTotal')) document.getElementById('saldoCaixaTotal').innerText = formatarMoeda(totalEntradas - totalSaidas);

    // também popular resumo (entradas/saídas acumulado e saldo do dia)
    if(document.getElementById('entradasAcumulado')) document.getElementById('entradasAcumulado').innerText = formatarMoeda(totalEntradas);
    if(document.getElementById('saidasAcumulado')) document.getElementById('saidasAcumulado').innerText = formatarMoeda(totalSaidas);
    if(document.getElementById('saldoTotalAcumulado')) document.getElementById('saldoTotalAcumulado').innerText = formatarMoeda(totalEntradas - totalSaidas);
    if(document.getElementById('saldoDoDia')) document.getElementById('saldoDoDia').innerText = formatarMoeda(entradasHoje - saidasHoje);
}
window.atualizarCaixa = atualizarCaixa;

/* ---------- Totais e Tabela de Clientes ---------- */
function calcularTotais(){
    let totalEmp=0, totalJuros=0, totalFinal=0;
    let pagos=0, pendentes=0, atrasados=0;
    let hoje = new Date();
    clientes.forEach(c=>{
        totalEmp += Number(c.valor || 0);
        totalJuros += Number(c.valorJuros || 0);
        totalFinal += Number(c.valorFinal || 0);
        if(c.pago) pagos++;
        else {
            pendentes++;
            if(c.dataVenc && new Date(c.dataVenc) < hoje) atrasados++;
        }
    });

    if(document.getElementById("totalEmpGeral")) document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    if(document.getElementById("totalJurosGeral")) document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    if(document.getElementById("totalReceberGeral")) document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalFinal);
    if(document.getElementById("totalClientes")) document.getElementById("totalClientes").innerText = clientes.length;
    if(document.getElementById("totalPagos")) document.getElementById("totalPagos").innerText = pagos;
    if(document.getElementById("totalPendentes")) document.getElementById("totalPendentes").innerText = pendentes;
    if(document.getElementById("totalAtrasados")) document.getElementById("totalAtrasados").innerText = atrasados;

    // atualizar resumo caixa também (caso tenha elementos)
    atualizarCaixa();
}
window.calcularTotais = calcularTotais;

function atualizarTabela(){
    let tbody = document.querySelector("#tabelaClientes tbody");
    if(!tbody) return;
    tbody.innerHTML = "";
    let busca = document.getElementById('buscar') ? document.getElementById('buscar').value.toLowerCase() : '';
    let filtro = document.getElementById('filtroStatus') ? document.getElementById('filtroStatus').value : 'todos';
    clientes.forEach((c,i)=>{
        let atrasado=false;
        let hoje=new Date();
        if(c.dataVenc && new Date(c.dataVenc) < hoje && !c.pago) atrasado=true;
        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        if(busca && !(String(c.nome||'').toLowerCase().includes(busca) || String(c.cpf||'').toLowerCase().includes(busca))) return;
        if(filtro==='pendente' && (c.pago || atrasado)) return;
        if(filtro==='pago' && !c.pago) return;
        if(filtro==='atrasado' && !atrasado) return;
        let tr = document.createElement('tr');
        tr.style.backgroundColor = corLinha;
        if(atrasado && !c.pago) tr.classList.add('piscar');
        // escapar apóstrofos no nome para onclick inline
        let nomeEsc = (c.nome || '').replace(/'/g,"\\'");
        tr.innerHTML = `
            <td>${c.nome}</td>
            <td class="valor-emp">${formatarMoeda(c.valor)}</td>
            <td class="valor-juros">${formatarMoeda(c.valorJuros)}</td>
            <td class="valor-total">${formatarMoeda(c.valorFinal)}</td>
            <td>${c.dataVenc || '-'}</td>
            <td class="historico-actions">
                <button class="small" onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${nomeEsc}')">💰 Cobrar</button>
                <button class="small btn-danger" onclick="excluirCliente(${i})">❌ Excluir</button>
                ${!c.pago ? `<button class="small btn-success" onclick="marcarPago(${i})">✅ Pago</button>` : `<button class="small" disabled>Pago</button>`}
            </td>
        `;
        tbody.appendChild(tr);
    });
}
window.atualizarTabela = atualizarTabela;

/* ---------- Ordenação, PDF ---------- */
function ordenarTabela(campo){
    if(ordemAtual === campo){
        clientes.reverse();
    } else {
        clientes.sort((a,b)=>{
            if(['valor','valorJuros','valorFinal'].includes(campo)){
                return Number(a[campo]||0) - Number(b[campo]||0);
            }
            if(campo === 'dataVenc'){
                let da = a.dataVenc ? new Date(a.dataVenc).getTime() : 0;
                let db = b.dataVenc ? new Date(b.dataVenc).getTime() : 0;
                return da - db;
            }
            let aa = String(a[campo] || '').toLowerCase();
            let bb = String(b[campo] || '').toLowerCase();
            if(aa < bb) return -1;
            if(aa > bb) return 1;
            return 0;
        });
        ordemAtual = campo;
    }
    atualizarTabela();
}
window.ordenarTabela = ordenarTabela;

function exportarPDF(){
    let conteudo = document.querySelector('#paginaClientes') ? document.querySelector('#paginaClientes').innerHTML : document.querySelector('.card').innerHTML;
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório</title></head><body style="font-family:Arial;padding:10px;">'+conteudo+'</body></html>');
    win.document.close();
    win.print();
}

/* ---------- Inicialização ---------- */
document.addEventListener('DOMContentLoaded', function(){
    atualizarTabela();
    calcularTotais();
    atualizarCaixa();
});
// chamar também logo
setTimeout(()=>{ atualizarTabela(); calcularTotais(); atualizarCaixa(); }, 120);
</script>

</body>
</html>

