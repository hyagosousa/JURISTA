<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Empréstimos Avançado</title>
<style>
    body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
    .container { max-width: 920px; margin: 20px auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    h2, h3 { text-align: center; }
    label { font-weight: bold; margin-top: 10px; display: block; }
    input, select { width: 100%; padding: 12px; margin-top: 5px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; box-sizing: border-box; }
    button { padding: 10px 14px; border: none; border-radius: 8px; margin-top: 10px; font-size: 15px; cursor: pointer; }
    button.full { width:100%; }
    .resultado, .dashboard { background: #eee; padding: 15px; border-radius: 10px; margin-top: 20px; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; white-space: nowrap; }
    th { background: #ddd; cursor: pointer; }

    /* ---- ABAS ---- */
    .tabs { display: flex; justify-content: center; gap: 8px; margin-top: 12px; padding: 0 10px; flex-wrap:wrap; }
    .tab-btn {
        padding: 10px 16px;
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

    @media(max-width:600px){
      input, select { font-size:14px; padding:10px; }
      th, td { font-size:12px; padding:6px; white-space:normal; }
    }
</style>
</head>
<body>

<!-- LOGIN -->
<script>
if (!localStorage.getItem("logado")) {
    document.body.innerHTML = `
    <div style='max-width:420px;margin:auto;margin-top:80px;background:white;padding:25px;border-radius:10px;box-shadow:0 0 10px #0003;'>
        <h2 style='text-align:center;'>Login</h2>
        <input id='loginUser' placeholder='Login' style='width:100%;padding:12px;margin-top:10px;border-radius:6px;border:1px solid #ccc;'>
        <input id='loginPass' type='password' placeholder='Senha' style='width:100%;padding:12px;margin-top:10px;border-radius:6px;border:1px solid #ccc;'>
        <button id='btnEntrar' style="width:100%;padding:12px;margin-top:18px;background:#007bff;color:white;border-radius:6px;">Entrar</button>
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

<!-- ABAS -->
<div class="tabs">
    <button class="tab-btn active" onclick="trocarAba('cadastro', event)">Cadastro de Clientes</button>
    <button class="tab-btn" onclick="trocarAba('resumo', event)">Resumo Geral</button>
    <button class="tab-btn" onclick="trocarAba('clientes', event)">Clientes Cadastrados</button>
</div>

<!-- ABA 1 - CADASTRO -->
<div id="cadastro" class="tab-content active">
    <div class="container">
        <h2>Sistema de Empréstimos Avançado</h2>

        <label>Nome do Cliente:</label>
        <input type="text" id="nome">

        <label>CPF:</label>
        <input type="text" id="cpf">

        <label>Telefone (somente números):</label>
        <input type="text" id="telefone" placeholder="Ex: 31999998888">

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

        <button class="full" style="background:#28a745;color:white;" onclick="salvarCliente()">Salvar Cliente</button>
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

        <div style="display:flex;gap:10px;margin-top:12px;flex-wrap:wrap;">
          <button style="background:#ffc107;color:black;" onclick="exportarPDF()">Exportar PDF</button>
          <button style="background:#17a2b8;color:white;" onclick="enviarAlertaAtrasados()">📩 Enviar alerta para atrasados</button>
          <button style="background:#6c757d;color:white;" onclick="limparFiltros()">Limpar filtros</button>
        </div>
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
function trocarAba(aba, event) {
    document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(div => div.classList.remove('active'));
    if(event && event.target) event.target.classList.add('active');
    document.getElementById(aba).classList.add('active');
}

/* ---- DADOS E HELPERS ---- */
let clientes = JSON.parse(localStorage.getItem("clientes") || "[]");
let ordemAtual = '';
function formatarMoeda(valor){ return Number(valor || 0).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' }); }

/* ---- CÁLCULO E PRÉ-VISUALIZAÇÃO ---- */
function atualizarCalculo(){
    const valor = parseFloat(document.getElementById('valor').value) || 0;
    const juros = parseFloat(document.getElementById('juros').value) || 0;
    const vJuros = (valor * juros) / 100;
    const vFinal = valor + vJuros;
    document.getElementById('prevValor').innerText = formatarMoeda(valor);
    document.getElementById('prevJuros').innerText = formatarMoeda(vJuros);
    document.getElementById('prevFinal').innerText = formatarMoeda(vFinal);
}

/* ---- SALVAR CLIENTE ---- */
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
        alert('Preencha todos os campos corretamente!');
        return;
    }

    const valorJuros = Number(((valor * juros) / 100).toFixed(2));
    const valorFinal = Number((valor + valorJuros).toFixed(2));

    clientes.push({
        nome,
        cpf,
        telefone,
        endereco,
        valor: Number(valor.toFixed(2)),
        juros: Number(juros.toFixed(2)),
        valorJuros,
        valorFinal,
        dataEmp,
        dataVenc,
        pago: false,
        recebido: 0.00
    });

    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
    calcularTotais();
    limparFormulario();
    alert("Cliente salvo com sucesso!");
}

/* ---- LIMPAR FORMULÁRIO ---- */
function limparFormulario(){
    document.getElementById("nome").value = '';
    document.getElementById("cpf").value = '';
    document.getElementById("telefone").value = '';
    document.getElementById("endereco").value = '';
    document.getElementById("valor").value = '';
    document.getElementById("juros").value = '';
    document.getElementById("dataEmp").value = '';
    document.getElementById("dataVenc").value = '';
    atualizarCalculo();
}

/* ---- EXCLUIR / MARCAR PAGO (AGORA COM RECEBIDO AUTOMÁTICO) ---- */
function excluirCliente(i){
    if(!confirm('Excluir este cliente?')) return;
    clientes.splice(i,1);
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
    calcularTotais();
}

function marcarPago(i){
    // marca como pago e registra recebido igual ao valorFinal
    clientes[i].pago = true;
    clientes[i].recebido = clientes[i].valorFinal;
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
    calcularTotais();
}

/* ---- COBRAR VIA WHATSAPP ---- */
function cobrar(telefone, valorFinal, dataVenc, nome){
    let tel = (telefone || '').replace(/\D/g,'');
    if(!tel) { alert('Telefone inválido'); return; }
    let valorFormatado = formatarMoeda(valorFinal);
    let msg = `Opa ${nome}, sua dívida de ${valorFormatado} vence em ${dataVenc}. Por favor, realize o pagamento via PIX (chave: Hyagosousasous@gmail.com). Obrigado!`;
    window.open(`https://wa.me/55${tel}?text=${encodeURIComponent(msg)}`, "_blank");
}

/* ---- ENVIAR ALERTA PARA ATRASADOS (abre várias janelas/abas do WhatsApp) ---- */
function enviarAlertaAtrasados(){
    let hoje = new Date();
    let atrasados = clientes.filter(c => !c.pago && c.dataVenc && new Date(c.dataVenc) < hoje);
    if(atrasados.length === 0){ alert("Não há clientes atrasados no momento!"); return; }
    atrasados.forEach(c => { cobrar(c.telefone, c.valorFinal, c.dataVenc, c.nome); });
}

/* ---- CALCULAR TOTAIS ---- */
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

    document.getElementById("totalEmpGeral").innerText = formatarMoeda(totalEmp);
    document.getElementById("totalJurosGeral").innerText = formatarMoeda(totalJuros);
    document.getElementById("totalReceberGeral").innerText = formatarMoeda(totalFinal);
    document.getElementById("totalClientes").innerText = clientes.length;
    document.getElementById("totalPagos").innerText = pagos;
    document.getElementById("totalPendentes").innerText = pendentes;
    document.getElementById("totalAtrasados").innerText = atrasados;
}

/* ---- ATUALIZAR TABELA ---- */
function atualizarTabela(){
    let tbody=document.querySelector("#tabelaClientes tbody");
    tbody.innerHTML="";
    let busca = (document.getElementById('buscar')?.value || '').toLowerCase();
    let filtro = document.getElementById('filtroStatus')?.value || 'todos';
    let hoje = new Date();

    clientes.forEach((c,i)=>{
        let atrasado = c.dataVenc && new Date(c.dataVenc) < hoje && !c.pago;
        // filtros de busca
        if(busca && !( (c.nome||'').toLowerCase().includes(busca) || (c.cpf||'').toLowerCase().includes(busca) )) return;
        if(filtro==='pendente' && (c.pago || atrasado)) return;
        if(filtro==='pago' && !c.pago) return;
        if(filtro==='atrasado' && !atrasado) return;

        let corLinha = c.pago ? '#c8f7c5' : (atrasado ? '#ffb3b3' : 'white');
        let tr = document.createElement('tr');
        tr.style.backgroundColor = corLinha;
        if(atrasado && !c.pago) tr.classList.add('piscar');

        tr.innerHTML = `
            <td>${c.nome}</td>
            <td>${formatarMoeda(c.valor)}</td>
            <td>${formatarMoeda(c.valorJuros)}</td>
            <td>${formatarMoeda(c.valorFinal)}</td>
            <td>${c.dataVenc || '-'}</td>
            <td style="display:flex;flex-direction:column;gap:6px;align-items:center;">
                <button onclick="cobrar('${c.telefone}', ${c.valorFinal}, '${c.dataVenc}', '${c.nome}')">💰 Cobrar</button>
                <button onclick="excluirCliente(${i})" style="background:red;color:white;">❌ Excluir</button>
                ${!c.pago ? `<button onclick="marcarPago(${i})" style="background:green;color:white;">✅ Pago</button>` : `<span style="background:#28a745;color:#fff;padding:6px;border-radius:6px;">Pago</span>`}
            </td>
        `;
        tbody.appendChild(tr);
    });
}

/* ---- ORDENAÇÃO SIMPLES ---- */
function ordenarTabela(campo){
    if(ordemAtual === campo){ clientes.reverse(); }
    else {
        clientes.sort((a,b)=>{
            let va = a[campo] === undefined ? '' : a[campo];
            let vb = b[campo] === undefined ? '' : b[campo];
            // se forem datas, converte
            if(campo.toLowerCase().includes('data')) return (''+va).localeCompare(''+vb);
            if(typeof va === 'number' && typeof vb === 'number') return va - vb;
            return (''+va).localeCompare(''+vb);
        });
        ordemAtual = campo;
    }
    localStorage.setItem("clientes", JSON.stringify(clientes));
    atualizarTabela();
}

/* ---- EXPORTAR PDF (abre impressão) ---- */
function exportarPDF(){
    let conteudo = document.querySelector('#clientes .historico')?.innerHTML || document.querySelector('.historico')?.innerHTML || '';
    if(!conteudo) conteudo = document.querySelector('#tabelaClientes')?.outerHTML || '';
    let win = window.open('', '', 'width=900,height=700');
    win.document.write('<html><head><title>Relatório de Clientes</title></head><body style="font-family:Arial,sans-serif;padding:20px;">' + conteudo + '</body></html>');
    win.document.close();
    win.print();
}

/* ---- UTILIDADES ---- */
function limparFiltros(){
    document.getElementById('buscar').value = '';
    document.getElementById('filtroStatus').value = 'todos';
    atualizarTabela();
}

/* ---- INICIALIZAÇÃO ---- */
atualizarCalculo();
atualizarTabela();
calcularTotais();
</script>

</body>
</html>


