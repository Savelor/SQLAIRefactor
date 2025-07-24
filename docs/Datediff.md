
<table>
  <tr>
    <td style="vertical-align: top; padding-right: 16px;">

<h4>🔹 Query A – Utenti attivi</h4>

<pre><code>
SELECT id, nome, email
FROM utenti
WHERE attivo = 1
ORDER BY nome;
</code></pre>

    </td>
    <td style="vertical-align: top; padding-left: 16px;">

<h4>🔹 Query B – Clienti iscritti</h4>

<pre><code>
SELECT nome, email
FROM clienti
WHERE iscritti = 1
ORDER BY data_iscrizione DESC;
</code></pre>

    </td>
  </tr>
</table>

