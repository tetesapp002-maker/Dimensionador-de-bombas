# Dimensionador-de-bombas
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dimensionador Conab+</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --navy: #1e3a5f;
            --navy-dark: #152a45;
            --green: #22c55e;
            --green-dark: #16a34a;
        }
        body { font-family: 'Inter', sans-serif; }
        .dark { --bg-primary: #1a1a2e; --bg-secondary: #16213e; --text-primary: #ffffff; --text-secondary: #a0aec0; }
        .light { --bg-primary: #f8fafc; --bg-secondary: #ffffff; --text-primary: #1e3a5f; --text-secondary: #64748b; }
        .tab-active { border-bottom: 3px solid #22c55e; color: white; background: rgba(34, 197, 94, 0.1); }
        .pump-card { transition: all 0.3s ease; }
        .pump-card:hover { transform: translateY(-5px); box-shadow: 0 10px 40px rgba(0,0,0,0.15); }
        .filter-input { transition: all 0.2s ease; }
        .filter-input:focus { border-color: #22c55e; box-shadow: 0 0 0 3px rgba(34, 197, 94, 0.2); }
        .btn-primary { background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%); }
        .btn-primary:hover { background: linear-gradient(135deg, #16a34a 0%, #15803d 100%); }
        .excellence { background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%); }
        .good { background: linear-gradient(135deg, #eab308 0%, #ca8a04 100%); }
        .bad { background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%); }
        .border-boa { border: 3px solid #22c55e; }
        .border-media { border: 3px solid #f97316; }
        .border-ruim { border: 3px solid #ef4444; }
        .modal-overlay { backdrop-filter: blur(5px); }
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #1e3a5f; }
        ::-webkit-scrollbar-thumb { background: #22c55e; border-radius: 4px; }
        .compare-badge { animation: pulse 2s infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.7; } }
    </style>
</head>
<body class="light bg-gray-100 min-h-screen">
    <!-- Tela de Login Inicial (obrigatório) -->
    <div id="loginScreen" class="fixed inset-0 bg-gradient-to-br from-[#1e3a5f] to-[#152a45] z-[100] flex items-center justify-center">
        <div class="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-md mx-4">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-bold text-[#1e3a5f]">
                    Dimensionador Conab<span class="text-green-500 text-4xl">+</span>
                </h1>
                <p class="text-gray-500 mt-2">Faça login para acessar o sistema</p>
            </div>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Usuário</label>
                    <input type="text" id="loginUser" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500" placeholder="Digite seu usuário">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Senha</label>
                    <input type="password" id="loginPass" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500" placeholder="Digite sua senha" onkeypress="if(event.key==='Enter')fazerLogin()">
                </div>
                <button onclick="fazerLogin()" class="w-full btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">
                    Entrar
                </button>
                <p id="loginErrorMsg" class="text-red-500 text-center text-sm hidden">Usuário ou senha incorretos</p>
            </div>
            <div class="mt-6 pt-6 border-t text-center text-xs text-gray-400">
                Contate o administrador para obter acesso
            </div>
        </div>
    </div>

    <!-- Header -->
    <header class="bg-gradient-to-r from-[#1e3a5f] to-[#152a45] text-white shadow-xl sticky top-0 z-50">
        <div class="container mx-auto px-4 py-4">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl md:text-3xl font-bold">
                    Dimensionador Conab<span class="text-green-500 text-4xl">+</span>
                </h1>
                <div class="flex items-center gap-4">
                    <!-- Firebase Status -->
                    <div id="firebaseStatus" class="hidden px-3 py-1 rounded-full text-xs font-semibold">
                        <span id="statusIcon">●</span> <span id="statusText">Offline</span>
                    </div>
                    <!-- Dark Mode Toggle -->
                    <button id="themeToggle" class="p-2 rounded-lg bg-white/10 hover:bg-white/20 transition">
                        <svg id="sunIcon" class="w-6 h-6 hidden" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"/>
                        </svg>
                        <svg id="moonIcon" class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"/>
                        </svg>
                    </button>
                    <!-- Compare Badge -->
                    <div id="compareBadge" class="hidden compare-badge bg-green-500 text-white px-3 py-1 rounded-full text-sm font-semibold cursor-pointer" onclick="openCompareModal()">
                        Comparar: <span id="compareCount">0</span>/5
                    </div>
                    <!-- Menu do Usuário -->
                    <div id="userMenu" class="hidden relative">
                        <button onclick="toggleUserDropdown()" class="flex items-center gap-2 bg-white/10 hover:bg-white/20 px-3 py-2 rounded-lg transition">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/>
                            </svg>
                            <span id="userNameDisplay" class="text-sm font-medium">Usuário</span>
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
                            </svg>
                        </button>
                        <div id="userDropdown" class="hidden absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-xl py-2 z-50">
                            <div class="px-4 py-2 border-b">
                                <p class="text-sm font-semibold text-gray-700" id="dropdownUserName">Usuário</p>
                                <p class="text-xs text-gray-500" id="dropdownUserRole">Função</p>
                            </div>
                            <button onclick="fazerLogout()" class="w-full text-left px-4 py-2 text-sm text-red-600 hover:bg-red-50 flex items-center gap-2">
                                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"/>
                                </svg>
                                Sair
                            </button>
                        </div>
                    </div>
                </div>
            </div>
            <!-- Tabs -->
            <nav class="flex mt-4 overflow-x-auto gap-1">
                <button onclick="showTab('dimensionamento')" class="tab-btn tab-active px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition" data-tab="dimensionamento">
                    Dimensionamento
                </button>
                <button onclick="showTab('cadastrar')" class="tab-btn px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition hover:bg-white/10" data-tab="cadastrar">
                    Cadastrar Bomba
                </button>
                <button onclick="showTab('orcamento')" class="tab-btn px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition hover:bg-white/10 relative" data-tab="orcamento">
                    Orçamento
                    <span id="orcamentoCount" class="hidden absolute -top-1 -right-1 bg-green-500 text-xs rounded-full w-5 h-5 flex items-center justify-center">0</span>
                </button>
                <button onclick="showTab('lista')" class="tab-btn px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition hover:bg-white/10" data-tab="lista">
                    Lista de Bombas
                </button>
                <button onclick="showTab('relatorios')" class="tab-btn px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition hover:bg-white/10" data-tab="relatorios">
                    Relatórios
                </button>
                <button onclick="showTab('configuracoes')" class="tab-btn px-4 py-2 rounded-t-lg font-medium whitespace-nowrap transition hover:bg-white/10" data-tab="configuracoes">
                    Configurações
                </button>
            </nav>
        </div>
    </header>

    <main class="container mx-auto px-4 py-6">
        <!-- Tab: Dimensionamento -->
        <section id="tab-dimensionamento" class="tab-content">
            <!-- Input Section -->
            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-4">🔍 Parâmetros de Dimensionamento</h2>
                <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Vazão (m³/h)</label>
                        <input type="number" id="inputVazao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: 15" step="0.1" oninput="buscarCompatibilidade()">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Altura Manométrica (mca)</label>
                        <input type="number" id="inputAltura" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: 25" step="0.1" oninput="buscarCompatibilidade()">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Tolerância (%)</label>
                        <input type="number" id="inputTolerancia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: 10" value="10" step="1" oninput="buscarCompatibilidade()">
                    </div>
                    <div class="flex items-end">
                        <button onclick="buscarCompatibilidade()" class="btn-primary w-full text-white font-semibold py-2 px-4 rounded-lg transition">
                            Buscar
                        </button>
                    </div>
                </div>
            </div>

            <!-- Filters Section -->
            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <div class="flex items-center justify-between mb-4">
                    <h2 class="text-xl font-bold text-[#1e3a5f]">🎛️ Filtros Avançados</h2>
                    <button onclick="limparFiltros()" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg text-sm font-medium transition">
                        Limpar Filtros
                    </button>
                </div>
                <div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-6 gap-4">
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Código</label>
                        <input type="text" id="filterCodigo" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Código" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Marca</label>
                        <select id="filterMarca" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Modelo</label>
                        <input type="text" id="filterModelo" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Modelo" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Potência (CV)</label>
                        <select id="filterPotencia" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Polos</label>
                        <select id="filterPolos" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="2">2 Polos</option>
                            <option value="4">4 Polos</option>
                            <option value="6">6 Polos</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Inversor</label>
                        <select id="filterInversor" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="sim">Com Inversor</option>
                            <option value="nao">Sem Inversor</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Fase</label>
                        <select id="filterFase" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                            <option value="monofasico">Monofásico</option>
                            <option value="trifasico">Trifásico</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Tensão (V)</label>
                        <select id="filterTensao" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                            <option value="127">127V</option>
                            <option value="220">220V</option>
                            <option value="380">380V</option>
                            <option value="440">440V</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Grau Proteção</label>
                        <select id="filterGrauProtecao" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="IP21">IP21</option>
                            <option value="IP44">IP44</option>
                            <option value="IP55">IP55</option>
                            <option value="IP65">IP65</option>
                            <option value="IP68">IP68</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Sucção (mm)</label>
                        <input type="number" id="filterSuccao" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Ø Sucção" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Recalque (mm)</label>
                        <input type="number" id="filterRecalque" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Ø Recalque" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Frequência (Hz)</label>
                        <select id="filterFrequencia" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                            <option value="50">50 Hz</option>
                            <option value="60">60 Hz</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Categoria</label>
                        <select id="filterCategoria" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="togglePressurizacaoFilter()">
                            <option value="">Todas</option>
                            <option value="centrifuga">Centrífuga</option>
                            <option value="submersa">Submersa</option>
                            <option value="autoaspirante">Autoaspirante</option>
                            <option value="multicelular">Multicelular</option>
                            <option value="pressurizacao">Pressurização</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Tipo Rotor</label>
                        <select id="filterTipoRotor" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="fechado">Fechado</option>
                            <option value="aberto">Aberto</option>
                            <option value="semiaberto">Semi-aberto</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Material</label>
                        <select id="filterMaterial" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="ferro">Ferro Fundido</option>
                            <option value="inox">Inox</option>
                            <option value="bronze">Bronze</option>
                            <option value="plastico">Plástico</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Posição</label>
                        <select id="filterPosicao" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todas</option>
                            <option value="horizontal">Horizontal</option>
                            <option value="vertical">Vertical</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Medida Rotor (mm)</label>
                        <input type="number" id="filterMedidaRotor" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Ø Rotor" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Alt. Máx Sucção ≥ (m)</label>
                        <input type="number" id="filterAltMaxSuccao" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Min" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Pressão Máx ≥ (mca)</label>
                        <input type="number" id="filterPressaoMax" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Min" oninput="aplicarFiltros()">
                    </div>
                    <div>
                        <label class="block text-xs font-medium text-gray-600 mb-1">Estoque</label>
                        <select id="filterEstoque" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" onchange="aplicarFiltros()">
                            <option value="">Todos</option>
                            <option value="disponivel">Em Estoque</option>
                            <option value="baixo">Baixo Estoque</option>
                            <option value="esgotado">Esgotado</option>
                        </select>
                    </div>
                </div>
                <!-- Filtros extras para Pressurização -->
                <div id="pressurizacaoFilters" class="hidden mt-4 p-4 bg-blue-50 border border-blue-200 rounded-lg">
                    <h3 class="font-bold text-blue-800 mb-3">Filtros de Sistema de Pressurização</h3>
                    <div class="grid grid-cols-1 gap-4">
                        <div>
                            <label class="block text-xs font-medium text-gray-600 mb-1">Quant. de Bombas do Sistema</label>
                            <input type="number" id="filterQtdBombasOp" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Ex: 2" min="1" oninput="aplicarFiltros()">
                        </div>
                    </div>
                </div>
            </div>

            <!-- Chart Section -->
            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-4">📈 Curvas de Desempenho</h2>
                <div class="relative" style="height: 400px;">
                    <canvas id="curvasChart"></canvas>
                </div>
                <div id="chartLegend" class="mt-4 flex flex-wrap gap-2 justify-center"></div>
            </div>

            <!-- Results Section -->
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex items-center justify-between mb-4">
                    <h2 class="text-xl font-bold text-[#1e3a5f]">🔧 Bombas Encontradas</h2>
                    <span id="resultCount" class="bg-[#1e3a5f] text-white px-3 py-1 rounded-full text-sm">0 resultados</span>
                </div>
                <div id="bombasGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    <!-- Pump cards will be inserted here -->
                </div>
            </div>
        </section>

        <!-- Tab: Cadastrar Bomba -->
        <section id="tab-cadastrar" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">➕ Cadastrar Nova Bomba</h2>
                <form id="formCadastro" class="space-y-6">
                    <!-- Basic Info -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Código *</label>
                            <input type="text" id="cadCodigo" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Marca *</label>
                            <input type="text" id="cadMarca" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Modelo *</label>
                            <input type="text" id="cadModelo" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                        </div>
                    </div>

                    <!-- Technical Specs -->
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Potência (CV) *</label>
                            <input type="number" id="cadPotencia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Polos *</label>
                            <select id="cadPolos" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                                <option value="2">2 Polos</option>
                                <option value="4">4 Polos</option>
                                <option value="6">6 Polos</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Inversor</label>
                            <select id="cadInversor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="nao">Não</option>
                                <option value="sim">Sim</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Fase *</label>
                            <select id="cadFase" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                                <option value="monofasico">Monofásico</option>
                                <option value="trifasico">Trifásico</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tensão (V) *</label>
                            <input type="text" id="cadTensao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: 220V, 380V" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Grau Proteção</label>
                            <select id="cadGrauProtecao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="IP21">IP21</option>
                                <option value="IP44">IP44</option>
                                <option value="IP55">IP55</option>
                                <option value="IP65">IP65</option>
                                <option value="IP68">IP68</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Sucção (mm)</label>
                            <input type="number" id="cadSuccao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Recalque (mm)</label>
                            <input type="number" id="cadRecalque" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Frequência (Hz)</label>
                            <select id="cadFrequencia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="60">60 Hz</option>
                                <option value="50">50 Hz</option>
                                <option value="50/60">50/60 Hz</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Categoria</label>
                            <select id="cadCategoria" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" onchange="togglePressurizacao()">
                                <option value="centrifuga">Centrífuga</option>
                                <option value="submersa">Submersa</option>
                                <option value="autoaspirante">Autoaspirante</option>
                                <option value="multicelular">Multicelular</option>
                                <option value="pressurizacao">Pressurização</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Rotor</label>
                            <select id="cadTipoRotor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="fechado">Fechado</option>
                                <option value="aberto">Aberto</option>
                                <option value="semiaberto">Semi-aberto</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Material</label>
                            <input type="text" id="cadMaterial" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: Ferro Fundido, Inox">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Posição</label>
                            <select id="cadPosicao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="horizontal">Horizontal</option>
                                <option value="vertical">Vertical</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Medida Rotor (mm)</label>
                            <input type="number" id="cadMedidaRotor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Alt. Máx Sucção (m)</label>
                            <input type="number" id="cadAltMaxSuccao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Pressão Máx (mca)</label>
                            <input type="number" id="cadPressaoMax" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                    </div>

                    <!-- Dimensions & Packaging -->
                    <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Comprimento (mm)</label>
                            <input type="number" id="cadComprimento" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Largura (mm)</label>
                            <input type="number" id="cadLargura" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Altura (mm)</label>
                            <input type="number" id="cadAlturaDim" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Peso (kg)</label>
                            <input type="number" id="cadPeso" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Embalagem</label>
                            <select id="cadEmbalagem" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="caixa_papelao">Caixa de Papelão</option>
                                <option value="pallet">Pallet</option>
                                <option value="caixa_madeira">Caixa de Madeira</option>
                                <option value="granel">Granel</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Conexão</label>
                            <select id="cadConexao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="rosca">Rosca</option>
                                <option value="flange">Flange</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Estoque</label>
                            <select id="cadEstoque" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="disponivel">Em Estoque</option>
                                <option value="baixo">Baixo Estoque</option>
                                <option value="esgotado">Esgotado</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Eficiência (%)</label>
                            <input type="number" id="cadEficiencia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1" max="100">
                        </div>
                    </div>

                    <!-- Photo -->
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Foto do Produto</label>
                        <div class="flex items-center gap-4">
                            <input type="file" id="cadFoto" accept="image/*" class="hidden" onchange="previewFoto(event)">
                            <button type="button" onclick="document.getElementById('cadFoto').click()" class="bg-gray-100 hover:bg-gray-200 text-gray-700 px-4 py-2 rounded-lg transition">
                                📷 Selecionar Foto
                            </button>
                            <img id="previewImg" class="w-24 h-24 object-cover rounded-lg hidden border-2 border-gray-300">
                        </div>
                    </div>

                    <!-- Curve Points -->
                    <div id="curvaSection">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Curva de Desempenho (mínimo 2 pontos) *</label>
                        <div id="curvaPoints" class="space-y-2">
                            <div class="flex gap-4 items-center curva-point">
                                <input type="number" placeholder="Vazão (m³/h)" class="curva-vazao filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                                <input type="number" placeholder="Altura (mca)" class="curva-altura filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                                <button type="button" onclick="removerPontoCurva(this)" class="text-red-500 hover:text-red-700 p-2">✕</button>
                            </div>
                            <div class="flex gap-4 items-center curva-point">
                                <input type="number" placeholder="Vazão (m³/h)" class="curva-vazao filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                                <input type="number" placeholder="Altura (mca)" class="curva-altura filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                                <button type="button" onclick="removerPontoCurva(this)" class="text-red-500 hover:text-red-700 p-2">✕</button>
                            </div>
                        </div>
                        <button type="button" onclick="adicionarPontoCurva()" class="mt-2 text-green-600 hover:text-green-700 font-medium">
                            + Adicionar Ponto
                        </button>
                    </div>

                    <!-- Campos Extras para Pressurização -->
                    <div id="pressurizacaoSection" class="hidden space-y-4">
                        <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
                            <h3 class="font-bold text-blue-800 mb-3">Configuração do Sistema de Pressurização</h3>
                            <div class="grid grid-cols-1 gap-4">
                                <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">Quant. de Bombas do Sistema *</label>
                                    <input type="number" id="cadQtdBombasOp" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" min="1" value="1" onchange="gerarCamposBombas()">
                                </div>
                            </div>
                        </div>

                        <!-- Pontos de Trabalho por Bomba -->
                        <div id="pontosPorBomba" class="space-y-4"></div>
                    </div>

                    <div class="flex gap-4">
                        <button type="submit" class="btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">
                            💾 Salvar Bomba
                        </button>
                        <button type="reset" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-8 rounded-lg transition">
                            🔄 Limpar
                        </button>
                    </div>
                </form>
            </div>
        </section>

        <!-- Tab: Orçamento -->
        <section id="tab-orcamento" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">💰 Novo Orçamento</h2>
                <form id="formOrcamento" class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Nome do Cliente *</label>
                        <input type="text" id="orcCliente" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">O.S</label>
                        <input type="text" id="orcOS" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Nº Orçamento</label>
                        <input type="text" id="orcNumero" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Referência (cliente)</label>
                        <input type="text" id="orcReferencia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Setor</label>
                        <input type="text" id="orcSetor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Data</label>
                        <input type="date" id="orcData" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                    </div>
                </form>
            </div>

            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h3 class="text-lg font-bold text-[#1e3a5f] mb-4">🔧 Bombas Selecionadas</h3>
                <div id="orcamentoBombas" class="space-y-4">
                    <p class="text-gray-500 text-center py-8">Nenhuma bomba selecionada. Selecione bombas na aba Dimensionamento.</p>
                </div>
            </div>

            <div class="flex gap-4">
                <button onclick="salvarOrcamento()" class="btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">
                    💾 Salvar Orçamento
                </button>
                <button onclick="exportarOrcamentoPDF()" class="bg-[#1e3a5f] hover:bg-[#152a45] text-white font-semibold py-3 px-8 rounded-lg transition">
                    📄 Exportar PDF
                </button>
            </div>
        </section>

        <!-- Tab: Lista de Bombas -->
        <section id="tab-lista" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex items-center justify-between mb-6">
                    <h2 class="text-xl font-bold text-[#1e3a5f]">📋 Lista de Bombas Cadastradas</h2>
                    <input type="text" id="searchLista" placeholder="Buscar..." class="filter-input px-4 py-2 border border-gray-300 rounded-lg" oninput="filtrarLista()">
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-sm">
                        <thead>
                            <tr class="bg-[#1e3a5f] text-white">
                                <th class="px-4 py-3 text-left rounded-tl-lg">Código</th>
                                <th class="px-4 py-3 text-left">Marca</th>
                                <th class="px-4 py-3 text-left">Modelo</th>
                                <th class="px-4 py-3 text-center">Potência</th>
                                <th class="px-4 py-3 text-center">Fase</th>
                                <th class="px-4 py-3 text-center">Tensão</th>
                                <th class="px-4 py-3 text-center">Estoque</th>
                                <th class="px-4 py-3 text-center rounded-tr-lg">Ações</th>
                            </tr>
                        </thead>
                        <tbody id="listaBombasTable">
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- Tab: Relatórios -->
        <section id="tab-relatorios" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">📈 Relatórios</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Filtrar por Cliente</label>
                        <select id="relCliente" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                            <option value="">Todos</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Filtrar por Marca</label>
                        <select id="relMarca" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                            <option value="">Todas</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Filtrar por Modelo</label>
                        <input type="text" id="relModelo" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Modelo">
                    </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                    <button onclick="gerarRelatorio('clientes')" class="bg-gradient-to-r from-blue-500 to-blue-600 text-white p-4 rounded-xl hover:shadow-lg transition">
                        <div class="text-3xl mb-2">👥</div>
                        <div class="font-semibold">Clientes</div>
                        <div class="text-sm opacity-80">Lista de clientes</div>
                    </button>
                    <button onclick="gerarRelatorio('orcamentos')" class="bg-gradient-to-r from-green-500 to-green-600 text-white p-4 rounded-xl hover:shadow-lg transition">
                        <div class="text-3xl mb-2">💰</div>
                        <div class="font-semibold">Orçamentos</div>
                        <div class="text-sm opacity-80">Orçamentos gerados</div>
                    </button>
                    <button onclick="gerarRelatorio('marcas')" class="bg-gradient-to-r from-purple-500 to-purple-600 text-white p-4 rounded-xl hover:shadow-lg transition">
                        <div class="text-3xl mb-2">🏭</div>
                        <div class="font-semibold">Marcas Orçadas</div>
                        <div class="text-sm opacity-80">Marcas mais vendidas</div>
                    </button>
                    <button onclick="gerarRelatorio('bombas')" class="bg-gradient-to-r from-orange-500 to-orange-600 text-white p-4 rounded-xl hover:shadow-lg transition">
                        <div class="text-3xl mb-2">🔧</div>
                        <div class="font-semibold">Todas as Bombas</div>
                        <div class="text-sm opacity-80">Catálogo completo</div>
                    </button>
                </div>
            </div>
            <div id="relatorioPreview" class="bg-white rounded-xl shadow-lg p-6 hidden">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-bold text-[#1e3a5f]">Prévia do Relatório</h3>
                    <button onclick="exportarRelatorioPDF()" class="btn-primary text-white font-semibold py-2 px-6 rounded-lg transition">
                        📥 Baixar PDF
                    </button>
                </div>
                <div id="relatorioContent"></div>
            </div>
        </section>

        <!-- Tab: Configurações -->
        <section id="tab-configuracoes" class="tab-content hidden">
            <!-- Login Admin -->
            <div id="adminLogin" class="bg-white rounded-xl shadow-lg p-6 mb-6">
                <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">Acesso Administrativo</h2>
                <div class="max-w-md mx-auto">
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-1">Usuário</label>
                        <input type="text" id="adminUser" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Digite o usuário">
                    </div>
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-1">Senha</label>
                        <input type="password" id="adminPass" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Digite a senha">
                    </div>
                    <button onclick="loginAdmin()" class="btn-primary w-full text-white font-semibold py-3 px-8 rounded-lg transition">
                        Entrar
                    </button>
                    <p id="loginError" class="text-red-500 text-center mt-3 hidden">Usuário ou senha incorretos</p>
                </div>
            </div>

            <!-- Painel Admin (oculto até login) -->
            <div id="adminPanel" class="hidden">
                <!-- Controle de Usuários -->
                <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                    <div class="flex items-center justify-between mb-6">
                        <h2 class="text-xl font-bold text-[#1e3a5f]">Controle de Usuários</h2>
                        <button onclick="logoutAdmin()" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg text-sm font-medium transition">
                            Sair
                        </button>
                    </div>
                    
                    <!-- Adicionar Usuário -->
                    <div class="bg-gray-50 rounded-lg p-4 mb-6">
                        <h3 class="font-semibold text-gray-700 mb-3">Adicionar Novo Usuário</h3>
                        <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">Nome do Usuário</label>
                                <input type="text" id="novoUserNome" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nome">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">Login</label>
                                <input type="text" id="novoUserLogin" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Login">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">Senha</label>
                                <input type="password" id="novoUserSenha" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Senha">
                            </div>
                            <div class="flex items-end">
                                <button onclick="adicionarUsuario()" class="btn-primary w-full text-white font-semibold py-2 px-4 rounded-lg transition text-sm">
                                    Adicionar
                                </button>
                            </div>
                        </div>
                    </div>

                    <!-- Lista de Usuários -->
                    <div class="overflow-x-auto">
                        <table class="w-full text-sm">
                            <thead>
                                <tr class="bg-[#1e3a5f] text-white">
                                    <th class="px-4 py-3 text-left rounded-tl-lg">Nome</th>
                                    <th class="px-4 py-3 text-left">Login</th>
                                    <th class="px-4 py-3 text-center">Alterar Bombas</th>
                                    <th class="px-4 py-3 text-center">Excluir Bombas</th>
                                    <th class="px-4 py-3 text-center">Excluir Orçamentos</th>
                                    <th class="px-4 py-3 text-center">Gerenciar Clientes</th>
                                    <th class="px-4 py-3 text-center rounded-tr-lg">Ações</th>
                                </tr>
                            </thead>
                            <tbody id="listaUsuarios">
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Gerenciamento de Listas Suspensas -->
                <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
                    <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">Gerenciamento de Listas Suspensas</h2>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                        <!-- Categorias -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Categorias</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaCategoria" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nova categoria">
                                <button onclick="adicionarItemLista('categorias', 'novaCategoria')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaCategorias" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Polos -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Polos</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novoPolos" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo valor">
                                <button onclick="adicionarItemLista('polos', 'novoPolos')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaPolos" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Fase -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Fase</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaFase" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nova fase">
                                <button onclick="adicionarItemLista('fases', 'novaFase')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaFases" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Grau de Proteção -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Grau de Proteção</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novoGrauProtecao" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo grau">
                                <button onclick="adicionarItemLista('grausProtecao', 'novoGrauProtecao')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaGrausProtecao" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Frequência -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Frequência</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaFrequencia" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nova frequência">
                                <button onclick="adicionarItemLista('frequencias', 'novaFrequencia')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaFrequencias" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Tipo de Rotor -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Tipo de Rotor</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novoTipoRotor" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo tipo">
                                <button onclick="adicionarItemLista('tiposRotor', 'novoTipoRotor')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaTiposRotor" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Posição -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Posição</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaPosicao" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nova posição">
                                <button onclick="adicionarItemLista('posicoes', 'novaPosicao')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaPosicoes" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Tipo de Embalagem -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Tipo de Embalagem</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaEmbalagem" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo tipo">
                                <button onclick="adicionarItemLista('embalagens', 'novaEmbalagem')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaEmbalagens" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Tipo de Conexão -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Tipo de Conexão</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaConexao" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo tipo">
                                <button onclick="adicionarItemLista('conexoes', 'novaConexao')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaConexoes" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Inversor -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Inversor</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novoInversor" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo valor">
                                <button onclick="adicionarItemLista('inversores', 'novoInversor')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaInversores" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Estoque -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Status de Estoque</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novoEstoque" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Novo status">
                                <button onclick="adicionarItemLista('estoques', 'novoEstoque')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaEstoques" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>

                        <!-- Tensões Comuns -->
                        <div class="bg-gray-50 rounded-lg p-4">
                            <h3 class="font-semibold text-gray-700 mb-3">Tensões Comuns</h3>
                            <div class="flex gap-2 mb-3">
                                <input type="text" id="novaTensao" class="filter-input flex-1 px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nova tensão">
                                <button onclick="adicionarItemLista('tensoes', 'novaTensao')" class="bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg text-sm">+</button>
                            </div>
                            <ul id="listaTensoes" class="space-y-2 max-h-40 overflow-y-auto"></ul>
                        </div>
                    </div>
                </div>

                <!-- Gerenciamento de Clientes -->
                <div class="bg-white rounded-xl shadow-lg p-6">
                    <h2 class="text-xl font-bold text-[#1e3a5f] mb-6">Gerenciamento de Clientes</h2>
                    
                    <!-- Adicionar Cliente -->
                    <div class="bg-gray-50 rounded-lg p-4 mb-6">
                        <h3 class="font-semibold text-gray-700 mb-3">Adicionar Novo Cliente</h3>
                        <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">Nome do Cliente</label>
                                <input type="text" id="novoClienteNome" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Nome">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">CNPJ/CPF</label>
                                <input type="text" id="novoClienteDoc" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Documento">
                            </div>
                            <div>
                                <label class="block text-xs font-medium text-gray-600 mb-1">Telefone</label>
                                <input type="text" id="novoClienteTel" class="filter-input w-full px-3 py-2 border border-gray-300 rounded-lg text-sm" placeholder="Telefone">
                            </div>
                            <div class="flex items-end">
                                <button onclick="adicionarCliente()" class="btn-primary w-full text-white font-semibold py-2 px-4 rounded-lg transition text-sm">
                                    Adicionar
                                </button>
                            </div>
                        </div>
                    </div>

                    <!-- Lista de Clientes -->
                    <div class="overflow-x-auto">
                        <table class="w-full text-sm">
                            <thead>
                                <tr class="bg-[#1e3a5f] text-white">
                                    <th class="px-4 py-3 text-left rounded-tl-lg">Nome</th>
                                    <th class="px-4 py-3 text-left">CNPJ/CPF</th>
                                    <th class="px-4 py-3 text-left">Telefone</th>
                                    <th class="px-4 py-3 text-center rounded-tr-lg">Ações</th>
                                </tr>
                            </thead>
                            <tbody id="listaClientes">
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <!-- Modal: Ficha Técnica -->
    <div id="modalFicha" class="fixed inset-0 bg-black/50 modal-overlay hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl max-w-4xl w-full max-h-[90vh] overflow-y-auto">
            <div class="bg-gradient-to-r from-[#1e3a5f] to-[#152a45] text-white p-6 rounded-t-2xl flex justify-between items-center">
                <h3 class="text-xl font-bold">Ficha Técnica</h3>
                <button onclick="fecharModal('modalFicha')" class="text-white hover:text-gray-300 text-2xl">&times;</button>
            </div>
            <div id="fichaContent" class="p-6"></div>
        </div>
    </div>

    <!-- Modal: Comparação -->
    <div id="modalCompare" class="fixed inset-0 bg-black/50 modal-overlay hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl max-w-6xl w-full max-h-[90vh] overflow-y-auto">
            <div class="bg-gradient-to-r from-[#1e3a5f] to-[#152a45] text-white p-6 rounded-t-2xl flex justify-between items-center">
                <h3 class="text-xl font-bold">⚖️ Comparar Bombas</h3>
                <button onclick="fecharModal('modalCompare')" class="text-white hover:text-gray-300 text-2xl">&times;</button>
            </div>
            <div id="compareContent" class="p-6 overflow-x-auto"></div>
        </div>
    </div>

    <!-- Modal: Editar Bomba -->
    <div id="modalEdit" class="fixed inset-0 bg-black/50 modal-overlay hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl max-w-4xl w-full max-h-[90vh] overflow-y-auto">
            <div class="bg-gradient-to-r from-[#1e3a5f] to-[#152a45] text-white p-6 rounded-t-2xl flex justify-between items-center">
                <h3 class="text-xl font-bold">✏️ Editar Bomba</h3>
                <button onclick="fecharModal('modalEdit')" class="text-white hover:text-gray-300 text-2xl">&times;</button>
            </div>
            <div id="editContent" class="p-6"></div>
        </div>
    </div>

    <script>
        // ==================== FIREBASE CONFIGURATION ====================
        const firebaseConfig = {
            apiKey: "AIzaSyBW8fDN8fUIKkW-FFoNSKVL5iocniAMi9I",
            authDomain: "dimensionador-conab.firebaseapp.com",
            databaseURL: "https://dimensionador-conab-default-rtdb.firebaseio.com",
            projectId: "dimensionador-conab",
            storageBucket: "dimensionador-conab.firebasestorage.app",
            messagingSenderId: "95624610629",
            appId: "1:95624610629:web:680d53a2173afe1b6ef42c",
            measurementId: "G-H1KXGJLMN1"
        };

        // Inicializar Firebase
        let database = null;
        let firebaseInitialized = false;

        try {
            firebase.initializeApp(firebaseConfig);
            database = firebase.database();
            firebaseInitialized = true;
            console.log('✅ Firebase conectado com sucesso!');
            updateFirebaseStatus(true);
        } catch (error) {
            console.warn('⚠️ Firebase não configurado. Usando localStorage. Erro:', error.message);
            firebaseInitialized = false;
            updateFirebaseStatus(false);
        }

        // Atualizar indicador de status do Firebase
        function updateFirebaseStatus(connected) {
            const statusDiv = document.getElementById('firebaseStatus');
            const statusIcon = document.getElementById('statusIcon');
            const statusText = document.getElementById('statusText');
            
            if (statusDiv && statusIcon && statusText) {
                statusDiv.classList.remove('hidden');
                if (connected) {
                    statusDiv.className = 'px-3 py-1 rounded-full text-xs font-semibold bg-green-500/20 text-green-300';
                    statusIcon.textContent = '●';
                    statusText.textContent = 'Online (Sincronizado)';
                } else {
                    statusDiv.className = 'px-3 py-1 rounded-full text-xs font-semibold bg-yellow-500/20 text-yellow-300';
                    statusIcon.textContent = '●';
                    statusText.textContent = 'Offline (Local)';
                }
            }
        }

        // ==================== STATE ====================
        let bombas = [];
        let bombasFiltradas = [];
        let orcamentoItems = [];
        let orcamentos = [];
        let compareList = [];
        let curvasChart = null;
        let isDarkMode = false;
        let pontoTrabalho = { vazao: 0, altura: 0 };

        // ==================== SISTEMA DE LOGIN ====================
        function fazerLogin() {
            const user = document.getElementById('loginUser').value.trim();
            const pass = document.getElementById('loginPass').value.trim();
            
            if (!user || !pass) {
                document.getElementById('loginErrorMsg').classList.remove('hidden');
                return;
            }
            
            // Verificar admin
            if (user === 'admin' && pass === 'admin123') {
                currentUser = { nome: 'Administrador', login: 'admin', isAdmin: true };
                completarLogin();
                return;
            }
            
            // Verificar usuários cadastrados
            const usuario = usuarios.find(u => u.login === user && u.senha === pass);
            if (usuario) {
                currentUser = usuario;
                completarLogin();
                return;
            }
            
            document.getElementById('loginErrorMsg').classList.remove('hidden');
        }
        
        function completarLogin() {
            document.getElementById('loginScreen').classList.add('hidden');
            document.getElementById('loginErrorMsg').classList.add('hidden');
            document.getElementById('userMenu').classList.remove('hidden');
            document.getElementById('userNameDisplay').textContent = currentUser.nome;
            document.getElementById('dropdownUserName').textContent = currentUser.nome;
            document.getElementById('dropdownUserRole').textContent = currentUser.isAdmin ? 'Administrador' : 'Usuário';
            
            atualizarVisibilidadeAbas();
            renderLista();
        }
        
        function fazerLogout() {
            currentUser = null;
            document.getElementById('loginScreen').classList.remove('hidden');
            document.getElementById('userMenu').classList.add('hidden');
            document.getElementById('loginUser').value = '';
            document.getElementById('loginPass').value = '';
            document.getElementById('userDropdown').classList.add('hidden');
            showTab('dimensionamento');
        }
        
        function toggleUserDropdown() {
            document.getElementById('userDropdown').classList.toggle('hidden');
        }
        
        // Fechar dropdown ao clicar fora
        document.addEventListener('click', (e) => {
            const userMenu = document.getElementById('userMenu');
            const dropdown = document.getElementById('userDropdown');
            if (userMenu && dropdown && !userMenu.contains(e.target)) {
                dropdown.classList.add('hidden');
            }
        });

        // ==================== SAMPLE DATA ====================
        const bombasExemplo = [
            {
                id: 'BC92-1CV',
                codigo: 'BC92-1CV',
                marca: 'Schneider',
                modelo: 'BC-92 1CV',
                potencia: 1,
                polos: 2,
                inversor: 'nao',
                fase: 'monofasico',
                tensao: '220',
                grauProtecao: 'IP21',
                succao: 32,
                recalque: 25,
                frequencia: '60',
                categoria: 'centrifuga',
                tipoRotor: 'fechado',
                material: 'ferro',
                posicao: 'horizontal',
                medidaRotor: 127,
                altMaxSuccao: 7,
                pressaoMax: 32,
                comprimento: 350,
                largura: 180,
                alturaDim: 200,
                peso: 12.5,
                embalagem: 'caixa_papelao',
                conexao: 'rosca',
                estoque: 'disponivel',
                eficiencia: 68,
                foto: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Crect fill="%231e3a5f" width="100" height="100"/%3E%3Ctext x="50" y="55" text-anchor="middle" fill="white" font-size="12"%3EBC-92%3C/text%3E%3C/svg%3E',
                curva: [
                    { vazao: 0, altura: 32 },
                    { vazao: 5, altura: 30 },
                    { vazao: 10, altura: 26 },
                    { vazao: 15, altura: 20 },
                    { vazao: 20, altura: 12 },
                    { vazao: 22, altura: 6 }
                ]
            },
            {
                id: 'BC92-2CV',
                codigo: 'BC92-2CV',
                marca: 'Schneider',
                modelo: 'BC-92 2CV',
                potencia: 2,
                polos: 2,
                inversor: 'nao',
                fase: 'trifasico',
                tensao: '380',
                grauProtecao: 'IP55',
                succao: 50,
                recalque: 32,
                frequencia: '60',
                categoria: 'centrifuga',
                tipoRotor: 'fechado',
                material: 'ferro',
                posicao: 'horizontal',
                medidaRotor: 152,
                altMaxSuccao: 8,
                pressaoMax: 45,
                comprimento: 420,
                largura: 210,
                alturaDim: 240,
                peso: 18.2,
                embalagem: 'caixa_papelao',
                conexao: 'flange',
                estoque: 'disponivel',
                eficiencia: 72,
                foto: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Crect fill="%2322c55e" width="100" height="100"/%3E%3Ctext x="50" y="55" text-anchor="middle" fill="white" font-size="12"%3EBC-92 2CV%3C/text%3E%3C/svg%3E',
                curva: [
                    { vazao: 0, altura: 45 },
                    { vazao: 8, altura: 42 },
                    { vazao: 16, altura: 36 },
                    { vazao: 24, altura: 28 },
                    { vazao: 32, altura: 18 },
                    { vazao: 38, altura: 8 }
                ]
            },
            {
                id: 'KSB-ETA50',
                codigo: 'KSB-ETA50',
                marca: 'KSB',
                modelo: 'ETA 50-160',
                potencia: 5,
                polos: 4,
                inversor: 'sim',
                fase: 'trifasico',
                tensao: '380',
                grauProtecao: 'IP55',
                succao: 65,
                recalque: 50,
                frequencia: '60',
                categoria: 'centrifuga',
                tipoRotor: 'fechado',
                material: 'inox',
                posicao: 'horizontal',
                medidaRotor: 160,
                altMaxSuccao: 7,
                pressaoMax: 52,
                comprimento: 550,
                largura: 280,
                alturaDim: 320,
                peso: 45,
                embalagem: 'pallet',
                conexao: 'flange',
                estoque: 'baixo',
                eficiencia: 78,
                foto: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Crect fill="%23dc2626" width="100" height="100"/%3E%3Ctext x="50" y="55" text-anchor="middle" fill="white" font-size="12"%3EKSB ETA%3C/text%3E%3C/svg%3E',
                curva: [
                    { vazao: 0, altura: 52 },
                    { vazao: 15, altura: 50 },
                    { vazao: 30, altura: 45 },
                    { vazao: 45, altura: 38 },
                    { vazao: 60, altura: 28 },
                    { vazao: 75, altura: 15 }
                ]
            }
        ];

        // ==================== INITIALIZATION ====================
        document.addEventListener('DOMContentLoaded', async () => {
            // Carregar dados de configuração primeiro
            await loadConfigData();
            
            await loadFromStorage();
            if (bombas.length === 0) {
                bombas = [...bombasExemplo];
                await saveToStorage();
            }
            initChart();
            populateFilters();
            renderBombas(); // Vai mostrar mensagem inicial pedindo para preencher parâmetros
            updateChart(); // Vai mostrar mensagem inicial no gráfico
            renderLista();
            updateOrcamentoView();
            updateRelatorioFilters();
            document.getElementById('orcData').valueAsDate = new Date();
            
            // Configurar listeners do Firebase para sincronização em tempo real
            setupFirebaseListeners();
            
            // Atualizar visibilidade das abas baseado no usuário
            atualizarVisibilidadeAbas();
        });

        // ==================== STORAGE ====================
        async function loadFromStorage() {
            if (firebaseInitialized) {
                try {
                    // Carregar bombas do Firebase
                    const bombasSnapshot = await database.ref('bombas').once('value');
                    if (bombasSnapshot.exists()) {
                        const bombasData = bombasSnapshot.val();
                        bombas = Object.values(bombasData);
                        console.log('✅ Bombas carregadas do Firebase:', bombas.length);
                    }

                    // Carregar orçamentos do Firebase
                    const orcamentosSnapshot = await database.ref('orcamentos').once('value');
                    if (orcamentosSnapshot.exists()) {
                        const orcamentosData = orcamentosSnapshot.val();
                        orcamentos = Object.values(orcamentosData);
                        console.log('✅ Orçamentos carregados do Firebase:', orcamentos.length);
                    }

                    // Carregar itens de orçamento do localStorage (temporário)
                    const storedItems = localStorage.getItem('conab_orcamento_items');
                    if (storedItems) orcamentoItems = JSON.parse(storedItems);
                } catch (error) {
                    console.error('Erro ao carregar do Firebase:', error);
                    loadFromLocalStorage();
                }
            } else {
                loadFromLocalStorage();
            }
        }

        function loadFromLocalStorage() {
            const stored = localStorage.getItem('conab_bombas');
            if (stored) bombas = JSON.parse(stored);
            const storedOrc = localStorage.getItem('conab_orcamentos');
            if (storedOrc) orcamentos = JSON.parse(storedOrc);
            const storedItems = localStorage.getItem('conab_orcamento_items');
            if (storedItems) orcamentoItems = JSON.parse(storedItems);
        }

        async function saveToStorage() {
            if (firebaseInitialized) {
                try {
                    // Salvar bombas no Firebase
                    const bombasObj = {};
                    bombas.forEach(b => {
                        bombasObj[b.id] = b;
                    });
                    await database.ref('bombas').set(bombasObj);

                    // Salvar orçamentos no Firebase
                    const orcamentosObj = {};
                    orcamentos.forEach(o => {
                        orcamentosObj[o.id] = o;
                    });
                    await database.ref('orcamentos').set(orcamentosObj);

                    console.log('✅ Dados salvos no Firebase');
                } catch (error) {
                    console.error('Erro ao salvar no Firebase:', error);
                    saveToLocalStorage();
                }
            } else {
                saveToLocalStorage();
            }
            
            // Sempre salvar itens de orçamento no localStorage
            localStorage.setItem('conab_orcamento_items', JSON.stringify(orcamentoItems));
        }

        function saveToLocalStorage() {
            localStorage.setItem('conab_bombas', JSON.stringify(bombas));
            localStorage.setItem('conab_orcamentos', JSON.stringify(orcamentos));
            localStorage.setItem('conab_orcamento_items', JSON.stringify(orcamentoItems));
        }

        // ==================== FIREBASE REAL-TIME SYNC ====================
        function setupFirebaseListeners() {
            if (!firebaseInitialized) return;

            // Listener para mudanças nas bombas
            database.ref('bombas').on('value', (snapshot) => {
                if (snapshot.exists()) {
                    const bombasData = snapshot.val();
                    const newBombas = Object.values(bombasData);
                    
                    // Apenas atualizar se houve mudança real
                    if (JSON.stringify(newBombas) !== JSON.stringify(bombas)) {
                        bombas = newBombas;
                        bombasFiltradas = [...bombas];
                        console.log('🔄 Bombas sincronizadas do Firebase');
                        populateFilters();
                        renderBombas();
                        renderLista();
                        updateChart();
                    }
                }
            });

            // Listener para mudanças nos orçamentos
            database.ref('orcamentos').on('value', (snapshot) => {
                if (snapshot.exists()) {
                    const orcamentosData = snapshot.val();
                    const newOrcamentos = Object.values(orcamentosData);
                    
                    if (JSON.stringify(newOrcamentos) !== JSON.stringify(orcamentos)) {
                        orcamentos = newOrcamentos;
                        console.log('🔄 Orçamentos sincronizados do Firebase');
                        updateRelatorioFilters();
                    }
                }
            });
        }

        // ==================== TABS ====================
        function showTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(tab => tab.classList.add('hidden'));
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('tab-active');
                btn.classList.add('hover:bg-white/10');
            });
            document.getElementById('tab-' + tabName).classList.remove('hidden');
            document.querySelector(`[data-tab="${tabName}"]`).classList.add('tab-active');
            document.querySelector(`[data-tab="${tabName}"]`).classList.remove('hover:bg-white/10');
            
            if (tabName === 'lista') renderLista();
            if (tabName === 'orcamento') updateOrcamentoView();
            
            // Se acessar configurações e for admin, mostrar painel automaticamente
            if (tabName === 'configuracoes' && currentUser && currentUser.isAdmin) {
                document.getElementById('adminLogin').classList.add('hidden');
                document.getElementById('adminPanel').classList.remove('hidden');
                renderUsuarios();
                renderClientes();
                renderTodasListas();
            }
        }

        // ==================== THEME ====================
        document.getElementById('themeToggle').addEventListener('click', () => {
            isDarkMode = !isDarkMode;
            document.body.classList.toggle('dark', isDarkMode);
            document.body.classList.toggle('light', !isDarkMode);
            document.body.style.background = isDarkMode ? '#1a1a2e' : '#f1f5f9';
            document.getElementById('sunIcon').classList.toggle('hidden', !isDarkMode);
            document.getElementById('moonIcon').classList.toggle('hidden', isDarkMode);
            document.querySelectorAll('.bg-white').forEach(el => {
                el.style.background = isDarkMode ? '#16213e' : 'white';
                el.style.color = isDarkMode ? 'white' : '';
            });
        });



        // ==================== FILTERS ====================
        function populateFilters() {
            const marcas = [...new Set(bombas.map(b => b.marca))];
            const potencias = [...new Set(bombas.map(b => b.potencia))].sort((a,b) => a-b);
            
            const filterMarca = document.getElementById('filterMarca');
            filterMarca.innerHTML = '<option value="">Todas</option>';
            marcas.forEach(m => filterMarca.innerHTML += `<option value="${m}">${m}</option>`);
            
            const filterPotencia = document.getElementById('filterPotencia');
            filterPotencia.innerHTML = '<option value="">Todas</option>';
            potencias.forEach(p => filterPotencia.innerHTML += `<option value="${p}">${p} CV</option>`);
        }

        function limparFiltros() {
            document.querySelectorAll('#tab-dimensionamento .filter-input').forEach(input => {
                if (input.tagName === 'SELECT') input.selectedIndex = 0;
                else if (input.id !== 'inputTolerancia') input.value = '';
            });
            document.getElementById('pressurizacaoFilters').classList.add('hidden');
            document.getElementById('filterQtdBombasOp').value = '';
            bombasFiltradas = [...bombas];
            renderBombas();
            updateChart();
        }

        function togglePressurizacaoFilter() {
            const categoria = document.getElementById('filterCategoria').value;
            const pressurizacaoFilters = document.getElementById('pressurizacaoFilters');
            
            if (categoria === 'pressurizacao') {
                pressurizacaoFilters.classList.remove('hidden');
            } else {
                pressurizacaoFilters.classList.add('hidden');
                document.getElementById('filterQtdBombasOp').value = '';
            }
            
            aplicarFiltros();
        }

        function aplicarFiltros() {
            const filters = {
                codigo: document.getElementById('filterCodigo').value.toLowerCase(),
                marca: document.getElementById('filterMarca').value,
                modelo: document.getElementById('filterModelo').value.toLowerCase(),
                potencia: document.getElementById('filterPotencia').value,
                polos: document.getElementById('filterPolos').value,
                inversor: document.getElementById('filterInversor').value,
                fase: document.getElementById('filterFase').value,
                tensao: document.getElementById('filterTensao').value,
                grauProtecao: document.getElementById('filterGrauProtecao').value,
                succao: document.getElementById('filterSuccao').value,
                recalque: document.getElementById('filterRecalque').value,
                frequencia: document.getElementById('filterFrequencia').value,
                categoria: document.getElementById('filterCategoria').value,
                tipoRotor: document.getElementById('filterTipoRotor').value,
                material: document.getElementById('filterMaterial').value,
                posicao: document.getElementById('filterPosicao').value,
                medidaRotor: document.getElementById('filterMedidaRotor').value,
                altMaxSuccao: document.getElementById('filterAltMaxSuccao').value,
                pressaoMax: document.getElementById('filterPressaoMax').value,
                estoque: document.getElementById('filterEstoque').value,
                qtdBombasOp: document.getElementById('filterQtdBombasOp')?.value
            };

            bombasFiltradas = bombas.filter(b => {
                // FILTRO DE CATEGORIA - Verifica primeiro se a categoria da bomba corresponde
                if (filters.categoria) {
                    // Se selecionou uma categoria específica, a bomba DEVE ser dessa categoria
                    if (b.categoria !== filters.categoria) {
                        return false;
                    }
                    
                    // Se é pressurização e especificou quantidade de bombas
                    if (filters.categoria === 'pressurizacao' && filters.qtdBombasOp) {
                        // Verifica se a bomba tem a quantidade exata de bombas
                        if (!b.qtdBombasOp || parseInt(b.qtdBombasOp) !== parseInt(filters.qtdBombasOp)) {
                            return false;
                        }
                    }
                }

                // Demais filtros
                return (
                    (!filters.codigo || b.codigo.toLowerCase().includes(filters.codigo)) &&
                    (!filters.marca || b.marca === filters.marca) &&
                    (!filters.modelo || b.modelo.toLowerCase().includes(filters.modelo)) &&
                    (!filters.potencia || b.potencia == filters.potencia) &&
                    (!filters.polos || b.polos == filters.polos) &&
                    (!filters.inversor || b.inversor === filters.inversor) &&
                    (!filters.fase || b.fase === filters.fase) &&
                    (!filters.tensao || b.tensao === filters.tensao) &&
                    (!filters.grauProtecao || b.grauProtecao === filters.grauProtecao) &&
                    (!filters.succao || b.succao >= parseFloat(filters.succao)) &&
                    (!filters.recalque || b.recalque >= parseFloat(filters.recalque)) &&
                    (!filters.frequencia || b.frequencia === filters.frequencia) &&
                    (!filters.tipoRotor || b.tipoRotor === filters.tipoRotor) &&
                    (!filters.material || b.material === filters.material) &&
                    (!filters.posicao || b.posicao === filters.posicao) &&
                    (!filters.medidaRotor || b.medidaRotor >= parseFloat(filters.medidaRotor)) &&
                    (!filters.altMaxSuccao || b.altMaxSuccao >= parseFloat(filters.altMaxSuccao)) &&
                    (!filters.pressaoMax || b.pressaoMax >= parseFloat(filters.pressaoMax)) &&
                    (!filters.estoque || b.estoque === filters.estoque)
                );
            });
            renderBombas();
            updateChart();
        }

        // ==================== DIMENSIONAMENTO ====================
        function buscarCompatibilidade() {
            const vazao = parseFloat(document.getElementById('inputVazao').value) || 0;
            const altura = parseFloat(document.getElementById('inputAltura').value) || 0;
            const tolerancia = parseFloat(document.getElementById('inputTolerancia').value) || 10;

            pontoTrabalho = { vazao, altura };

            if (vazao === 0 && altura === 0) {
                bombasFiltradas = [...bombas];
                aplicarFiltros();
                return;
            }

            bombasFiltradas = bombas.filter(b => {
                const compatibilidade = calcularCompatibilidade(b, vazao, altura, tolerancia);
                return compatibilidade !== null;
            }).map(b => ({
                ...b,
                compatibilidade: calcularCompatibilidade(b, vazao, altura, tolerancia)
            })).sort((a, b) => {
                const order = { excelente: 0, boa: 1, ruim: 2 };
                return order[a.compatibilidade.classificacao] - order[b.compatibilidade.classificacao];
            });

            renderBombas();
            updateChart();
        }

        function calcularCompatibilidade(bomba, vazao, altura, tolerancia) {
            const curva = bomba.curva;
            if (!curva || curva.length < 2) return null;

            // Interpolar altura na curva para a vazão desejada
            let alturaReal = null;
            for (let i = 0; i < curva.length - 1; i++) {
                if (vazao >= curva[i].vazao && vazao <= curva[i + 1].vazao) {
                    const t = (vazao - curva[i].vazao) / (curva[i + 1].vazao - curva[i].vazao);
                    alturaReal = curva[i].altura + t * (curva[i + 1].altura - curva[i].altura);
                    break;
                }
            }

            if (alturaReal === null) {
                if (vazao < curva[0].vazao) alturaReal = curva[0].altura;
                else if (vazao > curva[curva.length - 1].vazao) return null;
            }

            const sobra = ((alturaReal - altura) / altura) * 100;
            const tolMin = -tolerancia;
            const tolMax = tolerancia * 2;

            let classificacao;
            if (sobra >= 0 && sobra <= 10) classificacao = 'excelente';
            else if (sobra > 10 && sobra <= tolMax) classificacao = 'boa';
            else if (sobra >= tolMin && sobra < 0) classificacao = 'ruim';
            else return null;

            return {
                alturaReal: alturaReal.toFixed(2),
                sobra: sobra.toFixed(1),
                classificacao
            };
        }

        // ==================== CHART ====================
        function initChart() {
            const ctx = document.getElementById('curvasChart').getContext('2d');
            curvasChart = new Chart(ctx, {
                type: 'line',
                data: {
                    datasets: []
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {
                        intersect: false,
                        mode: 'nearest'
                    },
                    onClick: (event, activeElements, chart) => {
                        // Get click coordinates
                        const canvasPosition = Chart.helpers.getRelativePosition(event, chart);
                        
                        // Convert to data values
                        const dataX = chart.scales.x.getValueForPixel(canvasPosition.x);
                        const dataY = chart.scales.y.getValueForPixel(canvasPosition.y);
                        
                        // Update inputs and working point
                        if (dataX >= 0 && dataY >= 0) {
                            document.getElementById('inputVazao').value = dataX.toFixed(1);
                            document.getElementById('inputAltura').value = dataY.toFixed(1);
                            pontoTrabalho = { vazao: parseFloat(dataX.toFixed(1)), altura: parseFloat(dataY.toFixed(1)) };
                            buscarCompatibilidade();
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            backgroundColor: '#1e3a5f',
                            titleFont: { size: 14 },
                            bodyFont: { size: 12 },
                            callbacks: {
                                label: function(context) {
                                    return `${context.dataset.label}: ${context.parsed.y.toFixed(1)} mca @ ${context.parsed.x.toFixed(1)} m³/h`;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            type: 'linear',
                            title: {
                                display: true,
                                text: 'Vazão (m³/h)',
                                font: { size: 14, weight: 'bold' },
                                color: '#1e3a5f'
                            },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Altura Manométrica (mca)',
                                font: { size: 14, weight: 'bold' },
                                color: '#1e3a5f'
                            },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        }
                    }
                }
            });
        }

        function updateChart() {
            const vazao = parseFloat(document.getElementById('inputVazao').value) || 0;
            const altura = parseFloat(document.getElementById('inputAltura').value) || 0;
            
            // Só mostrar gráfico se vazão E altura foram preenchidos
            if (vazao === 0 || altura === 0) {
                curvasChart.data.datasets = [];
                curvasChart.update();
                document.getElementById('chartLegend').innerHTML = '<div class="text-center text-gray-500 py-8"><p>Preencha vazão e altura manométrica para visualizar as curvas</p></div>';
                return;
            }
            
            const colors = [
                '#22c55e', '#3b82f6', '#f59e0b', '#ef4444', '#8b5cf6',
                '#06b6d4', '#ec4899', '#14b8a6', '#f97316', '#6366f1'
            ];

            const datasets = [];
            let colorIndex = 0;

            bombasFiltradas.slice(0, 10).forEach((bomba, index) => {
                // Verificar se é bomba de pressurização com múltiplas curvas
                if (bomba.categoria === 'pressurizacao' && bomba.curvasPorBomba && bomba.curvasPorBomba.length > 0) {
                    // Desenhar curva SEPARADA e SUAVE para cada bomba individual do sistema
                    bomba.curvasPorBomba.forEach((curvaBomba, bombaNum) => {
                        if (curvaBomba && curvaBomba.length > 0) {
                            // Ordenar pontos por vazão para garantir linha contínua
                            const pontosSorted = [...curvaBomba].sort((a, b) => a.vazao - b.vazao);
                            
                            datasets.push({
                                label: `${bomba.marca} ${bomba.modelo} - Bomba ${bombaNum + 1}`,
                                data: pontosSorted.map(p => ({ x: p.vazao, y: p.altura })),
                                borderColor: colors[colorIndex % colors.length],
                                backgroundColor: colors[colorIndex % colors.length] + '20',
                                borderWidth: 3,
                                tension: 0.4, // Linha suavemente encurvada
                                pointRadius: 5,
                                pointHoverRadius: 7,
                                pointBackgroundColor: colors[colorIndex % colors.length],
                                pointBorderColor: '#ffffff',
                                pointBorderWidth: 2,
                                fill: false,
                                spanGaps: false // Não conectar pontos com gaps
                            });
                            colorIndex++;
                        }
                    });
                } else {
                    // Bombas normais mostram apenas a curva principal
                    if (bomba.curva && bomba.curva.length > 0) {
                        // Ordenar pontos por vazão
                        const pontosSorted = [...bomba.curva].sort((a, b) => a.vazao - b.vazao);
                        
                        datasets.push({
                            label: `${bomba.marca} ${bomba.modelo}`,
                            data: pontosSorted.map(p => ({ x: p.vazao, y: p.altura })),
                            borderColor: colors[colorIndex % colors.length],
                            backgroundColor: colors[colorIndex % colors.length] + '20',
                            borderWidth: 3,
                            tension: 0.4,
                            pointRadius: 5,
                            pointHoverRadius: 7,
                            pointBackgroundColor: colors[colorIndex % colors.length],
                            pointBorderColor: '#ffffff',
                            pointBorderWidth: 2,
                            fill: false,
                            spanGaps: false
                        });
                        colorIndex++;
                    }
                }
            });

            // Add working point if set
            if (pontoTrabalho.vazao > 0 && pontoTrabalho.altura > 0) {
                datasets.push({
                    label: 'Ponto de Trabalho',
                    data: [{ x: pontoTrabalho.vazao, y: pontoTrabalho.altura }],
                    borderColor: '#dc2626',
                    backgroundColor: '#dc2626',
                    pointRadius: 12,
                    pointStyle: 'circle',
                    showLine: false
                });

                // Add crosshair lines
                datasets.push({
                    label: '_vazaoLine',
                    data: [
                        { x: pontoTrabalho.vazao, y: 0 },
                        { x: pontoTrabalho.vazao, y: pontoTrabalho.altura }
                    ],
                    borderColor: '#dc262680',
                    borderDash: [5, 5],
                    borderWidth: 2,
                    pointRadius: 0,
                    showLine: true
                });
                datasets.push({
                    label: '_alturaLine',
                    data: [
                        { x: 0, y: pontoTrabalho.altura },
                        { x: pontoTrabalho.vazao, y: pontoTrabalho.altura }
                    ],
                    borderColor: '#dc262680',
                    borderDash: [5, 5],
                    borderWidth: 2,
                    pointRadius: 0,
                    showLine: true
                });
            }

            curvasChart.data.datasets = datasets;
            curvasChart.update();

            // Update legend
            const legendDiv = document.getElementById('chartLegend');
            let legendHTML = '';
            colorIndex = 0;

            bombasFiltradas.slice(0, 10).forEach((bomba, index) => {
                // Verificar se é bomba de pressurização com múltiplas curvas
                if (bomba.categoria === 'pressurizacao' && bomba.curvasPorBomba && bomba.curvasPorBomba.length > 0) {
                    // Mostrar legenda SEPARADA para cada bomba individual do sistema
                    bomba.curvasPorBomba.forEach((curvaBomba, bombaNum) => {
                        if (curvaBomba && curvaBomba.length > 0) {
                            legendHTML += `
                                <div class="flex items-center gap-2 px-3 py-1 bg-gray-100 rounded-full text-sm">
                                    <div class="w-4 h-4 rounded-full" style="background: ${colors[colorIndex % colors.length]}"></div>
                                    <span>${bomba.marca} ${bomba.modelo} - Bomba ${bombaNum + 1}</span>
                                </div>
                            `;
                            colorIndex++;
                        }
                    });
                } else {
                    if (bomba.curva && bomba.curva.length > 0) {
                        legendHTML += `
                            <div class="flex items-center gap-2 px-3 py-1 bg-gray-100 rounded-full text-sm">
                                <div class="w-4 h-4 rounded-full" style="background: ${colors[colorIndex % colors.length]}"></div>
                                <span>${bomba.marca} ${bomba.modelo}</span>
                            </div>
                        `;
                        colorIndex++;
                    }
                }
            });

            if (pontoTrabalho.vazao > 0) {
                legendHTML += `
                    <div class="flex items-center gap-2 px-3 py-1 bg-red-100 rounded-full text-sm">
                        <div class="w-4 h-4 rounded-full bg-red-600"></div>
                        <span>Ponto de Trabalho (${pontoTrabalho.vazao} m³/h, ${pontoTrabalho.altura} mca)</span>
                    </div>
                `;
            }

            legendDiv.innerHTML = legendHTML;
        }

        // ==================== RENDER BOMBAS ====================
        function renderBombas() {
            const grid = document.getElementById('bombasGrid');
            const count = document.getElementById('resultCount');
            
            const vazao = parseFloat(document.getElementById('inputVazao').value) || 0;
            const altura = parseFloat(document.getElementById('inputAltura').value) || 0;
            
            // Só mostrar resultados se vazão E altura foram preenchidos
            if (vazao === 0 || altura === 0) {
                grid.innerHTML = '<div class="col-span-full text-center py-16 text-gray-500"><svg class="w-24 h-24 mx-auto mb-4 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/></svg><p class="text-lg font-medium">Preencha os parâmetros de dimensionamento acima</p><p class="text-sm mt-2">Informe a vazão (m³/h) e altura manométrica (mca) para buscar bombas compatíveis</p></div>';
                count.textContent = '0 resultados';
                return;
            }
            
            if (bombasFiltradas.length === 0) {
                bombasFiltradas = [...bombas];
            }

            count.textContent = `${bombasFiltradas.length} resultados`;

            grid.innerHTML = bombasFiltradas.map(bomba => {
                const compat = bomba.compatibilidade;
                const classeBadge = compat ? 
                    (compat.classificacao === 'excelente' ? 'excellence' : 
                     compat.classificacao === 'boa' ? 'good' : 'bad') : '';
                const borderClass = compat ? 
                    (compat.classificacao === 'excelente' ? 'border-boa' : 
                     compat.classificacao === 'boa' ? 'border-media' : 'border-ruim') : 'border border-gray-200';
                const estoqueColor = bomba.estoque === 'disponivel' ? 'bg-green-100 text-green-800' :
                                    bomba.estoque === 'baixo' ? 'bg-yellow-100 text-yellow-800' : 
                                    'bg-red-100 text-red-800';
                const estoqueText = bomba.estoque === 'disponivel' ? 'Em Estoque' :
                                   bomba.estoque === 'baixo' ? 'Baixo Estoque' : 'Esgotado';
                const isInCompare = compareList.includes(bomba.id);
                const potenciaKW = (bomba.potencia * 0.7355).toFixed(2);
                const consumo = (potenciaKW / (bomba.eficiencia / 100 || 0.7)).toFixed(2);

                return `
                    <div class="pump-card bg-gray-50 rounded-xl p-4 ${borderClass}">
                        <div class="flex justify-between items-start mb-3">
                            <div class="flex items-center gap-3">
                                <img src="${bomba.foto || 'data:image/svg+xml,%3Csvg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 100 100\"%3E%3Crect fill=\"%23e5e7eb\" width=\"100\" height=\"100\"/%3E%3Ctext x=\"50\" y=\"55\" text-anchor=\"middle\" fill=\"%239ca3af\" font-size=\"10\"%3ESem Foto%3C/text%3E%3C/svg%3E'}" 
                                     class="w-16 h-16 object-cover rounded-lg border">
                                <div>
                                    <h3 class="font-bold text-[#1e3a5f]">${bomba.marca}</h3>
                                    <p class="text-sm text-gray-600">${bomba.modelo}</p>
                                </div>
                            </div>
                            <button onclick="abrirFicha('${bomba.id}')" class="text-gray-400 hover:text-[#1e3a5f] p-1" title="Ver Ficha Técnica">
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
                                </svg>
                            </button>
                        </div>
                        
                        ${compat ? `
                            <div class="mb-3 ${classeBadge} text-white px-3 py-2 rounded-lg text-center">
                                <span class="font-semibold">${compat.classificacao.toUpperCase()}</span>
                                <span class="text-sm ml-2">Sobra: ${compat.sobra}% | H: ${compat.alturaReal} mca</span>
                            </div>
                        ` : ''}

                        <div class="grid grid-cols-2 gap-2 text-xs text-gray-600 mb-3">
                            <div><strong>Potência:</strong> ${bomba.potencia} CV (${potenciaKW} kW)</div>
                            <div><strong>Tensão:</strong> ${bomba.tensao}V</div>
                            <div><strong>Fase:</strong> ${bomba.fase === 'monofasico' ? 'Mono' : 'Tri'}</div>
                            <div><strong>Eficiência:</strong> ${bomba.eficiencia || 70}%</div>
                            <div><strong>Consumo Est.:</strong> ${consumo} kW</div>
                            <div><strong>Rotor:</strong> ${bomba.medidaRotor || '-'} mm</div>
                        </div>

                        <div class="flex items-center justify-between mb-3">
                            <span class="text-xs px-2 py-1 rounded-full ${estoqueColor}">${estoqueText}</span>
                            <span class="text-xs text-gray-500">${bomba.codigo}</span>
                        </div>

                        <div class="flex items-center gap-2 border-t pt-3">
                            <input type="checkbox" id="sel-${bomba.id}" class="w-4 h-4 rounded border-gray-300 text-green-600 focus:ring-green-500" 
                                   onchange="toggleSelection('${bomba.id}', this.checked)" 
                                   ${orcamentoItems.find(i => i.id === bomba.id) ? 'checked' : ''}>
                            <input type="number" id="qty-${bomba.id}" class="w-16 px-2 py-1 border rounded text-sm" 
                                   placeholder="Qtd" min="1" value="${orcamentoItems.find(i => i.id === bomba.id)?.quantidade || 1}"
                                   onchange="updateQuantidade('${bomba.id}', this.value)">
                            <button onclick="toggleCompare('${bomba.id}')" 
                                    class="ml-auto px-3 py-1 rounded text-sm font-medium transition ${isInCompare ? 'bg-green-600 text-white' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'}">
                                ${isInCompare ? '✓ Comparar' : '⚖️ Comparar'}
                            </button>
                        </div>
                    </div>
                `;
            }).join('');
        }

        // ==================== SELECTION & COMPARE ====================
        function toggleSelection(id, checked) {
            const bomba = bombas.find(b => b.id === id);
            if (!bomba) return;

            if (checked) {
                const qty = parseInt(document.getElementById(`qty-${id}`).value) || 1;
                if (!orcamentoItems.find(i => i.id === id)) {
                    orcamentoItems.push({ ...bomba, quantidade: qty, pontoTrabalho: { ...pontoTrabalho } });
                }
            } else {
                orcamentoItems = orcamentoItems.filter(i => i.id !== id);
            }
            saveToStorage();
            updateOrcamentoCount();
        }

        function updateQuantidade(id, qty) {
            const item = orcamentoItems.find(i => i.id === id);
            if (item) {
                item.quantidade = parseInt(qty) || 1;
                saveToStorage();
            }
        }

        function toggleCompare(id) {
            if (compareList.includes(id)) {
                compareList = compareList.filter(i => i !== id);
            } else if (compareList.length < 5) {
                compareList.push(id);
            } else {
                alert('Máximo de 5 bombas para comparação!');
                return;
            }
            updateCompareBadge();
            renderBombas();
        }

        function updateCompareBadge() {
            const badge = document.getElementById('compareBadge');
            const count = document.getElementById('compareCount');
            count.textContent = compareList.length;
            badge.classList.toggle('hidden', compareList.length === 0);
        }

        function updateOrcamentoCount() {
            const badge = document.getElementById('orcamentoCount');
            badge.textContent = orcamentoItems.length;
            badge.classList.toggle('hidden', orcamentoItems.length === 0);
        }

        // ==================== MODALS ====================
        function fecharModal(id) {
            document.getElementById(id).classList.add('hidden');
        }

        function abrirFicha(id) {
            const bomba = bombas.find(b => b.id === id);
            if (!bomba) return;

            const potenciaKW = (bomba.potencia * 0.7355).toFixed(2);
            const consumo = (potenciaKW / (bomba.eficiencia / 100 || 0.7)).toFixed(2);
            const compat = bomba.compatibilidade;

            // Store current bomba for PDF export
            window.currentFichaBomba = bomba;

            document.getElementById('fichaContent').innerHTML = `
                <div id="fichaPrintArea">
                    <div class="grid md:grid-cols-3 gap-6">
                        <div class="text-center">
                            <img src="${bomba.foto || 'data:image/svg+xml,%3Csvg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 100 100\"%3E%3Crect fill=\"%23e5e7eb\" width=\"100\" height=\"100\"/%3E%3C/svg%3E'}" 
                                 class="w-full max-w-[200px] mx-auto rounded-xl border-2 border-gray-200 mb-4">
                            <h2 class="text-2xl font-bold text-[#1e3a5f]">${bomba.marca}</h2>
                            <p class="text-lg text-gray-600">${bomba.modelo}</p>
                            <p class="text-sm text-gray-500 mt-1">Código: ${bomba.codigo}</p>
                            ${compat ? `
                                <div class="mt-4 ${compat.classificacao === 'excelente' ? 'bg-green-500' : compat.classificacao === 'boa' ? 'bg-yellow-500' : 'bg-red-500'} text-white px-4 py-2 rounded-lg">
                                    <strong>${compat.classificacao.toUpperCase()}</strong>
                                    <div class="text-sm">Altura Real: ${compat.alturaReal} mca</div>
                                    <div class="text-sm">Sobra: ${compat.sobra}%</div>
                                </div>
                            ` : ''}
                        </div>
                        <div class="md:col-span-2">
                            <h3 class="font-bold text-lg text-[#1e3a5f] mb-3 border-b pb-2">Especificações Técnicas</h3>
                            <div class="grid grid-cols-2 md:grid-cols-3 gap-3 text-sm">
                                <div class="bg-gray-50 p-2 rounded"><strong>Potência:</strong> ${bomba.potencia} CV</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Potência:</strong> ${potenciaKW} kW</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Polos:</strong> ${bomba.polos}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Fase:</strong> ${bomba.fase === 'monofasico' ? 'Monofásico' : 'Trifásico'}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Tensão:</strong> ${bomba.tensao}V</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Frequência:</strong> ${bomba.frequencia} Hz</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Inversor:</strong> ${bomba.inversor === 'sim' ? 'Sim' : 'Não'}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Grau Proteção:</strong> ${bomba.grauProtecao}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Categoria:</strong> ${bomba.categoria}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Sucção:</strong> ${bomba.succao} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Recalque:</strong> ${bomba.recalque} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Tipo Rotor:</strong> ${bomba.tipoRotor}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Material:</strong> ${bomba.material}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Posição:</strong> ${bomba.posicao}</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Medida Rotor:</strong> ${bomba.medidaRotor} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Alt. Máx Sucção:</strong> ${bomba.altMaxSuccao} m</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Pressão Máx:</strong> ${bomba.pressaoMax} mca</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Eficiência:</strong> ${bomba.eficiencia || 70}%</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Consumo Est.:</strong> ${consumo} kW</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Conexão:</strong> ${bomba.conexao}</div>
                            </div>
                            
                            <h3 class="font-bold text-lg text-[#1e3a5f] mt-4 mb-3 border-b pb-2">Dimensões e Embalagem</h3>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-3 text-sm">
                                <div class="bg-gray-50 p-2 rounded"><strong>Comprimento:</strong> ${bomba.comprimento || '-'} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Largura:</strong> ${bomba.largura || '-'} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Altura:</strong> ${bomba.alturaDim || '-'} mm</div>
                                <div class="bg-gray-50 p-2 rounded"><strong>Peso:</strong> ${bomba.peso || '-'} kg</div>
                                <div class="bg-gray-50 p-2 rounded col-span-2"><strong>Embalagem:</strong> ${
                                    bomba.embalagem === 'caixa_papelao' ? 'Caixa de Papelão' :
                                    bomba.embalagem === 'pallet' ? 'Pallet' :
                                    bomba.embalagem === 'caixa_madeira' ? 'Caixa de Madeira' : 'Granel'
                                }</div>
                            </div>
                        </div>
                    </div>

                    <!-- Curva de Desempenho -->
                    <div class="mt-6">
                        <h3 class="font-bold text-lg text-[#1e3a5f] mb-3 border-b pb-2">Curva de Desempenho</h3>
                        <div class="relative bg-gray-50 rounded-lg p-4" style="height: 300px;">
                            <canvas id="fichaCurvaChart"></canvas>
                        </div>
                        ${pontoTrabalho.vazao > 0 ? `
                            <div class="mt-4 bg-red-50 border border-red-200 rounded-lg p-4">
                                <h4 class="font-semibold text-red-700 mb-2">Ponto de Trabalho Selecionado</h4>
                                <div class="grid grid-cols-2 gap-4 text-center">
                                    <div>
                                        <div class="text-2xl font-bold text-red-600">${pontoTrabalho.vazao}</div>
                                        <div class="text-sm text-gray-600">m³/h (Vazão)</div>
                                    </div>
                                    <div>
                                        <div class="text-2xl font-bold text-red-600">${pontoTrabalho.altura}</div>
                                        <div class="text-sm text-gray-600">mca (Altura)</div>
                                    </div>
                                </div>
                            </div>
                        ` : ''}
                    </div>
                </div>

                <!-- Botão Exportar PDF -->
                <div class="mt-6 pt-4 border-t flex justify-end">
                    <button id="btnExportarPDF" onclick="exportarFichaPDF()" class="btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">
                        Exportar PDF
                    </button>
                </div>
            `;
            document.getElementById('modalFicha').classList.remove('hidden');

            // Create chart for this pump
            setTimeout(() => {
                createFichaChart(bomba);
            }, 100);
        }

        function createFichaChart(bomba) {
            const ctx = document.getElementById('fichaCurvaChart');
            if (!ctx) return;

            // Destroy existing chart if exists
            if (window.fichaChartInstance) {
                window.fichaChartInstance.destroy();
            }

            const colors = [
                '#1e3a5f', '#22c55e', '#3b82f6', '#f59e0b', '#ef4444',
                '#8b5cf6', '#06b6d4', '#ec4899', '#14b8a6', '#f97316'
            ];

            const datasets = [];
            let colorIndex = 0;

            // Verificar se é bomba de pressurização com múltiplas curvas
            if (bomba.categoria === 'pressurizacao' && bomba.curvasPorBomba && Array.isArray(bomba.curvasPorBomba) && bomba.curvasPorBomba.length > 0) {
                console.log('Bomba de pressurização detectada:', bomba.curvasPorBomba.length, 'bombas');
                
                // Desenhar curva SEPARADA para cada bomba individual do sistema
                bomba.curvasPorBomba.forEach((curvaBomba, bombaNum) => {
                    if (curvaBomba && Array.isArray(curvaBomba) && curvaBomba.length > 0) {
                        // Ordenar pontos por vazão para garantir linha contínua
                        const pontosSorted = [...curvaBomba].sort((a, b) => a.vazao - b.vazao);
                        
                        console.log(`Bomba ${bombaNum + 1}:`, pontosSorted);
                        
                        datasets.push({
                            label: `${bomba.marca} ${bomba.modelo} - Bomba ${bombaNum + 1}`,
                            data: pontosSorted.map(p => ({ x: parseFloat(p.vazao), y: parseFloat(p.altura) })),
                            borderColor: colors[colorIndex % colors.length],
                            backgroundColor: colors[colorIndex % colors.length] + '20',
                            borderWidth: 3,
                            tension: 0.4,
                            pointRadius: 5,
                            pointHoverRadius: 7,
                            pointBackgroundColor: colors[colorIndex % colors.length],
                            pointBorderColor: '#ffffff',
                            pointBorderWidth: 2,
                            fill: false,
                            spanGaps: false
                        });
                        colorIndex++;
                    }
                });
            } else {
                // Bomba normal - curva única
                if (bomba.curva && Array.isArray(bomba.curva) && bomba.curva.length > 0) {
                    const pontosSorted = [...bomba.curva].sort((a, b) => a.vazao - b.vazao);
                    
                    datasets.push({
                        label: `${bomba.marca} ${bomba.modelo}`,
                        data: pontosSorted.map(p => ({ x: parseFloat(p.vazao), y: parseFloat(p.altura) })),
                        borderColor: '#1e3a5f',
                        backgroundColor: '#1e3a5f20',
                        borderWidth: 3,
                        tension: 0.4,
                        pointRadius: 5,
                        pointHoverRadius: 7,
                        pointBackgroundColor: '#1e3a5f',
                        pointBorderColor: '#ffffff',
                        pointBorderWidth: 2,
                        fill: true
                    });
                }
            }

            // Add working point if set
            if (pontoTrabalho.vazao > 0 && pontoTrabalho.altura > 0) {
                datasets.push({
                    label: 'Ponto de Trabalho',
                    data: [{ x: pontoTrabalho.vazao, y: pontoTrabalho.altura }],
                    borderColor: '#dc2626',
                    backgroundColor: '#dc2626',
                    pointRadius: 12,
                    pointStyle: 'circle',
                    showLine: false
                });

                // Add crosshair lines
                datasets.push({
                    label: '_vazaoLine',
                    data: [
                        { x: pontoTrabalho.vazao, y: 0 },
                        { x: pontoTrabalho.vazao, y: pontoTrabalho.altura }
                    ],
                    borderColor: '#dc262680',
                    borderDash: [5, 5],
                    borderWidth: 2,
                    pointRadius: 0,
                    showLine: true
                });
                datasets.push({
                    label: '_alturaLine',
                    data: [
                        { x: 0, y: pontoTrabalho.altura },
                        { x: pontoTrabalho.vazao, y: pontoTrabalho.altura }
                    ],
                    borderColor: '#dc262680',
                    borderDash: [5, 5],
                    borderWidth: 2,
                    pointRadius: 0,
                    showLine: true
                });
            }

            console.log('Datasets para o gráfico da ficha:', datasets.length);

            window.fichaChartInstance = new Chart(ctx, {
                type: 'line',
                data: { datasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    animation: {
                        duration: 0
                    },
                    interaction: {
                        intersect: false,
                        mode: 'nearest'
                    },
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top',
                            labels: {
                                filter: (item) => !item.text.startsWith('_')
                            }
                        },
                        tooltip: {
                            backgroundColor: '#1e3a5f',
                            callbacks: {
                                label: function(context) {
                                    if (context.dataset.label.startsWith('_')) return null;
                                    return `${context.dataset.label}: ${context.parsed.y.toFixed(1)} mca @ ${context.parsed.x.toFixed(1)} m³/h`;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            type: 'linear',
                            title: {
                                display: true,
                                text: 'Vazao (m3/h)',
                                font: { size: 12, weight: 'bold' },
                                color: '#1e3a5f'
                            },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Altura Manometrica (mca)',
                                font: { size: 12, weight: 'bold' },
                                color: '#1e3a5f'
                            },
                            grid: { color: 'rgba(0,0,0,0.1)' }
                        }
                    }
                }
            });
        }

        async function exportarFichaPDF() {
            const bomba = window.currentFichaBomba;
            if (!bomba) {
                alert('Erro: Bomba não encontrada.');
                return;
            }

            // Show loading message
            const btnExportar = document.getElementById('btnExportarPDF');
            if (btnExportar) {
                btnExportar.textContent = 'Gerando PDF...';
                btnExportar.disabled = true;
            }

            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF('p', 'mm', 'a4');
                const potenciaKW = (bomba.potencia * 0.7355).toFixed(2);
                const consumo = (potenciaKW / (bomba.eficiencia / 100 || 0.7)).toFixed(2);
                const hoje = new Date().toLocaleDateString('pt-BR');

                // Header
                doc.setFillColor(30, 58, 95);
                doc.rect(0, 0, 210, 30, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(20);
                doc.text('Dimensionador Conab', 15, 15);
                doc.setTextColor(34, 197, 94);
                doc.text('+', 85, 15);
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.text('Relatorio Tecnico do Equipamento', 15, 24);
                doc.text('Gerado em: ' + hoje, 155, 24);

                // Pump Photo and Info Section
                let y = 38;
                const fotoSize = 35;
                
                // Add pump photo on the left
                if (bomba.foto && bomba.foto.length > 0) {
                    try {
                        doc.addImage(bomba.foto, 'PNG', 15, y, fotoSize, fotoSize);
                    } catch (e) {
                        // If image fails, draw placeholder
                        doc.setFillColor(230, 230, 230);
                        doc.rect(15, y, fotoSize, fotoSize, 'F');
                        doc.setTextColor(150, 150, 150);
                        doc.setFontSize(8);
                        doc.text('Sem foto', 15 + fotoSize/2, y + fotoSize/2, { align: 'center' });
                    }
                } else {
                    // Draw placeholder if no photo
                    doc.setFillColor(230, 230, 230);
                    doc.rect(15, y, fotoSize, fotoSize, 'F');
                    doc.setTextColor(150, 150, 150);
                    doc.setFontSize(8);
                    doc.text('Sem foto', 15 + fotoSize/2, y + fotoSize/2, { align: 'center' });
                }

                // Pump Info (to the right of photo)
                const infoX = 15 + fotoSize + 8;
                doc.setTextColor(30, 58, 95);
                doc.setFontSize(16);
                doc.setFont(undefined, 'bold');
                doc.text(bomba.marca + ' ' + bomba.modelo, infoX, y + 10);
                doc.setFont(undefined, 'normal');
                doc.setFontSize(10);
                doc.setTextColor(100, 100, 100);
                doc.text('Codigo: ' + bomba.codigo, infoX, y + 18);
                doc.text('Potencia: ' + bomba.potencia + ' CV (' + potenciaKW + ' kW)', infoX, y + 25);
                doc.text('Categoria: ' + bomba.categoria, infoX, y + 32);
                
                y = y + fotoSize + 5;

                // Technical Specs
                doc.setFillColor(30, 58, 95);
                doc.rect(15, y, 180, 7, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('ESPECIFICACOES TECNICAS', 18, y + 5);
                
                y += 12;
                doc.setFont(undefined, 'normal');
                doc.setFontSize(8);
                doc.setTextColor(0, 0, 0);

                const faseTexto = bomba.fase === 'monofasico' ? 'Monofasico' : 'Trifasico';
                const inversorTexto = bomba.inversor === 'sim' ? 'Sim' : 'Nao';
                const embalagemTexto = bomba.embalagem === 'caixa_papelao' ? 'Caixa Papelao' : bomba.embalagem === 'pallet' ? 'Pallet' : bomba.embalagem === 'caixa_madeira' ? 'Caixa Madeira' : 'Granel';

                const specs = [
                    ['Potencia: ' + bomba.potencia + ' CV (' + potenciaKW + ' kW)', 'Polos: ' + bomba.polos, 'Fase: ' + faseTexto],
                    ['Tensao: ' + bomba.tensao + 'V', 'Frequencia: ' + bomba.frequencia + ' Hz', 'Inversor: ' + inversorTexto],
                    ['Grau Protecao: ' + bomba.grauProtecao, 'Categoria: ' + bomba.categoria, 'Material: ' + bomba.material],
                    ['Succao: ' + bomba.succao + ' mm', 'Recalque: ' + bomba.recalque + ' mm', 'Tipo Rotor: ' + bomba.tipoRotor],
                    ['Medida Rotor: ' + bomba.medidaRotor + ' mm', 'Posicao: ' + bomba.posicao, 'Conexao: ' + bomba.conexao],
                    ['Alt. Max Succao: ' + bomba.altMaxSuccao + ' m', 'Pressao Max: ' + bomba.pressaoMax + ' mca', 'Eficiencia: ' + (bomba.eficiencia || 70) + '%'],
                    ['Consumo Estimado: ' + consumo + ' kW', 'Peso: ' + (bomba.peso || '-') + ' kg', 'Embalagem: ' + embalagemTexto]
                ];

                specs.forEach((row, i) => {
                    if (i % 2 === 0) {
                        doc.setFillColor(245, 245, 245);
                        doc.rect(15, y - 3, 180, 6, 'F');
                    }
                    doc.text(row[0], 18, y);
                    doc.text(row[1], 78, y);
                    doc.text(row[2], 138, y);
                    y += 6;
                });

                // Dimensions
                y += 3;
                doc.setFillColor(30, 58, 95);
                doc.rect(15, y, 180, 7, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('DIMENSOES', 18, y + 5);
                
                y += 12;
                doc.setFont(undefined, 'normal');
                doc.setFontSize(8);
                doc.setTextColor(0, 0, 0);
                doc.setFillColor(245, 245, 245);
                doc.rect(15, y - 3, 180, 6, 'F');
                doc.text('Comprimento: ' + (bomba.comprimento || '-') + ' mm', 18, y);
                doc.text('Largura: ' + (bomba.largura || '-') + ' mm', 78, y);
                doc.text('Altura: ' + (bomba.alturaDim || '-') + ' mm', 138, y);

                // Working Point Section
                y += 12;
                doc.setFillColor(30, 58, 95);
                doc.rect(15, y, 180, 7, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('PONTO DE TRABALHO SELECIONADO', 18, y + 5);

                y += 12;
                if (pontoTrabalho.vazao > 0 && pontoTrabalho.altura > 0) {
                    doc.setFillColor(254, 226, 226);
                    doc.rect(15, y - 3, 180, 14, 'F');
                    doc.setTextColor(185, 28, 28);
                    doc.setFontSize(11);
                    doc.setFont(undefined, 'bold');
                    doc.text('Vazao: ' + pontoTrabalho.vazao + ' m3/h', 25, y + 3);
                    doc.text('Altura Manometrica: ' + pontoTrabalho.altura + ' mca', 100, y + 3);
                    y += 18;
                } else {
                    doc.setTextColor(128, 128, 128);
                    doc.setFontSize(9);
                    doc.setFont(undefined, 'italic');
                    doc.text('Nenhum ponto de trabalho selecionado', 18, y + 2);
                    y += 10;
                }

                // Curve Section Title
                y += 3;
                doc.setFillColor(30, 58, 95);
                doc.rect(15, y, 180, 7, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('CURVA DE DESEMPENHO', 18, y + 5);
                
                y += 10;

                // Capture the chart as image using Chart.js native method
                if (window.fichaChartInstance) {
                    try {
                        // Get the chart image directly from Chart.js
                        const imgData = window.fichaChartInstance.toBase64Image('image/png', 1);
                        
                        // Add chart image
                        const imgWidth = 180;
                        const imgHeight = 70;
                        
                        doc.addImage(imgData, 'PNG', 15, y, imgWidth, imgHeight);
                        y += imgHeight + 8;
                    } catch (e) {
                        console.log('Erro ao capturar grafico:', e);
                        // Draw a placeholder if chart capture fails
                        doc.setFillColor(245, 245, 245);
                        doc.rect(15, y, 180, 50, 'F');
                        doc.setTextColor(150, 150, 150);
                        doc.setFontSize(12);
                        doc.text('Grafico nao disponivel', 105, y + 25, { align: 'center' });
                        y += 55;
                    }
                } else {
                    // No chart instance available
                    doc.setFillColor(245, 245, 245);
                    doc.rect(15, y, 180, 50, 'F');
                    doc.setTextColor(150, 150, 150);
                    doc.setFontSize(12);
                    doc.text('Grafico nao disponivel', 105, y + 25, { align: 'center' });
                    y += 55;
                }

                // Footer
                const totalPages = doc.internal.getNumberOfPages();
                for (let i = 1; i <= totalPages; i++) {
                    doc.setPage(i);
                    doc.setFontSize(7);
                    doc.setTextColor(128, 128, 128);
                    doc.text('Documento gerado pelo Dimensionador Conab+ | www.conab.com.br', 105, 290, { align: 'center' });
                }

                doc.save('relatorio_' + bomba.codigo + '_' + Date.now() + '.pdf');
                
            } catch (error) {
                console.error('Erro ao gerar PDF:', error);
                alert('Erro ao gerar PDF: ' + error.message);
            } finally {
                if (btnExportar) {
                    btnExportar.textContent = 'Exportar PDF';
                    btnExportar.disabled = false;
                }
            }
        }

        function openCompareModal() {
            if (compareList.length < 2) {
                alert('Selecione pelo menos 2 bombas para comparar!');
                return;
            }

            const bombasCompare = compareList.map(id => bombas.find(b => b.id === id)).filter(Boolean);
            
            // Store for PDF export
            window.currentCompareBombas = bombasCompare;
            
            document.getElementById('compareContent').innerHTML = `
                <table class="w-full text-sm" id="compareTable">
                    <thead>
                        <tr class="bg-gray-100">
                            <th class="p-3 text-left font-semibold text-[#1e3a5f]">Característica</th>
                            ${bombasCompare.map(b => `<th class="p-3 text-center font-semibold text-[#1e3a5f]">${b.marca}<br>${b.modelo}</th>`).join('')}
                        </tr>
                    </thead>
                    <tbody>
                        <tr class="border-b"><td class="p-2 font-medium">Foto</td>${bombasCompare.map(b => `<td class="p-2 text-center"><img src="${b.foto}" class="w-16 h-16 mx-auto rounded"></td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Código</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.codigo}</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Potência</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.potencia} CV</td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Tensão</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.tensao}V</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Fase</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.fase === 'monofasico' ? 'Mono' : 'Tri'}</td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Polos</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.polos}</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Eficiência</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.eficiencia || 70}%</td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Sucção</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.succao} mm</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Recalque</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.recalque} mm</td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Pressão Máx</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.pressaoMax} mca</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Material</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.material}</td>`).join('')}</tr>
                        <tr class="border-b bg-gray-50"><td class="p-2 font-medium">Peso</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.peso || '-'} kg</td>`).join('')}</tr>
                        <tr class="border-b"><td class="p-2 font-medium">Estoque</td>${bombasCompare.map(b => `<td class="p-2 text-center">${b.estoque === 'disponivel' ? 'Disponivel' : b.estoque === 'baixo' ? 'Baixo' : 'Esgotado'}</td>`).join('')}</tr>
                        <tr class="border-b bg-blue-50">
                            <td class="p-2 font-medium align-top">Curva de Desempenho</td>
                            ${bombasCompare.map((b, idx) => `
                                <td class="p-2 text-center">
                                    <div class="bg-white rounded-lg p-2 border" style="height: 150px;">
                                        <canvas id="compareChart-${idx}"></canvas>
                                    </div>
                                </td>
                            `).join('')}
                        </tr>
                    </tbody>
                </table>
                <div class="mt-6 flex justify-center gap-4">
                    <button onclick="exportarComparacaoPDF()" class="btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">
                        Exportar PDF
                    </button>
                    <button onclick="compareList = []; updateCompareBadge(); renderBombas(); fecharModal('modalCompare');" class="bg-red-500 hover:bg-red-600 text-white font-semibold py-3 px-8 rounded-lg transition">
                        Limpar Comparação
                    </button>
                </div>
            `;
            document.getElementById('modalCompare').classList.remove('hidden');

            // Criar gráficos pequenos para cada bomba
            setTimeout(() => {
                bombasCompare.forEach((bomba, idx) => {
                    createCompareChart(bomba, idx);
                });
            }, 100);
        }

        // Função para criar gráfico pequeno na comparação
        function createCompareChart(bomba, idx) {
            const ctx = document.getElementById(`compareChart-${idx}`);
            if (!ctx) return;

            const colors = [
                '#1e3a5f', '#22c55e', '#3b82f6', '#f59e0b', '#ef4444',
                '#8b5cf6', '#06b6d4', '#ec4899', '#14b8a6', '#f97316'
            ];

            const datasets = [];
            let colorIndex = 0;

            // Verificar se é bomba de pressurização com múltiplas curvas
            if (bomba.categoria === 'pressurizacao' && bomba.curvasPorBomba && Array.isArray(bomba.curvasPorBomba) && bomba.curvasPorBomba.length > 0) {
                // Desenhar curva para cada bomba individual do sistema
                bomba.curvasPorBomba.forEach((curvaBomba, bombaNum) => {
                    if (curvaBomba && Array.isArray(curvaBomba) && curvaBomba.length > 0) {
                        const pontosSorted = [...curvaBomba].sort((a, b) => a.vazao - b.vazao);
                        
                        datasets.push({
                            label: `B${bombaNum + 1}`,
                            data: pontosSorted.map(p => ({ x: parseFloat(p.vazao), y: parseFloat(p.altura) })),
                            borderColor: colors[colorIndex % colors.length],
                            backgroundColor: colors[colorIndex % colors.length] + '20',
                            borderWidth: 2,
                            tension: 0.4,
                            pointRadius: 2,
                            pointHoverRadius: 4,
                            fill: false
                        });
                        colorIndex++;
                    }
                });
            } else {
                // Bomba normal - curva única
                if (bomba.curva && Array.isArray(bomba.curva) && bomba.curva.length > 0) {
                    const pontosSorted = [...bomba.curva].sort((a, b) => a.vazao - b.vazao);
                    
                    datasets.push({
                        label: bomba.modelo,
                        data: pontosSorted.map(p => ({ x: parseFloat(p.vazao), y: parseFloat(p.altura) })),
                        borderColor: '#1e3a5f',
                        backgroundColor: '#1e3a5f20',
                        borderWidth: 2,
                        tension: 0.4,
                        pointRadius: 2,
                        pointHoverRadius: 4,
                        fill: true
                    });
                }
            }

            // Add working point if set
            if (pontoTrabalho.vazao > 0 && pontoTrabalho.altura > 0) {
                datasets.push({
                    label: 'PT',
                    data: [{ x: pontoTrabalho.vazao, y: pontoTrabalho.altura }],
                    borderColor: '#dc2626',
                    backgroundColor: '#dc2626',
                    pointRadius: 5,
                    pointStyle: 'circle',
                    showLine: false
                });
            }

            new Chart(ctx, {
                type: 'line',
                data: { datasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    animation: { duration: 0 },
                    plugins: {
                        legend: {
                            display: true,
                            position: 'bottom',
                            labels: {
                                boxWidth: 8,
                                padding: 4,
                                font: { size: 8 }
                            }
                        },
                        tooltip: {
                            enabled: true,
                            callbacks: {
                                label: function(context) {
                                    return `${context.parsed.y.toFixed(1)} mca @ ${context.parsed.x.toFixed(1)} m³/h`;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            type: 'linear',
                            title: {
                                display: true,
                                text: 'm³/h',
                                font: { size: 8 }
                            },
                            ticks: { font: { size: 7 } },
                            grid: { display: false }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'mca',
                                font: { size: 8 }
                            },
                            ticks: { font: { size: 7 } },
                            grid: { color: 'rgba(0,0,0,0.05)' }
                        }
                    }
                }
            });
        }

        async function exportarComparacaoPDF() {
            const bombasCompare = window.currentCompareBombas;
            if (!bombasCompare || bombasCompare.length < 2) {
                alert('Erro: Bombas não encontradas.');
                return;
            }

            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF('p', 'mm', 'a4'); // Formato retrato A4
                const hoje = new Date().toLocaleDateString('pt-BR');
                const pageWidth = 210;
                const pageHeight = 297;
                const margin = 15;
                const contentWidth = pageWidth - (margin * 2);

                // Calcular largura de cada coluna de bomba
                const numBombas = bombasCompare.length;
                const bombaColWidth = contentWidth / numBombas;

                // ===================== PÁGINA 1: HEADER E FOTOS =====================
                // Header
                doc.setFillColor(30, 58, 95);
                doc.rect(0, 0, pageWidth, 25, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(18);
                doc.text('Dimensionador Conab', margin, 14);
                doc.setTextColor(34, 197, 94);
                doc.text('+', 78, 14);
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(9);
                doc.text('Comparacao de Bombas', margin, 21);
                doc.text('Gerado em: ' + hoje, pageWidth - margin - 35, 21);

                let y = 35;

                // ===================== SEÇÃO: FOTOS E IDENTIFICAÇÃO =====================
                doc.setFillColor(30, 58, 95);
                doc.rect(margin, y, contentWidth, 8, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('IDENTIFICACAO DOS EQUIPAMENTOS', margin + 3, y + 5.5);
                y += 12;

                // Fotos das bombas lado a lado - com proporção correta
                const fotoWidth = Math.min(bombaColWidth - 15, 35);
                const fotoHeight = fotoWidth * 0.8; // Proporção 5:4 para não espremer
                const cardHeight = fotoHeight + 28;
                
                bombasCompare.forEach((b, i) => {
                    const cardX = margin + (bombaColWidth * i) + 2;
                    const fotoX = margin + (bombaColWidth * i) + (bombaColWidth - fotoWidth) / 2;
                    
                    // Fundo do card
                    doc.setFillColor(248, 250, 252);
                    doc.roundedRect(cardX, y - 2, bombaColWidth - 4, cardHeight, 3, 3, 'F');
                    
                    // Foto com proporção correta (não espremida)
                    if (b.foto && b.foto.length > 0) {
                        try {
                            doc.addImage(b.foto, 'PNG', fotoX, y + 2, fotoWidth, fotoHeight);
                        } catch (e) {
                            doc.setFillColor(230, 230, 230);
                            doc.roundedRect(fotoX, y + 2, fotoWidth, fotoHeight, 2, 2, 'F');
                            doc.setTextColor(150, 150, 150);
                            doc.setFontSize(8);
                            doc.text('Sem foto', fotoX + fotoWidth/2, y + 2 + fotoHeight/2, { align: 'center' });
                        }
                    } else {
                        doc.setFillColor(230, 230, 230);
                        doc.roundedRect(fotoX, y + 2, fotoWidth, fotoHeight, 2, 2, 'F');
                        doc.setTextColor(150, 150, 150);
                        doc.setFontSize(8);
                        doc.text('Sem foto', fotoX + fotoWidth/2, y + 2 + fotoHeight/2, { align: 'center' });
                    }
                    
                    // Nome da bomba abaixo da foto
                    const nomeX = margin + (bombaColWidth * i) + bombaColWidth/2;
                    const nomeY = y + fotoHeight + 7;
                    
                    doc.setTextColor(30, 58, 95);
                    doc.setFontSize(9);
                    doc.setFont(undefined, 'bold');
                    doc.text(b.marca, nomeX, nomeY, { align: 'center' });
                    
                    doc.setFont(undefined, 'normal');
                    doc.setFontSize(8);
                    doc.text(b.modelo, nomeX, nomeY + 5, { align: 'center' });
                    
                    doc.setTextColor(100, 100, 100);
                    doc.setFontSize(7);
                    doc.text('Cod: ' + b.codigo, nomeX, nomeY + 10, { align: 'center' });
                });

                y += cardHeight + 5;

                // ===================== SEÇÃO: ESPECIFICAÇÕES TÉCNICAS =====================
                doc.setFillColor(30, 58, 95);
                doc.rect(margin, y, contentWidth, 8, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('ESPECIFICACOES TECNICAS', margin + 3, y + 5.5);
                y += 12;

                // Tabela de especificações
                const specs = [
                    { label: 'Potencia', values: bombasCompare.map(b => b.potencia + ' CV (' + (b.potencia * 0.7355).toFixed(2) + ' kW)') },
                    { label: 'Tensao', values: bombasCompare.map(b => b.tensao + 'V') },
                    { label: 'Fase', values: bombasCompare.map(b => b.fase === 'monofasico' ? 'Monofasico' : 'Trifasico') },
                    { label: 'Polos', values: bombasCompare.map(b => b.polos + ' polos') },
                    { label: 'Frequencia', values: bombasCompare.map(b => b.frequencia + ' Hz') },
                    { label: 'Eficiencia', values: bombasCompare.map(b => (b.eficiencia || 70) + '%') },
                    { label: 'Grau Protecao', values: bombasCompare.map(b => b.grauProtecao || '-') },
                    { label: 'Succao', values: bombasCompare.map(b => (b.succao || '-') + ' mm') },
                    { label: 'Recalque', values: bombasCompare.map(b => (b.recalque || '-') + ' mm') },
                    { label: 'Pressao Max', values: bombasCompare.map(b => (b.pressaoMax || '-') + ' mca') },
                    { label: 'Material', values: bombasCompare.map(b => b.material || '-') },
                    { label: 'Tipo Rotor', values: bombasCompare.map(b => b.tipoRotor || '-') },
                    { label: 'Peso', values: bombasCompare.map(b => (b.peso || '-') + ' kg') },
                    { label: 'Estoque', values: bombasCompare.map(b => b.estoque === 'disponivel' ? 'Em Estoque' : b.estoque === 'baixo' ? 'Baixo' : 'Esgotado') }
                ];

                doc.setFontSize(7);
                specs.forEach((spec, i) => {
                    // Alternar cor de fundo
                    if (i % 2 === 0) {
                        doc.setFillColor(248, 250, 252);
                        doc.rect(margin, y - 2, contentWidth, 6, 'F');
                    }
                    
                    // Label
                    doc.setTextColor(30, 58, 95);
                    doc.setFont(undefined, 'bold');
                    doc.text(spec.label + ':', margin + 2, y + 2);
                    
                    // Valores
                    doc.setFont(undefined, 'normal');
                    doc.setTextColor(0, 0, 0);
                    spec.values.forEach((val, j) => {
                        const x = margin + 35 + (bombaColWidth - 35/numBombas) * j;
                        doc.text(String(val), x, y + 2);
                    });
                    
                    y += 6;
                });

                // ===================== PÁGINA 2: CURVAS DE DESEMPENHO =====================
                doc.addPage();
                
                // Header página 2
                doc.setFillColor(30, 58, 95);
                doc.rect(0, 0, pageWidth, 25, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(18);
                doc.text('Dimensionador Conab', margin, 14);
                doc.setTextColor(34, 197, 94);
                doc.text('+', 78, 14);
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(9);
                doc.text('Comparacao de Bombas - Curvas de Desempenho', margin, 21);
                doc.text('Pagina 2', pageWidth - margin - 20, 21);

                y = 35;

                // Título da seção
                doc.setFillColor(30, 58, 95);
                doc.rect(margin, y, contentWidth, 8, 'F');
                doc.setTextColor(255, 255, 255);
                doc.setFontSize(10);
                doc.setFont(undefined, 'bold');
                doc.text('CURVAS DE DESEMPENHO', margin + 3, y + 5.5);
                y += 15;

                // Curvas de cada bomba - com tamanho proporcional e ajustado ao quadro
                const chartHeight = 65;
                const chartMargin = 8;
                const chartWidth = bombaColWidth - (chartMargin * 2);
                const cardChartHeight = chartHeight + 35;

                for (let i = 0; i < bombasCompare.length; i++) {
                    const b = bombasCompare[i];
                    const cardX = margin + (bombaColWidth * i) + 2;
                    const chartX = margin + (bombaColWidth * i) + chartMargin;
                    
                    // Card da bomba com borda
                    doc.setFillColor(248, 250, 252);
                    doc.roundedRect(cardX, y - 3, bombaColWidth - 4, cardChartHeight, 3, 3, 'F');
                    doc.setDrawColor(200, 200, 200);
                    doc.roundedRect(cardX, y - 3, bombaColWidth - 4, cardChartHeight, 3, 3, 'S');
                    
                    // Nome da bomba
                    doc.setTextColor(30, 58, 95);
                    doc.setFontSize(8);
                    doc.setFont(undefined, 'bold');
                    doc.text(b.marca + ' ' + b.modelo, cardX + (bombaColWidth - 4)/2, y + 5, { align: 'center' });
                    
                    // Área do gráfico com fundo branco
                    doc.setFillColor(255, 255, 255);
                    doc.roundedRect(chartX, y + 10, chartWidth, chartHeight, 2, 2, 'F');
                    
                    // Capturar gráfico
                    const canvas = document.getElementById(`compareChart-${i}`);
                    if (canvas) {
                        try {
                            const imgData = canvas.toDataURL('image/png', 1);
                            // Adiciona a imagem dentro da área branca
                            doc.addImage(imgData, 'PNG', chartX + 2, y + 12, chartWidth - 4, chartHeight - 4);
                        } catch (e) {
                            doc.setFillColor(230, 230, 230);
                            doc.rect(chartX + 2, y + 12, chartWidth - 4, chartHeight - 4, 'F');
                            doc.setTextColor(150, 150, 150);
                            doc.setFontSize(7);
                            doc.text('Grafico nao disponivel', cardX + (bombaColWidth - 4)/2, y + 10 + chartHeight/2, { align: 'center' });
                        }
                    }
                    
                    // Info do ponto de trabalho
                    if (pontoTrabalho.vazao > 0 && pontoTrabalho.altura > 0) {
                        doc.setTextColor(220, 38, 38);
                        doc.setFontSize(6);
                        doc.setFont(undefined, 'bold');
                        doc.text('PT: ' + pontoTrabalho.vazao + ' m3/h @ ' + pontoTrabalho.altura + ' mca', cardX + (bombaColWidth - 4)/2, y + chartHeight + 18, { align: 'center' });
                    }
                }

                // Footer
                doc.setFontSize(7);
                doc.setTextColor(128, 128, 128);
                doc.text('Documento gerado pelo Dimensionador Conab+ | www.conab.com.br', pageWidth/2, pageHeight - 10, { align: 'center' });

                // Adicionar número de páginas
                const totalPages = doc.internal.getNumberOfPages();
                for (let i = 1; i <= totalPages; i++) {
                    doc.setPage(i);
                    doc.setFontSize(7);
                    doc.setTextColor(128, 128, 128);
                    doc.text('Pagina ' + i + ' de ' + totalPages, pageWidth - margin, pageHeight - 10);
                }

                doc.save('comparacao_bombas_' + Date.now() + '.pdf');
                
            } catch (error) {
                console.error('Erro ao gerar PDF:', error);
                alert('Erro ao gerar PDF: ' + error.message);
            }
        }

        // ==================== CADASTRO ====================
        document.getElementById('formCadastro').addEventListener('submit', (e) => {
            e.preventDefault();
            
            const codigo = document.getElementById('cadCodigo').value.trim();
            const marca = document.getElementById('cadMarca').value.trim();
            const modelo = document.getElementById('cadModelo').value.trim();
            
            // Validação: Código único
            if (bombas.find(b => b.codigo.toLowerCase() === codigo.toLowerCase())) {
                alert('Código já existe! Use um código único.');
                return;
            }
            
            // Validação: Marca + Modelo não podem ser idênticos a outro cadastro
            if (bombas.find(b => b.marca.toLowerCase() === marca.toLowerCase() && b.modelo.toLowerCase() === modelo.toLowerCase())) {
                alert('Já existe uma bomba cadastrada com a mesma Marca e Modelo!\n\nMarca: ' + marca + '\nModelo: ' + modelo);
                return;
            }

            const categoria = document.getElementById('cadCategoria').value;
            let curva = [];
            let curvasPorBomba = [];
            let qtdBombasOp = null;
            let qtdBombasRes = null;

            // Se for pressurização, coletar dados das bombas individuais
            if (categoria === 'pressurizacao') {
                qtdBombasOp = parseInt(document.getElementById('cadQtdBombasOp').value) || 1;

                // Coletar pontos de cada bomba
                for (let i = 1; i <= qtdBombasOp; i++) {
                    const pontosInputs = document.querySelectorAll(`.vazao-bomba-${i}`);
                    const pontosBomba = [];
                    
                    pontosInputs.forEach((input, idx) => {
                        const vazao = parseFloat(input.value);
                        const altura = parseFloat(document.querySelectorAll(`.altura-bomba-${i}`)[idx].value);
                        if (!isNaN(vazao) && !isNaN(altura)) {
                            pontosBomba.push({ vazao, altura });
                        }
                    });

                    if (pontosBomba.length < 2) {
                        alert(`A curva da Bomba ${i} deve ter no mínimo 2 pontos!`);
                        return;
                    }

                    pontosBomba.sort((a, b) => a.vazao - b.vazao);
                    curvasPorBomba.push(pontosBomba);
                }

                // Usar a curva da primeira bomba como curva principal (para compatibilidade)
                curva = curvasPorBomba[0];
            } else {
                // Bomba normal - coletar curva única
                const curvaPoints = document.querySelectorAll('.curva-point');
                curvaPoints.forEach(point => {
                    const vazao = parseFloat(point.querySelector('.curva-vazao').value);
                    const altura = parseFloat(point.querySelector('.curva-altura').value);
                    if (!isNaN(vazao) && !isNaN(altura)) {
                        curva.push({ vazao, altura });
                    }
                });

                if (curva.length < 2) {
                    alert('A curva deve ter no mínimo 2 pontos!');
                    return;
                }

                curva.sort((a, b) => a.vazao - b.vazao);
            }

            const novaBomba = {
                id: codigo,
                codigo,
                marca: document.getElementById('cadMarca').value,
                modelo: document.getElementById('cadModelo').value,
                potencia: parseFloat(document.getElementById('cadPotencia').value),
                polos: parseInt(document.getElementById('cadPolos').value),
                inversor: document.getElementById('cadInversor').value,
                fase: document.getElementById('cadFase').value,
                tensao: document.getElementById('cadTensao').value,
                grauProtecao: document.getElementById('cadGrauProtecao').value,
                succao: parseInt(document.getElementById('cadSuccao').value) || 0,
                recalque: parseInt(document.getElementById('cadRecalque').value) || 0,
                frequencia: document.getElementById('cadFrequencia').value,
                categoria: categoria,
                tipoRotor: document.getElementById('cadTipoRotor').value,
                material: document.getElementById('cadMaterial').value,
                posicao: document.getElementById('cadPosicao').value,
                medidaRotor: parseInt(document.getElementById('cadMedidaRotor').value) || 0,
                altMaxSuccao: parseFloat(document.getElementById('cadAltMaxSuccao').value) || 0,
                pressaoMax: parseFloat(document.getElementById('cadPressaoMax').value) || 0,
                comprimento: parseInt(document.getElementById('cadComprimento').value) || 0,
                largura: parseInt(document.getElementById('cadLargura').value) || 0,
                alturaDim: parseInt(document.getElementById('cadAlturaDim').value) || 0,
                peso: parseFloat(document.getElementById('cadPeso').value) || 0,
                embalagem: document.getElementById('cadEmbalagem').value,
                conexao: document.getElementById('cadConexao').value,
                estoque: document.getElementById('cadEstoque').value,
                eficiencia: parseFloat(document.getElementById('cadEficiencia').value) || 70,
                foto: document.getElementById('previewImg').src || '',
                curva
            };

            // Adicionar dados extras de pressurização se aplicável
            if (categoria === 'pressurizacao') {
                novaBomba.qtdBombasOp = qtdBombasOp;
                novaBomba.curvasPorBomba = curvasPorBomba;
            }

            bombas.push(novaBomba);
            saveToStorage();
            populateFilters();
            bombasFiltradas = [...bombas];
            renderBombas();
            updateChart();
            
            e.target.reset();
            document.getElementById('previewImg').classList.add('hidden');
            alert('Bomba cadastrada com sucesso!');
        });

        function adicionarPontoCurva() {
            const container = document.getElementById('curvaPoints');
            const div = document.createElement('div');
            div.className = 'flex gap-4 items-center curva-point';
            div.innerHTML = `
                <input type="number" placeholder="Vazão (m³/h)" class="curva-vazao filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <input type="number" placeholder="Altura (mca)" class="curva-altura filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <button type="button" onclick="removerPontoCurva(this)" class="text-red-500 hover:text-red-700 p-2">✕</button>
            `;
            container.appendChild(div);
        }

        function removerPontoCurva(btn) {
            const points = document.querySelectorAll('.curva-point');
            if (points.length > 2) {
                btn.parentElement.remove();
            } else {
                alert('Mínimo de 2 pontos na curva!');
            }
        }

        function previewFoto(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    document.getElementById('previewImg').src = e.target.result;
                    document.getElementById('previewImg').classList.remove('hidden');
                };
                reader.readAsDataURL(file);
            }
        }

        // ==================== PRESSURIZAÇÃO ====================
        function togglePressurizacao() {
            const categoria = document.getElementById('cadCategoria').value;
            const curvaSection = document.getElementById('curvaSection');
            const pressurizacaoSection = document.getElementById('pressurizacaoSection');
            
            if (categoria === 'pressurizacao') {
                curvaSection.classList.add('hidden');
                pressurizacaoSection.classList.remove('hidden');
                gerarCamposBombas();
            } else {
                curvaSection.classList.remove('hidden');
                pressurizacaoSection.classList.add('hidden');
            }
        }

        function gerarCamposBombas() {
            const qtdBombas = parseInt(document.getElementById('cadQtdBombasOp').value) || 1;
            const container = document.getElementById('pontosPorBomba');
            
            container.innerHTML = '';
            
            for (let i = 1; i <= qtdBombas; i++) {
                const bombaDiv = document.createElement('div');
                bombaDiv.className = 'bg-gray-50 border border-gray-200 rounded-lg p-4';
                bombaDiv.innerHTML = `
                    <h4 class="font-bold text-gray-700 mb-3">Bomba ${i} - Pontos de Trabalho</h4>
                    <div class="pontos-bomba-${i} space-y-2">
                        <div class="flex gap-4 items-center ponto-item">
                            <input type="number" placeholder="Vazão (m³/h)" class="vazao-bomba-${i} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                            <input type="number" placeholder="Altura (mca)" class="altura-bomba-${i} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                            <button type="button" onclick="removerPontoBomba(this, ${i})" class="text-red-500 hover:text-red-700 p-2">✕</button>
                        </div>
                        <div class="flex gap-4 items-center ponto-item">
                            <input type="number" placeholder="Vazão (m³/h)" class="vazao-bomba-${i} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                            <input type="number" placeholder="Altura (mca)" class="altura-bomba-${i} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                            <button type="button" onclick="removerPontoBomba(this, ${i})" class="text-red-500 hover:text-red-700 p-2">✕</button>
                        </div>
                    </div>
                    <button type="button" onclick="adicionarPontoBomba(${i})" class="mt-2 text-green-600 hover:text-green-700 font-medium text-sm">
                        + Adicionar Ponto à Bomba ${i}
                    </button>
                `;
                container.appendChild(bombaDiv);
            }
        }

        function adicionarPontoBomba(numBomba) {
            const container = document.querySelector(`.pontos-bomba-${numBomba}`);
            const div = document.createElement('div');
            div.className = 'flex gap-4 items-center ponto-item';
            div.innerHTML = `
                <input type="number" placeholder="Vazão (m³/h)" class="vazao-bomba-${numBomba} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <input type="number" placeholder="Altura (mca)" class="altura-bomba-${numBomba} filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <button type="button" onclick="removerPontoBomba(this, ${numBomba})" class="text-red-500 hover:text-red-700 p-2">✕</button>
            `;
            container.appendChild(div);
        }

        function removerPontoBomba(btn, numBomba) {
            const container = document.querySelector(`.pontos-bomba-${numBomba}`);
            const pontos = container.querySelectorAll('.ponto-item');
            if (pontos.length > 2) {
                btn.parentElement.remove();
            } else {
                alert('Mínimo de 2 pontos por bomba!');
            }
        }

        // ==================== ORÇAMENTO ====================
        function updateOrcamentoView() {
            const container = document.getElementById('orcamentoBombas');
            updateOrcamentoCount();

            if (orcamentoItems.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-8">Nenhuma bomba selecionada. Selecione bombas na aba Dimensionamento.</p>';
                return;
            }

            container.innerHTML = orcamentoItems.map(item => `
                <div class="flex items-center gap-4 p-4 bg-gray-50 rounded-lg border">
                    <img src="${item.foto}" class="w-16 h-16 object-cover rounded-lg border">
                    <div class="flex-1">
                        <h4 class="font-bold text-[#1e3a5f]">${item.marca} ${item.modelo}</h4>
                        <p class="text-sm text-gray-600">${item.potencia} CV | ${item.tensao}V | ${item.fase === 'monofasico' ? 'Mono' : 'Tri'}</p>
                        ${item.pontoTrabalho?.vazao ? `<p class="text-xs text-gray-500">Ponto: ${item.pontoTrabalho.vazao} m³/h @ ${item.pontoTrabalho.altura} mca</p>` : ''}
                    </div>
                    <div class="flex items-center gap-2">
                        <label class="text-sm text-gray-600">Qtd:</label>
                        <input type="number" value="${item.quantidade}" min="1" 
                               class="w-16 px-2 py-1 border rounded text-center"
                               onchange="updateOrcamentoQty('${item.id}', this.value)">
                    </div>
                    <button onclick="removerDoOrcamento('${item.id}')" class="text-red-500 hover:text-red-700 p-2">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                        </svg>
                    </button>
                </div>
            `).join('');
        }

        function updateOrcamentoQty(id, qty) {
            const item = orcamentoItems.find(i => i.id === id);
            if (item) {
                item.quantidade = parseInt(qty) || 1;
                saveToStorage();
            }
        }

        function removerDoOrcamento(id) {
            orcamentoItems = orcamentoItems.filter(i => i.id !== id);
            saveToStorage();
            updateOrcamentoView();
            renderBombas();
        }

        function salvarOrcamento() {
            const cliente = document.getElementById('orcCliente').value.trim();
            if (!cliente) {
                alert('Informe o nome do cliente!');
                return;
            }
            if (orcamentoItems.length === 0) {
                alert('Adicione pelo menos uma bomba ao orçamento!');
                return;
            }

            const orcamento = {
                id: Date.now().toString(),
                cliente,
                os: document.getElementById('orcOS').value,
                numero: document.getElementById('orcNumero').value,
                referencia: document.getElementById('orcReferencia').value,
                setor: document.getElementById('orcSetor').value,
                data: document.getElementById('orcData').value,
                items: [...orcamentoItems]
            };

            orcamentos.push(orcamento);
            orcamentoItems = [];
            saveToStorage();
            updateOrcamentoView();
            renderBombas();
            updateRelatorioFilters();
            
            document.getElementById('formOrcamento').reset();
            document.getElementById('orcData').valueAsDate = new Date();
            
            alert('Orçamento salvo com sucesso!');
        }

        async function exportarOrcamentoPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'mm', 'a4');
            
            const cliente = document.getElementById('orcCliente').value || 'Cliente';
            const data = document.getElementById('orcData').value || new Date().toLocaleDateString('pt-BR');
            
            // Header
            doc.setFillColor(30, 58, 95);
            doc.rect(0, 0, 210, 35, 'F');
            doc.setTextColor(255, 255, 255);
            doc.setFontSize(24);
            doc.text('Dimensionador Conab', 15, 20);
            doc.setTextColor(34, 197, 94);
            doc.text('+', 100, 20);
            
            doc.setTextColor(255, 255, 255);
            doc.setFontSize(10);
            doc.text(`Orçamento para: ${cliente}`, 15, 30);
            doc.text(`Data: ${data}`, 160, 30);

            // Content
            doc.setTextColor(0, 0, 0);
            let y = 50;

            doc.setFontSize(14);
            doc.text('Bombas Selecionadas:', 15, y);
            y += 10;

            orcamentoItems.forEach((item, index) => {
                if (y > 260) {
                    doc.addPage();
                    y = 20;
                }
                
                doc.setFillColor(245, 245, 245);
                doc.rect(15, y - 5, 180, 25, 'F');
                
                doc.setFontSize(11);
                doc.setFont(undefined, 'bold');
                doc.text(`${index + 1}. ${item.marca} ${item.modelo}`, 20, y);
                doc.setFont(undefined, 'normal');
                doc.setFontSize(9);
                doc.text(`Código: ${item.codigo} | Potência: ${item.potencia} CV | Tensão: ${item.tensao}V`, 20, y + 6);
                doc.text(`Quantidade: ${item.quantidade} unidade(s)`, 20, y + 12);
                if (item.pontoTrabalho?.vazao) {
                    doc.text(`Ponto de Trabalho: ${item.pontoTrabalho.vazao} m³/h @ ${item.pontoTrabalho.altura} mca`, 20, y + 18);
                }
                
                y += 30;
            });

            // Footer
            doc.setFontSize(8);
            doc.setTextColor(128, 128, 128);
            doc.text('Documento gerado pelo Dimensionador Conab+ | www.conab.com.br', 105, 290, { align: 'center' });

            doc.save(`orcamento_${cliente.replace(/\s+/g, '_')}_${Date.now()}.pdf`);
        }

        // ==================== LISTA ====================
        function renderLista() {
            const tbody = document.getElementById('listaBombasTable');
            if (!tbody) {
                console.error('Elemento listaBombasTable não encontrado');
                return;
            }
            
            console.log('renderLista - iniciando');
            console.log('renderLista - bombas:', Array.isArray(bombas) ? bombas.length : 'não é array');
            console.log('renderLista - currentUser:', currentUser);
            
            // Verificar se bombas existe e é um array
            if (!Array.isArray(bombas)) {
                tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-8 text-center text-red-500">Erro: dados de bombas inválidos</td></tr>';
                return;
            }
            
            // Verificar se há bombas
            if (bombas.length === 0) {
                tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-8 text-center text-gray-500">Nenhuma bomba cadastrada</td></tr>';
                return;
            }
            
            // Verificar permissões
            const isAdmin = currentUser && currentUser.isAdmin === true;
            const canEdit = isAdmin || (currentUser && currentUser.permissoes && currentUser.permissoes.alterarBombas === true);
            const canDelete = isAdmin || (currentUser && currentUser.permissoes && currentUser.permissoes.excluirBombas === true);
            
            console.log('renderLista - isAdmin:', isAdmin);
            console.log('renderLista - canEdit:', canEdit);
            console.log('renderLista - canDelete:', canDelete);
            
            try {
                // Renderizar bombas
                const linhas = bombas.map((bomba, index) => {
                    // Proteção contra dados inválidos
                    const codigo = bomba.codigo || '';
                    const marca = bomba.marca || '';
                    const modelo = bomba.modelo || '';
                    const potencia = bomba.potencia || 0;
                    const fase = bomba.fase || 'trifasico';
                    const tensao = bomba.tensao || '';
                    const estoque = bomba.estoque || 'disponivel';
                    
                    // Determinar cor e texto do estoque
                    let estoqueColor = 'bg-green-100 text-green-800';
                    let estoqueText = 'Disponível';
                    
                    if (estoque === 'baixo') {
                        estoqueColor = 'bg-yellow-100 text-yellow-800';
                        estoqueText = 'Baixo';
                    } else if (estoque === 'esgotado') {
                        estoqueColor = 'bg-red-100 text-red-800';
                        estoqueText = 'Esgotado';
                    }
                    
                    // Determinar quais botões mostrar
                    const showEditBtn = isAdmin || canEdit;
                    const showDeleteBtn = isAdmin || canDelete;
                    
                    return `
                        <tr class="border-b hover:bg-gray-50">
                            <td class="px-4 py-3">${codigo}</td>
                            <td class="px-4 py-3">${marca}</td>
                            <td class="px-4 py-3">${modelo}</td>
                            <td class="px-4 py-3 text-center">${potencia} CV</td>
                            <td class="px-4 py-3 text-center">${fase === 'monofasico' ? 'Mono' : 'Tri'}</td>
                            <td class="px-4 py-3 text-center">${tensao}V</td>
                            <td class="px-4 py-3 text-center"><span class="px-2 py-1 rounded-full text-xs ${estoqueColor}">${estoqueText}</span></td>
                            <td class="px-4 py-3 text-center">
                                ${showEditBtn ? `<button onclick="editarBomba('${bomba.id}')" class="text-blue-600 hover:text-blue-800 mr-2" title="Editar">✏️</button>` : ''}
                                ${showDeleteBtn ? `<button onclick="excluirBomba('${bomba.id}')" class="text-red-600 hover:text-red-800" title="Excluir">🗑️</button>` : ''}
                                ${!showEditBtn && !showDeleteBtn ? '<span class="text-gray-400 text-xs">Sem permissão</span>' : ''}
                            </td>
                        </tr>
                    `;
                });
                
                tbody.innerHTML = linhas.join('');
                console.log('renderLista - concluído com sucesso');
                
            } catch (error) {
                console.error('Erro ao renderizar lista:', error);
                tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-8 text-center text-red-500">Erro ao carregar lista de bombas</td></tr>';
            }
        }

        function filtrarLista() {
            const query = document.getElementById('searchLista').value.toLowerCase();
            const rows = document.querySelectorAll('#listaBombasTable tr');
            rows.forEach(row => {
                const text = row.textContent.toLowerCase();
                row.style.display = text.includes(query) ? '' : 'none';
            });
        }

        function editarBomba(id) {
            const bomba = bombas.find(b => b.id === id);
            if (!bomba) return;

            const isPressurization = bomba.categoria === 'pressurizacao';
            
            let curvaHTML = '';
            if (isPressurization && bomba.curvasPorBomba && bomba.curvasPorBomba.length > 0) {
                // Edição de bombas de pressurização
                curvaHTML = `
                    <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
                        <h3 class="font-bold text-blue-800 mb-3">Configuração do Sistema de Pressurização</h3>
                        <div class="grid grid-cols-1 gap-4 mb-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Quant. de Bombas do Sistema</label>
                                <input type="number" id="editQtdBombasOp" value="${bomba.qtdBombasOp || 1}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" min="1">
                            </div>
                        </div>
                        <div id="editPontosPorBomba">
                            ${bomba.curvasPorBomba.map((curva, bombaNum) => `
                                <div class="mb-4 bg-white p-3 rounded">
                                    <h4 class="font-bold text-gray-700 mb-2">Bomba ${bombaNum + 1} - Pontos de Trabalho</h4>
                                    <div class="edit-pontos-bomba-${bombaNum} space-y-2">
                                        ${curva.map((ponto, i) => `
                                            <div class="flex gap-4 items-center edit-ponto-item">
                                                <input type="number" value="${ponto.vazao}" class="edit-vazao-${bombaNum} filter-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="Vazão (m³/h)" step="0.001">
                                                <input type="number" value="${ponto.altura}" class="edit-altura-${bombaNum} filter-input w-full px-3 py-2 border border-gray-300 rounded-lg" placeholder="Altura (mca)" step="0.001">
                                                <button type="button" onclick="removerPontoEditBomba(this, ${bombaNum})" class="text-red-500 hover:text-red-700 p-2">✕</button>
                                            </div>
                                        `).join('')}
                                    </div>
                                    <button type="button" onclick="adicionarPontoEditBomba(${bombaNum})" class="mt-2 text-green-600 hover:text-green-700 font-medium text-sm">
                                        + Adicionar Ponto à Bomba ${bombaNum + 1}
                                    </button>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            } else {
                // Edição de bombas normais
                curvaHTML = `
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Curva de Desempenho</label>
                        <div id="editCurvaPoints" class="space-y-2">
                            ${bomba.curva.map((ponto, i) => `
                                <div class="flex gap-4 items-center edit-curva-point">
                                    <input type="number" value="${ponto.vazao}" class="edit-curva-vazao filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Vazão (m³/h)" step="0.001">
                                    <input type="number" value="${ponto.altura}" class="edit-curva-altura filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Altura (mca)" step="0.001">
                                    <button type="button" onclick="removerPontoEdit(this)" class="text-red-500 hover:text-red-700 p-2">✕</button>
                                </div>
                            `).join('')}
                        </div>
                        <button type="button" onclick="adicionarPontoEdit()" class="mt-2 text-green-600 hover:text-green-700 font-medium">
                            + Adicionar Ponto
                        </button>
                    </div>
                `;
            }

            document.getElementById('editContent').innerHTML = `
                <form id="formEdit" class="space-y-6">
                    <input type="hidden" id="editId" value="${bomba.id}">
                    
                    <!-- Basic Info -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Código *</label>
                            <input type="text" id="editCodigo" value="${bomba.codigo}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" readonly>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Marca *</label>
                            <input type="text" id="editMarca" value="${bomba.marca}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Modelo *</label>
                            <input type="text" id="editModelo" value="${bomba.modelo}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                        </div>
                    </div>

                    <!-- Technical Specs -->
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Potência (CV) *</label>
                            <input type="number" id="editPotencia" value="${bomba.potencia}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Polos *</label>
                            <select id="editPolos" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                                <option value="2" ${bomba.polos == 2 ? 'selected' : ''}>2 Polos</option>
                                <option value="4" ${bomba.polos == 4 ? 'selected' : ''}>4 Polos</option>
                                <option value="6" ${bomba.polos == 6 ? 'selected' : ''}>6 Polos</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Inversor</label>
                            <select id="editInversor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="nao" ${bomba.inversor === 'nao' ? 'selected' : ''}>Não</option>
                                <option value="sim" ${bomba.inversor === 'sim' ? 'selected' : ''}>Sim</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Fase *</label>
                            <select id="editFase" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" required>
                                <option value="monofasico" ${bomba.fase === 'monofasico' ? 'selected' : ''}>Monofásico</option>
                                <option value="trifasico" ${bomba.fase === 'trifasico' ? 'selected' : ''}>Trifásico</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tensão (V) *</label>
                            <input type="text" id="editTensao" value="${bomba.tensao}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: 220V, 380V" required>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Grau Proteção</label>
                            <select id="editGrauProtecao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="IP21" ${bomba.grauProtecao === 'IP21' ? 'selected' : ''}>IP21</option>
                                <option value="IP44" ${bomba.grauProtecao === 'IP44' ? 'selected' : ''}>IP44</option>
                                <option value="IP55" ${bomba.grauProtecao === 'IP55' ? 'selected' : ''}>IP55</option>
                                <option value="IP65" ${bomba.grauProtecao === 'IP65' ? 'selected' : ''}>IP65</option>
                                <option value="IP68" ${bomba.grauProtecao === 'IP68' ? 'selected' : ''}>IP68</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Sucção (mm)</label>
                            <input type="number" id="editSuccao" value="${bomba.succao || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Recalque (mm)</label>
                            <input type="number" id="editRecalque" value="${bomba.recalque || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Frequência (Hz)</label>
                            <select id="editFrequencia" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="60" ${bomba.frequencia === '60' ? 'selected' : ''}>60 Hz</option>
                                <option value="50" ${bomba.frequencia === '50' ? 'selected' : ''}>50 Hz</option>
                                <option value="50/60" ${bomba.frequencia === '50/60' ? 'selected' : ''}>50/60 Hz</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Categoria</label>
                            <select id="editCategoria" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="centrifuga" ${bomba.categoria === 'centrifuga' ? 'selected' : ''}>Centrífuga</option>
                                <option value="submersa" ${bomba.categoria === 'submersa' ? 'selected' : ''}>Submersa</option>
                                <option value="autoaspirante" ${bomba.categoria === 'autoaspirante' ? 'selected' : ''}>Autoaspirante</option>
                                <option value="multicelular" ${bomba.categoria === 'multicelular' ? 'selected' : ''}>Multicelular</option>
                                <option value="pressurizacao" ${bomba.categoria === 'pressurizacao' ? 'selected' : ''}>Pressurização</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Rotor</label>
                            <select id="editTipoRotor" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="fechado" ${bomba.tipoRotor === 'fechado' ? 'selected' : ''}>Fechado</option>
                                <option value="aberto" ${bomba.tipoRotor === 'aberto' ? 'selected' : ''}>Aberto</option>
                                <option value="semiaberto" ${bomba.tipoRotor === 'semiaberto' ? 'selected' : ''}>Semi-aberto</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Material</label>
                            <input type="text" id="editMaterial" value="${bomba.material}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" placeholder="Ex: Ferro Fundido, Inox">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Posição</label>
                            <select id="editPosicao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="horizontal" ${bomba.posicao === 'horizontal' ? 'selected' : ''}>Horizontal</option>
                                <option value="vertical" ${bomba.posicao === 'vertical' ? 'selected' : ''}>Vertical</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Medida Rotor (mm)</label>
                            <input type="number" id="editMedidaRotor" value="${bomba.medidaRotor || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Alt. Máx Sucção (m)</label>
                            <input type="number" id="editAltMaxSuccao" value="${bomba.altMaxSuccao || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Pressão Máx (mca)</label>
                            <input type="number" id="editPressaoMax" value="${bomba.pressaoMax || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                    </div>

                    <!-- Dimensions & Packaging -->
                    <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Comprimento (mm)</label>
                            <input type="number" id="editComprimento" value="${bomba.comprimento || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Largura (mm)</label>
                            <input type="number" id="editLargura" value="${bomba.largura || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Altura (mm)</label>
                            <input type="number" id="editAlturaDim" value="${bomba.alturaDim || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Peso (kg)</label>
                            <input type="number" id="editPeso" value="${bomba.peso || ''}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1">
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Embalagem</label>
                            <select id="editEmbalagem" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="caixa_papelao" ${bomba.embalagem === 'caixa_papelao' ? 'selected' : ''}>Caixa de Papelão</option>
                                <option value="pallet" ${bomba.embalagem === 'pallet' ? 'selected' : ''}>Pallet</option>
                                <option value="caixa_madeira" ${bomba.embalagem === 'caixa_madeira' ? 'selected' : ''}>Caixa de Madeira</option>
                                <option value="granel" ${bomba.embalagem === 'granel' ? 'selected' : ''}>Granel</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Tipo Conexão</label>
                            <select id="editConexao" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="rosca" ${bomba.conexao === 'rosca' ? 'selected' : ''}>Rosca</option>
                                <option value="flange" ${bomba.conexao === 'flange' ? 'selected' : ''}>Flange</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Estoque</label>
                            <select id="editEstoque" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg">
                                <option value="disponivel" ${bomba.estoque === 'disponivel' ? 'selected' : ''}>Em Estoque</option>
                                <option value="baixo" ${bomba.estoque === 'baixo' ? 'selected' : ''}>Baixo Estoque</option>
                                <option value="esgotado" ${bomba.estoque === 'esgotado' ? 'selected' : ''}>Esgotado</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Eficiência (%)</label>
                            <input type="number" id="editEficiencia" value="${bomba.eficiencia || 70}" class="filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.1" max="100">
                        </div>
                    </div>

                    <!-- Curve Points -->
                    ${curvaHTML}

                    <div class="flex gap-4 pt-4 border-t">
                        <button type="submit" class="btn-primary text-white font-semibold py-3 px-8 rounded-lg transition">Salvar Alterações</button>
                        <button type="button" onclick="fecharModal('modalEdit')" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-8 rounded-lg transition">Cancelar</button>
                    </div>
                </form>
            `;

            document.getElementById('formEdit').addEventListener('submit', (e) => {
                e.preventDefault();
                const idx = bombas.findIndex(b => b.id === document.getElementById('editId').value);
                if (idx !== -1) {
                    // Update all fields
                    bombas[idx].marca = document.getElementById('editMarca').value;
                    bombas[idx].modelo = document.getElementById('editModelo').value;
                    bombas[idx].potencia = parseFloat(document.getElementById('editPotencia').value);
                    bombas[idx].polos = parseInt(document.getElementById('editPolos').value);
                    bombas[idx].inversor = document.getElementById('editInversor').value;
                    bombas[idx].fase = document.getElementById('editFase').value;
                    bombas[idx].tensao = document.getElementById('editTensao').value;
                    bombas[idx].grauProtecao = document.getElementById('editGrauProtecao').value;
                    bombas[idx].succao = parseInt(document.getElementById('editSuccao').value) || 0;
                    bombas[idx].recalque = parseInt(document.getElementById('editRecalque').value) || 0;
                    bombas[idx].frequencia = document.getElementById('editFrequencia').value;
                    bombas[idx].categoria = document.getElementById('editCategoria').value;
                    bombas[idx].tipoRotor = document.getElementById('editTipoRotor').value;
                    bombas[idx].material = document.getElementById('editMaterial').value;
                    bombas[idx].posicao = document.getElementById('editPosicao').value;
                    bombas[idx].medidaRotor = parseInt(document.getElementById('editMedidaRotor').value) || 0;
                    bombas[idx].altMaxSuccao = parseFloat(document.getElementById('editAltMaxSuccao').value) || 0;
                    bombas[idx].pressaoMax = parseFloat(document.getElementById('editPressaoMax').value) || 0;
                    bombas[idx].comprimento = parseInt(document.getElementById('editComprimento').value) || 0;
                    bombas[idx].largura = parseInt(document.getElementById('editLargura').value) || 0;
                    bombas[idx].alturaDim = parseInt(document.getElementById('editAlturaDim').value) || 0;
                    bombas[idx].peso = parseFloat(document.getElementById('editPeso').value) || 0;
                    bombas[idx].embalagem = document.getElementById('editEmbalagem').value;
                    bombas[idx].conexao = document.getElementById('editConexao').value;
                    bombas[idx].estoque = document.getElementById('editEstoque').value;
                    bombas[idx].eficiencia = parseFloat(document.getElementById('editEficiencia').value) || 70;

                    // Update curve based on type
                    if (isPressurization) {
                        bombas[idx].qtdBombasOp = parseInt(document.getElementById('editQtdBombasOp').value) || 1;
                        
                        const curvasPorBomba = [];
                        for (let i = 0; i < bombas[idx].qtdBombasOp; i++) {
                            const vazaoInputs = document.querySelectorAll(`.edit-vazao-${i}`);
                            const alturaInputs = document.querySelectorAll(`.edit-altura-${i}`);
                            const pontos = [];
                            vazaoInputs.forEach((input, idx) => {
                                const vazao = parseFloat(input.value);
                                const altura = parseFloat(alturaInputs[idx].value);
                                if (!isNaN(vazao) && !isNaN(altura)) {
                                    pontos.push({ vazao, altura });
                                }
                            });
                            pontos.sort((a, b) => a.vazao - b.vazao);
                            curvasPorBomba.push(pontos);
                        }
                        bombas[idx].curvasPorBomba = curvasPorBomba;
                        bombas[idx].curva = curvasPorBomba[0]; // Use first pump curve as main
                    } else {
                        const curvaPoints = document.querySelectorAll('.edit-curva-point');
                        const curva = [];
                        curvaPoints.forEach(point => {
                            const vazao = parseFloat(point.querySelector('.edit-curva-vazao').value);
                            const altura = parseFloat(point.querySelector('.edit-curva-altura').value);
                            if (!isNaN(vazao) && !isNaN(altura)) {
                                curva.push({ vazao, altura });
                            }
                        });
                        curva.sort((a, b) => a.vazao - b.vazao);
                        bombas[idx].curva = curva;
                    }

                    saveToStorage();
                    renderLista();
                    populateFilters();
                    bombasFiltradas = [...bombas];
                    renderBombas();
                    updateChart();
                    fecharModal('modalEdit');
                    alert('Bomba atualizada com sucesso!');
                }
            });

            document.getElementById('modalEdit').classList.remove('hidden');
        }

        function excluirBomba(id) {
            if (confirm('Tem certeza que deseja excluir esta bomba?')) {
                bombas = bombas.filter(b => b.id !== id);
                saveToStorage();
                renderLista();
                populateFilters();
                bombasFiltradas = [...bombas];
                renderBombas();
                updateChart();
            }
        }

        // Funções para adicionar/remover pontos na edição de bombas normais
        function adicionarPontoEdit() {
            const container = document.getElementById('editCurvaPoints');
            const div = document.createElement('div');
            div.className = 'flex gap-4 items-center edit-curva-point';
            div.innerHTML = `
                <input type="number" placeholder="Vazão (m³/h)" class="edit-curva-vazao filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <input type="number" placeholder="Altura (mca)" class="edit-curva-altura filter-input w-full px-4 py-2 border border-gray-300 rounded-lg" step="0.001">
                <button type="button" onclick="removerPontoEdit(this)" class="text-red-500 hover:text-red-700 p-2">✕</button>
            `;
            container.appendChild(div);
        }

        function removerPontoEdit(btn) {
            const points = document.querySelectorAll('.edit-curva-point');
            if (points.length > 2) {
                btn.parentElement.remove();
            } else {
                alert('Mínimo de 2 pontos na curva!');
            }
        }

        // Funções para adicionar/remover pontos na edição de bombas de pressurização
        function adicionarPontoEditBomba(numBomba) {
            const container = document.querySelector(`.edit-pontos-bomba-${numBomba}`);
            const div = document.createElement('div');
            div.className = 'flex gap-4 items-center edit-ponto-item';
            div.innerHTML = `
                <input type="number" placeholder="Vazão (m³/h)" class="edit-vazao-${numBomba} filter-input w-full px-3 py-2 border border-gray-300 rounded-lg" step="0.001">
                <input type="number" placeholder="Altura (mca)" class="edit-altura-${numBomba} filter-input w-full px-3 py-2 border border-gray-300 rounded-lg" step="0.001">
                <button type="button" onclick="removerPontoEditBomba(this, ${numBomba})" class="text-red-500 hover:text-red-700 p-2">✕</button>
            `;
            container.appendChild(div);
        }

        function removerPontoEditBomba(btn, numBomba) {
            const container = document.querySelector(`.edit-pontos-bomba-${numBomba}`);
            const pontos = container.querySelectorAll('.edit-ponto-item');
            if (pontos.length > 2) {
                btn.parentElement.remove();
            } else {
                alert('Mínimo de 2 pontos por bomba!');
            }
        }

        // ==================== RELATÓRIOS ====================
        function updateRelatorioFilters() {
            const clientes = [...new Set(orcamentos.map(o => o.cliente))];
            const marcas = [...new Set(bombas.map(b => b.marca))];

            document.getElementById('relCliente').innerHTML = '<option value="">Todos</option>' + 
                clientes.map(c => `<option value="${c}">${c}</option>`).join('');
            document.getElementById('relMarca').innerHTML = '<option value="">Todas</option>' + 
                marcas.map(m => `<option value="${m}">${m}</option>`).join('');
        }

        function gerarRelatorio(tipo) {
            const preview = document.getElementById('relatorioPreview');
            const content = document.getElementById('relatorioContent');
            preview.classList.remove('hidden');

            let html = '';
            const filtroCliente = document.getElementById('relCliente').value;
            const filtroMarca = document.getElementById('relMarca').value;
            const filtroModelo = document.getElementById('relModelo').value.toLowerCase();

            switch(tipo) {
                case 'clientes':
                    const clientes = [...new Set(orcamentos.map(o => o.cliente))];
                    html = `
                        <h4 class="font-bold text-lg mb-4">Lista de Clientes (${clientes.length})</h4>
                        <ul class="space-y-2">
                            ${clientes.map(c => {
                                const orcs = orcamentos.filter(o => o.cliente === c);
                                return `<li class="p-3 bg-gray-50 rounded-lg">
                                    <strong>${c}</strong> - ${orcs.length} orçamento(s)
                                </li>`;
                            }).join('')}
                        </ul>
                    `;
                    break;

                case 'orcamentos':
                    let orcs = orcamentos;
                    if (filtroCliente) orcs = orcs.filter(o => o.cliente === filtroCliente);
                    html = `
                        <h4 class="font-bold text-lg mb-4">Orçamentos Gerados (${orcs.length})</h4>
                        <div class="space-y-4">
                            ${orcs.map(o => `
                                <div class="p-4 bg-gray-50 rounded-lg border">
                                    <div class="flex justify-between items-start">
                                        <div>
                                            <strong>${o.cliente}</strong>
                                            <p class="text-sm text-gray-600">Nº ${o.numero || '-'} | O.S: ${o.os || '-'}</p>
                                        </div>
                                        <span class="text-sm text-gray-500">${o.data || '-'}</span>
                                    </div>
                                    <div class="mt-2 text-sm">
                                        <strong>${o.items.length}</strong> bomba(s): ${o.items.map(i => i.modelo).join(', ')}
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    `;
                    break;

                case 'marcas':
                    const marcasCount = {};
                    orcamentos.forEach(o => {
                        o.items.forEach(i => {
                            marcasCount[i.marca] = (marcasCount[i.marca] || 0) + i.quantidade;
                        });
                    });
                    const sorted = Object.entries(marcasCount).sort((a, b) => b[1] - a[1]);
                    html = `
                        <h4 class="font-bold text-lg mb-4">Marcas Mais Orçadas</h4>
                        <div class="space-y-2">
                            ${sorted.map(([marca, count]) => `
                                <div class="flex items-center gap-4 p-3 bg-gray-50 rounded-lg">
                                    <span class="font-semibold flex-1">${marca}</span>
                                    <span class="bg-green-100 text-green-800 px-3 py-1 rounded-full text-sm">${count} unid.</span>
                                </div>
                            `).join('')}
                        </div>
                    `;
                    break;

                case 'bombas':
                    let bombasList = [...bombas];
                    if (filtroMarca) bombasList = bombasList.filter(b => b.marca === filtroMarca);
                    if (filtroModelo) bombasList = bombasList.filter(b => b.modelo.toLowerCase().includes(filtroModelo));
                    html = `
                        <h4 class="font-bold text-lg mb-4">Catálogo de Bombas (${bombasList.length})</h4>
                        <div class="overflow-x-auto">
                            <table class="w-full text-sm">
                                <thead>
                                    <tr class="bg-gray-100">
                                        <th class="p-2 text-left">Código</th>
                                        <th class="p-2 text-left">Marca</th>
                                        <th class="p-2 text-left">Modelo</th>
                                        <th class="p-2 text-center">Potência</th>
                                        <th class="p-2 text-center">Tensão</th>
                                        <th class="p-2 text-center">Estoque</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    ${bombasList.map(b => `
                                        <tr class="border-b">
                                            <td class="p-2">${b.codigo}</td>
                                            <td class="p-2">${b.marca}</td>
                                            <td class="p-2">${b.modelo}</td>
                                            <td class="p-2 text-center">${b.potencia} CV</td>
                                            <td class="p-2 text-center">${b.tensao}V</td>
                                            <td class="p-2 text-center">${b.estoque}</td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    `;
                    break;
            }

            content.innerHTML = html;
            window.currentReportType = tipo;
        }

        async function exportarRelatorioPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF('p', 'mm', 'a4');
            
            const hoje = new Date().toLocaleDateString('pt-BR');
            
            // Header
            doc.setFillColor(30, 58, 95);
            doc.rect(0, 0, 210, 30, 'F');
            doc.setTextColor(255, 255, 255);
            doc.setFontSize(20);
            doc.text('Dimensionador Conab', 15, 18);
            doc.setTextColor(34, 197, 94);
            doc.text('+', 85, 18);
            doc.setTextColor(255, 255, 255);
            doc.setFontSize(10);
            doc.text(`Relatório gerado em: ${hoje}`, 140, 18);

            // Content
            doc.setTextColor(0, 0, 0);
            let y = 45;
            
            doc.setFontSize(14);
            doc.text(`Relatório: ${window.currentReportType?.toUpperCase() || 'GERAL'}`, 15, y);
            y += 15;

            // Simple text export of the report
            const content = document.getElementById('relatorioContent').innerText;
            const lines = doc.splitTextToSize(content, 180);
            
            doc.setFontSize(10);
            lines.forEach(line => {
                if (y > 280) {
                    doc.addPage();
                    y = 20;
                }
                doc.text(line, 15, y);
                y += 6;
            });

            // Footer
            doc.setFontSize(8);
            doc.setTextColor(128, 128, 128);
            const pageCount = doc.internal.getNumberOfPages();
            for (let i = 1; i <= pageCount; i++) {
                doc.setPage(i);
                doc.text(`Página ${i} de ${pageCount} | Dimensionador Conab+`, 105, 290, { align: 'center' });
            }

            doc.save(`relatorio_${window.currentReportType || 'geral'}_${Date.now()}.pdf`);
        }

        // Initialize
        bombasFiltradas = [...bombas];

        // ==================== CONFIGURAÇÕES - ADMIN ====================
        let usuarios = [];
        let clientes = [];
        let listasSuspensas = {
            categorias: ['centrifuga', 'submersa', 'autoaspirante', 'multicelular', 'pressurizacao'],
            polos: ['2', '4', '6'],
            fases: ['monofasico', 'trifasico'],
            grausProtecao: ['IP21', 'IP44', 'IP55', 'IP65', 'IP68'],
            frequencias: ['50', '60', '50/60'],
            tiposRotor: ['fechado', 'aberto', 'semiaberto'],
            posicoes: ['horizontal', 'vertical'],
            embalagens: ['caixa_papelao', 'pallet', 'caixa_madeira', 'granel'],
            conexoes: ['rosca', 'flange'],
            inversores: ['sim', 'nao'],
            estoques: ['disponivel', 'baixo', 'esgotado'],
            tensoes: ['127', '220', '380', '440']
        };
        let currentUser = null;
        const ADMIN_USER = 'admin';
        const ADMIN_PASS = 'admin123';

        // Carregar dados de configuração
        async function loadConfigData() {
            if (firebaseInitialized) {
                try {
                    const usuariosSnap = await database.ref('usuarios').once('value');
                    if (usuariosSnap.exists()) usuarios = Object.values(usuariosSnap.val());
                    
                    const clientesSnap = await database.ref('clientes').once('value');
                    if (clientesSnap.exists()) clientes = Object.values(clientesSnap.val());
                    
                    const listasSnap = await database.ref('listasSuspensas').once('value');
                    if (listasSnap.exists()) listasSuspensas = { ...listasSuspensas, ...listasSnap.val() };
                } catch (e) {
                    console.log('Erro ao carregar config:', e);
                }
            } else {
                const storedUsuarios = localStorage.getItem('conab_usuarios');
                if (storedUsuarios) usuarios = JSON.parse(storedUsuarios);
                const storedClientes = localStorage.getItem('conab_clientes');
                if (storedClientes) clientes = JSON.parse(storedClientes);
                const storedListas = localStorage.getItem('conab_listas');
                if (storedListas) listasSuspensas = { ...listasSuspensas, ...JSON.parse(storedListas) };
            }
        }

        async function saveConfigData() {
            if (firebaseInitialized) {
                try {
                    const usuariosObj = {};
                    usuarios.forEach(u => { usuariosObj[u.id] = u; });
                    await database.ref('usuarios').set(usuariosObj);
                    
                    const clientesObj = {};
                    clientes.forEach(c => { clientesObj[c.id] = c; });
                    await database.ref('clientes').set(clientesObj);
                    
                    await database.ref('listasSuspensas').set(listasSuspensas);
                } catch (e) {
                    console.log('Erro ao salvar config:', e);
                }
            } else {
                localStorage.setItem('conab_usuarios', JSON.stringify(usuarios));
                localStorage.setItem('conab_clientes', JSON.stringify(clientes));
                localStorage.setItem('conab_listas', JSON.stringify(listasSuspensas));
            }
        }

        // Login Admin (na aba de configurações - apenas para admin)
        function loginAdmin() {
            // Se o usuário atual é admin, mostrar o painel
            if (currentUser && currentUser.isAdmin) {
                document.getElementById('adminLogin').classList.add('hidden');
                document.getElementById('adminPanel').classList.remove('hidden');
                renderUsuarios();
                renderClientes();
                renderTodasListas();
            } else {
                // Verificar credenciais do admin
                const user = document.getElementById('adminUser').value;
                const pass = document.getElementById('adminPass').value;
                
                if (user === ADMIN_USER && pass === ADMIN_PASS) {
                    document.getElementById('adminLogin').classList.add('hidden');
                    document.getElementById('adminPanel').classList.remove('hidden');
                    document.getElementById('loginError').classList.add('hidden');
                    renderUsuarios();
                    renderClientes();
                    renderTodasListas();
                } else {
                    document.getElementById('loginError').classList.remove('hidden');
                }
            }
        }

        function logoutAdmin() {
            document.getElementById('adminLogin').classList.remove('hidden');
            document.getElementById('adminPanel').classList.add('hidden');
            document.getElementById('adminUser').value = '';
            document.getElementById('adminPass').value = '';
        }

        // Gerenciar Usuários
        function adicionarUsuario() {
            const nome = document.getElementById('novoUserNome').value.trim();
            const login = document.getElementById('novoUserLogin').value.trim();
            const senha = document.getElementById('novoUserSenha').value.trim();
            
            if (!nome || !login || !senha) {
                alert('Preencha todos os campos!');
                return;
            }
            
            if (usuarios.find(u => u.login === login)) {
                alert('Login já existe!');
                return;
            }
            
            usuarios.push({
                id: Date.now().toString(),
                nome,
                login,
                senha,
                permissoes: {
                    alterarBombas: false,
                    excluirBombas: false,
                    excluirOrcamentos: false,
                    gerenciarClientes: false
                }
            });
            
            saveConfigData();
            renderUsuarios();
            
            document.getElementById('novoUserNome').value = '';
            document.getElementById('novoUserLogin').value = '';
            document.getElementById('novoUserSenha').value = '';
        }

        function renderUsuarios() {
            const tbody = document.getElementById('listaUsuarios');
            tbody.innerHTML = usuarios.map(u => `
                <tr class="border-b hover:bg-gray-50">
                    <td class="px-4 py-3">${u.nome}</td>
                    <td class="px-4 py-3">${u.login}</td>
                    <td class="px-4 py-3 text-center">
                        <input type="checkbox" ${u.permissoes.alterarBombas ? 'checked' : ''} 
                               onchange="togglePermissao('${u.id}', 'alterarBombas', this.checked)"
                               class="w-4 h-4 rounded border-gray-300 text-green-600 focus:ring-green-500">
                    </td>
                    <td class="px-4 py-3 text-center">
                        <input type="checkbox" ${u.permissoes.excluirBombas ? 'checked' : ''} 
                               onchange="togglePermissao('${u.id}', 'excluirBombas', this.checked)"
                               class="w-4 h-4 rounded border-gray-300 text-green-600 focus:ring-green-500">
                    </td>
                    <td class="px-4 py-3 text-center">
                        <input type="checkbox" ${u.permissoes.excluirOrcamentos ? 'checked' : ''} 
                               onchange="togglePermissao('${u.id}', 'excluirOrcamentos', this.checked)"
                               class="w-4 h-4 rounded border-gray-300 text-green-600 focus:ring-green-500">
                    </td>
                    <td class="px-4 py-3 text-center">
                        <input type="checkbox" ${u.permissoes.gerenciarClientes ? 'checked' : ''} 
                               onchange="togglePermissao('${u.id}', 'gerenciarClientes', this.checked)"
                               class="w-4 h-4 rounded border-gray-300 text-green-600 focus:ring-green-500">
                    </td>
                    <td class="px-4 py-3 text-center">
                        <button onclick="excluirUsuario('${u.id}')" class="text-red-600 hover:text-red-800">🗑️</button>
                    </td>
                </tr>
            `).join('');
        }

        function togglePermissao(userId, permissao, valor) {
            const usuario = usuarios.find(u => u.id === userId);
            if (usuario) {
                usuario.permissoes[permissao] = valor;
                saveConfigData();
            }
        }

        function excluirUsuario(userId) {
            if (confirm('Tem certeza que deseja excluir este usuário?')) {
                usuarios = usuarios.filter(u => u.id !== userId);
                saveConfigData();
                renderUsuarios();
            }
        }

        // Gerenciar Listas Suspensas
        function adicionarItemLista(lista, inputId) {
            const input = document.getElementById(inputId);
            const valor = input.value.trim();
            
            if (!valor) return;
            
            if (!listasSuspensas[lista].includes(valor)) {
                listasSuspensas[lista].push(valor);
                saveConfigData();
                renderListaSuspensa(lista);
            }
            
            input.value = '';
        }

        function removerItemLista(lista, valor) {
            listasSuspensas[lista] = listasSuspensas[lista].filter(v => v !== valor);
            saveConfigData();
            renderListaSuspensa(lista);
        }

        function renderListaSuspensa(lista) {
            const containerId = 'lista' + lista.charAt(0).toUpperCase() + lista.slice(1);
            const container = document.getElementById(containerId);
            if (!container) return;
            
            container.innerHTML = listasSuspensas[lista].map(item => `
                <li class="flex items-center justify-between bg-white px-3 py-2 rounded border">
                    <span class="text-sm">${item}</span>
                    <button onclick="removerItemLista('${lista}', '${item}')" class="text-red-500 hover:text-red-700 text-sm">✕</button>
                </li>
            `).join('');
        }

        function renderTodasListas() {
            Object.keys(listasSuspensas).forEach(lista => renderListaSuspensa(lista));
        }

        // Gerenciar Clientes
        function adicionarCliente() {
            const nome = document.getElementById('novoClienteNome').value.trim();
            const doc = document.getElementById('novoClienteDoc').value.trim();
            const tel = document.getElementById('novoClienteTel').value.trim();
            
            if (!nome) {
                alert('Informe o nome do cliente!');
                return;
            }
            
            clientes.push({
                id: Date.now().toString(),
                nome,
                documento: doc,
                telefone: tel
            });
            
            saveConfigData();
            renderClientes();
            
            document.getElementById('novoClienteNome').value = '';
            document.getElementById('novoClienteDoc').value = '';
            document.getElementById('novoClienteTel').value = '';
        }

        function renderClientes() {
            const tbody = document.getElementById('listaClientes');
            tbody.innerHTML = clientes.map(c => `
                <tr class="border-b hover:bg-gray-50">
                    <td class="px-4 py-3">${c.nome}</td>
                    <td class="px-4 py-3">${c.documento || '-'}</td>
                    <td class="px-4 py-3">${c.telefone || '-'}</td>
                    <td class="px-4 py-3 text-center">
                        <button onclick="excluirCliente('${c.id}')" class="text-red-600 hover:text-red-800">🗑️</button>
                    </td>
                </tr>
            `).join('');
        }

        function excluirCliente(clienteId) {
            if (confirm('Tem certeza que deseja excluir este cliente?')) {
                clientes = clientes.filter(c => c.id !== clienteId);
                saveConfigData();
                renderClientes();
            }
        }

        // Verificar permissões
        function temPermissao(permissao) {
            // Se não há usuário logado, bloqueia tudo (precisa fazer login)
            if (!currentUser) return false;
            // Admin tem todas as permissões
            if (currentUser.isAdmin) return true;
            // Verifica permissão específica do usuário
            return currentUser.permissoes && currentUser.permissoes[permissao] === true;
        }

        // Função para atualizar visibilidade das abas baseado nas permissões
        function atualizarVisibilidadeAbas() {
            // Primeiro, mostrar todas as abas
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.style.display = '';
            });
            
            // Se é admin, mostra tudo
            if (currentUser && currentUser.isAdmin) {
                return;
            }
            
            // Para usuários com permissões limitadas, ajusta a visibilidade
            // Dimensionamento, Orçamento e Relatórios sempre visíveis
            
            // Cadastrar Bomba - precisa de permissão alterarBombas
            const tabCadastrar = document.querySelector('[data-tab="cadastrar"]');
            if (tabCadastrar) {
                tabCadastrar.style.display = temPermissao('alterarBombas') ? '' : 'none';
            }
            
            // Lista de Bombas - precisa de permissão alterarBombas OU excluirBombas
            const tabLista = document.querySelector('[data-tab="lista"]');
            if (tabLista) {
                tabLista.style.display = (temPermissao('alterarBombas') || temPermissao('excluirBombas')) ? '' : 'none';
            }
            
            // Configurações - só admin
            const tabConfig = document.querySelector('[data-tab="configuracoes"]');
            if (tabConfig) {
                tabConfig.style.display = (currentUser && currentUser.isAdmin) ? '' : 'none';
            }
        }

        // Inicializar configurações
        loadConfigData();
    </script>
</body>
</html>
