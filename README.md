# Scheda-contabile-S-B
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestione Finanze Associazione</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
    </style>
</head>
<body class="bg-slate-50 min-h-screen text-slate-900">

    <div class="max-w-5xl mx-auto px-4 py-8">
        <header class="mb-8">
            <h1 class="text-3xl font-bold text-slate-800">Associazione Tesoreria</h1>
            <p class="text-slate-500">Monitora entrate e uscite con riferimenti documentali.</p>
        </header>

        <!-- Statistiche -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
                <p class="text-sm text-slate-500 mb-1">Entrate Totali</p>
                <h3 id="total-income" class="text-2xl font-bold text-emerald-600">€ 0,00</h3>
            </div>
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
                <p class="text-sm text-slate-500 mb-1">Uscite Totali</p>
                <h3 id="total-expense" class="text-2xl font-bold text-rose-600">€ 0,00</h3>
            </div>
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
                <p class="text-sm text-slate-500 mb-1">Saldo Attuale</p>
                <h3 id="net-balance" class="text-2xl font-bold text-slate-800">€ 0,00</h3>
            </div>
        </div>

        <!-- Form di inserimento -->
        <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 mb-8">
            <h2 class="text-lg font-semibold mb-4">Nuova Operazione</h2>
            <form id="transaction-form" class="grid grid-cols-1 md:grid-cols-5 gap-4">
                <input type="text" id="desc" placeholder="Descrizione (es. Affitto)" required class="col-span-1 md:col-span-1 p-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none">
                <input type="text" id="ref" placeholder="Rif. (es. Scontrino #123)" class="p-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none">
                <select id="type" class="p-3 rounded-xl border border-slate-200">
                    <option value="income">Entrata</option>
                    <option value="expense">Uscita</option>
                </select>
                <select id="method" class="p-3 rounded-xl border border-slate-200">
                    <option value="Contanti">Contanti</option>
                    <option value="Carta">Carta di Credito</option>
                    <option value="Assegno">Assegno</option>
                </select>
                <input type="number" id="amount" placeholder="Importo (€)" step="0.01" required class="p-3 rounded-xl border border-slate-200 focus:ring-2 focus:ring-blue-500 outline-none">
                <button type="submit" class="bg-blue-600 text-white font-semibold py-3 rounded-xl hover:bg-blue-700 transition">Aggiungi</button>
            </form>
        </div>

        <!-- Lista Transazioni -->
        <div class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="p-6 border-b border-slate-100">
                <h2 class="text-lg font-semibold">Storico Movimenti</h2>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left">
                    <thead class="bg-slate-50 text-slate-500 uppercase text-xs font-semibold">
                        <tr>
                            <th class="px-6 py-4">Descrizione</th>
                            <th class="px-6 py-4">Riferimento</th>
                            <th class="px-6 py-4">Metodo</th>
                            <th class="px-6 py-4">Tipo</th>
                            <th class="px-6 py-4">Importo</th>
                            <th class="px-6 py-4 text-center">Azione</th>
                        </tr>
                    </thead>
                    <tbody id="transaction-list" class="divide-y divide-slate-100">
                        <!-- Le righe verranno iniettate qui -->
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        let transactions = [];

        const form = document.getElementById('transaction-form');
        const list = document.getElementById('transaction-list');
        const incomeEl = document.getElementById('total-income');
        const expenseEl = document.getElementById('total-expense');
        const balanceEl = document.getElementById('net-balance');

        function updateUI() {
            // Calcolo totali
            const income = transactions
                .filter(t => t.type === 'income')
                .reduce((acc, t) => acc + t.amount, 0);
            const expense = transactions
                .filter(t => t.type === 'expense')
                .reduce((acc, t) => acc + t.amount, 0);
            
            incomeEl.textContent = `€ ${income.toFixed(2)}`;
            expenseEl.textContent = `€ ${expense.toFixed(2)}`;
            balanceEl.textContent = `€ ${(income - expense).toFixed(2)}`;

            // Render lista
            list.innerHTML = '';
            transactions.forEach((t, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td class="px-6 py-4 font-medium">${t.desc}</td>
                    <td class="px-6 py-4 text-slate-500 text-sm italic">${t.ref || '-'}</td>
                    <td class="px-6 py-4 text-sm text-slate-600">${t.method}</td>
                    <td class="px-6 py-4">
                        <span class="px-2 py-1 rounded-full text-xs font-semibold ${t.type === 'income' ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}">
                            ${t.type === 'income' ? 'Entrata' : 'Uscita'}
                        </span>
                    </td>
                    <td class="px-6 py-4 font-bold ${t.type === 'income' ? 'text-emerald-600' : 'text-rose-600'}">
                        ${t.type === 'income' ? '+' : '-'} € ${t.amount.toFixed(2)}
                    </td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteTransaction(${index})" class="text-slate-400 hover:text-rose-600 transition">
                            <i data-lucide="trash-2" class="w-5 h-5"></i>
                        </button>
                    </td>
                `;
                list.appendChild(row);
            });
            
            // Re-inizializza icone
            lucide.createIcons();
        }

        function deleteTransaction(index) {
            transactions.splice(index, 1);
            updateUI();
        }

        form.addEventListener('submit', (e) => {
            e.preventDefault();
            const desc = document.getElementById('desc').value;
            const ref = document.getElementById('ref').value;
            const type = document.getElementById('type').value;
            const method = document.getElementById('method').value;
            const amount = parseFloat(document.getElementById('amount').value);

            if(amount > 0) {
                transactions.push({ desc, ref, type, method, amount });
                form.reset();
                updateUI();
            }
        });

        // Inizializzazione icone al caricamento
        lucide.createIcons();
    </script>
</body>
</html>
