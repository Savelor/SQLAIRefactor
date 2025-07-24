
<div style="display: flex; flex-wrap: wrap; gap: 16px;">

  <div style="flex: 1; min-width: 300px;">
    <h4>🔹 Query A – Utenti attivi</h4>
    <pre><code class="language-sql">
SELECT id, nome, email
FROM utenti
WHERE attivo = 1
ORDER BY nome;
    </code></pre>
  </div>

  <div style="flex: 1; min-width: 300px;">
    <h4>🔹 Query B – Clienti iscritti</h4>
    <pre><code class="language-sql">
SELECT nome, email
FROM clienti
WHERE iscritti = 1
ORDER BY data_iscrizione DESC;
    </code></pre>
  </div>

</div>
