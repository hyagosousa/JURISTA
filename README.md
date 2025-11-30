<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Sistema de Empréstimos + Controle de Caixa</title>
<style>
  body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
  .tabs { display:flex; gap:8px; justify-content:center; padding:12px; flex-wrap:wrap; }
  .tab-btn { padding:10px 14px; background:#007bff; color:#fff; border:none; border-radius:6px; cursor:pointer; }
  .tab-btn.active { background:#0056b3; }
  .tab-content { display:none; max-width:920px; margin: 12px auto; }
  .tab-content.active { display:block; }
  .container { background:#fff; padding:18px; border-radius:10px; box-shadow:0 0 12px rgba(0,0,0,0.08); }
  label{display:block;margin-top:10px;font-weight:600}
  input, select, textarea { width:100%; padding:10px; margin-top:6px; border-radius:6px; border:1px solid #ccc; box-sizing:border-box; }
  button { padding:10px 14px; border:none; border-radius:6px; cursor:pointer; }
  button.full { width:100%; }
  .resultado, .dashboard { background:#efefef; padding:12px; border-radius:8px; margin-top:12px; }
  table { width:100%; border-collapse:collapse; margin-top:12px; }
  th, td { border:1px solid #ddd; padding:8px; text-align:center; font-size:14px; }
  th { background:#f1f1f1; cursor:pointer; }
  .small { font-size:13px; }
  .piscar { animation: blink 1s infinite; }
  @keyframes blink { 0%{background:#ffb3b3}50%{background:#fff}100%{background:#ffb3b3} }
  .row { display:flex; gap:12px; }
  .col { flex:1; }
  @media(max-width:780px){ .row{flex-direction:column} .tabs{padding:8px} }
</style>
</head>
<body>

<!-- LOGIN -->
<script>
if(!localStorage.getItem("logado")){
  document.body.innerHTML = `
    <div style="max-width:420px;margin:80px auto;background:white;padding:22px;border-radius:8px;box-shadow:0 0 12px rgba(0,0,0,0.12)">
      <h2 style="text-align:center;margin:0 0 12px 0">Login</h2>
      <input id="loginUser" placeholder="Login" style="padding:10px;border-radius:6px;border:1px solid #ccc;margin-bottom:8px"/>
      <input id="loginPass" type="password" placeholder="Senha" style="padding:10px;border-radius:6px;border:1px solid #ccc;margin-bottom:12px"/>
      <button id="btnEntrar" style="width:100%;background:#007bff;color:white;padding:10px;border-radius:6px">Entrar</button>
    </div>`;
  document.getElementById("btnEntrar").onclick = () => {
    let user = document.getElementById("loginUser").value.trim();
    let pass = document.getElementById("loginPass").value.trim();
    if(user === "H07y0321" && pass === "Helo2020@"){
      localStorage.setItem("logado","true");
      location.reload();
    } else alert("Login ou senha incorretos!");
  };
}
</script>

<!-- TABS -->
<div class="tabs" role="tablist">
  <button class="tab-btn active" onclick="trocarAba('cadastro', event)">Cadastro de Clientes</button>
  <button class="tab-btn" onclick="trocarAba('resumo', event)">Resumo Geral</button>
  <button class="tab-btn" onclick="trocarAba('clientes', event)">Clientes Cadastrados</button>
  <button class="tab-btn" onclick="trocarAba('caixa', event)">Controle de Caixa</button>
</div>

<!-- ABA: CADASTRO -->
<div id="cadastro" class="tab-content active">
  <div class="container">
    <h2 style="margin-top:0">Cadastro de Clientes</h2>

    <label>Nome do Cliente:</label>
    <input id="nome" type="text">

    <label>CPF:</label>
    <input id="cpf" type="text">

    <label>Telefone (somente números):</label>
    <input id="telefone" type="text" placeholder="31999998888">

    <label>Endereço:</label>
    <input id="endereco" type="text">

    <div class="row">
      <div class="col">
        <label>Valor Emprestado (R$):</label>
        <input id="valor" type="number" step="0.01" oninput="atualizarCalculo()">
      </div>
      <div class="col">
        <label>Porcentagem de Juros (%):</label>
        <input id="juros" type="number" step="0.1" oninput="atualizarCalculo()">
      </div>
    </div>

    <div class="row">
      <div class="col">
        <label>Data do Empréstimo:</label>
        <input id="dataEmp" type="date">
      </div>
      <div class="col">
        <label>Data de Vencimento:</label>
        <input id="dataVenc" type="date">
      </div>
    </div>

    <div class="resultado">
      <p class="small">Valor: <strong id="prevValor">R$ 0,00</strong></p>
      <p class="small">Juros: <strong id="prevJuros">R$ 0,00</strong></p>
      <p class="small">Total a Receber: <strong id="prevFinal">R$ 0,00</strong></p>
    </div>

    <button class="full" style="background:#28a745;color:white;margin-top:12px" onclick="salvarCliente()">Salvar Cliente</button>
  </div>
</div>

<!-- ABA: RESUMO GERAL -->
<div id="resumo" class="tab-content">
  <div class="container">
    <h3 style="margin-top:0">Resumo Geral</h3>

    <div class="row">
      <div class="col dashboard">
        <p>Total de Clientes: <strong id="totalClientes">0</strong></p>
        <p>Clientes Pagos: <strong id="totalPagos">0</strong></p>
        <p>Clientes Pendentes: <strong id="totalPendentes">0</strong></p>
        <p>Clientes Atrasados: <strong id="totalAtrasados">0</strong></p>
      </div>

      <div class="col dashboard">
        <p>Valor de Saída (Emprestados): <strong id="totalSaidaEmp" class="small">R$ 0,00</strong></p>
        <p>Valor Recebido Total: <strong id="totalRecebidoTotal" class="small">R$ 0,00</strong></p>
        <p>Valor Recebido Só de Juros: <strong id="totalRecebidoJuros" class="small">R$ 0,00</strong></p>
        <p>Saldo Caixa: <strong id="saldoCaixa" class="small">R$ 0,00</strong></p>
      </div>
    </div>

    <label>Buscar Cliente (Nome ou CPF):</label>
    <input id="buscar" type="text" oninput="atualizarTabela()">

    <label>Filtrar por Status:</label>
    <select id="filtroStatus" onchange="atualizarTabela()">
      <option value="todos">Todos</option>
      <option value="pendente">Pendentes</option>
      <option value="pago">Pagos</option>
      <option value="atrasado">Atrasados</option>
    </select>

    <div style="display:flex;gap:10px;margin-top:12px;flex-wrap:wrap">
      <button style="background:#ffc107;border:none;padding:8px 12px;border-radius:6px" onclick="exportarPDF()">Exportar PDF</button>
      <button style="background:#17a2b8;border:none;padding:8px 12px;border-radius:6px;color:white" onclick="enviarAlertaAtrasados()">📩 Enviar alerta</button>
    </div>
  </div>
</div>

<!-- ABA: CLIENTES CADASTRADOS -->
<div id="clientes" class="tab-content">
  <div class="container">
    <h3 style="margin-top:0">Clientes Registrados</h3>
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

<!-- ABA: CONTROLE DE CAIXA -->
<div id="caixa" class="tab-content">
  <div class="container">
    <h3 style="margin-top:0">Controle de Caixa</h3>

    <!-- Totais do Caixa -->
    <div class="row">
      <div class="col dashboard">
        <p>Valor de Saída (Emprestados): <strong id="caixa_saida_emprestimos">R$ 0,00</strong></p>
        <p>Total Entradas (Pagamentos + Entradas manuais): <strong id="caixa_total_entradas">R$ 0,00</strong></p>
        <p>Total Entradas (Somente Juros): <strong id="caixa_total_juros">R$ 0,00</strong></p>
        <p>Total Entradas Manuais: <strong id="caixa_entradas_manuais">R$ 0,00</strong></p>
        <p>Total Saídas Manuais: <strong id="caixa_saidas_manuais">R$ 0,00</strong></p>
        <p>Saldo Final: <strong id="caixa_saldo_final">R$ 0,00</strong></p>
      </div>

      <div class="col dashboard">
        <p class="small"><strong>Resumo</strong></p>
        <p class="small">Entradas automáticas vindo de clientes pagos (valor + juros)</p>
        <p class="small">Saídas automáticas quando você registra um empréstimo (ao salvar cliente)</p>
        <p class="small">Lançamentos manuais abaixo</p>
      </div>
    </div>

    <!-- Formulário de Lançamento Manual -->
    <div style="margin-top:12px;">
      <h4 style="margin:6px 0">Lançamento Manual</h4>
      <label>Tipo:</label>
      <select id="mov_tipo">
        <option value="entrada">Entrada</option>
        <option value="saida">Saída</option>
      </select>

      <label>Categoria (ex: venda, compra material, pagamento cliente):</label>
      <input id="mov_categoria" type="text" placeholder="Categoria">

      <label>Descrição:</label>
      <input id="mov_descricao" type="text" placeholder="Descrição">

      <div class="row">
        <div class="col">
          <label>Valor (R$):</label>
          <input id="mov_valor" type="number" step="0.01">
        </div>
        <div class="col">
          <label>Data:</label>
          <input id="mov_data" type="date">
        </div>
      </div>

      <button style="background:#007bff;color:white;margin-top:10px" onclick="adicionarMovimentoManual()">Adicionar Movimento</button>
    </div>

    <!-- Histórico do Caixa -->
    <div style="margin-top:16px;">
      <h4 style="margin:6px 0">Histórico do Caixa</h4>
      <table id="tabelaCaixa">
        <thead>
          <tr>
            <th>Data</th>
            <th>Tipo</th>
            <th>Categoria</th>
            <th>Descrição</th>
            <th>Valor</th>
            <th>Origem</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <div style="margin-top:10px;display:flex;gap:8px;flex-wrap:wrap">
      <button style="background:#ffc107;padding:8px;border-radius:6px;border:none" onclick="exportarRelatorioCaixa()">Exportar relatório (impressão)</button>
      <button style="background:#dc3545;padding:8px;border-radius:6px;border:none;color:white" onclick="limparCaixaConfirm()">Limpar histórico do caixa</button>
    </div>
  </div>
</div>

<script>
/* ---------------------------
   Dados: clients e caixa
   --------------------------- */
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let caixa = JSON.parse(localStorage.getItem("caixa") || "[]"); // cada item: {data, tipo('entrada'|'saida'), categoria, descricao, valor, origem, juros:0.00}

/* ---------------------------
   Helpers
   --------------------------- */
function trocarAba(id, event){
  document.querySelectorAll('.tab-btn').forEach(b=>b.classList.remove('active'));
  document.querySelectorAll('.tab-content').forEach(c=>c.classList.remove('active'));
  if(event && event.target) event.target.classList.add('active');
  document.getElementById(id).classList.add('active');
}

function formatarMoeda(v){ return Number(v||0).toLocaleString('pt-BR',{style:'currency',currency:'BRL'}); }

/* ---------------------------
   Cálculo pré-visualização
   --------------------------- */
function atualizarCalculo(){
  const valor = parseFloat(document.getElementById('valor').value) || 0;
  const juros = parseFloat(document.getElementById('juros').value) || 0;
  const vJuros = Number(((valor * juros) / 100).toFixed(2));
  const vFinal = Number((valor + vJuros).toFixed(2));
  document.getElementById('prevValor').innerText = formatarMoeda(valor);
  document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
  document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}

/* ---------------------------
   Salvar Cliente (registra saída = empréstimo)
   --------------------------- */
function salvarCliente(){
  let nome = document.getElementById("nome").value.trim();
  let cpf = document.getElementById("cpf").value.trim();
  let telefone = document.getElementById("telefone").value.trim();
  let endereco = document.getElementById("endereco").value.trim();
  let valor = parseFloat(document.getElementById("valor").value) || 0;
  let juros = parseFloat(document.getElementById("juros").value) || 0;
  let dataEmp = document.getElementById("dataEmp").value;
  let dataVenc = document.getElementById("dataVenc").value;

  if(!nome || !cpf || !telefone || valor <= 0 || !dataEmp || !dataVenc){
    alert("Preencha todos os campos corretamente!");
    return;
  }

  const valorJuros = Number(((valor * juros) / 100).toFixed(2));
  const valorFinal = Number((valor + valorJuros).toFixed(2));

  const cliente = {
    id: Date.now() + Math.floor(Math.random()*999),
    nome, cpf, telefone, endereco,
    valor: Number(valor.toFixed(2)),
    juros: Number(juros.toFixed(2)),
    valorJuros,
    valorFinal,
    dataEmp, dataVenc,
    pago:false, recebido:0.00
  };

  clientes.push(cliente);
  localStorage.setItem("clientes", JSON.stringify(clientes));

  // Registra saída automática no caixa (emprestimo)
  const movSaida = {
    id: Date.now() + Math.floor(Math.random()*999),
    data: dataEmp || new Date().toISOString().slice(0,10),
    tipo: "saida",
    categoria: "emprestimo",
    descricao: `Empréstimo p/ ${cliente.nome}`,
    valor: Number(valor.toFixed(2)),
    origem: `cliente:${cliente.id}`,
    juros: 0.00
  };
  caixa.push(movSaida);
  localStorage.setItem("caixa", JSON.stringify(caixa));

  atualizarTabela();
  atualizarCaixaTabela();
  calcularTotais();
  limparFormulario();
  alert("Cliente salvo e saída registrada no caixa (empréstimo).");
}

/* ---------------------------
   Marcar pago (entrada automática)
   --------------------------- */
function marcarPago(i){
  if(!confirm('Confirmar marcação como PAGO para este cliente?')) return;
  const c = clientes[i];
  if(c.pago){ alert('Cliente já marcado como pago.'); return; }
  c.pago = true;
  c.recebido = c.valorFinal;
  localStorage.setItem("clientes", JSON.stringify(clientes));

  // Registrar entrada automática no caixa (valorFinal)
  const movEntrada = {
    id: Date.now() + Math.floor(Math.random()*999),
    data: new Date().toISOString().slice(0,10),
    tipo: "entrada",
    categoria: "pagamento_cliente",
    descricao: `Pagamento de ${c.nome}`,
    valor: Number(c.valorFinal),
    origem: `cliente:${c.id}`,
    juros: Number(c.valorJuros)
  };
  caixa.push(movEntrada);
  localStorage.setItem("caixa", JSON.stringify(caixa));

  atualizarTabela();
  atualizarCaixaTabela();
  calcularTotais();
  alert('Registro de pagamento inserido no caixa automaticamente.');
}

/* ---------------------------
   Cobrar via WhatsApp
   --------------------------- */
function cobrar(telefone, valorFinal, dataVenc, nome){
  let tel = (telefone || '').replace(/\D/g,'');
  if(!tel){ alert('Telefone inválido'); return; }
  let valorFormatado = formatarMoeda(valorFinal);
  let msg = `Opa ${nome}, sua dívida de ${valorFormatado} vence em ${dataVenc}. Por favor, realize o pagamento via PIX (chave: Hyagosousasous@gmail.com). Obrigado!`;
  window.open(`https://wa.me/55${tel}?text=${encodeURIComponent(msg)}`, "_blank");
}

/* ---------------------------
   Adicionar movimento manual no caixa
   --------------------------- */
function adicionarMovimentoManual(){
  const tipo = document.getElementById('mov_tipo').value;
  const categoria = document.getElementById('mov_categoria').value.trim() || (tipo === 'entrada' ? 'entrada_manual' : 'saida_manual');
  const descricao = document.getElementById('mov_descricao').value.trim() || '-';
  const valor = Number((parseFloat(document.getElementById('mov_valor').value) || 0).toFixed(2));
  const data = document.getElementById('mov_data').value || new Date().toISOString().slice(0,10);

  if(valor <= 0){ alert('Informe um valor maior que zero.'); return; }

  const mov = {
    id: Date.now() + Math.floor(Math.random()*999),
    data,
    tipo,
    categoria,
    descricao,
    valor,
    origem: 'manual',
    juros: 0.00
  };
  caixa.push(mov);
  localStorage.setItem("caixa", JSON.stringify(caixa));
  atualizarCaixaTabela();
  calcularTotais();
  // limpar campos
  document.getElementById('mov_categoria').value = '';
  document.getElementById('mov_descricao').value = '';
  document.getElementById('mov_valor').value = '';
  document.getElementById('mov_data').value = '';
  alert('Movimento manual adicionado ao caixa.');
}

/* ---------------------------
   Tabela de clientes
   --------------------------- */
function atualizarTabela(){
  const tbody = document.querySelector('#tabelaClientes tbody');
  tbody.innerHTML = '';
  const busca = (document.getElementById('buscar')?.value || '').toLowerCase();
  const filtro = document.getElementById('filtroStatus')?.value || 'todos';
  const hoje = new Date();

  clientes.forEach((c,i)=>{
    const atrasado = c.dataVenc && new Date(c.dataVenc) < hoje && !c.pago;
    if(busca && !(c.nome.toLowerCase().includes(busca) || (c.cpf||'').toLowerCase().includes(busca))) return;
    if(filtro==='pendente' && (c.pago || atrasado)) return;
    if(filtro==='pago' && !c.pago) return;
    if(filtro==='atrasado' && !atrasado) return;

    const tr = document.createElement('tr');
    tr.style.backgroundColor = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
    if(atrasado && !c.pago) tr.classList.add('piscar');

    tr.innerHTML = `
      <td>${c.nome}</td>
      <td>${formatarMoeda(c.valor)}</td>
      <td>${formatarMoeda(c.valorJuros)}</td>
      <td>${formatarMoeda(c.valorFinal)}</td>
      <td>${c.dataVenc || '-'}</td>
      <td style="display:flex;flex-direction:column;gap:6px;align-items:center">
        <button onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${c.nome}')">💰 Cobrar</button>
        <button onclick="excluirCliente(${i})" style="background:#dc3545;color:white;">❌ Excluir</button>
        ${!c.pago ? `<button onclick="marcarPago(${i})" style="background:#28a745;color:white;">✅ Pago</button>` : `<span style="background:#28a745;color:#fff;padding:6px;border-radius:6px">Pago</span>`}
      </td>
    `;
    tbody.appendChild(tr);
  });
}

/* ---------------------------
   Excluir cliente (e opcionalmente limpar origem no caixa)
   --------------------------- */
function excluirCliente(i){
  if(!confirm('Excluir este cliente? Isso não removerá automaticamente os lançamentos já registrados no caixa.')) return;
  const cliente = clientes[i];
  // NÃO removemos automaticamente lançamentos do caixa para preservar histórico
  clientes.splice(i,1);
  localStorage.setItem("clientes", JSON.stringify(clientes));
  atualizarTabela();
  calcularTotais();
  alert('Cliente excluído. Movimentos no caixa permanecem para histórico.');
}

/* ---------------------------
   Caixa: tabela e totais
   --------------------------- */
function atualizarCaixaTabela(){
  const tbody = document.querySelector('#tabelaCaixa tbody');
  tbody.innerHTML = '';
  // ordenar por data desc
  const arr = [...caixa].sort((a,b)=> (''+b.data).localeCompare(''+a.data) || b.id - a.id);
  arr.forEach((m,i)=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${m.data}</td>
      <td>${m.tipo === 'entrada' ? 'Entrada' : 'Saída'}</td>
      <td>${m.categoria}</td>
      <td>${m.descricao}</td>
      <td>${formatarMoeda(m.valor)}</td>
      <td>${m.origem || '-'}</td>
      <td>
        <button onclick="removerMovimento('${m.id}')" style="background:#dc3545;color:white;padding:6px;border-radius:6px;border:none">Remover</button>
      </td>
    `;
    tbody.appendChild(tr);
  });
}

function removerMovimento(id){
  if(!confirm('Remover este movimento do caixa?')) return;
  caixa = caixa.filter(m=>String(m.id) !== String(id));
  localStorage.setItem("caixa", JSON.stringify(caixa));
  atualizarCaixaTabela();
  calcularTotais();
}

/* ---------------------------
   Cálculo Totais específicos solicitados
   - Valor de Saída (Emprestimos) = soma de movimentos tipo=saida e categoria=emprestimo
   - Valor Recebido Total = soma de movimentos tipo=entrada e (categoria pagamento_cliente + outras entradas)
   - Valor Recebido Só de Juros = soma do campo juros das entradas com origem cliente
   - Entradas Manuais / Saídas Manuais
   - Saldo = (Entradas totais) - (Saídas totais)
   --------------------------- */
function calcularTotais(){
  // Totais do cliente (resumo rápido)
  let pagos=0, pendentes=0, atrasados=0;
  const hoje = new Date();
  clientes.forEach(c=>{
    if(c.pago) pagos++; else { pendentes++; if(c.dataVenc && new Date(c.dataVenc) < hoje) atrasados++; }
  });
  document.getElementById('totalClientes').innerText = clientes.length;
  document.getElementById('totalPagos').innerText = pagos;
  document.getElementById('totalPendentes').innerText = pendentes;
  document.getElementById('totalAtrasados').innerText = atrasados;

  // Totais do caixa
  let saidaEmp = 0;
  let entradasTotais = 0;
  let entradasJuros = 0;
  let entradasManuais = 0;
  let saidasManuais = 0;
  let saidasTotais = 0;

  caixa.forEach(m=>{
    if(m.tipo === 'entrada'){
      entradasTotais += Number(m.valor || 0);
      if(m.origem && String(m.origem).startsWith('cliente:')){
        // se origem cliente, soma juros registrados no campo juros
        entradasJuros += Number(m.juros || 0);
      } else {
        entradasManuais += Number(m.valor || 0);
      }
    } else if(m.tipo === 'saida'){
      saidasTotais += Number(m.valor || 0);
      if(m.categoria === 'emprestimo') saidaEmp += Number(m.valor || 0);
      else saidasManuais += Number(m.valor || 0);
    }
  });

  const saldo = entradasTotais - saidasTotais;

  // Atualiza elementos do Caixa (aba Caixa)
  document.getElementById('caixa_saida_emprestimos').innerText = formatarMoeda(saidaEmp);
  document.getElementById('caixa_total_entradas').innerText = formatarMoeda(entradasTotais);
  document.getElementById('caixa_total_juros').innerText = formatarMoeda(entradasJuros);
  document.getElementById('caixa_entradas_manuais').innerText = formatarMoeda(entradasManuais);
  document.getElementById('caixa_saidas_manuais').innerText = formatarMoeda(saidasManuais);
  document.getElementById('caixa_saldo_final').innerText = formatarMoeda(saldo);

  // Atualiza elementos do Resumo Geral (aba Resumo)
  document.getElementById('totalSaidaEmp').innerText = formatarMoeda(saidaEmp);
  document.getElementById('totalRecebidoTotal').innerText = formatarMoeda(entradasTotais);
  document.getElementById('totalRecebidoJuros').innerText = formatarMoeda(entradasJuros);
  document.getElementById('saldoCaixa').innerText = formatarMoeda(saldo);
  // Totais gerais (valores emprestados / juros / total a receber - se quiser mais: podia calcular separado)
  // Também atualiza totais (antigos)
  // se existirem campos extras
}

/* ---------------------------
   Exportar caixa (impressão)
   --------------------------- */
function exportarRelatorioCaixa(){
  let html = `<h2>Relatório do Caixa</h2>`;
  html += `<p>Gerado em: ${new Date().toLocaleString()}</p>`;
  html += `<table border="1" cellpadding="6" cellspacing="0" style="border-collapse:collapse;width:100%"><thead><tr><th>Data</th><th>Tipo</th><th>Categoria</th><th>Descrição</th><th>Valor</th><th>Origem</th></tr></thead><tbody>`;
  caixa.slice().reverse().forEach(m=>{
    html += `<tr>
      <td>${m.data}</td>
      <td>${m.tipo}</td>
      <td>${m.categoria}</td>
      <td>${m.descricao}</td>
      <td>${formatarMoeda(m.valor)}</td>
      <td>${m.origem || '-'}</td>
    </tr>`;
  });
  html += `</tbody></table>`;
  const win = window.open('','_blank','width=900,height=700');
  win.document.write('<html><head><title>Relatório Caixa</title></head><body style="font-family:Arial;padding:16px">'+html+'</body></html>');
  win.document.close();
  win.print();
}

/* ---------------------------
   Limpar caixa (com confirm)
   --------------------------- */
function limparCaixaConfirm(){
  if(!confirm('Tem certeza que quer apagar todo o histórico do caixa? Esta ação não pode ser desfeita.')) return;
  caixa = [];
  localStorage.setItem("caixa", JSON.stringify(caixa));
  atualizarCaixaTabela();
  calcularTotais();
  alert('Histórico do caixa limpo.');
}

/* ---------------------------
   Exportar PDF (clientes)
   --------------------------- */
function exportarPDF(){
  let conteudo = document.querySelector('#clientes .container')?.innerHTML || document.querySelector('#tabelaClientes')?.outerHTML || '';
  const win = window.open('','_blank','width=900,height=700');
  win.document.write('<html><head><title>Relatório Clientes</title></head><body style="font-family:Arial;padding:16px">'+conteudo+'</body></html>');
  win.document.close();
  win.print();
}

/* ---------------------------
   Enviar alerta atrasados (whatsapp)
   --------------------------- */
function enviarAlertaAtrasados(){
  const hoje = new Date();
  const atrasados = clientes.filter(c => !c.pago && c.dataVenc && new Date(c.dataVenc) < hoje);
  if(atrasados.length === 0){ alert('Não há clientes atrasados no momento!'); return; }
  atrasados.forEach(c => cobrar(c.telefone, c.valorFinal, c.dataVenc, c.nome));
}

/* ---------------------------
   Inicialização: atualiza tabelas e totais
   --------------------------- */
function inicializar(){
  atualizarCalculo();
  atualizarTabela();
  atualizarCaixaTabela();
  calcularTotais();
}
inicializar();

</script>

</body>
</html>



