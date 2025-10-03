<html lang="pt-BR">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#2b6cb0" />
    <link rel="manifest" href="manifest.webmanifest" />
    <title>Calculadora de Recibo - Fernanda Dantas</title>
    <style>
      :root {
        --bg: #f7fafc;
        --card: #ffffff;
        --text: #1a202c;
        --muted: #4a5568;
        --primary: #2b6cb0;
        --accent: #38b2ac;
        --border: #e2e8f0;
      }

      * { box-sizing: border-box; }
      body {
        margin: 0;
        font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', Arial, sans-serif;
        background: var(--bg);
        color: var(--text);
      }
      header {
        background: linear-gradient(135deg, var(--primary), var(--accent));
        color: white;
        padding: 28px 20px;
      }
      header h1 { margin: 0 0 6px; font-size: 24px; }
      header p { margin: 0; opacity: 0.9; }

      .container { max-width: 1000px; margin: 24px auto; padding: 0 16px; }
      .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
      @media (max-width: 860px) { .grid { grid-template-columns: 1fr; } }

      .card {
        background: var(--card);
        border: 1px solid var(--border);
        border-radius: 12px;
        padding: 16px;
        box-shadow: 0 2px 10px rgba(0,0,0,0.04);
      }
      .card h2 { margin: 0; font-size: 18px; display: flex; align-items: center; justify-content: space-between; gap: 12px; }
      .card-content { margin-top: 12px; }
      .card.collapsed .card-content { display: none; }
      .card-toggle { font-size: 12px; padding: 6px 10px; border: 1px solid var(--border); border-radius: 8px; background: #fff; color: var(--text); cursor: pointer; }
      .card-toggle:hover { background: #f0f4f8; }
      .hint { color: var(--muted); font-size: 13px; }

      .item { display: grid; grid-template-columns: auto 1fr auto 90px; gap: 10px; align-items: center; padding: 8px 0; border-bottom: 1px dashed var(--border); }
      .item:last-child { border-bottom: 0; }
      .price { color: var(--muted); }
      .qty { width: 80px; }
      .qty input { width: 100%; padding: 6px 8px; border: 1px solid var(--border); border-radius: 8px; }
      .price-edit {
        width: 120px;
        padding: 8px 10px;
        border: 1px solid var(--border);
        border-radius: 10px;
        background: #fff;
        color: var(--text);
      }
      .price-edit:focus, .qty input:focus, textarea:focus {
        outline: none;
        border-color: var(--accent);
        box-shadow: 0 0 0 3px rgba(38, 178, 172, 0.25);
      }

      .summary {
        background: var(--card);
        border: 2px solid var(--border);
        border-radius: 12px;
        padding: 16px;
        position: sticky;
        top: 16px;
      }
      .summary h3 { margin: 0 0 8px; font-size: 18px; }
      .summary-list { margin: 0; padding-left: 18px; color: var(--muted); }
      .summary-list li { margin: 4px 0; }
      .total { display: flex; justify-content: space-between; align-items: center; margin-top: 12px; padding-top: 12px; border-top: 1px solid var(--border); }
      .total-amount { font-weight: 700; font-size: 20px; }

      .actions { display: flex; gap: 10px; margin-top: 12px; }
      button {
        padding: 10px 14px;
        border: 1px solid var(--border);
        border-radius: 10px;
        background: #fff;
        cursor: pointer;
      }
      button.primary { background: var(--primary); color: white; border-color: var(--primary); }
      button.secondary { background: #fff; color: var(--text); }

      .note { margin-top: 12px; }
      textarea { width: 100%; min-height: 80px; padding: 10px; border: 1px solid var(--border); border-radius: 10px; }
      .footnote { margin-top: 8px; font-size: 12px; color: var(--muted); }
      /* Form inputs gerais */
      .form-row { display: grid; grid-template-columns: 140px 1fr; gap: 10px; align-items: center; padding: 6px 0; }
      .text-input, select { width: 100%; padding: 8px 10px; border: 1px solid var(--border); border-radius: 10px; background: #fff; color: var(--text); }
      .text-input:focus, select:focus { outline: none; border-color: var(--accent); box-shadow: 0 0 0 3px rgba(38, 178, 172, 0.25); }

      /* Recibo (print-only) */
      .recibo { display: none; padding: 24px; color: #000; }
      .recibo-header h1 { margin: 0 0 6px; font-size: 22px; }
      .recibo-header p { margin: 4px 0; }
      .recibo-info p { margin: 2px 0; }
      .recibo-table { width: 100%; border-collapse: collapse; margin-top: 12px; }
      .recibo-table th, .recibo-table td { border: 1px solid #000; padding: 8px; }
      .recibo-table th { background: #f0f0f0; text-align: left; }
      .recibo-total { margin-top: 12px; text-align: right; font-weight: 700; }

      @media print {
        header, .container { display: none !important; }
        .recibo { display: block; }
        body { background: #fff; }
      }
    </style>
  </head>
  <body>
    <header>
      <h1>Fernanda Dantas — Calculadora de Recibo</h1>
      <p>Selecione os serviços realizados para calcular o valor da consulta.</p>
    </header>

    <div class="container">
      <div class="grid">
        <!-- Coluna esquerda: seleção -->
        <section class="card" aria-labelledby="consultas-title">
          <h2 id="consultas-title">Consultas <button class="card-toggle" aria-expanded="true" aria-controls="consultas-content">Recolher</button></h2>
          <div id="consultas-content" class="card-content">
          <p class="hint">Escolha o tipo de consulta.</p>

          <div class="item">
            <input type="radio" name="consulta" id="consulta-geral" data-label="Consulta geral" data-price="120" checked />
            <label for="consulta-geral">Consulta geral</label>
            <div class="price">R$ 120,00</div>
            <div></div>
          </div>

          <div class="item">
            <input type="radio" name="consulta" id="consulta-retorno" data-label="Consulta de retorno" data-price="0" />
            <label for="consulta-retorno">Consulta de retorno</label>
            <div class="price">R$ 0,00</div>
            <div></div>
          </div>

          <div class="item">
            <input type="radio" name="consulta" id="consulta-emergencia" data-label="Atendimento de emergência" data-price="200" />
            <label for="consulta-emergencia">Atendimento de emergência</label>
            <div class="price">R$ 200,00</div>
            <div></div>
          </div>
          </div>
        </section>

        <section class="card" aria-labelledby="exames-title">
          <h2 id="exames-title">Exames <button class="card-toggle" aria-expanded="true" aria-controls="exames-content">Recolher</button></h2>
          <div id="exames-content" class="card-content">
          <p class="hint">Marque os exames realizados e ajuste o valor e a quantidade quando necessário.</p>

          <div class="item">
            <input type="checkbox" class="service" id="ex-hemograma" data-label="Hemograma completo" data-cost="65" data-price="90" />
            <label for="ex-hemograma">Hemograma completo</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="90"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="ex-painel" data-label="Painel completo (hemograma + 4 bioquímicas)" data-cost="160" data-price="160" />
            <label for="ex-painel">Painel completo</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="160"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="ex-bioq" data-label="Bioquímica (cada)" data-cost="35" data-price="35" />
            <label for="ex-bioq">Bioquímica</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="35"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="ex-citologia" data-label="Citologia (valor variável)" data-price="110" data-editable="true" />
            <label for="ex-citologia">Citologia</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="110"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>
          </div>
        </section>

        <section class="card" aria-labelledby="vacinas-title">
          <h2 id="vacinas-title">Vacinas <button class="card-toggle" aria-expanded="true" aria-controls="vacinas-content">Recolher</button></h2>
          <div id="vacinas-content" class="card-content">
          <p class="hint">Marque as vacinas realizadas e ajuste o valor e a quantidade quando necessário.</p>

          <div class="item">
            <input type="checkbox" class="service" id="vac-v5" data-label="V5" data-cost="59" data-price="155" />
            <label for="vac-v5">V5</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="155"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-v4-domicilio" data-label="V4 (domicílio)" data-cost="45.63" data-price="120" />
            <label for="vac-v4-domicilio">V4</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="120"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-gripe-oral" data-label="Gripe oral" data-cost="39" data-price="95" />
            <label for="vac-gripe-oral">Gripe oral</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="95"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-giardia" data-label="Giardia" data-cost="50" data-price="135" />
            <label for="vac-giardia">Giardia</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="135"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-antirrabica" data-label="Antirrábica" data-cost="18" data-price="85" />
            <label for="vac-antirrabica">Antirrábica</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="85"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-v10-recombitek" data-label="V10 Recombitek" data-cost="40.8" data-price="115" />
            <label for="vac-v10-recombitek">V10 Recombitek</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="115"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-v10-vanguard" data-label="V10 Vanguard" data-cost="47" data-price="115" />
            <label for="vac-v10-vanguard">V10 Vanguard</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="115"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="vac-v8-nobivac" data-label="Nobivac DHPPi (V8)" data-cost="38" data-price="105" />
            <label for="vac-v8-nobivac">Nobivac DHPPi (V8)</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="105"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>
          </div>
        </section>

        <section class="card" aria-labelledby="testes-title">
          <h2 id="testes-title">Testes rápidos <button class="card-toggle" aria-expanded="true" aria-controls="testes-content">Recolher</button></h2>
          <div id="testes-content" class="card-content">
          <p class="hint">Marque os testes rápidos realizados e ajuste o valor e a quantidade quando necessário.</p>

          <div class="item">
            <input type="checkbox" class="service" id="tst-fiv-felv" data-label="Teste FIV e FeLV" data-cost="68" data-price="155" />
            <label for="tst-fiv-felv">Teste FIV e FeLV</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="155"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="tst-parvo" data-label="Teste parvovirose" data-price="0" data-editable="true" />
            <label for="tst-parvo">Teste parvovirose</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="0" placeholder="Defina o preço"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="tst-cinomose" data-label="Teste cinomose" data-price="0" data-editable="true" />
            <label for="tst-cinomose">Teste cinomose</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="0" placeholder="Defina o preço"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="tst-erliquia-anaplasma" data-label="Teste de erliquia e anaplasma" data-cost="52" data-price="120" />
            <label for="tst-erliquia-anaplasma">Teste de erliquia e anaplasma</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="120"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>

          <div class="item">
            <input type="checkbox" class="service" id="tst-leish" data-label="Teste de leish" data-price="0" data-editable="true" />
            <label for="tst-leish">Teste de leish</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="0" placeholder="Defina o preço"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>
          </div>
        </section>

        <section class="card" aria-labelledby="aplicacoes-title">
          <h2 id="aplicacoes-title">Aplicações e medicações <button class="card-toggle" aria-expanded="true" aria-controls="aplicacoes-content">Recolher</button></h2>
          <div id="aplicacoes-content" class="card-content">
          <p class="hint">Marque as aplicações realizadas e ajuste o valor e a quantidade quando necessário..</p>

          <div class="item">
            <input type="checkbox" class="service" id="apl-dipirona-10kg" data-label="Dipirona (até 10kg)" data-cost="30" data-price="32" />
            <label for="apl-dipirona-10kg">Dipirona (até 10kg)</label>
            <div class="price"><input class="price-edit" type="number" min="0" step="0.01" value="32"></div>
            <div class="qty"><input type="number" min="1" value="1"></div>
          </div>
          </div>
        </section>
      </div>

      <div class="grid" style="margin-top: 16px;">
        <section class="card" aria-labelledby="dados-title">
          <h2 id="dados-title">Dados do recibo <button class="card-toggle" aria-expanded="true" aria-controls="dados-content">Recolher</button></h2>
          <div id="dados-content" class="card-content">
            <div class="form-row">
              <label for="cliente-nome">Responsável</label>
              <input id="cliente-nome" class="text-input" type="text" placeholder="Nome do responsável" />
            </div>
            <div class="form-row">
              <label for="animal-nome">Animal</label>
              <input id="animal-nome" class="text-input" type="text" placeholder="Nome do animal" />
            </div>
            <div class="form-row">
              <label for="forma-pagamento">Forma de pagamento</label>
              <select id="forma-pagamento" class="text-input">
                <option>Dinheiro</option>
                <option>PIX</option>
                <option>Cartão de crédito</option>
                <option>Cartão de débito</option>
              </select>
            </div>
            <div class="form-row">
              <label for="status-pagamento">Status</label>
              <select id="status-pagamento" class="text-input">
                <option>Pago</option>
                <option>Pendente</option>
              </select>
            </div>
          </div>
        </section>
        <section class="card" aria-labelledby="obs-title">
          <h2 id="obs-title">Observações <button class="card-toggle" aria-expanded="true" aria-controls="obs-content">Recolher</button></h2>
          <div id="obs-content" class="card-content">
          <div class="note">
            <textarea id="observacoes" placeholder="Inclua observações relevantes sobre o atendimento..."></textarea>
            <p class="footnote">Valores indicativos. O orçamento final pode variar conforme a avaliação clínica.</p>
          </div>
          </div>
        </section>

        <aside class="summary" aria-live="polite">
          <h3>Resumo do orçamento</h3>
          <ul id="resumo" class="summary-list"></ul>
          <div class="total">
            <div>Total</div>
            <div id="total" class="total-amount">R$ 0,00</div>
          </div>
          <div class="actions">
            <button class="primary" id="btn-imprimir">Imprimir recibo</button>
            <button class="secondary" id="btn-reset">Limpar seleção</button>
          </div>
        </aside>
      </div>
    </div>

    <!-- Recibo para impressão -->
    <section id="recibo-print" class="recibo" aria-hidden="true">
      <div class="recibo-header">
        <h1>Recibo</h1>
        <p><strong>Fernanda Dantas</strong></p>
        <p id="recibo-id"></p>
        <p id="recibo-data"></p>
      </div>
      <div class="recibo-info">
        <p><strong>Tutor:</strong> <span id="recibo-cliente">—</span></p>
        <p><strong>Animal:</strong> <span id="recibo-animal">—</span></p>
        <p><strong>Forma de pagamento:</strong> <span id="recibo-pagamento">—</span></p>
        <p><strong>Status:</strong> <span id="recibo-status">—</span></p>
      </div>
      <table class="recibo-table">
        <thead>
          <tr>
            <th>Serviço</th>
            <th>Qtd</th>
            <th>Valor unitário</th>
            <th>Subtotal</th>
          </tr>
        </thead>
        <tbody id="recibo-items"></tbody>
      </table>
      <div class="recibo-total">Total: <span id="recibo-total"></span></div>
      <div class="recibo-observacoes" style="margin-top: 12px;">
        <strong>Observações</strong>
        <p id="recibo-observacoes"></p>
      </div>
    </section>

    <script>
      const brl = new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' });

      function getSelectedConsulta() {
        const selected = document.querySelector('input[name="consulta"]:checked');
        if (!selected) return null;
        return {
          label: selected.dataset.label,
          price: Number(selected.dataset.price),
          qty: 1
        };
      }

      function getSelectedServicos() {
        return Array.from(document.querySelectorAll('input[type="checkbox"].service'))
          .filter(el => el.checked)
          .map(el => {
            const item = el.closest('.item');
            const qtyInput = item.querySelector('.qty input');
            const priceEdit = item.querySelector('.price-edit');
            const qty = Number(qtyInput?.value || 1);
            const price = priceEdit ? Number(priceEdit.value || el.dataset.price || 0) : Number(el.dataset.price || 0);
            return { label: el.dataset.label, price, qty };
          });
      }

      function updateQtyState() {
        // Deixa todos os campos editáveis, sem travar por seleção
        document.querySelectorAll('input[type="checkbox"].service').forEach(cb => {
          const item = cb.closest('.item');
          const qtyInput = item.querySelector('.qty input');
          const priceEdit = item.querySelector('.price-edit');
          if (qtyInput) qtyInput.disabled = false;
          if (priceEdit) priceEdit.disabled = false;
        });
      }

      function renderResumo(items) {
        const ul = document.getElementById('resumo');
        ul.innerHTML = '';
        items.forEach(({ label, price, qty }) => {
          const li = document.createElement('li');
          li.textContent = `${label} — ${qty} × ${brl.format(price)} = ${brl.format(price * qty)}`;
          ul.appendChild(li);
        });
        if (items.length === 0) {
          const li = document.createElement('li');
          li.textContent = 'Nenhum serviço selecionado.';
          ul.appendChild(li);
        }
      }

      function updateTotal() {
        const items = [];
        const consulta = getSelectedConsulta();
        if (consulta) items.push(consulta);
        const servicos = getSelectedServicos();
        items.push(...servicos);

        const total = items.reduce((sum, it) => sum + it.price * it.qty, 0);
        document.getElementById('total').textContent = brl.format(total);
        renderResumo(items);
      }

      function buildRecibo() {
        const items = [];
        const consulta = getSelectedConsulta();
        if (consulta) items.push(consulta);
        const servicos = getSelectedServicos();
        items.push(...servicos);

        const tbody = document.getElementById('recibo-items');
        tbody.innerHTML = '';
        items.forEach(({ label, price, qty }) => {
          const tr = document.createElement('tr');
          const subtotal = price * qty;
          tr.innerHTML = `<td>${label}</td><td>${qty}</td><td>${brl.format(price)}</td><td>${brl.format(subtotal)}</td>`;
          tbody.appendChild(tr);
        });

        const total = items.reduce((sum, it) => sum + it.price * it.qty, 0);
        document.getElementById('recibo-total').textContent = brl.format(total);

        const now = new Date();
        function nextReciboId() {
          const raw = localStorage.getItem('reciboCounter');
          let n = parseInt(raw || '0', 10);
          if (!Number.isFinite(n)) n = 0;
          n += 1;
          localStorage.setItem('reciboCounter', String(n));
          return n;
        }
        const seq = nextReciboId();
        document.getElementById('recibo-id').textContent = `Recibo #${seq}`;
        document.getElementById('recibo-data').textContent = now.toLocaleString('pt-BR');
        document.getElementById('recibo-observacoes').textContent = document.getElementById('observacoes').value || '—';

        const cliente = (document.getElementById('cliente-nome')?.value || '').trim();
        const animal = (document.getElementById('animal-nome')?.value || '').trim();
        const pagamento = document.getElementById('forma-pagamento')?.value || '';
        const status = document.getElementById('status-pagamento')?.value || '';
        document.getElementById('recibo-cliente').textContent = cliente || '—';
        document.getElementById('recibo-animal').textContent = animal || '—';
        document.getElementById('recibo-pagamento').textContent = pagamento || '—';
        document.getElementById('recibo-status').textContent = status || '—';
      }

      function initCardToggles() {
        document.querySelectorAll('.card').forEach(card => {
          const toggle = card.querySelector('.card-toggle');
          const contentId = toggle?.getAttribute('aria-controls');
          const content = contentId ? document.getElementById(contentId) : card.querySelector('.card-content');
          if (!toggle || !content) return;
          toggle.addEventListener('click', () => {
            const collapsed = card.classList.toggle('collapsed');
            toggle.setAttribute('aria-expanded', (!collapsed).toString());
            toggle.textContent = collapsed ? 'Expandir' : 'Recolher';
          });
        });
      }

      function bindEvents() {
        // Consultas
        document.querySelectorAll('input[name="consulta"]').forEach(r => {
          r.addEventListener('change', updateTotal);
        });
        // Serviços (exames, vacinas, testes) + quantidade/preço editável
        document.querySelectorAll('input[type="checkbox"].service').forEach(cb => {
          cb.addEventListener('change', () => { updateQtyState(); updateTotal(); });
          const item = cb.closest('.item');
          const qtyInput = item.querySelector('.qty input');
          if (qtyInput) qtyInput.addEventListener('input', updateTotal);
          const priceEdit = item.querySelector('.price-edit');
          if (priceEdit) priceEdit.addEventListener('input', updateTotal);
        });
        // Ações
        document.getElementById('btn-imprimir').addEventListener('click', () => { buildRecibo(); window.print(); });
        document.getElementById('btn-reset').addEventListener('click', () => {
          document.querySelector('#consulta-geral').checked = true;
          document.querySelectorAll('input[type="checkbox"].service').forEach(cb => cb.checked = false);
          updateQtyState();
          updateTotal();
        });
      }

      // Inicialização
      bindEvents();
      initCardToggles();
      updateQtyState();
      updateTotal();

      // PWA: registra service worker
      if ('serviceWorker' in navigator) {
        window.addEventListener('load', () => {
          navigator.serviceWorker.register('./sw.js').catch(console.error);
        });
      }
    </script>
  </body>
</html>
