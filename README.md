<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Apostas</title>
    <style>
        /* Estilo Geral */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }

        header {
            background-color: #004d99;
            color: #fff;
            padding: 1rem;
            text-align: center;
        }

        header h1 {
            margin: 0;
        }

        main {
            padding: 2rem;
        }

        form {
            margin-bottom: 2rem;
        }

        label {
            display: block;
            margin-bottom: 0.5rem;
        }

        input, select {
            display: block;
            margin-bottom: 1rem;
            padding: 0.5rem;
            width: 100%;
            box-sizing: border-box;
        }

        button {
            background-color: #004d99;
            color: #fff;
            border: none;
            padding: 0.5rem 1rem;
            cursor: pointer;
        }

        button:hover {
            background-color: #003366;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        table, th, td {
            border: 1px solid #ddd;
        }

        th, td {
            padding: 0.5rem;
            text-align: left;
        }

        th {
            background-color: #004d99;
            color: #fff;
        }

        #summary {
            margin-top: 2rem;
        }

        /* Media Queries */
        @media (max-width: 768px) {
            /* Estilos para dispositivos móveis */
            header {
                padding: 1rem;
                text-align: center;
            }

            main {
                padding: 1rem;
            }

            button {
                padding: 0.5rem;
                font-size: 0.9rem;
            }

            table, th, td {
                font-size: 0.9rem;
            }
        }

        @media (max-width: 480px) {
            /* Estilos para telas muito pequenas */
            header h1 {
                font-size: 1.5rem;
            }

            main {
                padding: 0.5rem;
            }

            button {
                width: 100%;
                padding: 0.75rem;
                font-size: 1rem;
            }

            table, th, td {
                font-size: 0.8rem;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Calculadora de Apostas</h1>
    </header>
    <main>
        <form id="calculatorForm">
            <div>
                <label for="initialAmount">Valor da Aposta Inicial (R$):</label>
                <input type="number" id="initialAmount" step="0.01" min="0" required>
            </div>
            <div>
                <label for="odds">Odds:</label>
                <input type="number" id="odds" step="0.01" min="1" required>
            </div>
            <div>
                <label for="entries">Número de Entradas:</label>
                <input type="number" id="entries" min="1" required>
            </div>
            <button type="submit">Calcular</button>
            <button type="button" id="clearButton">Limpar</button>
        </form>
        <div id="result">
            <h2>Resultado</h2>
            <table id="resultTable">
                <thead>
                    <tr>
                        <th>Selecione</th>
                        <th>Entrada</th>
                        <th>Valor da Aposta</th>
                        <th>Odds</th>
                        <th>Retorno</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Linhas da tabela de resultados -->
                </tbody>
            </table>
            <p>Lucro Total: <span id="totalProfit">R$0.00</span></p>
        </div>
    </main>
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            document.getElementById('calculatorForm').addEventListener('submit', function(event) {
                event.preventDefault();

                // Obtém e valida os valores do formulário
                const initialAmount = parseFloat(document.getElementById('initialAmount').value);
                const odds = parseFloat(document.getElementById('odds').value);
                const entries = parseInt(document.getElementById('entries').value);

                // Verifica se os valores são válidos
                if (isNaN(initialAmount) || initialAmount <= 0) {
                    alert('O valor da aposta inicial deve ser um número positivo.');
                    return;
                }
                if (isNaN(odds) || odds <= 1) {
                    alert('As odds devem ser um número maior que 1.');
                    return;
                }
                if (isNaN(entries) || entries <= 0) {
                    alert('O número de entradas deve ser um número inteiro positivo.');
                    return;
                }

                let currentAmount = initialAmount;
                let totalProfit = 0;
                let totalAmountApostado = 0;
                const resultTableBody = document.querySelector('#resultTable tbody');
                const bets = [];

                resultTableBody.innerHTML = ''; // Limpa a tabela antes de adicionar novas linhas

                // Calcula os resultados
                for (let i = 0; i < entries; i++) {
                    const retorno = currentAmount * odds;
                    totalAmountApostado += currentAmount;
                    const lucro = retorno - currentAmount;

                    totalProfit += lucro;

                    // Adiciona o cálculo ao array de apostas
                    bets.push({
                        currentAmount,
                        odds,
                        retorno,
                        checked: false // Inicialmente sem marcado
                    });

                    // Cria e adiciona a linha à tabela
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td><input type="checkbox" class="resultCheckbox" data-index="${i}"></td>
                        <td>${i + 1}</td>
                        <td>${formatCurrency(currentAmount)}</td>
                        <td>${odds.toFixed(2)}</td>
                        <td>${formatCurrency(retorno)}</td>
                    `;
                    resultTableBody.appendChild(row);

                    currentAmount = retorno; // Reinveste o retorno na próxima aposta
                }

                // Atualiza o lucro total
                document.getElementById('totalProfit').textContent = formatCurrency(totalProfit);

                // Salva os resultados no localStorage
                localStorage.setItem('calculatedBets', JSON.stringify(bets));
            });

            // Função para formatar valores como moeda
            function formatCurrency(value) {
                return `R$${value.toFixed(2).replace('.', ',')}`;
            }

            // Recupera dados do localStorage e atualiza a tabela
            function loadPreviousResults() {
                const storedData = localStorage.getItem('calculatedBets');
                if (storedData) {
                    const bets = JSON.parse(storedData);
                    updateTableAndSummary(bets);
                }
            }

            // Atualiza a tabela com os dados armazenados
            function updateTableAndSummary(bets) {
                const resultTableBody = document.querySelector('#resultTable tbody');
                resultTableBody.innerHTML = '';
                let totalProfit = 0;

                bets.forEach((bet, index) => {
                    const lucro = bet.retorno - bet.currentAmount;

                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td><input type="checkbox" class="resultCheckbox" ${bet.checked ? 'checked' : ''} data-index="${index}"></td>
                        <td>${index + 1}</td>
                        <td>${formatCurrency(bet.currentAmount)}</td>
                        <td>${bet.odds.toFixed(2)}</td>
                        <td>${formatCurrency(bet.retorno)}</td>
                    `;
                    resultTableBody.appendChild(row);

                    totalProfit += lucro;
                });

                document.getElementById('totalProfit').textContent = formatCurrency(totalProfit);
            }

            // Adiciona evento de clique no botão de limpar
            document.getElementById('clearButton').addEventListener('click', function() {
                localStorage.removeItem('calculatedBets');
                document.querySelector('#resultTable tbody').innerHTML = '';
                document.getElementById('totalProfit').textContent = 'R$0.00';
                document.getElementById('calculatorForm').reset();
            });

            // Adiciona evento de mudança nos checkboxes
            document.querySelector('#resultTable tbody').addEventListener('change', function(event) {
                if (event.target.classList.contains('resultCheckbox')) {
                    const index = event.target.getAttribute('data-index');
                    const storedData = localStorage.getItem('calculatedBets');
                    if (storedData) {
                        const bets = JSON.parse(storedData);
                        bets[index].checked = event.target.checked;
                        localStorage.setItem('calculatedBets', JSON.stringify(bets));
                    }
                }
            });

            loadPreviousResults(); // Carrega resultados anteriores ao iniciar
        });
    </script>
</body>
</html>
