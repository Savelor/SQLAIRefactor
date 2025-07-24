
<table>
  <tr> <!-- Riga 1 -->
    <td>
      <h4>Query A – Utenti attivi</h4>
      <pre><code>
   
SELECT id, nome, email
FROM utenti
WHERE attivo = 1
ORDER BY nome;
      </code></pre>
    </td>
    <td>
      <h4>Query B – Utenti attivi</h4>
      <pre><code>

SELECT nome, email
FROM clienti
WHERE iscritti = 1
ORDER BY data_iscrizione DESC;
</code></pre>
    </td>
  </tr>
</table>

![Time_1](https://github.com/user-attachments/assets/914fde7b-710f-4c92-87ba-3fbbbcbaa23e)
