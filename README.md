<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Multiplayer para Família</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .card {
            background-color: white;
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            padding: 2rem;
            transition: all 0.3s ease;
        }
        .btn {
            display: inline-flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
            font-weight: 600;
            text-align: left;
            user-select: none;
            border: 1px solid transparent;
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            line-height: 1.5;
            border-radius: 0.5rem;
            transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
        }
        .btn:disabled {
            opacity: 0.8;
            cursor: not-allowed;
        }
        .btn-primary {
            color: #fff;
            background-color: #4f46e5;
            border-color: #4f46e5;
        }
        .btn-primary:hover:not(:disabled) {
            background-color: #4338ca;
            border-color: #3730a3;
        }
        .btn-secondary {
            color: #4f46e5;
            background-color: #e0e7ff;
            border-color: transparent;
        }
        .btn-secondary:hover:not(:disabled) {
            background-color: #c7d2fe;
        }
        .quiz-option {
            font-size: 1rem;
            padding: 0.75rem 1rem;
            justify-content: flex-start; /* Align text to the left */
        }
        .quiz-option.selected {
            background-color: #4f46e5;
            color: white;
            border-color: #4338ca;
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.4);
        }
        .input-field {
            display: block;
            width: 100%;
            padding: 0.75rem 1rem;
            font-size: 1rem;
            line-height: 1.5;
            color: #111827;
            background-color: #f9fafb;
            border: 1px solid #d1d5db;
            border-radius: 0.5rem;
            transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
        }
        .input-field:focus {
            outline: none;
            border-color: #4f46e5;
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.2);
        }
        #game-code-display {
            background-color: #f3f4f6;
            border: 2px dashed #d1d5db;
            padding: 1rem;
            border-radius: 8px;
            font-size: 1.5rem;
            font-weight: 700;
            color: #374151;
            cursor: pointer;
            text-align: center;
        }
        .vote-count-badge {
            display: flex;
            align-items: center;
            justify-content: center;
            min-width: 24px;
            height: 24px;
            padding: 0 8px;
            border-radius: 9999px;
            font-size: 0.75rem;
            font-weight: 700;
        }
        /* Visual question image styling */
        .question-image {
            max-height: 10rem; /* 160px */
            width: auto;
            margin-left: auto;
            margin-right: auto;
            margin-bottom: 1rem; /* 16px */
            border-radius: 0.375rem; /* rounded-md */
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4">

    <div id="app" class="max-w-3xl w-full mx-auto">
        <!-- Tela de Início / Lobby -->
        <div id="lobby-screen" class="card text-center">
            <h1 class="text-4xl font-bold text-gray-800 mb-4">Quiz em Família!</h1>
            <p class="text-gray-600 mb-8">Crie uma sala ou junte-se a uma existente para começar a diversão.</p>

            <div class="space-y-4">
                <input type="text" id="player-name" class="input-field text-center" placeholder="Digite seu nome">
                <button id="create-game-btn" class="btn btn-primary w-full justify-center">Criar Novo Jogo</button>
                <div class="flex items-center my-4">
                    <hr class="flex-grow border-t border-gray-300">
                    <span class="px-4 text-gray-500 font-semibold">OU</span>
                    <hr class="flex-grow border-t border-gray-300">
                </div>
                <input type="text" id="game-code-input" class="input-field text-center" placeholder="Digite o código da sala">
                <button id="join-game-btn" class="btn btn-secondary w-full justify-center">Entrar no Jogo</button>
            </div>
            <div id="error-message" class="text-red-500 mt-4 font-semibold"></div>
            <div class="mt-6 text-xs text-gray-400">
                <p>Seu ID de Jogador: <span id="user-id-display"></span></p>
            </div>
        </div>

        <!-- Tela de Espera -->
        <div id="waiting-room-screen" class="hidden card">
            <h2 class="text-2xl font-bold text-gray-800 mb-2">Sala de Espera</h2>
            <p class="text-gray-600 mb-4">Compartilhe o código com outros jogadores!</p>
            <div id="game-code-display" title="Clique para copiar"><span id="game-code-text"></span></div>
            <p class="text-sm text-gray-500 mt-2 text-center">Clique no código para copiar</p>
            
            <h3 class="text-xl font-semibold text-gray-700 mt-8 mb-4">Jogadores na Sala:</h3>
            <ul id="player-list" class="space-y-2"></ul>
            
            <div id="host-controls" class="hidden mt-8">
                <div id="host-quiz-selection">
                    <h3 class="text-xl font-semibold text-gray-700 mb-4">Escolha o Tema do Quiz:</h3>
                    <div id="quiz-options-container" class="grid grid-cols-1 gap-2 mb-6"></div>
                </div>
                <button id="start-game-btn" class="btn btn-primary w-full justify-center" disabled>Começar o Jogo!</button>
            </div>
            <p id="waiting-for-host" class="mt-8 text-center text-gray-600 font-semibold">Aguardando o anfitrião escolher um tema e iniciar o jogo...</p>
        </div>

        <!-- Tela do Jogo -->
        <div id="game-screen" class="hidden card">
            <div class="flex justify-between items-center mb-4">
                 <div>
                    <div class="text-sm font-medium text-gray-500" id="quiz-name-display"></div>
                    <div class="text-lg font-bold text-indigo-600">Pergunta <span id="question-number"></span>/<span id="total-questions"></span></div>
                 </div>
                <div id="scoreboard-toggle" class="cursor-pointer font-semibold text-indigo-600 hover:underline">Ver Pontuação</div>
            </div>
            <div id="question-container" class="mb-6">
                <!-- O conteúdo da pergunta (texto e/ou imagem) será inserido aqui -->
            </div>
            <div id="answers-container" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
            <div id="feedback-container" class="mt-6 text-center text-xl font-bold"></div>
            <div id="next-question-controls" class="hidden mt-6 text-center">
                 <button id="next-question-btn" class="btn btn-primary justify-center">Próxima Pergunta</button>
            </div>
        </div>

        <!-- Tela de Pontuação (Modal) -->
        <div id="scoreboard-modal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
            <div class="card w-full max-w-md">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">Pontuação</h2>
                <ul id="scoreboard-list" class="space-y-2"></ul>
                <button id="close-scoreboard-btn" class="btn btn-secondary mt-6 w-full justify-center">Fechar</button>
            </div>
        </div>
        
        <!-- Tela Final -->
        <div id="end-screen" class="hidden card text-center">
            <h1 class="text-4xl font-bold text-gray-800 mb-4">Fim de Jogo!</h1>
            <h2 class="text-2xl font-semibold text-indigo-600 mb-6">O grande vencedor é... <span id="winner-name" class="text-3xl"></span>!</h2>
            <h3 class="text-xl font-semibold text-gray-700 mt-8 mb-4">Placar Final:</h3>
            <ul id="final-scoreboard-list" class="space-y-3"></ul>
            <button id="play-again-btn" class="btn btn-primary mt-8 w-full justify-center">Jogar Novamente</button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, collection, onSnapshot, writeBatch, serverTimestamp, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // =================================================================
        //  PASSO IMPORTANTE: COLE A SUA CONFIGURAÇÃO DO FIREBASE AQUI
        // =================================================================
        const firebaseConfig = {
            apiKey: "AIzaSyBu2s6DU9d-hc5HTqSuyYLV6Ut0oOwRIls",
            authDomain: "quizmultijogador.firebaseapp.com",
            projectId: "quizmultijogador",
            storageBucket: "quizmultijogador.appspot.com",
            messagingSenderId: "245519129065",
            appId: "1:245519129065:web:95727cf109edad07f29378",
            measurementId: "G-0BRETH5CW3"
        };
        // =================================================================

        const appId = 'default-quiz-app';
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        setLogLevel('debug');

        let currentUser = null;
        let currentGameId = null;
        let unsubscribeGame = null;
        let unsubscribePlayers = null;
        let selectedQuizId = null;
        let currentPlayers = [];
        let currentGameData = null;
        
        const quizzes = {
            "geral": {
                name: "Conhecimentos Gerais",
                questions: [
                    { question: "Qual é a capital do Brasil?", answers: ["São Paulo", "Rio de Janeiro", "Brasília", "Salvador"], correct: 2 },
                    { question: "Quem pintou a Mona Lisa?", answers: ["Van Gogh", "Picasso", "Da Vinci", "Michelangelo"], correct: 2 },
                    { question: "Qual o maior animal do mundo?", answers: ["Elefante", "Tubarão Baleia", "Baleia Azul", "Girafa"], correct: 2 },
                    { question: "Quem escreveu 'Dom Quixote'?", answers: ["Machado de Assis", "Miguel de Cervantes", "Shakespeare", "Fernando Pessoa"], correct: 1 },
                    { question: "Em que ano o homem pisou na Lua?", answers: ["1969", "1972", "1965", "1980"], correct: 0 },
                    { question: "Qual é o rio mais longo do mundo?", answers: ["Rio Nilo", "Rio Amazonas", "Rio Mississipi", "Rio Yangtzé"], correct: 1 },
                    { question: "Qual a montanha mais alta do mundo?", answers: ["Monte Everest", "K2", "Kangchenjunga", "Lhotse"], correct: 0 },
                    { question: "Qual a fórmula química da água?", answers: ["CO2", "O2", "H2O2", "H2O"], correct: 3 },
                    { question: "Qual país tem o formato de uma bota?", answers: ["Grécia", "Portugal", "Itália", "Espanha"], correct: 2},
                    { question: "Quem foi o inventor da lâmpada?", answers: ["Nikola Tesla", "Thomas Edison", "Graham Bell", "Santos Dumont"], correct: 1},
                    { question: "Qual a capital de Portugal?", answers: ["Madrid", "Porto", "Roma", "Lisboa"], correct: 3},
                    { question: "Quem descobriu o Brasil?", answers: ["Cristóvão Colombo", "Vasco da Gama", "Pedro Álvares Cabral", "Américo Vespúcio"], correct: 2},
                    { question: "Qual a moeda oficial do Japão?", answers: ["Yuan", "Won", "Iene", "Rupia"], correct: 2},
                    { question: "Qual o nome do oceano que banha o Brasil?", answers: ["Pacífico", "Índico", "Atlântico", "Ártico"], correct: 2 },
                    { question: "Qual o deserto mais quente do mundo?", answers: ["Saara", "Atacama", "Arábia", "Gobi"], correct: 0},
                    { question: "Qual o plural de 'cidadão'?", answers: ["Cidadãos", "Cidadões", "Cidades", "Cidadãs"], correct: 0 },
                    { question: "Em que continente fica o Egito?", answers: ["Ásia", "Europa", "África", "Oceania"], correct: 2 },
                    { question: "Qual o coletivo de cães?", answers: ["Manada", "Cardume", "Alcateia", "Rebanho"], correct: 2 },
                    { question: "Qual o ponto mais alto do Brasil?", answers: ["Pico da Neblina", "Pico 31 de Março", "Monte Roraima", "Pico da Bandeira"], correct: 0 },
                    { question: "Quantos lados tem um heptágono?", answers: ["5", "6", "7", "8"], correct: 2 }
                ]
            },
             "geral2": {
                name: "Conhecimentos Gerais 2",
                questions: [
                    { question: "Qual o livro mais vendido no mundo, a seguir à Bíblia?", answers: ["Dom Quixote", "O Senhor dos Anéis", "O Pequeno Príncipe", "Harry Potter"], correct: 0 },
                    { question: "Qual a nacionalidade de Pablo Picasso?", answers: ["Italiana", "Francesa", "Espanhola", "Portuguesa"], correct: 2 },
                    { question: "Qual o metal mais abundante na crosta terrestre?", answers: ["Ferro", "Cobre", "Ouro", "Alumínio"], correct: 3 },
                    { question: "Em que país nasceu a pizza?", answers: ["Grécia", "Itália", "Estados Unidos", "França"], correct: 1 },
                    { question: "Qual destes planetas não tem anéis?", answers: ["Saturno", "Júpiter", "Vénus", "Urano"], correct: 2 },
                    { question: "Quantas patas tem uma aranha?", answers: ["6", "8", "10", "12"], correct: 1 },
                    { question: "Qual a cidade conhecida como a 'Big Apple'?", answers: ["Los Angeles", "Chicago", "Miami", "Nova Iorque"], correct: 3 },
                    { question: "Quem foi a primeira mulher a ganhar um Prémio Nobel?", answers: ["Marie Curie", "Rosalind Franklin", "Ada Lovelace", "Dorothy Hodgkin"], correct: 0 },
                    { question: "Qual o nome do maior osso do corpo humano?", answers: ["Tíbia", "Fêmur", "Úmero", "Pelve"], correct: 1 },
                    { question: "Quantos dentes tem um ser humano adulto?", answers: ["28", "30", "32", "26"], correct: 2 },
                    { question: "Qual o nome do processo de conversão de açúcar em álcool?", answers: ["Fotossíntese", "Respiração", "Digestão", "Fermentação"], correct: 3 },
                    { question: "Em que cidade se encontram as ruínas de Machu Picchu?", answers: ["México", "Chile", "Bolívia", "Peru"], correct: 3 },
                    { question: "Qual a capital da Austrália?", answers: ["Sydney", "Melbourne", "Camberra", "Brisbane"], correct: 2 },
                    { question: "Qual o animal que consegue sobreviver mais tempo sem água?", answers: ["Camelo", "Rato-canguru", "Elefante", "Girafa"], correct: 1 },
                    { question: "Qual o nome do criador da personagem de BD 'Tintim'?", answers: ["Goscinny", "Uderzo", "Hergé", "Franquin"], correct: 2 },
                    { question: "Qual o nome do deus do trovão na mitologia nórdica?", answers: ["Odin", "Loki", "Thor", "Freyr"], correct: 2 },
                    { question: "Qual o oceano mais frio do mundo?", answers: ["Pacífico", "Atlântico", "Índico", "Ártico"], correct: 3 },
                    { question: "Qual a cor que resulta da mistura de azul e amarelo?", answers: ["Roxo", "Laranja", "Verde", "Cinzento"], correct: 2 },
                    { question: "Qual o nome da primeira pessoa a viajar para o espaço?", answers: ["Neil Armstrong", "Buzz Aldrin", "Yuri Gagarin", "John Glenn"], correct: 2 },
                    { question: "Qual o animal que tem o maior cérebro em proporção ao seu corpo?", answers: ["Humano", "Golfinho", "Elefante", "Formiga"], correct: 3 }
                ]
            },
            "ciencia": {
                name: "Ciência e Tecnologia",
                questions: [
                    { question: "Qual planeta é conhecido como 'Planeta Vermelho'?", answers: ["Vênus", "Marte", "Júpiter", "Saturno"], correct: 1 },
                    { question: "Qual o gás mais abundante na atmosfera terrestre?", answers: ["Oxigénio", "Dióxido de Carbono", "Hidrogénio", "Nitrogénio"], correct: 3 },
                    { question: "Quem desenvolveu a teoria da relatividade?", answers: ["Isaac Newton", "Galileu Galilei", "Albert Einstein", "Nikola Tesla"], correct: 2 },
                    { question: "Qual o primeiro animal a ser clonado?", answers: ["Cachorro", "Gato", "Ovelha", "Vaca"], correct: 2 },
                    { question: "O que mede um sismógrafo?", answers: ["Ventos", "Marés", "Temperatura", "Terremotos"], correct: 3 },
                    { question: "Qual o nome do processo que as plantas usam para se alimentar?", answers: ["Respiração", "Transpiração", "Fotossíntese", "Digestão"], correct: 2 },
                    { question: "Qual a função dos glóbulos brancos?", answers: ["Transportar oxigénio", "Defender o corpo", "Coagular o sangue", "Produzir hormonas"], correct: 1 },
                    { question: "O que é um 'byte'?", answers: ["Unidade de som", "Unidade de cor", "Unidade de informação digital", "Unidade de velocidade"], correct: 2 },
                    { question: "Qual desses não é um sistema operativo?", answers: ["Windows", "Linux", "Microsoft Office", "Android"], correct: 2 },
                    { question: "Quem inventou o telefone?", answers: ["Thomas Edison", "Nikola Tesla", "Santos Dumont", "Alexander Graham Bell"], correct: 3 },
                    { question: "Qual o osso mais longo do corpo humano?", answers: ["Tíbia", "Fêmur", "Úmero", "Rádio"], correct: 1 },
                    { question: "O que é a 'penicilina'?", answers: ["Um vírus", "Uma bactéria", "Um analgésico", "Um antibiótico"], correct: 3 },
                    { question: "Qual o nome do primeiro satélite artificial lançado ao espaço?", answers: ["Apollo 11", "Explorer 1", "Sputnik 1", "Vostok 1"], correct: 2 },
                    { question: "De que é feito o vidro?", answers: ["Plástico derretido", "Argila aquecida", "Areia de sílica", "Metal líquido"], correct: 2 },
                    { question: "O que significa a sigla 'DNA'?", answers: ["Ácido desoxirribonucleico", "Ácido ribonucleico", "Densidade nuclear atómica", "Digital new age"], correct: 0},
                    { question: "Qual a velocidade do som no ar?", answers: ["Aprox. 343 m/s", "Aprox. 1235 m/s", "Aprox. 3000 m/s", "É instantânea"], correct: 0},
                    { question: "Qual elemento químico tem o símbolo 'Fe'?", answers: ["Flúor", "Hélio", "Ferro", "Fósforo"], correct: 2},
                    { question: "Quem é considerado o 'pai da computação'?", answers: ["Bill Gates", "Alan Turing", "Steve Jobs", "Tim Berners-Lee"], correct: 1},
                    { question: "O que estuda a entomologia?", answers: ["Rochas", "Pássaros", "Insetos", "Peixes"], correct: 2},
                    { question: "Qual o nome da nossa galáxia?", answers: ["Andrómeda", "Via Láctea", "Centaurus A", "Galáxia do Triângulo"], correct: 1}
                ]
            },
            "artes": {
                name: "Artes e Entretenimento",
                questions: [
                    { question: "Qual banda de rock é conhecida como 'The Fab Four'?", answers: ["The Rolling Stones", "The Beatles", "Queen", "Led Zeppelin"], correct: 1 },
                    { question: "Quem escreveu a saga 'Harry Potter'?", answers: ["J. R. R. Tolkien", "J. K. Rowling", "George R. R. Martin", "C. S. Lewis"], correct: 1 },
                    { question: "Qual o nome do vocalista da banda Queen?", answers: ["Freddie Mercury", "Elton John", "David Bowie", "Mick Jagger"], correct: 0 },
                    { question: "Qual destes pintores é famoso por cortar a própria orelha?", answers: ["Pablo Picasso", "Claude Monet", "Salvador Dalí", "Vincent van Gogh"], correct: 3 },
                    { question: "Qual filme ganhou o primeiro Oscar de Melhor Animação?", answers: ["A Bela e a Fera", "O Rei Leão", "Toy Story", "Shrek"], correct: 3 },
                    { question: "Quem é o autor da famosa peça 'Romeu e Julieta'?", answers: ["Machado de Assis", "Charles Dickens", "William Shakespeare", "Dante Alighieri"], correct: 2 },
                    { question: "Qual série de TV se passa em Westeros?", answers: ["The Witcher", "Game of Thrones", "Vikings", "The Last Kingdom"], correct: 1 },
                    { question: "Qual cantor é conhecido como o 'Rei do Pop'?", answers: ["Elvis Presley", "James Brown", "Michael Jackson", "Prince"], correct: 2 },
                    { question: "De qual país o tango é uma dança tradicional?", answers: ["Espanha", "Brasil", "México", "Argentina"], correct: 3 },
                    { question: "Qual o nome do bruxo que é o principal vilão em Harry Potter?", answers: ["Dumbledore", "Snape", "Voldemort", "Sirius Black"], correct: 2 },
                    { question: "Quem dirigiu 'Pulp Fiction'?", answers: ["Steven Spielberg", "Martin Scorsese", "Quentin Tarantino", "James Cameron"], correct: 2},
                    { question: "Qual destes não é um membro dos Vingadores originais do cinema?", answers: ["Capitão América", "Homem-Formiga", "Viúva Negra", "Hulk"], correct: 1},
                    { question: "Quem canta a música 'Like a Virgin'?", answers: ["Whitney Houston", "Cyndi Lauper", "Mariah Carey", "Madonna"], correct: 3},
                    { question: "Qual o nome do boneco de madeira que queria ser um menino de verdade?", answers: ["Gepeto", "Grilo Falante", "Pinóquio", "Figaro"], correct: 2},
                    { question: "Em que cidade se passa a série 'La Casa de Papel'?", answers: ["Barcelona", "Madrid", "Lisboa", "Rio de Janeiro"], correct: 1},
                    { question: "Qual o nome do leão protagonista de 'O Rei Leão'?", answers: ["Mufasa", "Scar", "Simba", "Timão"], correct: 2},
                    { question: "Que artista é conhecido pela obra 'O Grito'?", answers: ["Edvard Munch", "Salvador Dalí", "Frida Kahlo", "Claude Monet"], correct: 0},
                    { question: "Qual o nome do carro de 'Regresso ao Futuro'?", answers: ["Ford Mustang", "Ferrari 250 GTO", "Pontiac Trans Am", "DeLorean"], correct: 3},
                    { question: "Qual o nome da personagem principal da saga 'Jogos da Fome'?", answers: ["Primrose Everdeen", "Katniss Everdeen", "Gale Hawthorne", "Peeta Mellark"], correct: 1},
                    { question: "Quem compôs 'As Quatro Estações'?", answers: ["Bach", "Mozart", "Beethoven", "Vivaldi"], correct: 3}
                ]
            },
            "filmes": {
                name: "Filmes e Séries",
                questions: [
                    { question: "Qual filme apresenta a frase 'Que a Força esteja com você'?", answers: ["Star Trek", "Star Wars", "Blade Runner", "Duna"], correct: 1 },
                    { question: "Na série 'Breaking Bad', qual é o pseudónimo de Walter White?", answers: ["Cap'n Cook", "Jesse", "Gus", "Heisenberg"], correct: 3 },
                    { question: "Qual o nome da inteligência artificial no filme '2001: Uma Odisseia no Espaço'?", answers: ["J.A.R.V.I.S.", "Skynet", "HAL 9000", "GLaDOS"], correct: 2 },
                    { question: "Quem é o realizador da trilogia 'O Cavaleiro das Trevas'?", answers: ["Tim Burton", "Zack Snyder", "J.J. Abrams", "Christopher Nolan"], correct: 3 },
                    { question: "Qual o nome do navio em 'Piratas das Caraíbas' comandado por Jack Sparrow?", answers: ["O Vingança da Rainha Ana", "O Holandês Voador", "O Pérola Negra", "O Interceptor"], correct: 2 },
                    { question: "Em 'Friends', qual o nome do café onde o grupo se encontra?", answers: ["The Coffee Shop", "Central Perk", "Monk's Café", "The Grind"], correct: 1 },
                    { question: "Que ator interpreta o Homem de Ferro no Universo Cinematográfico Marvel?", answers: ["Chris Evans", "Chris Hemsworth", "Mark Ruffalo", "Robert Downey Jr."], correct: 3 },
                    { question: "Qual o nome do protagonista da série 'The Office' (versão EUA)?", answers: ["Dwight Schrute", "Jim Halpert", "Michael Scott", "Andy Bernard"], correct: 2 },
                    { question: "Qual o primeiro filme a cores a ganhar o Oscar de Melhor Filme?", answers: ["O Feiticeiro de Oz", "E Tudo o Vento Levou", "Serenata à Chuva", "Branca de Neve"], correct: 1 },
                    { question: "Qual o nome do mundo fantástico da série 'A Guerra dos Tronos'?", answers: ["Terra Média", "Nárnia", "Westeros", "Alagaësia"], correct: 2 },
                    { question: "Qual o nome do tubarão no filme de Steven Spielberg de 1975?", answers: ["Jaws", "Bruce", "Deep Blue", "Moby Dick"], correct: 1},
                    { question: "Na série 'Stranger Things', qual o nome do mundo invertido?", answers: ["The Underneath", "The Abyss", "The Upside Down", "The Nether"], correct: 2},
                    { question: "Qual destes filmes não foi realizado por Martin Scorsese?", answers: ["Taxi Driver", "O Poderoso Chefão", "Touro Enraivecido", "Os Bons Companheiros"], correct: 1},
                    { question: "Qual a cor do sabre de luz de Luke Skywalker em 'O Regresso de Jedi'?", answers: ["Azul", "Vermelho", "Roxo", "Verde"], correct: 3},
                    { question: "Qual o nome da personagem de Keanu Reeves em 'Matrix'?", answers: ["Morpheus", "Cypher", "Neo", "Agente Smith"], correct: 2},
                    { question: "Qual o nome da casa de Hogwarts a que Harry Potter pertence?", answers: ["Slytherin", "Hufflepuff", "Gryffindor", "Ravenclaw"], correct: 2},
                    { question: "Qual o nome do palhaço assassino no filme 'It'?", answers: ["Coringa", "Pennywise", "Krusty", "Bozo"], correct: 1},
                    { question: "Quem interpreta a personagem 'Eleven' na série 'Stranger Things'?", answers: ["Sadie Sink", "Millie Bobby Brown", "Maya Hawke", "Natalia Dyer"], correct: 1},
                    { question: "Qual o nome do império do mal em 'Star Wars'?", answers: ["A Primeira Ordem", "A Aliança Rebelde", "O Império Galáctico", "A República"], correct: 2},
                    { question: "Em que filme a personagem diz 'Eu vejo gente morta'?", answers: ["O Exorcista", "O Iluminado", "Poltergeist", "O Sexto Sentido"], correct: 3}
                ]
            },
            "desporto": {
                name: "Esportes",
                questions: [
                    { question: "Quantos jogadores tem uma equipa de basquetebol em campo?", answers: ["5", "6", "7", "11"], correct: 0 },
                    { question: "Qual país venceu o primeiro Campeonato do Mundo de Futebol?", answers: ["Brasil", "Argentina", "Itália", "Uruguai"], correct: 3 },
                    { question: "Em que desporto se usa um 'puck'?", answers: ["Basebol", "Críquete", "Pólo Aquático", "Hóquei no Gelo"], correct: 3 },
                    { question: "Quem é o maior medalhista olímpico de todos os tempos?", answers: ["Usain Bolt", "Carl Lewis", "Larisa Latynina", "Michael Phelps"], correct: 3 },
                    { question: "Qual o nome do famoso estádio de futebol em Londres?", answers: ["Old Trafford", "Anfield", "Wembley", "Stamford Bridge"], correct: 2 },
                    { question: "Qual o nome do torneio de ténis mais antigo do mundo?", answers: ["Roland Garros", "US Open", "Australian Open", "Wimbledon"], correct: 3 },
                    { question: "Quantos pontos vale um 'touchdown' no futebol americano?", answers: ["3", "6", "7", "2"], correct: 1 },
                    { question: "Qual o país de origem do piloto de Fórmula 1, Ayrton Senna?", answers: ["Argentina", "Itália", "Alemanha", "Brasil"], correct: 3 },
                    { question: "Em que arte marcial se usa um 'judogi'?", answers: ["Karaté", "Taekwondo", "Judo", "Kung Fu"], correct: 2 },
                    { question: "Qual é a corrida de ciclismo mais famosa do mundo?", answers: ["Giro d'Italia", "Vuelta a España", "Tour de France", "Paris-Roubaix"], correct: 2 },
                    { question: "Qual jogador de futebol é conhecido como 'O Rei'?", answers: ["Maradona", "Messi", "Pelé", "Cristiano Ronaldo"], correct: 2},
                    { question: "Quantas casas tem um tabuleiro de xadrez?", answers: ["32", "64", "100", "48"], correct: 1},
                    { question: "Qual o nome da bola usada no basebol?", answers: ["A bola não tem nome", "Slugger", "Rawlings", "É só 'bola de basebol'"], correct: 3},
                    { question: "Qual a duração de uma maratona oficial?", answers: ["42.195 metros", "42.195 km", "40 km", "21 km"], correct: 1},
                    { question: "Qual equipa da NBA tem mais títulos?", answers: ["Chicago Bulls", "Golden State Warriors", "Los Angeles Lakers", "Boston Celtics"], correct: 2},
                    { question: "Onde foram realizados os primeiros Jogos Olímpicos da era moderna?", answers: ["Paris", "Atenas", "Londres", "Roma"], correct: 1},
                    { question: "Qual o nome do golpe final no voleibol?", answers: ["Manchete", "Serviço", "Bloqueio", "Remate"], correct: 3},
                    { question: "Qual o surfista com mais títulos mundiais?", answers: ["Gabriel Medina", "Kelly Slater", "Mick Fanning", "Andy Irons"], correct: 1},
                    { question: "Qual a pontuação máxima numa partida de bowling?", answers: ["100", "150", "200", "300"], correct: 3},
                    { question: "Em que desporto se compete na 'corrida das 500 milhas de Indianápolis'?", answers: ["NASCAR", "Fórmula 1", "Motociclismo", "IndyCar Series"], correct: 3}
                ]
            },
            "historia": {
                name: "História do Mundo",
                questions: [
                    { question: "Em que ano começou a Primeira Guerra Mundial?", answers: ["1912", "1914", "1916", "1918"], correct: 1 },
                    { question: "Quem foi o primeiro imperador de Roma?", answers: ["Júlio César", "Nero", "Augusto", "Calígula"], correct: 2 },
                    { question: "Qual civilização antiga construiu Machu Picchu?", answers: ["Astecas", "Maias", "Incas", "Egípcios"], correct: 2 },
                    { question: "A Queda do Muro de Berlim ocorreu em que ano?", answers: ["1985", "1987", "1989", "1991"], correct: 2 },
                    { question: "Quem foi o líder da Revolução Cubana?", answers: ["Che Guevara", "Fulgencio Batista", "Fidel Castro", "Camilo Cienfuegos"], correct: 2 },
                    { question: "Qual o nome do navio em que Charles Darwin viajou?", answers: ["HMS Victory", "HMS Endeavour", "HMS Beagle", "HMS Discovery"], correct: 2 },
                    { question: "Qual o nome da famosa rainha do Egito que se aliou a Roma?", answers: ["Nefertiti", "Hatshepsut", "Cleópatra", "Nefertari"], correct: 2 },
                    { question: "Em que país começou a Renascença?", answers: ["França", "Espanha", "Portugal", "Itália"], correct: 3 },
                    { question: "Quem era o presidente dos EUA durante a Guerra Civil Americana?", answers: ["Thomas Jefferson", "Andrew Jackson", "Abraham Lincoln", "Ulysses S. Grant"], correct: 2 },
                    { question: "Qual o nome da doença que causou a 'Peste Negra' na Europa?", answers: ["Varíola", "Tuberculose", "Peste Bubónica", "Cólera"], correct: 2 },
                    { question: "Qual o nome da última rainha da França?", answers: ["Catarina de Médici", "Joana d'Arc", "Maria Antonieta", "Isabel I"], correct: 2},
                    { question: "Em que ano terminou a Segunda Guerra Mundial?", answers: ["1943", "1944", "1945", "1946"], correct: 2},
                    { question: "Qual império foi governado por Gengis Khan?", answers: ["Império Romano", "Império Mongol", "Império Otomano", "Império Persa"], correct: 1},
                    { question: "Qual a cidade que foi soterrada pela erupção do vulcão Vesúvio em 79 d.C.?", answers: ["Roma", "Florença", "Atenas", "Pompeia"], correct: 3},
                    { question: "Quem proferiu a famosa frase 'Eu tenho um sonho'?", answers: ["Nelson Mandela", "Malcolm X", "Martin Luther King Jr.", "Barack Obama"], correct: 2},
                    { question: "A 'Guerra dos Cem Anos' foi travada entre que países?", answers: ["Inglaterra e Espanha", "França e Alemanha", "Inglaterra e França", "Espanha e Portugal"], correct: 2},
                    { question: "Qual o nome da primeira mulher a voar para o espaço?", answers: ["Sally Ride", "Valentina Tereshkova", "Svetlana Savitskaya", "Mae Jemison"], correct: 1},
                    { question: "Qual o nome do código de leis da Babilónia antiga?", answers: ["Código de Hamurabi", "As Doze Tábuas", "As Leis de Manu", "O Livro dos Mortos"], correct: 0},
                    { question: "Qual o nome do movimento liderado por Martinho Lutero no século XVI?", answers: ["A Contrarreforma", "O Iluminismo", "A Reforma Protestante", "O Humanismo"], correct: 2},
                    { question: "Quem foi a figura central do Apartheid na África do Sul?", answers: ["Desmond Tutu", "F. W. de Klerk", "Nelson Mandela", "Thabo Mbeki"], correct: 2}
                ]
            },
            "geografia 1": {
                name: "Geografia 1",
                questions: [
                    { question: "Qual é o maior oceano do mundo?", answers: ["Atlântico", "Índico", "Pacífico", "Ártico"], correct: 2 },
                    { question: "Qual é a capital da Argentina?", answers: ["Santiago", "Montevidéu", "Bogotá", "Buenos Aires"], correct: 3 },
                    { question: "O Rio Nilo desagua em qual mar?", answers: ["Mar Vermelho", "Mar Mediterrâneo", "Mar Negro", "Mar Cáspio"], correct: 1 },
                    { question: "Qual é o país mais populoso do mundo?", answers: ["Índia", "Estados Unidos", "Indonésia", "China"], correct: 0 },
                    { question: "Qual o nome da cordilheira que atravessa a América do Sul?", answers: ["Himalaias", "Andes", "Alpes", "Montanhas Rochosas"], correct: 1 },
                    { question: "Em que continente se situa a Austrália?", answers: ["Ásia", "América", "África", "Oceânia"], correct: 3 },
                    { question: "Qual é a capital do Canadá?", answers: ["Toronto", "Vancouver", "Montreal", "Ottawa"], correct: 3 },
                    { question: "Qual é o maior deserto do mundo (incluindo desertos polares)?", answers: ["Saara", "Deserto da Arábia", "Deserto Antártico", "Gobi"], correct: 2 },
                    { question: "Qual é o rio que passa por Paris?", answers: ["Tâmisa", "Danúbio", "Sena", "Reno"], correct: 2 },
                    { question: "Qual o ponto mais baixo de terra firme do planeta?", answers: ["Vale da Morte", "Depressão de Afar", "Mar Morto", "Fossa das Marianas"], correct: 2 },
                    { question: "Qual o país com a maior área territorial do mundo?", answers: ["Canadá", "China", "Estados Unidos", "Rússia"], correct: 3 },
                    { question: "Qual é o único país que é também um continente?", answers: ["Gronelândia", "Austrália", "Madagáscar", "Islândia"], correct: 1 },
                    { question: "Qual a capital do Japão?", answers: ["Quioto", "Osaka", "Hiroshima", "Tóquio"], correct: 3 },
                    { question: "Qual o nome do estreito que separa a Ásia da América?", answers: ["Estreito de Gibraltar", "Estreito de Bering", "Canal de Suez", "Canal do Panamá"], correct: 1 },
                    { question: "Qual o país conhecido como 'Terra do Sol Nascente'?", answers: ["China", "Coreia do Sul", "Japão", "Vietname"], correct: 2 },
                    { question: "Qual a capital da Rússia?", answers: ["São Petersburgo", "Kiev", "Moscovo", "Varsóvia"], correct: 2 },
                    { question: "Qual a linha imaginária que divide a Terra em hemisfério norte e sul?", answers: ["Trópico de Câncer", "Círculo Polar Ártico", "Linha do Equador", "Meridiano de Greenwich"], correct: 2 },
                    { question: "Qual o maior lago de água doce do mundo em volume?", answers: ["Lago Superior", "Lago Vitória", "Mar Cáspio", "Lago Baikal"], correct: 3 },
                    { question: "Em que país se encontra o Monte Fuji?", answers: ["China", "Coreia do Sul", "Japão", "Nepal"], correct: 2 },
                    { question: "Qual o país mais pequeno do mundo?", answers: ["Mónaco", "Nauru", "São Marino", "Vaticano"], correct: 3 }
                ]
            },
  "geografia 2": {
    "name": "Geografia 2",
    "questions": [
      { "question": "Qual é a capital da Austrália?", "answers": ["Sydney", "Melbourne", "Camberra", "Brisbane"], "correct": 2 },
      { "question": "Que país é conhecido como a 'Terra dos Mil Lagos'?", "answers": ["Suécia", "Noruega", "Canadá", "Finlândia"], "correct": 3 },
      { "question": "O Monte Everest está localizado em qual cordilheira?", "answers": ["Andes", "Himalaia", "Alpes", "Montanhas Rochosas"], "correct": 1 },
      { "question": "Qual é o maior país da América do Sul em área territorial?", "answers": ["Argentina", "Peru", "Colômbia", "Brasil"], "correct": 3 },
      { "question": "Qual oceano banha a costa leste dos Estados Unidos?", "answers": ["Pacífico", "Índico", "Atlântico", "Ártico"], "correct": 2 },
      { "question": "A Grande Muralha é um famoso ponto turístico de qual país?", "answers": ["Japão", "Índia", "China", "Mongólia"], "correct": 2 },
      { "question": "Qual é a capital da Espanha?", "answers": ["Barcelona", "Lisboa", "Roma", "Madrid"], "correct": 3 },
      { "question": "Qual o rio mais longo da Europa?", "answers": ["Danúbio", "Volga", "Reno", "Sena"], "correct": 1 },
      { "question": "Em que país se encontram as pirâmides de Gizé?", "answers": ["Sudão", "Líbia", "Egito", "Marrocos"], "correct": 2 },
      { "question": "Qual é o menor continente do mundo?", "answers": ["Europa", "Antártida", "América do Sul", "Oceania"], "correct": 3 },
      { "question": "Qual o nome do fenômeno que causa as estações do ano?", "answers": ["Rotação", "Translação", "Precessão", "Nutação"], "correct": 1 },
      { "question": "Qual país tem o maior número de ilhas no mundo?", "answers": ["Indonésia", "Filipinas", "Suécia", "Japão"], "correct": 2 },
      { "question": "A cidade de Istambul está localizada em dois continentes. Quais são eles?", "answers": ["África e Ásia", "Europa e África", "Europa e Ásia", "Ásia e Oceania"], "correct": 2 },
      { "question": "Qual o deserto mais seco do mundo?", "answers": ["Saara", "Gobi", "Atacama", "Kalahari"], "correct": 2 },
      { "question": "Qual a capital da Itália?", "answers": ["Milão", "Nápoles", "Veneza", "Roma"], "correct": 3 },
      { "question": "Que mar separa a Europa da África?", "answers": ["Mar Negro", "Mar Mediterrâneo", "Mar Vermelho", "Mar Adriático"], "correct": 1 },
      { "question": "Qual o ponto mais alto do Brasil?", "answers": ["Pico da Bandeira", "Monte Roraima", "Pico da Neblina", "Pico 31 de Março"], "correct": 2 },
      { "question": "Qual o nome dado à camada de ar que envolve a Terra?", "answers": ["Litosfera", "Hidrosfera", "Atmosfera", "Biosfera"], "correct": 2 },
      { "question": "A Península Ibérica é formada por quais países?", "answers": ["França e Espanha", "Portugal e Marrocos", "Espanha e Portugal", "Itália e Grécia"], "correct": 2 },
      { "question": "Qual o maior arquipélago do mundo?", "answers": ["Filipinas", "Japão", "Maldivas", "Indonésia"], "correct": 3 }
    ]
  },
  "geografia 3": {
    "name": "Geografia 3",
    "questions": [
      { "question": "Qual a capital da Alemanha?", "answers": ["Munique", "Hamburgo", "Frankfurt", "Berlim"], "correct": 3 },
      { "question": "O Canal de Suez conecta quais dois mares?", "answers": ["Mar Negro e Mar Cáspio", "Mar Mediterrâneo e Mar Vermelho", "Mar Báltico e Mar do Norte", "Mar Adriático e Mar Jônico"], "correct": 1 },
      { "question": "Qual é o maior estado dos Estados Unidos em área?", "answers": ["Texas", "Alasca", "Califórnia", "Montana"], "correct": 1 },
      { "question": "Em que continente fica a Groenlândia?", "answers": ["Europa", "Ásia", "América do Norte", "Oceania"], "correct": 2 },
      { "question": "Qual país é o maior produtor de café do mundo?", "answers": ["Colômbia", "Vietnã", "Brasil", "Etiópia"], "correct": 2 },
      { "question": "Qual é o nome do maior vulcão ativo da Europa?", "answers": ["Vesúvio", "Etna", "Stromboli", "Hekla"], "correct": 1 },
      { "question": "Qual cidade é famosa por seus canais e gôndolas?", "answers": ["Amsterdã", "Bruges", "Estocolmo", "Veneza"], "correct": 3 },
      { "question": "Qual é a capital do Egito?", "answers": ["Alexandria", "Luxor", "Cairo", "Gizé"], "correct": 2 },
      { "question": "O rio Amazonas deságua em qual oceano?", "answers": ["Pacífico", "Atlântico", "Índico", "Ártico"], "correct": 1 },
      { "question": "Qual a montanha mais alta do Japão?", "answers": ["Monte Kita", "Monte Fuji", "Monte Okuhotaka", "Monte Aino"], "correct": 1 },
      { "question": "Qual o nome da linha imaginária oposta ao Meridiano de Greenwich?", "answers": ["Linha do Equador", "Trópico de Capricórnio", "Linha Internacional de Data", "Círculo Polar Antártico"], "correct": 2 },
      { "question": "Que país se estende ao longo da costa oeste da América do Sul?", "answers": ["Peru", "Equador", "Colômbia", "Chile"], "correct": 3 },
      { "question": "Qual a capital da Coreia do Sul?", "answers": ["Busan", "Incheon", "Seul", "Daegu"], "correct": 2 },
      { "question": "Qual é a maior ilha do Mar Mediterrâneo?", "answers": ["Sardenha", "Córsega", "Chipre", "Sicília"], "correct": 3 },
      { "question": "A Floresta Amazônica está localizada principalmente em qual país?", "answers": ["Peru", "Colômbia", "Equador", "Brasil"], "correct": 3 },
      { "question": "Qual o país mais visitado do mundo?", "answers": ["Espanha", "Estados Unidos", "China", "França"], "correct": 3 },
      { "question": "Em que país está localizado o Taj Mahal?", "answers": ["Paquistão", "Índia", "Bangladesh", "Nepal"], "correct": 1 },
      { "question": "Qual é o maior lago da África?", "answers": ["Lago Tanganica", "Lago Malawi", "Lago Vitória", "Lago Turkana"], "correct": 2 },
      { "question": "Qual a capital do México?", "answers": ["Guadalajara", "Monterrey", "Cidade do México", "Puebla"], "correct": 2 },
      { "question": "Qual o nome do movimento das placas tectônicas?", "answers": ["Erosão", "Intemperismo", "Tectonismo", "Vulkanismo"], "correct": 2 }
    ]
  },
  "geografia 4": {
    "name": "Geografia 4",
    "questions": [
      { "question": "Qual a capital da Turquia?", "answers": ["Istambul", "Ancara", "Izmir", "Bursa"], "correct": 1 },
      { "question": "O Estreito de Gibraltar separa a Espanha de qual país africano?", "answers": ["Argélia", "Líbia", "Egito", "Marrocos"], "correct": 3 },
      { "question": "Qual o rio mais longo dos Estados Unidos?", "answers": ["Mississippi", "Missouri", "Yukon", "Rio Grande"], "correct": 1 },
      { "question": "Qual o país com a maior costa litorânea do mundo?", "answers": ["Rússia", "Austrália", "Indonésia", "Canadá"], "correct": 3 },
      { "question": "Qual a capital da Tailândia?", "answers": ["Chiang Mai", "Phuket", "Bangkok", "Pattaya"], "correct": 2 },
      { "question": "Qual é a cidade mais populosa do mundo?", "answers": ["Xangai", "Delhi", "São Paulo", "Tóquio"], "correct": 3 },
      { "question": "A Patagônia é uma região compartilhada por quais dois países?", "answers": ["Brasil e Argentina", "Chile e Peru", "Argentina e Chile", "Bolívia e Paraguai"], "correct": 2 },
      { "question": "Qual é o oceano mais frio do mundo?", "answers": ["Atlântico", "Pacífico", "Índico", "Ártico"], "correct": 3 },
      { "question": "Em que país fica a cidade de Machu Picchu?", "answers": ["Bolívia", "Equador", "Peru", "Colômbia"], "correct": 2 },
      { "question": "Qual é o maior país da África em área territorial?", "answers": ["Sudão", "República Democrática do Congo", "Argélia", "Líbia"], "correct": 2 },
      { "question": "Qual o nome da península onde se localizam a Noruega e a Suécia?", "answers": ["Península Itálica", "Península Balcânica", "Península Escandinava", "Península da Jutlândia"], "correct": 2 },
      { "question": "Qual a capital da Nova Zelândia?", "answers": ["Auckland", "Christchurch", "Wellington", "Queenstown"], "correct": 2 },
      { "question": "As Cataratas do Iguaçu fazem fronteira entre o Brasil e qual outro país?", "answers": ["Paraguai", "Uruguai", "Bolívia", "Argentina"], "correct": 3 },
      { "question": "Qual o mar interior localizado entre a Europa Oriental e a Ásia Ocidental?", "answers": ["Mar Negro", "Mar Cáspio", "Mar de Aral", "Mar Báltico"], "correct": 1 },
      { "question": "Qual é a língua mais falada no mundo (como primeira língua)?", "answers": ["Inglês", "Mandarim", "Espanhol", "Hindi"], "correct": 1 },
      { "question": "Qual o nome do processo de transformação de rochas pelo calor e pressão?", "answers": ["Sedimentação", "Erosão", "Metamorfismo", "Magmatismo"], "correct": 2 },
      { "question": "Qual o país com mais fusos horários?", "answers": ["Rússia", "Estados Unidos", "Canadá", "França"], "correct": 3 },
      { "question": "A cidade de Petra, um famoso sítio arqueológico, fica em qual país?", "answers": ["Síria", "Jordânia", "Líbano", "Israel"], "correct": 1 },
      { "question": "Qual a capital da Irlanda?", "answers": ["Belfast", "Cork", "Galway", "Dublin"], "correct": 3 },
      { "question": "Qual é a ilha mais populosa do mundo?", "answers": ["Grã-Bretanha", "Honshu", "Java", "Luzon"], "correct": 2 }
    ]
  },
            "corpo_humano": {
                name: "Corpo Humano",
                questions: [
                    { question: "Qual é o maior órgão do corpo humano?", answers: ["Fígado", "Pele", "Cérebro", "Intestino"], correct: 1 },
                    { question: "Quantos ossos tem um ser humano adulto?", answers: ["206", "212", "198", "250"], correct: 0 },
                    { question: "Qual o músculo mais forte do corpo humano em proporção ao seu tamanho?", answers: ["Masseter (mandíbula)", "Glúteo", "Quadríceps", "Coração"], correct: 0 },
                    { question: "Onde se localizam as papilas gustativas?", answers: ["Nariz", "Pele", "Estômago", "Língua"], correct: 3 },
                    { question: "Qual o nome dos vasos sanguíneos que levam o sangue do coração para o resto do corpo?", answers: ["Veias", "Artérias", "Capilares", "Vênulas"], correct: 1 },
                    { question: "Qual órgão produz a insulina?", answers: ["Fígado", "Estômago", "Pâncreas", "Rim"], correct: 2 },
                    { question: "Qual é o nome da substância que dá cor à nossa pele, cabelo e olhos?", answers: ["Queratina", "Colagénio", "Melanina", "Hemoglobina"], correct: 2 },
                    { question: "Quantos dentes tem um adulto, incluindo os dentes do siso?", answers: ["28", "30", "32", "34"], correct: 2 },
                    { question: "Qual o nome do osso do crânio que protege o cérebro?", answers: ["Mandíbula", "Maxilar", "Caixa craniana", "Clavícula"], correct: 2 },
                    { question: "Qual a principal função dos rins?", answers: ["Digestionar alimentos", "Bombear sangue", "Filtrar o sangue", "Produzir hormonas"], correct: 2 },
                    { question: "Qual o nome do osso popularmente conhecido como 'maçã de Adão'?", answers: ["Hióide", "Cartilagem tiroide", "Epiglote", "Traqueia"], correct: 1 },
                    { question: "Qual a parte do olho responsável por focar a luz?", answers: ["Íris", "Retina", "Pupila", "Cristalino"], correct: 3 },
                    { question: "Qual o nome do processo de respiração a nível celular?", answers: ["Fotossíntese", "Respiração celular", "Ventilação", "Hematose"], correct: 1 },
                    { question: "Quantos litros de sangue tem, em média, um adulto?", answers: ["2-3 litros", "5-6 litros", "8-10 litros", "1-2 litros"], correct: 1 },
                    { question: "Qual o nome do osso do antebraço do lado do polegar?", answers: ["Ulna", "Fêmur", "Tíbia", "Rádio"], correct: 3 },
                    { question: "Qual a parte do cérebro responsável pelo equilíbrio e coordenação?", answers: ["Córtex", "Cerebelo", "Tronco cerebral", "Hipocampo"], correct: 1 },
                    { question: "O que são os alvéolos?", answers: ["Células nervosas", "Ossos do ouvido", "Sacos de ar nos pulmões", "Células musculares"], correct: 2 },
                    { question: "Qual o nome do ácido presente no estômago?", answers: ["Ácido sulfúrico", "Ácido acético", "Ácido cítrico", "Ácido clorídrico"], correct: 3 },
                    { question: "Qual a vitamina produzida pelo corpo quando exposto ao sol?", answers: ["Vitamina C", "Vitamina A", "Vitamina D", "Vitamina B12"], correct: 2 },
                    { question: "Qual o nome do maior osso do corpo humano?", answers: ["Fêmur", "Tíbia", "Úmero", "Pelve"], correct: 0 }
                ]
            },
            "reino_animal": {
                name: "Reino Animal",
                questions: [
                    { question: "Qual é o mamífero mais rápido do mundo?", answers: ["Leão", "Guepardo", "Cavalo", "Antílope"], correct: 1 },
                    { question: "Qual o único mamífero que pode voar?", answers: ["Esquilo-voador", "Galinha", "Morcego", "Pinguim"], correct: 2 },
                    { question: "Qual é o maior réptil do mundo?", answers: ["Jacaré-açu", "Crocodilo-de-água-salgada", "Dragão-de-komodo", "Anaconda"], correct: 1 },
                    { question: "Qual animal tem a maior longevidade?", answers: ["Tartaruga-das-galápagos", "Baleia-da-gronelândia", "Tubarão-da-gronelândia", "Elefante"], correct: 2 },
                    { question: "Qual pássaro é conhecido pela sua capacidade de imitar a fala humana?", answers: ["Corvo", "Papagaio", "Arara", "Gralha"], correct: 1 },
                    { question: "Como se chama o macho da abelha?", answers: ["Vespa", "Zangão", "Operário", "Soldado"], correct: 1 },
                    { question: "Qual destes animais é um marsupial?", answers: ["Coelho", "Urso", "Canguru", "Rato"], correct: 2 },
                    { question: "Qual é o maior peixe do mundo?", answers: ["Tubarão-baleia", "Tubarão-branco", "Baleia-azul", "Atum"], correct: 0 },
                    { question: "Qual animal dorme de pé?", answers: ["Cão", "Gato", "Cavalo", "Urso"], correct: 2 },
                    { question: "De que cor é a pele de um urso polar?", answers: ["Branca", "Amarela", "Cinzenta", "Preta"], correct: 3 },
                    { question: "Quantos corações tem um polvo?", answers: ["1", "2", "3", "4"], correct: 2 },
                    { question: "Qual o animal terrestre mais alto do mundo?", answers: ["Elefante", "Camelo", "Girafa", "Alce"], correct: 2 },
                    { question: "O que é um animal herbívoro?", answers: ["Come carne", "Come plantas", "Come de tudo", "Não come"], correct: 1 },
                    { question: "Qual o nome do processo de mudança de pele em cobras?", answers: ["Muda", "Hibernação", "Ecdise", "Metamorfose"], correct: 2 },
                    { question: "Qual o animal que representa o signo de Leão?", answers: ["Tigre", "Lobo", "Urso", "Leão"], correct: 3 },
                    { question: "Qual é o som que as rãs fazem?", answers: ["Crocitar", "Chiar", "Coaxar", "Grasnar"], correct: 2 },
                    { question: "Qual destes animais é venenoso?", answers: ["Sapo", "Rã", "Cobra-coral", "Lagartixa"], correct: 2 },
                    { question: "Qual o nome do filhote de um cavalo?", answers: ["Bezerro", "Poldro", "Cordeiro", "Pinto"], correct: 1 },
                    { question: "Qual o animal que constrói barragens em rios?", answers: ["Lontra", "Capivara", "Castor", "Rato-do-banhado"], correct: 2 },
                    { question: "Qual o animal que tem o pescoço mais comprido?", answers: ["Avestruz", "Garça", "Flamingo", "Girafa"], correct: 3 }
                ]
            },
            "comida_bebida": {
                name: "Comida e Bebida",
                questions: [
                    { question: "De que país é originária a pizza?", answers: ["França", "Estados Unidos", "Grécia", "Itália"], correct: 3 },
                    { question: "Qual é o principal ingrediente do guacamole?", answers: ["Tomate", "Abacate", "Cebola", "Coentros"], correct: 1 },
                    { question: "Qual é a bebida nacional do Brasil?", answers: ["Cerveja", "Cachaça", "Caipirinha", "Guaraná"], correct: 2 },
                    { question: "O sushi é um prato típico de que país?", answers: ["China", "Japão", "Coreia do Sul", "Tailândia"], correct: 1 },
                    { question: "Qual o principal ingrediente do chocolate?", answers: ["Leite", "Açúcar", "Cacau", "Manteiga"], correct: 2 },
                    { question: "Qual destes queijos não é italiano?", answers: ["Mussarela", "Parmesão", "Gorgonzola", "Roquefort"], correct: 3 },
                    { question: "A paella é um prato tradicional de que país?", answers: ["Portugal", "Espanha", "México", "Itália"], correct: 1 },
                    { question: "Qual é a fruta mais consumida no mundo?", answers: ["Banana", "Maçã", "Tomate", "Laranja"], correct: 2 },
                    { question: "De que é feito o vinho?", answers: ["Maçãs", "Uvas", "Peras", "Morangos"], correct: 1 },
                    { question: "Qual o nome do pão achatado e redondo usado em kebabs?", answers: ["Baguete", "Pão pita", "Ciabatta", "Focaccia"], correct: 1 },
                    { question: "Qual a base da sopa japonesa 'ramen'?", answers: ["Arroz", "Massa", "Tofu", "Algas"], correct: 1 },
                    { question: "Qual o prato mais famoso de Portugal?", answers: ["Francesinha", "Cozido à Portuguesa", "Bacalhau", "Sardinha assada"], correct: 2 },
                    { question: "Qual o ingrediente principal da feijoada brasileira?", answers: ["Feijão preto", "Feijão branco", "Grão-de-bico", "Lentilhas"], correct: 0 },
                    { question: "De que país são originários os 'croissants'?", answers: ["França", "Suíça", "Áustria", "Bélgica"], correct: 2 },
                    { question: "Qual a bebida mais popular do mundo, a seguir à água?", answers: ["Café", "Chá", "Sumo de laranja", "Coca-Cola"], correct: 1 },
                    { question: "Qual o nome do prato mexicano que consiste numa tortilha enrolada?", answers: ["Taco", "Quesadilla", "Burrito", "Nacho"], correct: 2 },
                    { question: "Qual o principal ingrediente para fazer pão?", answers: ["Farinha", "Ovos", "Leite", "Manteiga"], correct: 0 },
                    { question: "Qual o tipo de massa em forma de laço?", answers: ["Penne", "Spaghetti", "Farfalle", "Fusilli"], correct: 2 },
                    { question: "Qual o principal ingrediente do húmus?", answers: ["Feijão", "Lentilhas", "Grão-de-bico", "Ervilhas"], correct: 2 },
                    { question: "Qual a bebida destilada feita a partir do agave?", answers: ["Rum", "Vodka", "Gin", "Tequila"], correct: 3 }
                ]
            },
            "marcas": {
                name: "Marcas Famosas",
                questions: [
                    { question: "Qual marca tem o slogan 'Just Do It'?", answers: ["Adidas", "Puma", "Reebok", "Nike"], correct: 3 },
                    { question: "Qual o nome da empresa de tecnologia fundada por Steve Jobs?", answers: ["Microsoft", "Apple", "Google", "Amazon"], correct: 1 },
                    { question: "Qual marca de automóveis tem um cavalo empinado como logótipo?", answers: ["Lamborghini", "Porsche", "Ferrari", "Maserati"], correct: 2 },
                    { question: "Qual o nome da empresa que criou a personagem Mickey Mouse?", answers: ["Warner Bros.", "Universal", "Disney", "Pixar"], correct: 2 },
                    { question: "Qual a marca de refrigerante mais famosa do mundo?", answers: ["Pepsi", "Coca-Cola", "Sprite", "Fanta"], correct: 1 },
                    { question: "Qual a empresa de mobiliário sueca conhecida pelas suas lojas enormes?", answers: ["H&M", "Zara Home", "IKEA", "Fnac"], correct: 2 },
                    { question: "Qual marca de telemóveis produz o 'Galaxy'?", answers: ["Apple", "Huawei", "Xiaomi", "Samsung"], correct: 3 },
                    { question: "Qual o nome da empresa de streaming com um 'N' vermelho como logótipo?", answers: ["YouTube", "Hulu", "Netflix", "Amazon Prime"], correct: 2 },
                    { question: "Qual a marca de brinquedos famosa pelos seus blocos de montar?", answers: ["Playmobil", "LEGO", "Mattel", "Hasbro"], correct: 1 },
                    { question: "Qual a rede social que usa um pássaro azul como logótipo?", answers: ["Facebook", "Instagram", "LinkedIn", "Twitter (X)"], correct: 3 },
                    { question: "Qual a marca de fast food cujo slogan é 'I'm Lovin' It'?", answers: ["Burger King", "Wendy's", "KFC", "McDonald's"], correct: 3 },
                    { question: "Qual a marca de carros alemã cujo logótipo tem quatro argolas?", answers: ["BMW", "Mercedes-Benz", "Audi", "Volkswagen"], correct: 2 },
                    { question: "Qual empresa é dona do motor de busca mais popular do mundo?", answers: ["Microsoft", "Yahoo", "Google", "DuckDuckGo"], correct: 2 },
                    { question: "Qual marca de café tem uma sereia verde como logótipo?", answers: ["Costa Coffee", "Dunkin' Donuts", "Nespresso", "Starbucks"], correct: 3 },
                    { question: "Qual o nome da empresa de comércio eletrónico fundada por Jeff Bezos?", answers: ["eBay", "Alibaba", "Amazon", "Shopify"], correct: 2 },
                    { question: "Qual marca desportiva tem um logótipo com três listras?", answers: ["Puma", "Adidas", "Nike", "Under Armour"], correct: 1 },
                    { question: "Qual a marca de chocolate conhecida pelo seu formato triangular?", answers: ["Milka", "Lindt", "Cadbury", "Toblerone"], correct: 3 },
                    { question: "Qual empresa de videojogos criou o 'Super Mario'?", answers: ["Sony", "Microsoft", "Sega", "Nintendo"], correct: 3 },
                    { question: "Qual o nome da empresa de carros elétricos liderada por Elon Musk?", answers: ["Rivian", "Lucid", "Tesla", "Nio"], correct: 2 },
                    { question: "Qual a marca de batatas fritas cujo logótipo é um bigode com um laço?", answers: ["Lay's", "Ruffles", "Pringles", "Doritos"], correct: 2 }
                ]
            },
            "2025_acontecimentos": {
                name: "Acontecimentos de 2025",
                questions: [
                    { question: "Qual cidade europeia está programada para ser a Capital Europeia da Cultura em 2025?", answers: ["Lisboa, Portugal", "Chemnitz, Alemanha", "Viena, Áustria", "Atenas, Grécia"], correct: 1 },
                    { question: "Qual missão da NASA está programada para levar astronautas a orbitar a Lua em 2025?", answers: ["Artemis I", "Artemis II", "Artemis III", "Apollo 18"], correct: 1 },
                    { question: "Que grande evento desportivo mundial terá lugar no Japão em 2025?", answers: ["Jogos Olímpicos de Verão", "Campeonato do Mundo de Futebol", "Campeonato Mundial de Atletismo", "Campeonato do Mundo de Rugby"], correct: 2 },
                    { question: "A Expo 2025, uma feira mundial, está programada para acontecer em que cidade japonesa?", answers: ["Tóquio", "Quioto", "Osaka", "Sapporo"], correct: 2 },
                    { question: "Em 2025, qual grande jubileu a Igreja Católica celebrará em Roma?", answers: ["Jubileu da Esperança", "Jubileu da Fé", "Jubileu da Misericórdia", "Jubileu da Caridade"], correct: 0 },
                    { question: "Qual país está programado para sediar o Campeonato Europeu de Futebol Feminino da UEFA em 2025?", answers: ["Alemanha", "França", "Suíça", "Inglaterra"], correct: 2 },
                    { question: "Qual tecnologia emergente espera-se que veja uma adoção ainda mais ampla em smartphones e dispositivos em 2025?", answers: ["5G", "Wi-Fi 7", "Inteligência Artificial Generativa", "Realidade Virtual"], correct: 2 },
                    { question: "Qual filme de super-heróis do DC Universe está programado para ser lançado em 2025, reiniciando o universo cinematográfico?", answers: ["The Batman Part II", "Joker: Folie à Deux", "Superman", "Wonder Woman 3"], correct: 2 },
                    { question: "Que ano marcante a Organização das Nações Unidas (ONU) celebra em 2025?", answers: ["50º aniversário", "75º aniversário", "80º aniversário", "100º aniversário"], correct: 2 },
                    { question: "O Ano Santo Compostelano, um grande evento de peregrinação, será celebrado em que ano, incluindo 2025?", answers: ["Apenas em 2025", "2021 e 2022", "2027", "Não se aplica"], correct: 1 },
                    { question: "Qual será a principal tendência na indústria automobilística em 2025?", answers: ["Carros a diesel", "Expansão de carros elétricos", "Carros a hidrogénio", "Carros voadores"], correct: 1 },
                    { question: "A sonda espacial 'Europa Clipper' da NASA está programada para ser lançada em direção a que lua de Júpiter?", answers: ["Ganimedes", "Calisto", "Io", "Europa"], correct: 3 },
                    { question: "Qual é o nome da próxima grande atualização do sistema operativo Windows esperada para 2025?", answers: ["Windows 12", "Windows 11 Update", "Windows Neptune", "Ainda não anunciado"], correct: 0 },
                    { question: "Que país sediará o Campeonato Mundial de Natação em 2025?", answers: ["Austrália", "Rússia", "Singapura", "Estados Unidos"], correct: 2 },
                    { question: "Qual o nome do telescópio espacial que sucederá o Hubble e o James Webb, com lançamento previsto para a década de 2020?", answers: ["Nancy Grace Roman", "Kepler", "Spitzer", "TESS"], correct: 0 },
                    { question: "A cidade do Rio de Janeiro foi nomeada Capital Mundial de quê para 2025 pela UNESCO?", answers: ["Capital Mundial do Livro", "Capital Mundial da Arquitetura", "Capital Mundial da Música", "Capital Mundial do Desporto"], correct: 1 },
                    { question: "Qual será o foco principal das discussões sobre mudanças climáticas na COP30 em Belém, Brasil?", answers: ["Redução de plástico", "Financiamento climático", "Energia nuclear", "Agricultura sustentável"], correct: 1 },
                    { question: "Que banda famosa anunciou uma turné de despedida que passará pela Europa em 2025?", answers: ["The Rolling Stones", "Aerosmith", "U2", "Coldplay"], correct: 1 },
                    { question: "O que se espera que a inteligência artificial seja capaz de fazer de forma mais autónoma em 2025?", answers: ["Conduzir carros em todas as condições", "Diagnosticar doenças com precisão", "Escrever romances completos", "Gerar código de programação complexo"], correct: 3 },
                    { question: "Qual a data prevista para a conclusão da Basílica da Sagrada Família em Barcelona?", answers: ["2026", "2030", "2035", "Ainda sem data"], correct: 0 }
                ]
            },
            "bandeiras_facil": {
                name: "Bandeiras (Fácil)",
                questions: [
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/br.png', answers: ["Portugal", "Argentina", "Brasil", "Cabo Verde"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/us.png', answers: ["Reino Unido", "Estados Unidos", "Libéria", "Malásia"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/pt.png', answers: ["Espanha", "Brasil", "Portugal", "Angola"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/jp.png', answers: ["China", "Japão", "Coreia do Sul", "Vietname"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ca.png', answers: ["Estados Unidos", "Austrália", "Canadá", "Peru"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/it.png', answers: ["México", "Irlanda", "Hungria", "Itália"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/de.png', answers: ["Bélgica", "Áustria", "Alemanha", "Luxemburgo"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/fr.png', answers: ["Países Baixos", "Rússia", "França", "Luxemburgo"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/gb.png', answers: ["Austrália", "Reino Unido", "Nova Zelândia", "Islândia"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/es.png', answers: ["Portugal", "Espanha", "Andorra", "Mónaco"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ar.png', answers: ["Uruguai", "Paraguai", "Argentina", "Honduras"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/cn.png', answers: ["Vietname", "China", "Turquia", "Marrocos"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/kr.png', answers: ["Japão", "Coreia do Norte", "Coreia do Sul", "Taiwan"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/au.png', answers: ["Nova Zelândia", "Reino Unido", "Fiji", "Austrália"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/mx.png', answers: ["Itália", "México", "Honduras", "Espanha"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ru.png', answers: ["Eslovénia", "Sérvia", "Eslováquia", "Rússia"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/in.png', answers: ["Paquistão", "Índia", "Bangladesh", "Irlanda"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/za.png', answers: ["Sudão do Sul", "Etiópia", "África do Sul", "Zimbábue"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ch.png', answers: ["Dinamarca", "Suíça", "Suécia", "Áustria"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/gr.png', answers: ["Chipre", "Finlândia", "Israel", "Grécia"], correct: 3 }
                ]
            },
            "bandeiras_medio": {
                name: "Bandeiras (Médio)",
                questions: [
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/se.png', answers: ["Dinamarca", "Noruega", "Suécia", "Finlândia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/no.png', answers: ["Islândia", "Noruega", "Dinamarca", "Finlândia"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/nl.png', answers: ["Luxemburgo", "França", "Países Baixos", "Rússia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ie.png', answers: ["Itália", "Costa do Marfim", "Irlanda", "Índia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/cl.png', answers: ["Texas (EUA)", "Porto Rico", "Chile", "Cuba"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/co.png', answers: ["Equador", "Venezuela", "Colômbia", "Roménia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/eg.png', answers: ["Síria", "Iémen", "Egito", "Iraque"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/tr.png', answers: ["Tunísia", "Turquia", "Azerbaijão", "Paquistão"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/il.png', answers: ["Grécia", "Israel", "Argentina", "Somália"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/jm.png', answers: ["Gana", "África do Sul", "Jamaica", "Brasil"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/cu.png', answers: ["República Dominicana", "Porto Rico", "Filipinas", "Cuba"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ua.png', answers: ["Suécia", "Colômbia", "Ucrânia", "Palau"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/pl.png', answers: ["Indonésia", "Mónaco", "Polónia", "Áustria"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/vn.png', answers: ["China", "Vietname", "Marrocos", "Somália"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/th.png', answers: ["Costa Rica", "Malásia", "Tailândia", "Camboja"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/at.png', answers: ["Polónia", "Áustria", "Letónia", "Peru"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/pk.png', answers: ["Arábia Saudita", "Irão", "Paquistão", "Argélia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/my.png', answers: ["Estados Unidos", "Libéria", "Malásia", "Uruguai"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ke.png', answers: ["Quénia", "Uganda", "Sudão do Sul", "Etiópia"], correct: 0 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/hu.png', answers: ["Bulgária", "Itália", "Irão", "Hungria"], correct: 3 }
                ]
            },
            "bandeiras_dificil": {
                name: "Bandeiras (Difícil)",
                questions: [
                    { question: "A que país pertence esta bandeira (tem uma faixa azul ligeiramente mais escura)?", imageUrl: 'https://flagcdn.com/w160/ro.png', answers: ["Roménia", "Chade", "Andorra", "Moldávia"], correct: 0 },
                    { question: "A que país pertence esta bandeira (é mais comprida que a do Mónaco)?", imageUrl: 'https://flagcdn.com/w160/id.png', answers: ["Indonésia", "Mónaco", "Polónia", "Singapura"], correct: 0 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/si.png', answers: ["Rússia", "Eslováquia", "Sérvia", "Eslovénia"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ci.png', answers: ["Irlanda", "Costa do Marfim", "Níger", "Itália"], correct: 1 },
                    { question: "A que país pertence esta bandeira (a única não retangular do mundo)?", imageUrl: 'https://flagcdn.com/w160/np.png', answers: ["Índia", "Butão", "Nepal", "Tibete"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/bt.png', answers: ["País de Gales", "Butão", "Malta", "Albânia"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/sc.png', answers: ["Comores", "Maurícia", "Seychelles", "Maldivas"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/bb.png', answers: ["Trindade e Tobago", "Barbados", "Santa Lúcia", "Bahamas"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/cy.png', answers: ["Creta", "Malta", "Chipre", "Kosovo"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/mz.png', answers: ["Angola", "Moçambique", "Zimbábue", "Timor-Leste"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/bz.png', answers: ["Guatemala", "Honduras", "Belize", "El Salvador"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/lk.png', answers: ["Índia", "Sri Lanka", "Bangladesh", "Nepal"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/kz.png', answers: ["Uzbequistão", "Turquemenistão", "Quirguistão", "Cazaquistão"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/lu.png', answers: ["Países Baixos", "Paraguai", "Luxemburgo", "Croácia"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ee.png', answers: ["Finlândia", "Letónia", "Lituânia", "Estónia"], correct: 3 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/va.png', answers: ["San Marino", "Vaticano", "Malta", "Andorra"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/pg.png', answers: ["Ilhas Salomão", "Vanuatu", "Papua-Nova Guiné", "Fiji"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/kg.png', answers: ["Cazaquistão", "Quirguistão", "Tajiquistão", "Mongólia"], correct: 1 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/lv.png', answers: ["Áustria", "Polónia", "Letónia", "Líbano"], correct: 2 },
                    { question: "A que país pertence esta bandeira?", imageUrl: 'https://flagcdn.com/w160/ml.png', answers: ["Senegal", "Guiné", "Mali", "Camarões"], correct: 2 }
                ]
            },
            "logos": {
                name: "Adivinhe a Marca",
                questions: [
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/a/a6/Logo_NIKE.svg', answers: ["Adidas", "Puma", "Nike", "Reebok"], correct: 2 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/3/36/McDonald%27s_Golden_Arches.svg/200px-McDonald%27s_Golden_Arches.svg.png', answers: ["Burger King", "Wendy's", "McDonald's", "Subway"], correct: 2 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/f/fa/Apple_logo_black.svg', answers: ["Microsoft", "Apple", "Samsung", "Google"], correct: 1 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/b8/2021_Facebook_icon.svg/200px-2021_Facebook_icon.svg.png', answers: ["Twitter (X)", "Instagram", "LinkedIn", "Facebook"], correct: 3 },
                    { question: "A que marca de automóveis pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/6/6d/Volkswagen_logo_2019.svg/200px-Volkswagen_logo_2019.svg.png', answers: ["Volvo", "Volkswagen", "BMW", "Audi"], correct: 1 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/0/09/YouTube_full-color_icon_%282017%29.svg/200px-YouTube_full-color_icon_%282017%29.svg.png', answers: ["Netflix", "Vimeo", "YouTube", "Hulu"], correct: 2 },
                    { question: "A que marca de automóveis pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/en/thumb/d/d1/Ferrari-Logo.svg/200px-Ferrari-Logo.svg.png', answers: ["Porsche", "Lamborghini", "Maserati", "Ferrari"], correct: 3 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/en/thumb/d/d3/Starbucks_Corporation_Logo_2011.svg/200px-Starbucks_Corporation_Logo_2011.svg.png', answers: ["Costa Coffee", "Starbucks", "Dunkin'", "Nespresso"], correct: 1 },
                    { question: "A que marca pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/6/6f/Logo_of_Twitter.svg/200px-Logo_of_Twitter.svg.png', answers: ["WhatsApp", "Telegram", "Twitter (X)", "Signal"], correct: 2 },
                    { question: "A que empresa pertence este logótipo?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/a9/Amazon_logo.svg/200px-Amazon_logo.svg.png', answers: ["Alibaba", "Amazon", "eBay", "Mercado Livre"], correct: 1 }
                ]
            },
            "monumentos": {
                name: "Monumentos do Mundo",
                questions: [
                    { question: "Onde fica este monumento?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Tour_Eiffel_Wikimedia_Commons_%28cropped%29.jpg/200px-Tour_Eiffel_Wikimedia_Commons_%28cropped%29.jpg', answers: ["Londres", "Roma", "Paris", "Berlim"], correct: 2 },
                    { question: "Qual o nome desta estátua?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/a1/Statue_of_Liberty_7.jpg/200px-Statue_of_Liberty_7.jpg', answers: ["O Pensador", "David", "Vénus de Milo", "Estátua da Liberdade"], correct: 3 },
                    { question: "Onde fica este antigo anfiteatro?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/d/de/Colosseo_2020.jpg/200px-Colosseo_2020.jpg', answers: ["Atenas", "Roma", "Istambul", "Cairo"], correct: 1 },
                    { question: "Qual o nome desta estrutura?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/2/23/The_Great_Wall_of_China_at_Jinshanling-edit.jpg/200px-The_Great_Wall_of_China_at_Jinshanling-edit.jpg', answers: ["Aqueduto de Segóvia", "Muralha da China", "Muralhas de Adriano", "Muralhas de Ávila"], correct: 1 },
                    { question: "Em que país fica este mausoléu?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Taj_Mahal%2C_Agra%2C_India_edit3.jpg/200px-Taj_Mahal%2C_Agra%2C_India_edit3.jpg', answers: ["Índia", "Paquistão", "Irão", "Turquia"], correct: 0 },
                    { question: "Qual o nome destas pirâmides?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/All_Gizah_Pyramids.jpg/200px-All_Gizah_Pyramids.jpg', answers: ["Pirâmides de Teotihuacan", "Pirâmides de Gizé", "Pirâmides da Núbia", "Zigurate de Ur"], correct: 1 },
                    { question: "Em que cidade fica esta ópera?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/4/40/Sydney_Opera_House_Sails.jpg/200px-Sydney_Opera_House_Sails.jpg', answers: ["Melbourne", "Auckland", "Sydney", "Wellington"], correct: 2 },
                    { question: "Qual o nome desta estátua no Rio de Janeiro?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/Christ_the_Redeemer_-_new_view.jpg/200px-Christ_the_Redeemer_-_new_view.jpg', answers: ["Cristo Rei", "Cristo Redentor", "Cristo da Concórdia", "Cristo de Havana"], correct: 1 },
                    { question: "Em que país fica esta cidade antiga?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/2/2f/Treasury_of_Petra.jpg/200px-Treasury_of_Petra.jpg', answers: ["Egito", "Israel", "Jordânia", "Arábia Saudita"], correct: 2 },
                    { question: "Qual o nome desta antiga cidadela inca?", imageUrl: 'https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/Machu_Picchu%2C_Peru.jpg/200px-Machu_Picchu%2C_Peru.jpg', answers: ["Chichén Itzá", "Tikal", "Palenque", "Machu Picchu"], correct: 3 }
                ]
            },
"celebridades": {
    name: "Quem é ? (Celebridades)",
    questions: [
        { question: "Quem é esta atriz?", imageUrl: 'https://image.tmdb.org/t/p/w200/A1kSoi1s2Z424w3yc69e2jK2yS.jpg', answers: ["Jennifer Aniston", "Angelina Jolie", "Charlize Theron", "Nicole Kidman"], correct: 1 },
        { question: "Quem é este ator?", imageUrl: 'https://image.tmdb.org/t/p/w200/5Brc5d6tT4a5gIadFDBdI2iI63s.jpg', answers: ["Brad Pitt", "Matt Damon", "Leonardo DiCaprio", "Johnny Depp"], correct: 2 },
        { question: "Quem é esta cantora?", imageUrl: 'https://image.tmdb.org/t/p/w200/1E12YsoM4c5C65nWEpPAcbPz2B.jpg', answers: ["Katy Perry", "Ariana Grande", "Selena Gomez", "Taylor Swift"], correct: 3 },
        { question: "Quem é este jogador de futebol?", imageUrl: 'https://image.tmdb.org/t/p/w200/8UvEgx4jI0T2i0cW2Hw2d3yIu3B.jpg', answers: ["Lionel Messi", "Neymar Jr.", "Cristiano Ronaldo", "Kylian Mbappé"], correct: 2 },
        { question: "Quem é esta artista?", imageUrl: 'https://image.tmdb.org/t/p/w200/3oKk1Y0s2wI8iF8ih23gY222wLp.jpg', answers: ["Rihanna", "Beyoncé", "Nicki Minaj", "Cardi B"], correct: 0 },
        { question: "Quem é este ator?", imageUrl: 'https://image.tmdb.org/t/p/w200/1A8jPY9d1t2XRA2x2h2n42O9p24.jpg', answers: ["Vin Diesel", "Jason Statham", "Dwayne Johnson", "John Cena"], correct: 2 },
        { question: "Quem é esta cantora?", imageUrl: 'https://image.tmdb.org/t/p/w200/3oMOCY0iQd5D61s22b0aGEmXnES.jpg', answers: ["Sia", "Adele", "Lady Gaga", "Katy Perry"], correct: 1 },
        { question: "Quem é este CEO?", imageUrl: 'https://image.tmdb.org/t/p/w200/pIqBfSLYn2hJAcVsoxAb3Tca40B.jpg', answers: ["Jeff Bezos", "Mark Zuckerberg", "Bill Gates", "Elon Musk"], correct: 3 },
        { question: "Quem é este ator?", imageUrl: 'https://image.tmdb.org/t/p/w200/5qHNjhtjMD4YWH3xSkj5j2dGZ3Y.jpg', answers: ["Chris Evans", "Robert Downey Jr.", "Mark Ruffalo", "Chris Hemsworth"], correct: 1 },
        { question: "Quem é este piloto de Fórmula 1?", imageUrl: 'https://image.tmdb.org/t/p/w200/3qT35yF13nAR9yCn4Gj3L8s1aB3.jpg', answers: ["Max Verstappen", "Sebastian Vettel", "Fernando Alonso", "Lewis Hamilton"], correct: 3 },
        { question: "Quem é esta atriz?", imageUrl: 'https://image.tmdb.org/t/p/w200/6NsMnJbv215GgrjN8A9SBxA6JkC.jpg', answers: ["Jennifer Lawrence", "Margot Robbie", "Scarlett Johansson", "Emma Watson"], correct: 2 },
        { question: "Quem é este cantor?", imageUrl: 'https://image.tmdb.org/t/p/w200/gKq9s4n6P093AuBf1f6i2g2PjG.jpg', answers: ["Justin Bieber", "Shawn Mendes", "Ed Sheeran", "Harry Styles"], correct: 2 },
        { question: "Quem é esta apresentadora?", imageUrl: 'https://image.tmdb.org/t/p/w200/1h2r27i1qV2V2B299aA5K42gS0g.jpg', answers: ["Oprah Winfrey", "Ellen DeGeneres", "Tyra Banks", "Whoopi Goldberg"], correct: 0 },
        { question: "Quem é este ator?", imageUrl: 'https://image.tmdb.org/t/p/w200/qtuFcM919EDT227f04H9nB4k00s.jpg', answers: ["Tom Cruise", "Tom Hanks", "Brad Pitt", "George Clooney"], correct: 1 },
        { question: "Quem é esta cantora?", imageUrl: 'https://image.tmdb.org/t/p/w200/uILe2sAl3rKcpv3aI2R90g3d2D.jpg', answers: ["Rihanna", "Alicia Keys", "Mariah Carey", "Beyoncé"], correct: 3 },
        { question: "Quem é este ator?", imageUrl: 'https://image.tmdb.org/t/p/w200/9Jsvb20rV1R5y20dGzS2r92C32z.jpg', answers: ["Will Smith", "Denzel Washington", "Jamie Foxx", "Eddie Murphy"], correct: 0 },
        { question: "Quem é esta tenista?", imageUrl: 'https://image.tmdb.org/t/p/w200/h2Oh1m3P032S8g1A4aJpGs5sLd1.jpg', answers: ["Venus Williams", "Naomi Osaka", "Serena Williams", "Maria Sharapova"], correct: 2 },
        { question: "Quem é este cantor?", imageUrl: 'https://image.tmdb.org/t/p/w200/1XGI1Ncr1Vn9Ag2rI1iB6nBTMXo.jpg', answers: ["Justin Timberlake", "Justin Bieber", "Shawn Mendes", "Zayn Malik"], correct: 1 },
        { question: "Quem é esta atriz?", imageUrl: 'https://image.tmdb.org/t/p/w200/wzcP3vKtywrg4b2hJAcVsoxAb3Tca40B.jpg', answers: ["Jennifer Aniston", "Jennifer Lopez", "Jennifer Garner", "Jennifer Lawrence"], correct: 3 },
        { question: "Quem é este jogador de futebol?", imageUrl: 'https://image.tmdb.org/t/p/w200/c2PLiYxOstseI4GZnoo4k2V2yXo.jpg', answers: ["Cristiano Ronaldo", "Lionel Messi", "Neymar Jr.", "Luis Suárez"], correct: 1 }
    ]
            },
            "famosos": {
    name: "Perguntas sobre Famosos",
    questions: [
        { question: "Qual atriz ganhou um Oscar por 'La La Land'?", answers: ["Emma Watson", "Emma Stone", "Jennifer Lawrence", "Scarlett Johansson"], correct: 1 },
        { question: "Qual é o nome verdadeiro do cantor 'The Weeknd'?", answers: ["Aubrey Graham", "Donald Glover", "Abel Tesfaye", "Bruno Mars"], correct: 2 },
        { question: "Que famoso chef britânico é conhecido por programas como 'Hell's Kitchen'?", answers: ["Jamie Oliver", "Gordon Ramsay", "Marco Pierre White", "Heston Blumenthal"], correct: 1 },
        { question: "Qual o nome da cantora conhecida como 'Rainha do Pop'?", answers: ["Beyoncé", "Rihanna", "Madonna", "Lady Gaga"], correct: 2 },
        { question: "Que ator interpretou James Bond antes de Daniel Craig?", answers: ["Sean Connery", "Roger Moore", "Timothy Dalton", "Pierce Brosnan"], correct: 3 },
        { question: "Qual o nome da apresentadora de TV americana que tinha um famoso 'talk show' diário?", answers: ["Ellen DeGeneres", "Oprah Winfrey", "Tyra Banks", "Wendy Williams"], correct: 1 },
        { question: "De que banda era vocalista Kurt Cobain?", answers: ["Pearl Jam", "Soundgarden", "Nirvana", "Alice in Chains"], correct: 2 },
        { question: "Qual o nome da famosa cantora e compositora espanhola de 'Malamente'?", answers: ["Shakira", "Rosalía", "Camila Cabello", "Natti Natasha"], correct: 1 },
        { question: "Que ator é famoso por interpretar o Capitão Jack Sparrow?", answers: ["Orlando Bloom", "Geoffrey Rush", "Johnny Depp", "Javier Bardem"], correct: 2 },
        { question: "Qual é o nome da irmã mais nova de Kim Kardashian?", answers: ["Kourtney", "Khloé", "Kendall", "Kylie"], correct: 3 },
        { question: "Qual jogador de basquetebol é apelidado de 'King James'?", answers: ["Michael Jordan", "Kobe Bryant", "LeBron James", "Stephen Curry"], correct: 2 },
        { question: "Qual o nome do criador do Facebook?", answers: ["Bill Gates", "Larry Page", "Jeff Bezos", "Mark Zuckerberg"], correct: 3 },
        { question: "Qual o nome do famoso físico que desenvolveu a teoria da relatividade geral?", answers: ["Isaac Newton", "Galileu Galilei", "Stephen Hawking", "Albert Einstein"], correct: 3 },
        { question: "Qual o nome do realizador de 'Titanic' e 'Avatar'?", answers: ["Steven Spielberg", "George Lucas", "James Cameron", "Christopher Nolan"], correct: 2 },
        { question: "Qual o nome da cantora de 'Rolling in the Deep'?", answers: ["Adele", "Sia", "Amy Winehouse", "Duffy"], correct: 0 },
        { question: "Qual o ator que interpreta 'Thor' nos filmes da Marvel?", answers: ["Chris Evans", "Chris Hemsworth", "Chris Pratt", "Tom Hiddleston"], correct: 1 },
        { question: "Qual o nome da famosa tenista que tem 23 títulos de Grand Slam?", answers: ["Venus Williams", "Serena Williams", "Maria Sharapova", "Naomi Osaka"], correct: 1 },
        { question: "Qual o nome do vocalista da banda Coldplay?", answers: ["Thom Yorke", "Bono", "Chris Martin", "Eddie Vedder"], correct: 2 },
        { question: "Qual o nome do famoso mágico e ilusionista conhecido por ter feito a Estátua da Liberdade desaparecer?", answers: ["David Blaine", "Criss Angel", "Penn & Teller", "David Copperfield"], correct: 3 },
        { question: "Qual o nome do ator que interpretou 'Forrest Gump'?", answers: ["Tom Cruise", "Brad Pitt", "Tom Hanks", "Leonardo DiCaprio"], correct: 2 }
    ]
},
"mpb": {
    name: "Músicas MPB",
    questions: [
        { question: "Qual destes artistas é conhecido como 'O Síndico'?", answers: ["Jorge Ben Jor", "Tim Maia", "Wilson Simonal", "Raul Seixas"], correct: 1 },
        { question: "Quem compôs a música 'Águas de Março'?", answers: ["Tom Jobim", "Vinicius de Moraes", "Chico Buarque", "Caetano Veloso"], correct: 0 },
        { question: "Qual o nome do famoso álbum de Caetano Veloso e Gilberto Gil gravado em exílio em Londres?", answers: ["Expresso 2222", "Tropicália", "Transa", "London, London"], correct: 3 },
        { question: "A canção 'O Bêbado e a Equilibrista' tornou-se um hino da anistia no Brasil. Quem a interpretou?", answers: ["Elis Regina", "Gal Costa", "Maria Bethânia", "Simone"], correct: 0 },
        { question: "Qual banda de rock rural foi liderada por Zé Ramalho e Alceu Valença?", answers: ["Novos Baianos", "Secos & Molhados", "A Cor do Som", "O Grande Encontro"], correct: 3 },
        { question: "De qual álbum dos Tribalistas é a música 'Velha Infância'?", answers: ["Tribalistas (2002)", "Tribalistas (2017)", "Já Sei Namorar", "É Você"], correct: 0 },
        { question: "Quem canta a música 'Sozinho', um grande sucesso no final dos anos 90?", answers: ["Djavan", "Caetano Veloso", "Peninha", "Fábio Jr."], correct: 1 },
        { question: "Qual o nome do movimento musical que revelou artistas como Caetano Veloso, Gilberto Gil e Gal Costa?", answers: ["Bossa Nova", "Jovem Guarda", "Tropicália", "Clube da Esquina"], correct: 2 },
        { question: "A música 'Construção' é uma das obras-primas de qual compositor?", answers: ["Milton Nascimento", "Chico Buarque", "Tom Zé", "Belchior"], correct: 1 },
        { question: "'Como Nossos Pais' é uma canção de Belchior que ficou famosa na voz de quem?", answers: ["Zizi Possi", "Fagner", "Elis Regina", "Amelinha"], correct: 2 },
        { question: "Qual artista mineiro é uma das figuras centrais do 'Clube da Esquina'?", answers: ["Lô Borges", "Milton Nascimento", "Beto Guedes", "Todos os anteriores"], correct: 3 },
        { question: "Quem é o autor da canção 'País Tropical'?", answers: ["Jorge Ben Jor", "Gilberto Gil", "Toquinho", "Chico Science"], correct: 0 },
        { question: "Qual o nome do grupo formado por Marisa Monte, Arnaldo Antunes e Carlinhos Brown?", answers: ["Os Tribalistas", "Titãs", "Paralamas do Sucesso", "Nação Zumbi"], correct: 0 },
        { question: "A música 'Sampa', de Caetano Veloso, é uma homenagem a qual cidade?", answers: ["Salvador", "Rio de Janeiro", "São Paulo", "Belo Horizonte"], correct: 2 },
        { question: "Qual destes artistas não fez parte do grupo Novos Baianos?", answers: ["Moraes Moreira", "Baby do Brasil", "Pepeu Gomes", "Raul Seixas"], correct: 3 },
        { question: "Quem canta a música 'Oceano'?", answers: ["Caetano Veloso", "Gilberto Gil", "Milton Nascimento", "Djavan"], correct: 3 },
        { question: "Qual o nome do famoso teatro no Rio de Janeiro que foi palco de muitos shows de MPB?", answers: ["Teatro Municipal", "Canecão", "Teatro Oficina", "Teatro Amazonas"], correct: 1 },
        { question: "Qual o nome da canção de Cássia Eller que fala sobre um 'Errado que deu certo'?", answers: ["O Segundo Sol", "Malandragem", "Por Enquanto", "All Star"], correct: 1 },
        { question: "Qual o nome do primeiro álbum de Maria Bethânia, lançado em 1965?", answers: ["Carcará", "Maria Bethânia", "Edu e Bethânia", "A Tua Presença"], correct: 1 },
        { question: "Qual o nome da música de Legião Urbana que cita 'a gente se encontra todo dia na mesma praça'?", answers: ["Eduardo e Mônica", "Faroeste Caboclo", "Tempo Perdido", "Pais e Filhos"], correct: 0 }
    ]
},
"musica_br": {
    name: "Músicas Brasileiras",
    questions: [
        { question: "Qual o gênero musical de Luiz Gonzaga, o 'Rei do Baião'?", answers: ["Samba", "Forró", "Sertanejo", "Bossa Nova"], correct: 1 },
        { question: "Qual dupla sertaneja é conhecida como 'os filhos de Francisco'?", answers: ["Chitãozinho & Xororó", "Leandro & Leonardo", "Zezé Di Camargo & Luciano", "Bruno & Marrone"], correct: 2 },
        { question: "Qual banda de rock brasileira lançou o álbum 'Cabeça Dinossauro'?", answers: ["Legião Urbana", "Titãs", "Paralamas do Sucesso", "Barão Vermelho"], correct: 1 },
        { question: "Qual o nome da cantora que ficou conhecida como 'Rainha do Rock Brasileiro'?", answers: ["Pitty", "Paula Toller", "Cássia Eller", "Rita Lee"], correct: 3 },
        { question: "Qual o nome da cantora baiana que é uma das maiores estrelas do Axé Music?", answers: ["Daniela Mercury", "Claudia Leitte", "Ivete Sangalo", "Margareth Menezes"], correct: 2 },
        { question: "Qual o nome do grupo de pagode dos anos 90 que tinha Alexandre Pires como vocalista?", answers: ["Raça Negra", "Só Pra Contrariar", "Exaltasamba", "Art Popular"], correct: 1 },
        { question: "Qual o nome do famoso sambista autor de 'Não Deixe o Samba Morrer'?", answers: ["Cartola", "Paulinho da Viola", "Martinho da Vila", "Alcione"], correct: 3 },
        { question: "Qual o nome da cantora que ficou famosa com a música 'Show das Poderosas'?", answers: ["Ludmilla", "Iza", "Pabllo Vittar", "Anitta"], correct: 3 },
        { question: "Qual o nome do rapper brasileiro que é membro do grupo Racionais MC's?", answers: ["Criolo", "Emicida", "Mano Brown", "Projota"], correct: 2 },
        { question: "Qual o nome do gênero musical que mistura maracatu, rock, hip hop e manguebeat?", answers: ["Axé", "Forró Eletrónico", "Funk Carioca", "Manguebeat"], correct: 3 },
        { question: "Qual o nome do cantor e compositor que é considerado o 'Poeta da Vila'?", answers: ["Noel Rosa", "Cartola", "Nelson Cavaquinho", "Adoniran Barbosa"], correct: 0 },
        { question: "Qual o nome da cantora que ficou conhecida pela música 'Evidências'?", answers: ["Roberta Miranda", "Ana Paula Valadão", "Simone Mendes", "Não foi uma cantora, e sim a dupla Chitãozinho & Xororó"], correct: 3 },
        { question: "Qual o nome do famoso festival de música que acontece no Rio de Janeiro?", answers: ["Lollapalooza", "Rock in Rio", "The Town", "Villa Mix"], correct: 1 },
        { question: "Qual o nome da dupla de palhaços que fazia sucesso com músicas infantis nos anos 80?", answers: ["Patati Patatá", "Atchim & Espirro", "Teleco & Teco", "Carequinha & Arrelia"], correct: 1 },
        { question: "Qual o nome da cantora que é considerada a 'Madrinha do Samba'?", answers: ["Dona Ivone Lara", "Clara Nunes", "Elza Soares", "Beth Carvalho"], correct: 3 },
        { question: "Qual o nome do funkeiro que ficou famoso com a música 'Rap da Felicidade'?", answers: ["MC Marcinho", "Cidinho & Doca", "MC Gorila", "MC Sapão"], correct: 1 },
        { question: "Qual o nome do famoso compositor de 'Garota de Ipanema'?", answers: ["Tom Jobim e Vinicius de Moraes", "Roberto Carlos e Erasmo Carlos", "Lennon e McCartney", "Chico Buarque e Edu Lobo"], correct: 0 },
        { question: "Qual o nome da banda de reggae liderada por Hélio Bentes?", answers: ["Cidade Negra", "Natiruts", "Ponto de Equilíbrio", "Chimarruts"], correct: 2 },
        { question: "Qual o nome do cantor sertanejo que é conhecido como 'O Embaixador'?", answers: ["Luan Santana", "Michel Teló", "Gusttavo Lima", "Wesley Safadão"], correct: 2 },
        { question: "Qual o nome da famosa cantora de 'Amor I Love You'?", answers: ["Adriana Calcanhotto", "Marisa Monte", "Vanessa da Mata", "Ana Carolina"], correct: 1 }
    ]
},
"cantores_br": {
    name: "Tudo sobre Cantores Brasileiros 1",
    questions: [
        { question: "Qual cantor brasileiro era conhecido como 'Maluco Beleza'?", answers: ["Raul Seixas", "Tim Maia", "Belchior", "Chico Science"], correct: 0 },
        { question: "Qual cantora é filha de Elis Regina e César Camargo Mariano?", answers: ["Maria Rita", "Luiza Possi", "Sandy", "Wanessa Camargo"], correct: 0 },
        { question: "Qual destes cantores nasceu em Minas Gerais?", answers: ["Caetano Veloso", "Gilberto Gil", "Milton Nascimento", "Djavan"], correct: 2 },
        { question: "Qual o nome do cantor que liderou a banda Legião Urbana?", answers: ["Dinho Ouro Preto", "Humberto Gessinger", "Renato Russo", "Cazuza"], correct: 2 },
        { question: "Qual cantor e compositor é famoso pela sua habilidade com o violão e por músicas como 'Oceano'?", answers: ["Toquinho", "Djavan", "João Bosco", "Ivan Lins"], correct: 1 },
        { question: "Qual cantora ficou famosa no grupo 'É o Tchan!' antes de seguir carreira a solo?", answers: ["Carla Perez", "Scheila Carvalho", "Sheila Mello", "Nenhuma das anteriores"], correct: 3 },
        { question: "Qual o nome do cantor que formava dupla com Cazuza no Barão Vermelho?", answers: ["Frejat", "Dinho Ouro Preto", "Herbert Vianna", "Samuel Rosa"], correct: 0 },
        { question: "Qual cantora popularizou a música 'Fogo e Paixão'?", answers: ["Alcione", "Fafá de Belém", "Wando", "Gal Costa"], correct: 2 },
        { question: "Qual cantor e compositor baiano é um dos criadores da Tropicália?", answers: ["Gilberto Gil", "Tom Zé", "Jorge Mautner", "Todos os anteriores"], correct: 3 },
        { question: "Qual o nome da cantora que é irmã de Caetano Veloso?", answers: ["Gal Costa", "Maria Bethânia", "Simone", "Zizi Possi"], correct: 1 },
        { question: "Qual cantor é famoso pela música 'Descobridor dos Sete Mares'?", answers: ["Lulu Santos", "Tim Maia", "Jorge Ben Jor", "Ed Motta"], correct: 1 },
        { question: "Qual cantora é conhecida como 'A Marrom'?", answers: ["Elza Soares", "Clara Nunes", "Alcione", "Beth Carvalho"], correct: 2 },
        { question: "Qual o nome do cantor que liderava a banda Skank?", answers: ["Dinho Ouro Preto", "Samuel Rosa", "Nando Reis", "Humberto Gessinger"], correct: 1 },
        { question: "Qual cantor e compositor cearense é autor de 'A Palo Seco'?", answers: ["Fagner", "Ednardo", "Belchior", "Amelinha"], correct: 2 },
        { question: "Qual o nome do cantor que ficou famoso com a música 'Ai Se Eu Te Pego'?", answers: ["Gusttavo Lima", "Luan Santana", "Michel Teló", "Lucas Lucco"], correct: 2 },
        { question: "Qual cantora é conhecida pela sua voz rouca e por sucessos como 'Brasil'?", answers: ["Cássia Eller", "Gal Costa", "Zélia Duncan", "Sandra de Sá"], correct: 1 },
        { question: "Qual cantor e compositor é conhecido como o 'Rei'?", answers: ["Roberto Carlos", "Fábio Jr.", "Amado Batista", "Reginaldo Rossi"], correct: 0 },
        { question: "Qual o nome da cantora que é filha de Zezé Di Camargo?", answers: ["Sandy", "Wanessa Camargo", "Paula Fernandes", "Marília Mendonça"], correct: 1 },
        { question: "Qual o nome do cantor que liderou a banda Charlie Brown Jr.?", answers: ["Chorão", "Champignon", "Thiago Castanho", "Marcão"], correct: 0 },
        { question: "Qual o nome do cantor que é famoso pela música 'Evidências'?", answers: ["Chitãozinho & Xororó", "Leandro & Leonardo", "Zezé Di Camargo & Luciano", "Daniel"], correct: 0 }
    ]
},
"musica_int": {
    name: "Músicas Internacionais",
    questions: [
        { question: "Qual banda britânica lançou o álbum 'The Dark Side of the Moon'?", answers: ["The Beatles", "Led Zeppelin", "Pink Floyd", "The Rolling Stones"], correct: 2 },
        { question: "Quem é o 'Rei do Pop'?", answers: ["Elvis Presley", "Michael Jackson", "Prince", "James Brown"], correct: 1 },
        { question: "Qual o nome da cantora de 'I Will Always Love You'?", answers: ["Mariah Carey", "Whitney Houston", "Celine Dion", "Aretha Franklin"], correct: 1 },
        { question: "Qual banda lançou a música 'Bohemian Rhapsody'?", answers: ["U2", "Queen", "The Eagles", "ABBA"], correct: 1 },
        { question: "De que país é a banda U2?", answers: ["Inglaterra", "Escócia", "Irlanda", "País de Gales"], correct: 2 },
        { question: "Qual o nome do rapper de 'Lose Yourself'?", answers: ["Jay-Z", "Dr. Dre", "Snoop Dogg", "Eminem"], correct: 3 },
        { question: "Qual o nome da banda de Kurt Cobain?", answers: ["Soundgarden", "Pearl Jam", "Nirvana", "Alice in Chains"], correct: 2 },
        { question: "Qual o nome da cantora de 'Like a Virgin'?", answers: ["Cyndi Lauper", "Madonna", "Janet Jackson", "Cher"], correct: 1 },
        { question: "Qual o nome da banda sueca que ficou famosa com 'Dancing Queen'?", answers: ["Roxette", "Ace of Base", "ABBA", "The Cardigans"], correct: 2 },
        { question: "Qual o nome do cantor de 'Shape of You'?", answers: ["Justin Bieber", "Ed Sheeran", "Shawn Mendes", "Bruno Mars"], correct: 1 },
        { question: "Qual o nome da banda de 'Stairway to Heaven'?", answers: ["The Who", "Led Zeppelin", "Deep Purple", "Black Sabbath"], correct: 1 },
        { question: "Qual o nome da cantora de 'Bad Guy'?", answers: ["Lana Del Rey", "Lorde", "Halsey", "Billie Eilish"], correct: 3 },
        { question: "Qual o nome da banda de 'Hotel California'?", answers: ["The Doors", "The Eagles", "Fleetwood Mac", "Creedence Clearwater Revival"], correct: 1 },
        { question: "Qual o nome do cantor de 'Uptown Funk'?", answers: ["Pharrell Williams", "Justin Timberlake", "Bruno Mars", "The Weeknd"], correct: 2 },
        { question: "Qual o nome da banda de 'Wonderwall'?", answers: ["Blur", "Oasis", "Pulp", "The Verve"], correct: 1 },
        { question: "Qual o nome da cantora de 'Single Ladies (Put a Ring on It)'?", answers: ["Rihanna", "Beyoncé", "Alicia Keys", "Mariah Carey"], correct: 1 },
        { question: "Qual o nome do cantor de 'Hallelujah'?", answers: ["Bob Dylan", "Leonard Cohen", "Jeff Buckley", "Neil Young"], correct: 1 },
        { question: "Qual o nome da banda de 'Smells Like Teen Spirit'?", answers: ["Nirvana", "Pearl Jam", "Soundgarden", "Foo Fighters"], correct: 0 },
        { question: "Qual o nome da cantora de 'Rolling in the Deep'?", answers: ["Duffy", "Amy Winehouse", "Adele", "Florence + The Machine"], correct: 2 },
        { question: "Qual o nome do cantor de 'Billie Jean'?", answers: ["Prince", "Stevie Wonder", "Michael Jackson", "Marvin Gaye"], correct: 2 }
    ]
},
"cantores_int": {
    name: "Tudo sobre Cantores Internacionais 1",
    questions: [
        { question: "Qual o país de origem da cantora Björk?", answers: ["Noruega", "Suécia", "Finlândia", "Islândia"], correct: 3 },
        { question: "Qual cantor era o vocalista da banda The Police?", answers: ["Bono", "Sting", "Phil Collins", "Peter Gabriel"], correct: 1 },
        { question: "Qual destes cantores fazia parte dos Beatles?", answers: ["Elton John", "David Bowie", "Paul McCartney", "Freddie Mercury"], correct: 2 },
        { question: "Qual o nome da cantora que ficou conhecida como a 'Rainha da Soul'?", answers: ["Diana Ross", "Aretha Franklin", "Tina Turner", "Etta James"], correct: 1 },
        { question: "De que país era o cantor Bob Marley?", answers: ["Trindade e Tobago", "Barbados", "Haiti", "Jamaica"], correct: 3 },
        { question: "Qual o nome do vocalista da banda Rolling Stones?", answers: ["Robert Plant", "Roger Daltrey", "Mick Jagger", "Jim Morrison"], correct: 2 },
        { question: "Qual o nome do famoso pianista e cantor de 'Rocket Man'?", answers: ["Elton John", "Billy Joel", "Stevie Wonder", "Ray Charles"], correct: 0 },
        { question: "Qual cantora é conhecida como 'A Voz'?", answers: ["Mariah Carey", "Whitney Houston", "Celine Dion", "Christina Aguilera"], correct: 1 },
        { question: "Qual destes cantores é de nacionalidade canadiana?", answers: ["Ed Sheeran", "Justin Bieber", "Harry Styles", "Shawn Mendes"], correct: 1 },
        { question: "Qual o nome do cantor que liderou a banda Queen?", answers: ["David Bowie", "Freddie Mercury", "Elton John", "George Michael"], correct: 1 },
        { question: "Qual o nome do famoso cantor de 'My Way'?", answers: ["Elvis Presley", "Frank Sinatra", "Dean Martin", "Tony Bennett"], correct: 1 },
        { question: "Qual o nome da cantora de 'Ironic'?", answers: ["Sheryl Crow", "Alanis Morissette", "Gwen Stefani", "Fiona Apple"], correct: 1 },
        { question: "Qual o nome do famoso cantor de blues conhecido pela sua guitarra 'Lucille'?", answers: ["B.B. King", "Muddy Waters", "John Lee Hooker", "Howlin' Wolf"], correct: 0 },
        { question: "Qual o nome da cantora que é considerada a 'Madrinha do Punk'?", answers: ["Joan Jett", "Debbie Harry", "Patti Smith", "Siouxsie Sioux"], correct: 2 },
        { question: "Qual o nome do famoso tenor italiano que popularizou a ópera?", answers: ["Andrea Bocelli", "Plácido Domingo", "José Carreras", "Luciano Pavarotti"], correct: 3 },
        { question: "Qual o nome da cantora de 'Wuthering Heights'?", answers: ["Kate Bush", "Tori Amos", "Björk", "PJ Harvey"], correct: 0 },
        { question: "Qual o nome do famoso rapper que foi assassinado em 1996?", answers: ["The Notorious B.I.G.", "Tupac Shakur", "Eazy-E", "Big L"], correct: 1 },
        { question: "Qual o nome do famoso cantor de 'What's Going On'?", answers: ["Marvin Gaye", "Stevie Wonder", "Al Green", "Bill Withers"], correct: 0 },
        { question: "Qual o nome da famosa cantora de 'La Vie en Rose'?", answers: ["Edith Piaf", "Brigitte Bardot", "Françoise Hardy", "Juliette Gréco"], correct: 0 },
        { question: "Qual o nome do famoso cantor de 'Imagine'?", answers: ["Paul McCartney", "George Harrison", "Ringo Starr", "John Lennon"], correct: 3 }
    ]
},
    "vencedores_oscar": {
        "name": "Vencedores do Oscar",
        "questions": [
            { "question": "Qual filme foi o grande vencedor do Oscar de Melhor Filme em 2020?", "answers": ["1917", "Coringa", "Parasita", "O Irlandês"], "correct": 2 },
            { "question": "Quantos Oscars o filme 'Titanic' (1997) ganhou?", "answers": ["8", "9", "10", "11"], "correct": 3 },
            { "question": "Qual ator ganhou o Oscar de Melhor Ator por seu papel em 'O Regresso' (2015)?", "answers": ["Tom Hardy", "Michael Fassbender", "Eddie Redmayne", "Leonardo DiCaprio"], "correct": 3 },
            { "question": "Qual foi o primeiro filme de super-herói a ser indicado a Melhor Filme?", "answers": ["O Cavaleiro das Trevas", "Pantera Negra", "Vingadores: Ultimato", "Logan"], "correct": 1 },
            { "question": "Qual diretora se tornou a segunda mulher na história a ganhar o Oscar de Melhor Direção, por 'Nomadland'?", "answers": ["Greta Gerwig", "Sofia Coppola", "Chloé Zhao", "Jane Campion"], "correct": 2 },
            { "question": "Qual filme de animação da Disney foi o primeiro a ser indicado para Melhor Filme?", "answers": ["O Rei Leão", "A Bela e a Fera", "Aladdin", "A Pequena Sereia"], "correct": 1 },
            { "question": "Qual ator ganhou um Oscar póstumo por sua atuação como o Coringa em 'O Cavaleiro das Trevas'?", "answers": ["Jack Nicholson", "Heath Ledger", "Cesar Romero", "Jared Leto"], "correct": 1 },
            { "question": "Qual filme detém o recorde de mais indicações ao Oscar, com 14 nomeações, junto com 'Titanic' e 'La La Land'?", "answers": ["Ben-Hur", "A Malvada", "O Senhor dos Anéis: O Retorno do Rei", "E o Vento Levou"], "correct": 1 },
            { "question": "Qual o único filme da história a ganhar os 'Cinco Grandes' prêmios do Oscar (Melhor Filme, Diretor, Ator, Atriz e Roteiro)?", "answers": ["O Poderoso Chefão", "Aconteceu Naquela Noite", "O Silêncio dos Inocentes", "Todos os anteriores"], "correct": 3 },
            { "question": "Qual atriz detém o recorde de mais vitórias no Oscar de Melhor Atriz?", "answers": ["Meryl Streep", "Ingrid Bergman", "Bette Davis", "Katharine Hepburn"], "correct": 3 },
            { "question": "Em que ano a atriz brasileira Fernanda Montenegro foi indicada ao Oscar de Melhor Atriz?", "answers": ["1997", "1998", "1999", "2000"], "correct": 2 },
            { "question": "Qual filme de Steven Spielberg sobre o Holocausto ganhou o Oscar de Melhor Filme em 1994?", "answers": ["O Resgate do Soldado Ryan", "A Lista de Schindler", "E.T. - O Extraterrestre", "Tubarão"], "correct": 1 },
            { "question": "Qual ator ganhou seu primeiro Oscar por 'Forrest Gump - O Contador de Histórias'?", "answers": ["Tom Cruise", "Tom Hanks", "Robin Williams", "Gary Sinise"], "correct": 1 },
            { "question": "Qual o primeiro filme em língua não inglesa a ganhar o Oscar de Melhor Filme?", "answers": ["Roma", "O Tigre e o Dragão", "A Vida é Bela", "Parasita"], "correct": 3 },
            { "question": "Qual filme de Martin Scorsese finalmente lhe rendeu o Oscar de Melhor Diretor?", "answers": ["Touro Indomável", "Os Bons Companheiros", "O Lobo de Wall Street", "Os Infiltrados"], "correct": 3 },
            { "question": "Qual atriz ganhou o Oscar de Melhor Atriz Coadjuvante por '12 Anos de Escravidão'?", "answers": ["Viola Davis", "Octavia Spencer", "Lupita Nyong'o", "Regina King"], "correct": 2 },
            { "question": "Qual filme de 2017 teve uma famosa confusão no anúncio de Melhor Filme, que foi entregue a 'Moonlight'?", "answers": ["La La Land", "A Chegada", "Manchester à Beira-Mar", "Até o Último Homem"], "correct": 0 },
            { "question": "Qual destes filmes de Quentin Tarantino ganhou o Oscar de Melhor Roteiro Original?", "answers": ["Cães de Aluguel", "Kill Bill: Volume 1", "Pulp Fiction: Tempo de Violência", "Bastardos Inglórios"], "correct": 2 },
            { "question": "Qual o nome do compositor que mais ganhou Oscars na história?", "answers": ["John Williams", "Alfred Newman", "Hans Zimmer", "Walt Disney"], "correct": 3 },
            { "question": "Qual filme de 2024 foi o grande vencedor do Oscar, levando 7 estatuetas, incluindo Melhor Filme?", "answers": ["Pobres Criaturas", "Anatomia de uma Queda", "Zona de Interesse", "Oppenheimer"], "correct": 3 }
        ]
    },
    "historia_brasil": {
        "name": "História do Brasil",
        "questions": [
            { "question": "Qual foi a primeira capital do Brasil?", "answers": ["Rio de Janeiro", "São Paulo", "Salvador", "Olinda"], "correct": 2 },
            { "question": "A Lei Áurea, que aboliu a escravidão no Brasil, foi assinada em que ano?", "answers": ["1822", "1888", "1889", "1850"], "correct": 1 },
            { "question": "Quem proclamou a Independência do Brasil?", "answers": ["Dom João VI", "Dom Pedro I", "Dom Pedro II", "Marechal Deodoro da Fonseca"], "correct": 1 },
            { "question": "Qual o nome da principal revolta nativista ocorrida em Minas Gerais no século XVIII?", "answers": ["Guerra dos Mascates", "Revolta de Beckman", "Inconfidência Mineira", "Conjuração Baiana"], "correct": 2 },
            { "question": "O período conhecido como 'Estado Novo' foi uma ditadura liderada por qual presidente?", "answers": ["Juscelino Kubitschek", "Jânio Quadros", "João Goulart", "Getúlio Vargas"], "correct": 3 },
            { "question": "Qual o nome do tratado que dividia as terras recém-descobertas entre Portugal e Espanha?", "answers": ["Tratado de Madri", "Tratado de Tordesilhas", "Tratado de Utrecht", "Tratado de Santo Ildefonso"], "correct": 1 },
            { "question": "A Proclamação da República do Brasil ocorreu em qual data?", "answers": ["7 de Setembro de 1822", "13 de Maio de 1888", "15 de Novembro de 1889", "21 de Abril de 1792"], "correct": 2 },
            { "question": "Qual foi o principal ciclo econômico do Brasil Colonial no século XVIII?", "answers": ["Ciclo do Pau-Brasil", "Ciclo do Açúcar", "Ciclo do Ouro", "Ciclo do Café"], "correct": 2 },
            { "question": "Como ficou conhecida a guerra que envolveu o Brasil, Argentina e Uruguai contra o Paraguai?", "answers": ["Guerra do Chaco", "Guerra do Pacífico", "Guerra da Cisplatina", "Guerra do Paraguai"], "correct": 3 },
            { "question": "Qual presidente foi responsável pela construção de Brasília?", "answers": ["Getúlio Vargas", "Eurico Gaspar Dutra", "Juscelino Kubitschek", "João Goulart"], "correct": 2 },
            { "question": "Quem foi Zumbi dos Palmares?", "answers": ["Um bandeirante famoso", "O último líder do Quilombo dos Palmares", "Um poeta do arcadismo", "Um inconfidente mineiro"], "correct": 1 },
            { "question": "O que foi a 'Revolta da Vacina'?", "answers": ["Um protesto contra a falta de vacinas", "Uma revolta popular contra a vacinação obrigatória", "Uma campanha de vacinação em massa", "Uma guerra por patentes de vacinas"], "correct": 1 },
            { "question": "Qual o nome do movimento político que pedia a volta das eleições diretas durante a Ditadura Militar?", "answers": ["Caras-Pintadas", "Movimento dos Sem Terra", "Diretas Já", "Tropicália"], "correct": 2 },
            { "question": "Qual foi a principal atividade econômica dos primeiros 30 anos da colonização portuguesa no Brasil?", "answers": ["Mineração de ouro", "Cultivo de cana-de-açúcar", "Extração de pau-brasil", "Criação de gado"], "correct": 2 },
            { "question": "O que foram as 'capitanias hereditárias'?", "answers": ["Províncias governadas por militares", "Territórios indígenas demarcados", "Lotes de terra doados pela Coroa a nobres", "Cidades fundadas pelos jesuítas"], "correct": 2 },
            { "question": "Qual imperador brasileiro reinou por quase 50 anos, num período de grande progresso cultural e material?", "answers": ["Dom Pedro I", "Dom João VI", "Dom Pedro II", "Dom Sebastião"], "correct": 2 },
            { "question": "A 'Semana de Arte Moderna' de 1922, que revolucionou a cultura brasileira, ocorreu em qual cidade?", "answers": ["Rio de Janeiro", "Salvador", "Recife", "São Paulo"], "correct": 3 },
            { "question": "Qual o nome dado aos exploradores que adentravam o sertão em busca de riquezas e indígenas para escravizar?", "answers": ["Jesuítas", "Tropeiros", "Bandeirantes", "Coronéis"], "correct": 2 },
            { "question": "Qual foi o motivo da transferência da corte portuguesa para o Brasil em 1808?", "answers": ["Uma crise econômica em Portugal", "A busca por ouro no Brasil", "A invasão de Portugal pelas tropas de Napoleão", "A proclamação da república em Portugal"], "correct": 2 },
            { "question": "Qual presidente sofreu impeachment em 1992, após uma grande mobilização popular?", "answers": ["José Sarney", "Fernando Collor de Mello", "Itamar Franco", "Fernando Henrique Cardoso"], "correct": 1 }
        ]
    },
    "historia_mundo_novo": {
        "name": "História do Mundo 2",
        "questions": [
            { "question": "Qual império construiu a famosa estrada 'Caminho Real Persa'?", "answers": ["Império Romano", "Império Aquemênida (Persa)", "Império Babilônico", "Império Egípcio"], "correct": 1 },
            { "question": "A Guerra Fria foi um conflito ideológico entre quais duas superpotências?", "answers": ["Alemanha e Japão", "Reino Unido e França", "Estados Unidos e União Soviética", "China e Índia"], "correct": 2 },
            { "question": "Qual foi a invenção de Johannes Gutenberg que revolucionou a disseminação do conhecimento?", "answers": ["O telescópio", "A bússola", "A pólvora", "A prensa de tipos móveis"], "correct": 3 },
            { "question": "Quem foi o líder dos hunos, conhecido como o 'Flagelo de Deus'?", "answers": ["Gengis Khan", "Átila", "Alexandre, o Grande", "Tamerlão"], "correct": 1 },
            { "question": "Em que cidade antiga se localizavam os Jardins Suspensos, uma das Sete Maravilhas do Mundo Antigo?", "answers": ["Atenas", "Roma", "Babilônia", "Alexandria"], "correct": 2 },
            { "question": "Qual evento marcou o início da Idade Média?", "answers": ["A queda do Império Romano do Ocidente", "A descoberta da América", "A invenção da escrita", "A Revolução Francesa"], "correct": 0 },
            { "question": "A Revolução Industrial começou em que país?", "answers": ["França", "Alemanha", "Estados Unidos", "Inglaterra"], "correct": 3 },
            { "question": "Qual filósofo grego foi o mestre de Platão?", "answers": ["Aristóteles", "Sócrates", "Heráclito", "Pitágoras"], "correct": 1 },
            { "question": "O que foi o 'Dia D' durante a Segunda Guerra Mundial?", "answers": ["O ataque a Pearl Harbor", "O desembarque dos Aliados na Normandia", "A rendição da Alemanha", "O lançamento da bomba atômica"], "correct": 1 },
            { "question": "Qual civilização pré-colombiana é famosa por seu avançado conhecimento em astronomia e matemática, incluindo o conceito do zero?", "answers": ["Incas", "Astecas", "Maias", "Olmedas"], "correct": 2 },
            { "question": "Quem foi o primeiro presidente dos Estados Unidos?", "answers": ["Thomas Jefferson", "Abraham Lincoln", "George Washington", "John Adams"], "correct": 2 },
            { "question": "A 'Rota da Seda' era uma rede de rotas comerciais que ligava a China a qual continente?", "answers": ["África", "América", "Europa", "Oceania"], "correct": 2 },
            { "question": "Qual era o principal objetivo das Cruzadas medievais?", "answers": ["Estabelecer novas rotas comerciais", "Conquistar Jerusalém e a Terra Santa", "Espalhar o budismo pela Europa", "Explorar o continente americano"], "correct": 1 },
            { "question": "Qual evento em 1929 desencadeou a 'Grande Depressão'?", "answers": ["O início da Primeira Guerra Mundial", "A quebra da Bolsa de Valores de Nova Iorque", "A Revolução Russa", "A assinatura do Tratado de Versalhes"], "correct": 1 },
            { "question": "O que foi a 'Primavera Árabe'?", "answers": ["Um festival cultural no Oriente Médio", "Uma série de protestos e revoluções populares no mundo árabe", "A descoberta de petróleo na Arábia Saudita", "Um acordo de paz entre Israel e Palestina"], "correct": 1 },
            { "question": "Qual faraó egípcio é famoso por seu túmulo intacto, descoberto em 1922?", "answers": ["Ramsés II", "Cleópatra", "Quéops", "Tutancâmon"], "correct": 3 },
            { "question": "Quem foi o líder da União Soviética durante a Segunda Guerra Mundial?", "answers": ["Lenin", "Trotsky", "Stalin", "Khrushchev"], "correct": 2 },
            { "question": "O que estabeleceu a Magna Carta, assinada na Inglaterra em 1215?", "answers": ["A abolição da monarquia", "A independência das colônias americanas", "Limites ao poder do rei e garantia de direitos", "A criação da Igreja Anglicana"], "correct": 2 },
            { "question": "Em que país ocorreu a Revolução dos Cravos, um golpe militar pacífico em 1974?", "answers": ["Espanha", "Grécia", "Itália", "Portugal"], "correct": 3 },
            { "question": "Qual império governou a Índia antes da colonização britânica?", "answers": ["Império Otomano", "Império Mongol", "Império Gupta", "Império Mogol"], "correct": 3 }
        ]
    },
    "eventos_historicos": {
        "name": "Eventos Históricos",
        "questions": [
            { "question": "A Tomada da Bastilha é o evento que marca o início de qual revolução?", "answers": ["Revolução Russa", "Revolução Americana", "Revolução Francesa", "Revolução Industrial"], "correct": 2 },
            { "question": "A Viagem de Colombo ao 'Novo Mundo' ocorreu em que ano?", "answers": ["1492", "1500", "1488", "1607"], "correct": 0 },
            { "question": "Qual evento deu início à Primeira Guerra Mundial em 1914?", "answers": ["A invasão da Polônia", "O assassinato do arquiduque Francisco Ferdinando", "O afundamento do Lusitania", "A Revolução Russa"], "correct": 1 },
            { "question": "O fim do Apartheid na África do Sul levou à eleição de qual líder em 1994?", "answers": ["Desmond Tutu", "F.W. de Klerk", "Thabo Mbeki", "Nelson Mandela"], "correct": 3 },
            { "question": "Qual foi o propósito da Conferência de Berlim de 1884-1885?", "answers": ["Discutir o fim da escravidão", "Dividir o continente africano entre as potências europeias", "Criar a Liga das Nações", "Estabelecer a paz após a Primeira Guerra Mundial"], "correct": 1 },
            { "question": "A Crise dos Mísseis de Cuba foi um confronto tenso entre quais dois países?", "answers": ["Cuba e Brasil", "Estados Unidos e Cuba", "União Soviética e China", "Estados Unidos e União Soviética"], "correct": 3 },
            { "question": "O Renascimento foi um movimento cultural que floresceu em qual período histórico?", "answers": ["Idade Antiga", "Idade Média", "Idade Moderna", "Idade Contemporânea"], "correct": 2 },
            { "question": "Qual evento de 1969 é considerado um marco na Corrida Espacial?", "answers": ["O lançamento do Sputnik 1", "O primeiro homem no espaço", "A chegada do homem à Lua", "A criação da NASA"], "correct": 2 },
            { "question": "A Batalha de Stalingrado foi um ponto de virada crucial em qual guerra?", "answers": ["Primeira Guerra Mundial", "Guerra do Vietnã", "Segunda Guerra Mundial", "Guerra da Coreia"], "correct": 2 },
            { "question": "O que foi a 'Noite dos Cristais' na Alemanha Nazista?", "answers": ["Uma celebração de Ano Novo", "Um ataque violento contra judeus e suas propriedades", "A assinatura de um acordo de paz", "A inauguração de uma catedral de cristal"], "correct": 1 },
            { "question": "Qual foi o resultado da Batalha de Waterloo em 1815?", "answers": ["A vitória de Napoleão Bonaparte", "A derrota final de Napoleão Bonaparte", "O início da Revolução Francesa", "A unificação da Alemanha"], "correct": 1 },
            { "question": "O que foi a 'Marcha para o Oeste' nos Estados Unidos?", "answers": ["Uma peregrinação religiosa", "A expansão territorial em direção ao Oceano Pacífico", "Uma campanha militar na Europa", "Uma corrida do ouro na África"], "correct": 1 },
            { "question": "A construção do Muro de Berlim em 1961 teve como objetivo:", "answers": ["Proteger a cidade de inundações", "Celebrar a unificação da Alemanha", "Impedir a fuga de pessoas da Alemanha Oriental para a Ocidental", "Criar uma barreira comercial"], "correct": 2 },
            { "question": "A Peste Negra, que devastou a Europa no século XIV, foi causada por:", "answers": ["Um vírus transportado pelo ar", "Uma bactéria transmitida por pulgas de ratos", "Contaminação da água", "Uma maldição divina"], "correct": 1 },
            { "question": "O Massacre da Praça da Paz Celestial em 1989 ocorreu em qual país?", "answers": ["Japão", "Coreia do Sul", "China", "Vietnã"], "correct": 2 },
            { "question": "A Declaração de Independência dos Estados Unidos foi adotada em qual ano?", "answers": ["1776", "1789", "1812", "1865"], "correct": 0 },
            { "question": "O que foi o Iluminismo?", "answers": ["Um movimento artístico", "Um período de descobertas científicas", "Um movimento intelectual que valorizava a razão", "Uma seita religiosa"], "correct": 2 },
            { "question": "Qual foi a causa da Guerra do Vietnã?", "answers": ["Disputas por petróleo", "Conflitos religiosos", "A divisão do país entre regimes comunista e não-comunista", "Uma invasão da China"], "correct": 2 },
            { "question": "O que foi a 'Caça às Bruxas de Salém'?", "answers": ["Um festival de Halloween", "Um período de histeria e perseguição por bruxaria na América colonial", "Uma peça de teatro famosa", "Uma batalha histórica"], "correct": 1 },
            { "question": "O Tratado de Versalhes, assinado em 1919, impôs duras sanções a qual país?", "answers": ["Rússia", "França", "Reino Unido", "Alemanha"], "correct": 3 }
        ]
    },
    "figuras_historicas": {
        "name": "Figuras Históricas",
        "questions": [
            { "question": "Quem foi a rainha egípcia famosa por sua aliança com os líderes romanos Júlio César e Marco Antônio?", "answers": ["Nefertiti", "Hatshepsut", "Cleópatra", "Nefertari"], "correct": 2 },
            { "question": "Qual líder macedônio conquistou um vasto império que se estendia da Grécia à Índia?", "answers": ["Leônidas", "Péricles", "Alexandre, o Grande", "Filipe II"], "correct": 2 },
            { "question": "Quem foi o líder do movimento pelos direitos civis nos Estados Unidos, conhecido pelo discurso 'Eu Tenho um Sonho'?", "answers": ["Malcolm X", "Martin Luther King Jr.", "Rosa Parks", "Nelson Mandela"], "correct": 1 },
            { "question": "Qual cientista desenvolveu a teoria da evolução através da seleção natural?", "answers": ["Gregor Mendel", "Louis Pasteur", "Isaac Newton", "Charles Darwin"], "correct": 3 },
            { "question": "Quem foi a heroína francesa que liderou o exército contra os ingleses durante a Guerra dos Cem Anos?", "answers": ["Maria Antonieta", "Catarina de Médici", "Joana d'Arc", "Eleonor da Aquitânia"], "correct": 2 },
            { "question": "Qual imperador romano é famoso por, supostamente, ter incendiado Roma?", "answers": ["Augusto", "Calígula", "Nero", "Trajano"], "correct": 2 },
            { "question": "Quem foi o primeiro homem a circum-navegar a Terra?", "answers": ["Cristóvão Colombo", "Vasco da Gama", "Fernão de Magalhães", "James Cook"], "correct": 2 },
            { "question": "Qual artista e inventor do Renascimento é famoso por pintar a 'Mona Lisa' e 'A Última Ceia'?", "answers": ["Michelangelo", "Rafael", "Donatello", "Leonardo da Vinci"], "correct": 3 },
            { "question": "Quem foi o líder da Revolução Bolchevique na Rússia em 1917?", "answers": ["Stalin", "Trotsky", "Lenin", "Czar Nicolau II"], "correct": 2 },
            { "question": "Qual imperador francês tentou conquistar a Europa no início do século XIX?", "answers": ["Luís XIV", "Carlos Magno", "Napoleão Bonaparte", "Luís XVI"], "correct": 2 },
            { "question": "Quem foi a física e química polonesa-francesa que foi pioneira na pesquisa sobre radioatividade?", "answers": ["Rosalind Franklin", "Marie Curie", "Lise Meitner", "Dorothy Hodgkin"], "correct": 1 },
            { "question": "Qual líder indiano usou a desobediência civil não-violenta para alcançar a independência da Índia?", "answers": ["Jawaharlal Nehru", "Subhas Chandra Bose", "Mahatma Gandhi", "Indira Gandhi"], "correct": 2 },
            { "question": "Quem foi o dramaturgo inglês autor de obras como 'Hamlet' e 'Romeu e Julieta'?", "answers": ["Christopher Marlowe", "Ben Jonson", "William Shakespeare", "John Milton"], "correct": 2 },
            { "question": "Qual líder mongol criou o maior império contíguo da história?", "answers": ["Kublai Khan", "Gengis Khan", "Tamerlão", "Átila, o Huno"], "correct": 1 },
            { "question": "Quem foi o presidente americano que aboliu a escravidão?", "answers": ["George Washington", "Thomas Jefferson", "Abraham Lincoln", "Theodore Roosevelt"], "correct": 2 },
            { "question": "Qual filósofo alemão é o autor de 'O Capital' e um dos fundadores do comunismo?", "answers": ["Friedrich Nietzsche", "Immanuel Kant", "Karl Marx", "Georg Hegel"], "correct": 2 },
            { "question": "Quem foi o líder sul-africano que lutou contra o Apartheid e se tornou o primeiro presidente negro do país?", "answers": ["Desmond Tutu", "Steve Biko", "Nelson Mandela", "Thabo Mbeki"], "correct": 2 },
            { "question": "Qual rainha da Inglaterra reinou por 63 anos, um período conhecido como a 'Era Vitoriana'?", "answers": ["Elizabeth I", "Maria I", "Rainha Vitória", "Elizabeth II"], "correct": 2 },
            { "question": "Quem foi o líder da Reforma Protestante na Alemanha?", "answers": ["João Calvino", "Henrique VIII", "Martinho Lutero", "Ulrico Zuínglio"], "correct": 2 },
            { "question": "Qual revolucionário argentino foi uma figura chave na Revolução Cubana ao lado de Fidel Castro?", "answers": ["Camilo Cienfuegos", "Ernesto 'Che' Guevara", "Hugo Chávez", "Augusto Pinochet"], "correct": 1 }
        ]
    },
    "ditados_populares": {
        "name": "Ditados Populares",
        "questions": [
            { "question": "Complete o ditado: 'Água mole em pedra dura...'", "answers": ["...tanto fura que seca.", "...tanto bate até que fura.", "...molha a planta toda.", "...desce o rio inteiro."], "correct": 1 },
            { "question": "O que significa o ditado 'Quem tem boca vai a Roma'?", "answers": ["Que falando se consegue chegar a qualquer lugar.", "Que é preciso gritar para ser ouvido.", "Que Roma é um destino fácil de encontrar.", "Que pessoas falantes viajam muito."], "correct": 0 },
            { "question": "Complete o ditado: 'Em terra de cego...'", "answers": ["...todo mundo tropeça.", "...quem tem um olho é rei.", "...a escuridão é normal.", "...se vende óculos escuros."], "correct": 1 },
            { "question": "O que significa a expressão 'Matar dois coelhos com uma cajadada só'?", "answers": ["Ser cruel com os animais.", "Resolver dois problemas de uma só vez.", "Ser muito forte.", "Ter uma pontaria excelente."], "correct": 1 },
            { "question": "Complete o ditado: 'De grão em grão...'", "answers": ["...a galinha enche o papo.", "...se faz um grande deserto.", "...o celeiro fica vazio.", "...o pássaro voa alto."], "correct": 0 },
            { "question": "O que significa o ditado 'Cavalo dado não se olha os dentes'?", "answers": ["Que cavalos velhos não servem para presente.", "Que é preciso verificar a idade de um cavalo.", "Que não se deve questionar a qualidade de um presente.", "Que presentes de animais são complicados."], "correct": 2 },
            { "question": "Complete o ditado: 'Mais vale um pássaro na mão...'", "answers": ["...do que o céu inteiro.", "...do que a gaiola vazia.", "...do que dois voando.", "...do que um gato no telhado."], "correct": 2 },
            { "question": "O que significa a expressão 'A curiosidade matou o gato'?", "answers": ["Que gatos são animais frágeis.", "Que ser curioso pode levar a situações perigosas.", "Que não se deve ter gatos de estimação.", "Que gatos não gostam de pessoas curiosas."], "correct": 1 },
            { "question": "Complete o ditado: 'Casa de ferreiro...'", "answers": ["...espeto de pau.", "...sempre tem fogo.", "...tem muito barulho.", "...a porta é de ferro."], "correct": 0 },
            { "question": "O que significa o ditado 'Pimenta nos olhos dos outros é refresco'?", "answers": ["Que pimenta faz bem para os olhos.", "Que é fácil subestimar o sofrimento alheio.", "Que as pessoas gostam de ver os outros sofrerem.", "Que pimenta e refresco combinam."], "correct": 1 },
            { "question": "Complete o ditado: 'Quem não tem cão...'", "answers": ["...compra um gato.", "...vive sozinho.", "...caça com gato.", "...não gosta de animais."], "correct": 2 },
            { "question": "O que significa a expressão 'Salvar pela pele'?", "answers": ["Fazer um tratamento de pele.", "Escapar de uma situação difícil por muito pouco.", "Comprar roupas de couro.", "Ficar muito bronzeado."], "correct": 1 },
            { "question": "Complete o ditado: 'Filho de peixe...'", "answers": ["...sabe nadar.", "...gosta de água.", "...peixinho é.", "...mora no mar."], "correct": 2 },
            { "question": "O que significa o ditado 'Uma andorinha só não faz verão'?", "answers": ["Que o verão precisa de muitos pássaros.", "Que uma pessoa sozinha não consegue mudar uma situação.", "Que andorinhas não gostam do calor.", "Que o verão é uma estação solitária."], "correct": 1 },
            { "question": "Complete o ditado: 'A pressa é...'", "answers": ["...amiga da perfeição.", "...a virtude dos fortes.", "...inimiga da perfeição.", "...o tempero da vida."], "correct": 2 },
            { "question": "O que significa a expressão 'Tirar o cavalinho da chuva'?", "answers": ["Guardar um brinquedo.", "Desistir de uma ideia ou plano.", "Proteger um animal da chuva.", "Dar um banho no cavalo."], "correct": 1 },
            { "question": "Complete o ditado: 'Para bom entendedor...'", "answers": ["...a palavra basta.", "...meia palavra basta.", "...o silêncio é ouro.", "...toda frase é um livro."], "correct": 1 },
            { "question": "O que significa 'Não chore sobre o leite derramado'?", "answers": ["Que não adianta se lamentar por algo que já aconteceu.", "Que leite derramado faz muita sujeira.", "Que é preciso ter cuidado ao servir leite.", "Que chorar não limpa a bagunça."], "correct": 0 },
            { "question": "Complete o ditado: 'Onde há fumaça...'", "answers": ["...há churrasco.", "...o céu está cinzento.", "...há fogo.", "...alguém está a cozinhar."], "correct": 2 },
            { "question": "O que significa o ditado 'Cada macaco no seu galho'?", "answers": ["Que macacos são organizados.", "Que cada um deve cuidar da sua própria vida.", "Que os zoológicos devem separar os macacos.", "Que não se deve misturar diferentes espécies."], "correct": 1 }
        ]
    },
    "doencas": {
        "name": "Doenças",
        "questions": [
            { "question": "Qual doença é causada pela falta de insulina ou pela incapacidade do corpo de usá-la eficientemente?", "answers": ["Hipertensão", "Asma", "Diabetes", "Anemia"], "correct": 2 },
            { "question": "A malária é uma doença transmitida pela picada de qual inseto?", "answers": ["Mosca Tsé-tsé", "Barbeiro", "Mosquito Aedes aegypti", "Mosquito Anopheles"], "correct": 3 },
            { "question": "Qual vitamina é essencial para prevenir o escorbuto?", "answers": ["Vitamina A", "Vitamina B12", "Vitamina C", "Vitamina D"], "correct": 2 },
            { "question": "A tuberculose afeta principalmente qual órgão do corpo humano?", "answers": ["Fígado", "Coração", "Pulmões", "Rins"], "correct": 2 },
            { "question": "Qual destas doenças é viral e pode ser prevenida com a vacina tríplice viral (SRC)?", "answers": ["Catapora", "Sarampo", "Coqueluche", "Difteria"], "correct": 1 },
            { "question": "A febre amarela é uma doença causada por:", "answers": ["Bactéria", "Fungo", "Vírus", "Protozoário"], "correct": 2 },
            { "question": "Qual doença autoimune crônica pode causar inflamação nas articulações, pele, rins e cérebro?", "answers": ["Lúpus", "Fibromialgia", "Esclerose Múltipla", "Artrite Reumatoide"], "correct": 0 },
            { "question": "A Doença de Chagas é causada por um protozoário transmitido por qual inseto?", "answers": ["Carrapato", "Pulga", "Barbeiro", "Piolho"], "correct": 2 },
            { "question": "O que é a hipertensão?", "answers": ["Nível baixo de açúcar no sangue", "Pressão arterial elevada", "Nível alto de colesterol", "Frequência cardíaca lenta"], "correct": 1 },
            { "question": "A AIDS (Síndrome da Imunodeficiência Adquirida) é causada por qual vírus?", "answers": ["HPV", "HIV", "Influenza", "Ebola"], "correct": 1 },
            { "question": "Qual é a principal forma de transmissão da dengue?", "answers": ["Contato com sangue infectado", "Picada do mosquito Aedes aegypti", "Ingestão de água contaminada", "Pelo ar, através da tosse ou espirro"], "correct": 1 },
            { "question": "A Peste Bubônica, ou Peste Negra, que assolou a Europa na Idade Média, é causada por qual tipo de microrganismo?", "answers": ["Vírus", "Bactéria", "Protozoário", "Fungo"], "correct": 1 },
            { "question": "Qual destas condições é caracterizada pela perda progressiva da memória e outras funções cognitivas?", "answers": ["Mal de Parkinson", "Esquizofrenia", "Doença de Alzheimer", "Epilepsia"], "correct": 2 },
            { "question": "A varíola foi a primeira doença infecciosa a ser erradicada do mundo através de qual medida?", "answers": ["Saneamento básico", "Uso de antibióticos", "Vacinação em massa", "Isolamento social"], "correct": 2 },
            { "question": "Qual é o nome da doença inflamatória crônica das vias aéreas que causa dificuldade para respirar?", "answers": ["Bronquite", "Pneumonia", "Asma", "Enfisema"], "correct": 2 },
            { "question": "A anemia falciforme é uma doença:", "answers": ["Infecciosa", "Causada por má nutrição", "Genética e hereditária", "Autoimune"], "correct": 2 },
            { "question": "Qual doença é causada por uma bactéria chamada 'Vibrio cholerae' e geralmente transmitida por água ou alimentos contaminados?", "answers": ["Febre Tifoide", "Cólera", "Leptospirose", "Hepatite A"], "correct": 1 },
            { "question": "A poliomielite, também conhecida como paralisia infantil, é uma doença viral que afeta principalmente:", "answers": ["O sistema digestivo", "O sistema nervoso", "O sistema respiratório", "O sistema circulatório"], "correct": 1 },
            { "question": "O que é um AVC (Acidente Vascular Cerebral)?", "answers": ["Um ataque cardíaco", "Uma convulsão", "Uma interrupção do fluxo sanguíneo para o cérebro", "Um coágulo de sangue na perna"], "correct": 2 },
            { "question": "Qual é a doença caracterizada pelo crescimento descontrolado de células anormais no corpo?", "answers": ["Diabetes", "Câncer", "Arteriosclerose", "Cirrose"], "correct": 1 }
        ]
    },
  "mitologia_hard_1": {
    "name": "Mitologias do Mundo (Hard - Parte 1)",
    "questions": [
      { "question": "Na mitologia suméria, quem foi o herói da Epopeia de Gilgamesh que sobreviveu ao dilúvio?", "answers": ["Enkidu", "Utnapishtim", "Humbaba", "Lugalbanda"], "correct": 1 },
      { "question": "Qual criatura da mitologia irlandesa era um espírito feminino cujo grito agourento anunciava uma morte na família?", "answers": ["Leprechaun", "Banshee", "Pooka", "Dullahan"], "correct": 1 },
      { "question": "No panteão egípcio, qual deusa era a personificação da escrita, da sabedoria e da medição?", "answers": ["Seshat", "Mafdet", "Taweret", "Meretseger"], "correct": 0 },
      { "question": "Segundo a mitologia asteca, de qual deus Huitzilopochtli era irmão e rival, representando a noite e a feitiçaria?", "answers": ["Quetzalcoatl", "Tlaloc", "Xipe Totec", "Tezcatlipoca"], "correct": 3 },
      { "question": "Na mitologia hindu, qual é o nome do arco divino de Vishnu, que foi empunhado por Rama para vencer um torneio?", "answers": ["Pinaka", "Vijaya", "Sharanga", "Gandiva"], "correct": 2 },
      { "question": "Quem era o deus do trovão e da guerra na mitologia eslava, cujo símbolo era o machado?", "answers": ["Svarog", "Veles", "Perun", "Dazhbog"], "correct": 2 },
      { "question": "Na mitologia japonesa, como se chama o 'kami' (deus) do mar e das tempestades, irmão de Amaterasu?", "answers": ["Tsukuyomi", "Susanoo", "Inari", "Hachiman"], "correct": 1 },
      { "question": "Na mitologia grega, quem foi o arquiteto do Labirinto de Creta?", "answers": ["Ícaro", "Teseu", "Dédalo", "Minos"], "correct": 2 },
      { "question": "Qual dos Quatro Cavaleiros do Apocalipse, segundo a tradição cristã, monta um cavalo vermelho e tem o poder de tirar a paz da Terra?", "answers": ["Peste", "Guerra", "Fome", "Morte"], "correct": 1 },
      { "question": "Na mitologia Tupi-Guarani, quem é a deusa da lua e protetora das plantas e dos amantes?", "answers": ["Iara", "Jaci", "Ceuci", "Anhangá"], "correct": 1 },
      { "question": "Qual é o nome do cavalo de oito patas de Odin na mitologia nórdica?", "answers": ["Gullinbursti", "Hrimfaxi", "Sleipnir", "Skinfaxi"], "correct": 2 },
      { "question": "Na mitologia romana, quem eram os Lares?", "answers": ["Espíritos malignos do submundo", "Deuses das colheitas", "Divindades protetoras do lar e da família", "Sacerdotes do templo de Júpiter"], "correct": 2 },
      { "question": "Qual herói da mitologia persa, do épico 'Shahnameh', matou o dragão Aži Dahāka?", "answers": ["Rostam", "Zal", "Fereydun", "Siyavash"], "correct": 2 },
      { "question": "Qual titânide grega era a deusa da memória e mãe das Musas?", "answers": ["Tétis", "Febe", "Reia", "Mnemosine"], "correct": 3 },
      { "question": "Na mitologia chinesa, Nezha é uma divindade protetora conhecida por lutar usando qual arma nos pés?", "answers": ["Botas de Vento e Fogo", "Sandálias de Sete Léguas", "Patins de Jade", "Rodas de Vento e Fogo"], "correct": 3 },
      { "question": "No zoroastrismo, quem é a divindade suprema, o criador não criado, representando a luz e a sabedoria?", "answers": ["Angra Mainyu", "Ahura Mazda", "Mithra", "Zaratustra"], "correct": 1 },
      { "question": "Qual gigante de fogo da mitologia nórdica, governante de Muspelheim, terá um papel crucial no Ragnarok?", "answers": ["Ymir", "Thrym", "Surtr", "Hrym"], "correct": 2 },
      { "question": "Na mitologia fenícia, qual deus da ressurreição passava parte do ano no submundo, simbolizando o ciclo das estações?", "answers": ["Baal", "Melqart", "Adonis", "El"], "correct": 2 },
      { "question": "Qual figura do folclore eslavo é uma velha bruxa que vive em uma casa com pés de galinha?", "answers": ["Rusalka", "Alkonost", "Baba Yaga", "Gamayun"], "correct": 2 },
      { "question": "Na mitologia dos iorubás, quem é o 'orixá' da justiça, dos raios e do trovão?", "answers": ["Ogum", "Oxóssi", "Exu", "Xangô"], "correct": 3 }
    ]
  },
  "mitologia_hard_2": {
    "name": "Mitologias do Mundo (Hard - Parte 2)",
    "questions": [
      { "question": "Qual é o nome da espada de Freyr, uma arma mágica que podia lutar sozinha, na mitologia nórdica?", "answers": ["Tyrfing", "Gram", "Sumarbrander", "Dáinsleif"], "correct": 2 },
      { "question": "No Livro dos Mortos egípcio, o coração do falecido era pesado contra qual objeto para determinar seu destino?", "answers": ["Um Ankh", "Um Escaravelho", "A Pena de Ma'at", "O Olho de Hórus"], "correct": 2 },
      { "question": "Na mitologia grega, quem foi o único dos Sete Contra Tebas a sobreviver ao assalto à cidade?", "answers": ["Tideu", "Capaneu", "Partenopeu", "Adrasto"], "correct": 3 },
      { "question": "Na mitologia hindu, o deus Ganesha tem a cabeça de qual animal?", "answers": ["Macaco", "Elefante", "Tigre", "Boi"], "correct": 1 },
      { "question": "Qual o nome do cão de três cabeças que guardava a entrada de Mictlan, o submundo asteca?", "answers": ["Cipactli", "Xolotl", "Huehuecoyotl", "Mictlantecuhtli"], "correct": 1 },
      { "question": "Na mitologia celta, qual era o nome do 'Outro Mundo', a terra da eterna juventude e do prazer?", "answers": ["Avalon", "Tir na nÓg", "Camelot", "Lyonesse"], "correct": 1 },
      { "question": "Qual o nome da lança de Lugh, uma das Quatro Jóias dos Tuatha Dé Danann, que era insaciável por batalha?", "answers": ["Gáe Bulg", "Caladbolg", "Areadbhar", "Fragarach"], "correct": 2 },
      { "question": "No épico finlandês 'Kalevala', qual é o nome do artefato mágico, um moinho que produzia sal, grãos e ouro, forjado por Ilmarinen?", "answers": ["Sampo", "Kantele", "Otso", "Väinämöinen"], "correct": 0 },
      { "question": "Na mitologia romana, quem era a deusa do esgoto e da purificação, celebrada no festival da Cloacina?", "answers": ["Vênus Cloacina", "Cardea", "Carmenta", "Laverna"], "correct": 0 },
      { "question": "Qual o nome do demônio do vento do sudoeste na mitologia mesopotâmica, frequentemente representado com cabeça de leão e asas de águia?", "answers": ["Lamashtu", "Pazuzu", "Asakku", "Humbaba"], "correct": 1 },
      { "question": "Na mitologia grega, quem eram as Erínias (Fúrias), as personificações da vingança?", "answers": ["As Moiras", "As Graças", "As Górgonas", "As Eumênides"], "correct": 3 },
      { "question": "Qual dos trabalhos de Hércules envolvia limpar os estábulos de Áugias em um único dia?", "answers": ["Quinto Trabalho", "Sexto Trabalho", "Sétimo Trabalho", "Oitavo Trabalho"], "correct": 0 },
      { "question": "Na cosmologia inca, como era chamado o 'mundo superior' ou o reino dos deuses?", "answers": ["Kay Pacha", "Uku Pacha", "Hanan Pacha", "Inti Pacha"], "correct": 2 },
      { "question": "Na mitologia do voodoo haitiano, quem é o 'loa' (espírito) que se situa na encruzilhada entre os mundos e abre a comunicação?", "answers": ["Baron Samedi", "Papa Legba", "Damballa", "Erzulie"], "correct": 1 },
      { "question": "Qual o nome do caldeirão mágico do deus galês Brân, o Abençoado, que podia ressuscitar os mortos?", "answers": ["Caldeirão de Dagda", "Caldeirão de Cerridwen", "Caldeirão do Renascimento", "Caldeirão de Gundestrup"], "correct": 2 },
      { "question": "Na demonologia islâmica, quem é o líder dos 'jinn' (gênios), que se recusou a se curvar perante Adão?", "answers": ["Marid", "Ifrit", "Iblis", "Ghul"], "correct": 2 },
      { "question": "Qual herói grego usou um escudo polido como espelho para derrotar a górgona Medusa?", "answers": ["Hércules", "Teseu", "Perseu", "Belerofonte"], "correct": 2 },
      { "question": "Na mitologia aborígene australiana, o que é o 'Tempo do Sonho' (Dreamtime)?", "answers": ["Um festival anual de colheita", "O período sagrado da criação do mundo", "Um estado de transe dos xamãs", "A noite em que as estrelas são mais brilhantes"], "correct": 1 },
      { "question": "Qual o nome do primeiro homem na mitologia persa, análogo a Adão?", "answers": ["Yima", "Gayomartan", "Jamshid", "Zahhak"], "correct": 1 },
      { "question": "Na mitologia grega, Sísifo foi condenado a qual castigo eterno no Tártaro?", "answers": ["Ter seu fígado comido por uma águia", "Ficar eternamente com fome e sede", "Empurrar uma rocha montanha acima", "Tentar encher uma jarra furada"], "correct": 2 }
    ]
  },
  "o_intruso_hard_1": {
    "name": "O Intruso (Hard - Parte 1)",
    "questions": [
      { "question": "Qual destes filósofos é o 'intruso' no grupo de existencialistas?", "answers": ["Jean-Paul Sartre", "Albert Camus", "Søren Kierkegaard", "Baruch Spinoza"], "correct": 3 },
      { "question": "Qual destas batalhas é a 'intrusa' por não ter ocorrido durante as Guerras Napoleônicas?", "answers": ["Batalha de Waterloo", "Batalha de Austerlitz", "Batalha de Trafalgar", "Batalha de Gettysburg"], "correct": 3 },
      { "question": "Qual destes elementos químicos é o 'intruso' no grupo dos gases nobres?", "answers": ["Hélio", "Neônio", "Hidrogênio", "Argônio"], "correct": 2 },
      { "question": "Qual destes diretores de cinema é o 'intruso' no grupo de cineastas italianos?", "answers": ["Federico Fellini", "Luchino Visconti", "Ingmar Bergman", "Vittorio De Sica"], "correct": 2 },
      { "question": "Qual destas obras é a 'intrusa' por não pertencer ao período do Renascimento?", "answers": ["Mona Lisa", "A Criação de Adão", "O Nascimento de Vênus", "A Liberdade Guiando o Povo"], "correct": 3 },
      { "question": "Qual destes instrumentos é o 'intruso' em uma orquestra sinfônica tradicional?", "answers": ["Violino", "Fagote", "Saxofone", "Trompa"], "correct": 2 },
      { "question": "Qual destes impérios é o 'intruso' por não ter sua principal base territorial na Mesopotâmia?", "answers": ["Império Acádio", "Império Babilônico", "Império Assírio", "Império Aquemênida"], "correct": 3 },
      { "question": "Qual destes ossos é o 'intruso' por não fazer parte do braço humano (úmero, rádio, ulna)?", "answers": ["Úmero", "Rádio", "Tíbia", "Ulna"], "correct": 2 },
      { "question": "Qual destes planetas é o 'intruso' no grupo dos 'gigantes gasosos'?", "answers": ["Júpiter", "Saturno", "Marte", "Netuno"], "correct": 2 },
      { "question": "Qual destes marcos arquitetônicos é o 'intruso' por não ser um Patrimônio Mundial da UNESCO?", "answers": ["Grande Muralha da China", "Torre Eiffel", "Machu Picchu", "Pirâmides de Gizé"], "correct": 1 },
      { "question": "Qual destes prêmios Nobel é o 'intruso' por não ter sido um dos prêmios originais estabelecidos por Alfred Nobel?", "answers": ["Paz", "Física", "Química", "Economia"], "correct": 3 },
      { "question": "Qual destes animais é o 'intruso' no grupo de mamíferos marinhos?", "answers": ["Baleia Azul", "Golfinho", "Tubarão-Branco", "Peixe-boi"], "correct": 2 },
      { "question": "Qual destas linguagens de programação é a 'intrusa' por ser primariamente de marcação e não de programação?", "answers": ["Python", "JavaScript", "HTML", "Java"], "correct": 2 },
      { "question": "Qual destes compostos é o 'intruso' por ser uma base e não um ácido?", "answers": ["Ácido Sulfúrico (H₂SO₄)", "Hidróxido de Sódio (NaOH)", "Ácido Clorídrico (HCl)", "Ácido Nítrico (HNO₃)"], "correct": 1 },
      { "question": "Qual destas capitais é a 'intrusa' por não ser banhada pelo Rio Danúbio?", "answers": ["Viena", "Budapeste", "Belgrado", "Praga"], "correct": 3 },
      { "question": "Qual destas peças de Shakespeare é a 'intrusa' por ser uma comédia e não uma tragédia?", "answers": ["Hamlet", "Romeu e Julieta", "Sonho de uma Noite de Verão", "Macbeth"], "correct": 2 },
      { "question": "Qual destas revoluções é a 'intrusa' por não ter ocorrido no século XVIII?", "answers": ["Revolução Americana", "Revolução Francesa", "Revolução Industrial", "Revolução Russa"], "correct": 3 },
      { "question": "Qual destas células é a 'intrusa' por ser procarionte?", "answers": ["Neurônio", "Hemácia", "Bactéria", "Osteócito"], "correct": 2 },
      { "question": "Qual destes deuses é o 'intruso' no panteão olímpico grego principal?", "answers": ["Zeus", "Ares", "Hades", "Apolo"], "correct": 2 },
      { "question": "Qual destes movimentos artísticos é o 'intruso' por ter surgido no século XX?", "answers": ["Barroco", "Romantismo", "Cubismo", "Neoclassicismo"], "correct": 2 }
    ]
  },
  "o_intruso_hard_2": {
    "name": "O Intruso (Hard - Parte 2)",
    "questions": [
      { "question": "Qual destes autores é o 'intruso' no grupo da 'Geração Perdida' da literatura americana?", "answers": ["Ernest Hemingway", "F. Scott Fitzgerald", "John Steinbeck", "Gertrude Stein"], "correct": 2 },
      { "question": "Qual destas unidades de medida é a 'intrusa' por não pertencer ao Sistema Internacional (SI)?", "answers": ["Metro", "Quilograma", "Segundo", "Polegada"], "correct": 3 },
      { "question": "Qual destes acordos de paz é o 'intruso' por ter encerrado a Primeira Guerra Mundial?", "answers": ["Paz de Vestfália", "Congresso de Viena", "Tratado de Versalhes", "Pactos de Latrão"], "correct": 2 },
      { "question": "Qual destes tipos de nuvens é o 'intruso' por se formar em altitudes muito elevadas (acima de 6.000m)?", "answers": ["Cumulus", "Stratus", "Cirrus", "Nimbostratus"], "correct": 2 },
      { "question": "Qual destes filósofos é o 'intruso' no grupo dos contratualistas?", "answers": ["Thomas Hobbes", "John Locke", "Platão", "Jean-Jacques Rousseau"], "correct": 2 },
      { "question": "Qual destas vitaminas é a 'intrusa' por ser lipossolúvel?", "answers": ["Vitamina C", "Vitamina B12", "Vitamina A", "Vitamina B9 (Ácido Fólico)"], "correct": 2 },
      { "question": "Qual destes filmes é o 'intruso' por não ter sido dirigido por Alfred Hitchcock?", "answers": ["Psicose", "Cidadão Kane", "Um Corpo que Cai", "Os Pássaros"], "correct": 1 },
      { "question": "Qual destes países é o 'intruso' por não fazer parte da Escandinávia?", "answers": ["Dinamarca", "Suécia", "Finlândia", "Noruega"], "correct": 2 },
      { "question": "Qual destas estruturas celulares é a 'intrusa' por estar presente em células vegetais, mas não em animais?", "answers": ["Mitocôndria", "Núcleo", "Membrana Plasmática", "Parede Celular"], "correct": 3 },
      { "question": "Qual destes imperadores é o 'intruso' na Dinastia Nerva-Antonina de Roma?", "answers": ["Trajano", "Adriano", "Marco Aurélio", "Vespasiano"], "correct": 3 },
      { "question": "Qual destas cordilheiras é a 'intrusa' por não fazer parte do 'Círculo de Fogo do Pacífico'?", "answers": ["Andes", "Montanhas Rochosas", "Alpes", "Cordilheiras do Japão"], "correct": 2 },
      { "question": "Qual destas escalas de medição é a 'intrusa' por não ser usada para medir a intensidade de terremotos?", "answers": ["Escala Richter", "Escala de Mercalli", "Escala de Magnitude de Momento", "Escala de Saffir-Simpson"], "correct": 3 },
      { "question": "Qual destas linguagens é a 'intrusa' no grupo de línguas românicas (derivadas do latim)?", "answers": ["Espanhol", "Francês", "Romeno", "Alemão"], "correct": 3 },
      { "question": "Qual destas escolas de pensamento econômico é a 'intrusa'?", "answers": ["Keynesianismo", "Escola Austríaca", "Marxismo", "Estoicismo"], "correct": 3 },
      { "question": "Qual destas partes do cérebro é a 'intrusa' no lobo frontal?", "answers": ["Córtex pré-frontal", "Área de Broca", "Córtex motor primário", "Córtex visual primário"], "correct": 3 },
      { "question": "Qual destes escritores é o 'intruso' no movimento modernista brasileiro?", "answers": ["Oswald de Andrade", "Mário de Andrade", "Manuel Bandeira", "José de Alencar"], "correct": 3 },
      { "question": "Qual destes oceanos é o 'intruso' por não banhar a costa da Ásia?", "answers": ["Pacífico", "Índico", "Ártico", "Atlântico"], "correct": 3 },
      { "question": "Qual destes princípios da física é o 'intruso' nas Leis de Newton?", "answers": ["Princípio da Inércia", "Princípio Fundamental da Dinâmica", "Princípio da Ação e Reação", "Princípio da Incerteza"], "correct": 3 },
      { "question": "Qual destas obras é a 'intrusa' por ser uma escultura de Michelangelo?", "answers": ["Davi", "Moisés", "Pietà", "O Pensador"], "correct": 3 },
      { "question": "Qual destes países é o 'intruso' no G7 (Grupo dos Sete)?", "answers": ["Canadá", "Japão", "Rússia", "Itália"], "correct": 2 }
    ]
  },
  "qual_veio_primeiro_hard_1": {
    "name": "Qual Veio Primeiro? (Hard - Parte 1)",
    "questions": [
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A publicação de 'A Origem das Espécies' de Darwin", "A Guerra Civil Americana"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A fundação da Universidade de Harvard", "A invenção do cálculo por Newton e Leibniz"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O primeiro voo dos Irmãos Wright", "A inauguração do Canal do Panamá"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção da guilhotina", "A composição de 'Eine kleine Nachtmusik' de Mozart"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A patente do telefone por Alexander Graham Bell", "A Batalha de Little Bighorn"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A construção do Coliseu de Roma", "A erupção do Vesúvio que destruiu Pompeia"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O fim da Dinastia Tudor na Inglaterra", "A fundação de Jamestown, a primeira colônia inglesa permanente na América"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A publicação de 'Frankenstein' de Mary Shelley", "A invenção do estetoscópio"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O lançamento do satélite Sputnik 1", "A criação da Comunidade Econômica Europeia (precursora da UE)"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A morte de Cleópatra", "O nascimento de Jesus Cristo"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção do código Morse", "O início da corrida do ouro na Califórnia"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A primeira apresentação da peça 'Hamlet' de Shakespeare", "A viagem do Mayflower para a América"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A descoberta da penicilina por Fleming", "A quebra da Bolsa de Valores de 1929"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A coroação de Carlos Magno como Imperador", "A fundação da cidade de Bagdá"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A primeira fotografia permanente", "A independência do Brasil"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A publicação do 'Manifesto Comunista'", "A Revolução Francesa de 1848"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção do motor a diesel", "A primeira edição dos Jogos Olímpicos modernos"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A descoberta da estrutura do DNA", "A coroação da Rainha Elizabeth II"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O fim da Ordem dos Cavaleiros Templários", "O início da Peste Negra na Europa"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A viagem de Marco Polo à China", "A fundação do Império Otomano"], "correct": 0 }
    ]
  },
  "qual_veio_primeiro_hard_2": {
    "name": "Qual Veio Primeiro? (Hard - Parte 2)",
    "questions": [
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção da máquina de fax", "A fundação da Itália como um reino unificado"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O nascimento de Platão", "A Guerra do Peloponeso"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A primeira transmissão de rádio transatlântica", "A teoria da relatividade especial de Einstein"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A Batalha de Hastings", "A Primeira Cruzada"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção do submarino movido a energia nuclear", "A descoberta da fissão nuclear"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A publicação de 'O Príncipe' de Maquiavel", "O início da Reforma Protestante por Martinho Lutero"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A construção do Palácio de Versalhes", "A Grande Peste de Londres"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O desenvolvimento da Tabela Periódica por Mendeleiev", "A abertura do Canal de Suez"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O primeiro uso do gás mostarda na guerra", "A entrada dos Estados Unidos na Primeira Guerra Mundial"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A queda de Constantinopla", "A invenção da prensa de Gutenberg"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A fundação da Nintendo (como uma empresa de cartas)", "A construção da Estátua da Liberdade"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A escrita da Magna Carta", "A fundação da Universidade de Oxford"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A criação do personagem Sherlock Holmes", "A invenção da Coca-Cola"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A revolta de Espártaco", "O assassinato de Júlio César"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção do cronômetro marítimo por John Harrison", "A Guerra dos Sete Anos"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A criação do primeiro videogame (Tennis for Two)", "O discurso 'Eu Tenho um Sonho' de Martin Luther King Jr."], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A publicação de 'Os Lusíadas' de Camões", "A Batalha de Lepanto"], "correct": 1 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["O primeiro transplante de coração bem-sucedido", "O festival de Woodstock"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A invenção do garfo de mesa na Itália", "O fim da Idade Média"], "correct": 0 },
      { "question": "Qual destes eventos veio primeiro?", "answers": ["A descoberta da Antártida", "A Doutrina Monroe"], "correct": 0 }
    ]
  },
  "literatura_hard_1": {
    "name": "Literatura Clássica (Hard - Brasil)",
    "questions": [
      { "question": "Em 'Dom Casmurro', qual era a profissão de Escobar, amigo de Bentinho e suposto pai de Ezequiel?", "answers": ["Advogado", "Comerciante", "Médico", "Militar"], "correct": 1 },
      { "question": "Qual o nome do movimento literário ao qual pertence o poeta Gregório de Matos, conhecido como 'Boca do Inferno'?", "answers": ["Arcadismo", "Barroco", "Quinhentismo", "Romantismo"], "correct": 1 },
      { "question": "Em 'Vidas Secas', de Graciliano Ramos, como se chama a cachorra que é considerada um membro da família?", "answers": ["Vitória", "Sinha", "Baleia", "Fabiana"], "correct": 2 },
      { "question": "Qual autor de 'O Ateneu' é considerado o único representante do Impressionismo na literatura brasileira?", "answers": ["Aluísio Azevedo", "Raul Pompeia", "Adolfo Caminha", "Inglês de Sousa"], "correct": 1 },
      { "question": "Na primeira fase do Modernismo brasileiro, qual era o objetivo do 'Manifesto Antropófago' de Oswald de Andrade?", "answers": ["Copiar os modelos europeus", "Rejeitar toda influência estrangeira", "Devorar a cultura estrangeira e digeri-la de forma brasileira", "Promover a literatura de cordel"], "correct": 2 },
      { "question": "Em 'Macunaíma', de Mário de Andrade, qual é a principal busca do herói após perder seu amuleto?", "answers": ["O amor de Ci", "O tesouro dos Incas", "A muiraquitã", "A imortalidade"], "correct": 2 },
      { "question": "Qual destas obras não faz parte da 'trilogia da alma' de Clarice Lispector?", "answers": ["Perto do Coração Selvagem", "O Lustre", "A Cidade Sitiada", "A Hora da Estrela"], "correct": 3 },
      { "question": "Quem é o autor do poema épico 'Caramuru', uma das obras mais importantes do Arcadismo brasileiro?", "answers": ["Basílio da Gama", "Cláudio Manuel da Costa", "Tomás Antônio Gonzaga", "Santa Rita Durão"], "correct": 3 },
      { "question": "Em 'O Cortiço', de Aluísio Azevedo, quem é o personagem português que é dono da estalagem e representa a ganância capitalista?", "answers": ["Jerônimo", "Miranda", "João Romão", "Botelho"], "correct": 2 },
      { "question": "Qual poeta da segunda geração do Romantismo ficou conhecido como o 'poeta dos escravos'?", "answers": ["Álvares de Azevedo", "Casimiro de Abreu", "Castro Alves", "Fagundes Varela"], "correct": 2 },
      { "question": "Na obra 'Sagarana', de Guimarães Rosa, qual o nome do conto que narra a história do boi que vai para o matadouro?", "answers": ["A hora e vez de Augusto Matraga", "O burrinho pedrês", "Duelo", "Corpo fechado"], "correct": 1 },
      { "question": "Qual o heterônimo de Fernando Pessoa que escreveu o poema 'Tabacaria'?", "answers": ["Alberto Caeiro", "Ricardo Reis", "Bernardo Soares", "Álvaro de Campos"], "correct": 3 },
      { "question": "Qual o movimento literário brasileiro que tinha como lema a 'arte pela arte' e valorizava a forma e o rigor métrico?", "answers": ["Simbolismo", "Parnasianismo", "Realismo", "Naturalismo"], "correct": 1 },
      { "question": "Em 'Grande Sertão: Veredas', Riobaldo narra sua história para qual interlocutor silencioso?", "answers": ["Um doutor", "Um padre", "Um compadre", "Um vaqueiro"], "correct": 0 },
      { "question": "Qual romance de Jorge Amado narra a história de um triângulo amoroso envolvendo Vadinho, Teodoro e a personagem principal?", "answers": ["Gabriela, Cravo e Canela", "Tieta do Agreste", "Dona Flor e Seus Dois Maridos", "Tenda dos Milagres"], "correct": 2 },
      { "question": "Qual poeta simbolista brasileiro é autor de 'Broquéis' e 'Faróis'?", "answers": ["Alphonsus de Guimaraens", "Cruz e Sousa", "Augusto dos Anjos", "Olavo Bilac"], "correct": 1 },
      { "question": "Em 'Memórias de um Sargento de Milícias', de Manuel Antônio de Almeida, qual é o nome do protagonista?", "answers": ["Leonardo (pai)", "Leonardo (filho)", "Major Vidigal", "Tomás"], "correct": 1 },
      { "question": "Qual o nome do personagem principal em 'Triste Fim de Policarpo Quaresma', de Lima Barreto?", "answers": ["Ricardo Coração dos Outros", "Major Quaresma", "Vicente Coleoni", "General Albernaz"], "correct": 1 },
      { "question": "A obra 'Eu', de Augusto dos Anjos, é considerada precursora de qual vanguarda literária no Brasil?", "answers": ["Surrealismo", "Dadaísmo", "Expressionismo", "Futurismo"], "correct": 2 },
      { "question": "Quem escreveu 'A Moreninha', considerado o primeiro romance romântico brasileiro?", "answers": ["José de Alencar", "Machado de Assis", "Joaquim Manuel de Macedo", "Álvares de Azevedo"], "correct": 2 }
    ]
  },
  "literatura_hard_2": {
    "name": "Literatura Clássica (Hard - Mundo)",
    "questions": [
      { "question": "Na 'Odisseia' de Homero, qual o nome do filho de Ulisses que o ajuda a derrotar os pretendentes?", "answers": ["Pátroclo", "Telêmaco", "Astíanax", "Neoptólemo"], "correct": 1 },
      { "question": "Em 'Dom Quixote', de Cervantes, qual o nome do cavalo do protagonista?", "answers": ["Bucéfalo", "Babieca", "Pegasus", "Rocinante"], "correct": 3 },
      { "question": "Qual o nome do primeiro círculo do 'Inferno' na 'Divina Comédia' de Dante, onde residem os pagãos virtuosos?", "answers": ["Vale dos Ventos", "Limbo", "Círculo da Gula", "Pântano Estige"], "correct": 1 },
      { "question": "Em 'Crime e Castigo', de Dostoiévski, qual o nome da personagem que Raskólnikov assassina com um machado?", "answers": ["Sônia Marmeládova", "Pulkhéria Raskólnikova", "Aliona Ivánovna", "Dúnia Raskólnikova"], "correct": 2 },
      { "question": "Qual o nome do capitão do navio baleeiro 'Pequod' no romance 'Moby Dick'?", "answers": ["Starbuck", "Ismael", "Ahab", "Queequeg"], "correct": 2 },
      { "question": "Qual poeta inglês é autor do poema épico 'Paraíso Perdido'?", "answers": ["William Shakespeare", "John Milton", "Geoffrey Chaucer", "John Donne"], "correct": 1 },
      { "question": "No romance 'O Apanhador no Campo de Centeio', qual é o nome do protagonista adolescente?", "answers": ["Jay Gatsby", "Atticus Finch", "Holden Caulfield", "Philip Marlowe"], "correct": 2 },
      { "question": "Em 'O Retrato de Dorian Gray', de Oscar Wilde, quem é o pintor que cria o retrato mágico?", "answers": ["Lord Henry Wotton", "Alan Campbell", "Basil Hallward", "James Vane"], "correct": 2 },
      { "question": "Qual o nome do protagonista de 'A Metamorfose' de Franz Kafka, que acorda transformado em um inseto?", "answers": ["Josef K.", "Karl Rossmann", "Gregor Samsa", "K."], "correct": 2 },
      { "question": "Em 'Cem Anos de Solidão', de Gabriel García Márquez, qual é o nome da cidade fictícia onde a história se passa?", "answers": ["Santa María", "Comala", "Macondo", "Coronel Aureliano"], "correct": 2 },
      { "question": "Qual dramaturgo norueguês, autor de 'Casa de Bonecas', é considerado o pai do realismo moderno no teatro?", "answers": ["August Strindberg", "Henrik Ibsen", "Anton Chekhov", "George Bernard Shaw"], "correct": 1 },
      { "question": "Em 'Os Miseráveis', de Victor Hugo, qual o número de prisioneiro de Jean Valjean?", "answers": ["90210", "12345", "777", "24601"], "correct": 3 },
      { "question": "Qual destes romances não foi escrito por Jane Austen?", "answers": ["Orgulho e Preconceito", "Razão e Sensibilidade", "O Morro dos Ventos Uivantes", "Emma"], "correct": 2 },
      { "question": "Quem escreveu a 'Eneida', o poema épico que narra a fundação de Roma?", "answers": ["Homero", "Ovídio", "Virgílio", "Horácio"], "correct": 2 },
      { "question": "Qual o nome do monstro criado pelo Dr. Victor Frankenstein no romance de Mary Shelley?", "answers": ["Igor", "Prometeu", "O monstro não tem nome", "Frankenstein"], "correct": 2 },
      { "question": "Na peça 'Édipo Rei', de Sófocles, qual a terrível profecia que o protagonista cumpre?", "answers": ["Destruir Atenas", "Tornar-se um deus", "Matar o pai e casar com a mãe", "Perder o trono para o irmão"], "correct": 2 },
      { "question": "Qual o nome do poeta italiano que escreveu a coletânea 'Canzoniere', dedicada à sua amada Laura?", "answers": ["Dante Alighieri", "Boccaccio", "Petrarca", "Leopardi"], "correct": 2 },
      { "question": "Em 'Guerra e Paz' de Tolstói, qual família nobre russa está no centro da narrativa?", "answers": ["Os Karamázov", "Os Oblônski", "Os Rostov", "Os Karenin"], "correct": 2 },
      { "question": "Qual filósofo e escritor francês é o autor de 'O Estrangeiro'?", "answers": ["Jean-Paul Sartre", "Albert Camus", "Voltaire", "Marcel Proust"], "correct": 1 },
      { "question": "Qual o nome do narrador do romance 'O Grande Gatsby'?", "answers": ["Jay Gatsby", "Tom Buchanan", "George Wilson", "Nick Carraway"], "correct": 3 }
    ]
  },
  "invencoes_hard_1": {
    "name": "Invenções e Descobrimentos (Hard - Parte 1)",
    "questions": [
      { "question": "Qual cientista é creditado com a invenção do processo de pasteurização?", "answers": ["Robert Koch", "Louis Pasteur", "Joseph Lister", "Alexander Fleming"], "correct": 1 },
      { "question": "A invenção do para-raios é atribuída a qual figura histórica americana?", "answers": ["Thomas Edison", "Nikola Tesla", "Benjamin Franklin", "Alexander Graham Bell"], "correct": 2 },
      { "question": "Qual o nome do processo químico, inventado por Fritz Haber e Carl Bosch, que permite a produção de amônia em larga escala para fertilizantes?", "answers": ["Processo Solvay", "Processo de Contato", "Processo Haber-Bosch", "Processo Hall-Héroult"], "correct": 2 },
      { "question": "Embora Samuel Morse seja famoso pelo código, quem foi o inventor do telégrafo elétrico?", "answers": ["Charles Wheatstone e William Cooke", "Michael Faraday", "James Clerk Maxwell", "Guglielmo Marconi"], "correct": 0 },
      { "question": "Qual material revolucionário, o primeiro plástico totalmente sintético, foi inventado por Leo Baekeland em 1907?", "answers": ["Nylon", "Celuloide", "Baquelite", "Poliéster"], "correct": 2 },
      { "question": "Quem é considerado o 'pai do motor de combustão interna' por ter construído o primeiro motor de quatro tempos bem-sucedido?", "answers": ["Karl Benz", "Henry Ford", "Rudolf Diesel", "Nikolaus Otto"], "correct": 3 },
      { "question": "O 'Tornel de Arquimedes', usado para transferir água de um nível baixo para um mais alto, é um exemplo de qual máquina simples?", "answers": ["Plano Inclinado", "Alavanca", "Roda e Eixo", "Parafuso"], "correct": 3 },
      { "question": "Qual o nome da primeira vacina desenvolvida, criada por Edward Jenner para combater qual doença?", "answers": ["Poliomielite", "Raiva", "Varíola", "Tuberculose"], "correct": 2 },
      { "question": "A dinamite, um explosivo mais seguro que a nitroglicerina pura, foi inventada e patenteada por quem?", "answers": ["Richard Gatling", "Hiram Maxim", "Alfred Nobel", "J. Robert Oppenheimer"], "correct": 2 },
      { "question": "Qual matemático e filósofo grego é creditado com a descoberta do princípio do empuxo?", "answers": ["Pitágoras", "Euclides", "Arquimedes", "Ptolomeu"], "correct": 2 },
      { "question": "Quem inventou a World Wide Web (WWW) em 1989, incluindo protocolos como HTTP e HTML?", "answers": ["Bill Gates", "Steve Jobs", "Vint Cerf", "Tim Berners-Lee"], "correct": 3 },
      { "question": "Qual o nome da estrutura molecular do benzeno, um anel de seis carbonos, cuja descoberta foi inspirada por um sonho de Friedrich August Kekulé?", "answers": ["Cadeia linear", "Anel de Kekulé", "Estrutura tetraédrica", "Fulereno"], "correct": 1 },
      { "question": "Quem inventou o revólver com cilindro rotativo que revolucionou as armas de fogo?", "answers": ["John Browning", "Samuel Colt", "Oliver Winchester", "Eliphalet Remington"], "correct": 1 },
      { "question": "A descoberta dos raios-X em 1895 é creditada a qual físico alemão?", "answers": ["Max Planck", "Heinrich Hertz", "Wilhelm Röntgen", "Albert Einstein"], "correct": 2 },
      { "question": "Qual a invenção de Elisha Otis, apresentada em 1854, que tornou os arranha-céus uma realidade segura?", "answers": ["Vigas de aço", "Concreto armado", "Ar condicionado", "O freio de segurança para elevadores"], "correct": 3 },
      { "question": "O transistor, componente fundamental de todos os eletrônicos modernos, foi inventado em qual laboratório?", "answers": ["Xerox PARC", "IBM Research", "Bell Labs", "MIT Lincoln Laboratory"], "correct": 2 },
      { "question": "Quem é creditado com a invenção do primeiro telescópio refrator funcional, embora não o tenha inventado?", "answers": ["Galileu Galilei", "Johannes Kepler", "Isaac Newton", "Hans Lippershey"], "correct": 3 },
      { "question": "Qual foi a grande contribuição de Ignaz Semmelweis para a medicina no século XIX?", "answers": ["A descoberta da anestesia", "A invenção do estetoscópio", "A introdução da antissepsia (lavagem das mãos)", "O desenvolvimento da teoria dos germes"], "correct": 2 },
      { "question": "A invenção do tear mecânico por Edmund Cartwright foi um passo crucial para a mecanização de qual indústria?", "answers": ["Siderúrgica", "Automotiva", "Têxtil", "Alimentícia"], "correct": 2 },
      { "question": "Qual físico italiano inventou a primeira bateria elétrica, a 'pilha voltaica'?", "answers": ["Luigi Galvani", "Alessandro Volta", "Guglielmo Marconi", "Enrico Fermi"], "correct": 1 }
    ]
  },
  "invencoes_hard_2": {
    "name": "Invenções e Descobrimentos (Hard - Parte 2)",
    "questions": [
      { "question": "Qual o nome do processo desenvolvido por Henry Bessemer que permitiu a produção em massa de aço a baixo custo?", "answers": ["Processo de Siemens-Martin", "Processo de Bessemer", "Processo de puddling", "Processo de cadinho"], "correct": 1 },
      { "question": "Quem descobriu as leis da hereditariedade através de seus experimentos com ervilhas?", "answers": ["Charles Darwin", "Alfred Wallace", "Gregor Mendel", "Thomas Morgan"], "correct": 2 },
      { "question": "A descoberta de qual elemento radioativo por Marie e Pierre Curie deu início à era atômica?", "answers": ["Urânio", "Tório", "Polônio e Rádio", "Plutônio"], "correct": 2 },
      { "question": "Qual astrônomo polonês desenvolveu o modelo heliocêntrico do universo, tirando a Terra do centro?", "answers": ["Tycho Brahe", "Johannes Kepler", "Galileu Galilei", "Nicolau Copérnico"], "correct": 3 },
      { "question": "A invenção do 'motor a vapor de movimento rotativo' por James Watt foi fundamental para qual evento histórico?", "answers": ["Renascimento", "Revolução Francesa", "Revolução Industrial", "Iluminismo"], "correct": 2 },
      { "question": "Qual o nome do primeiro antibiótico descoberto, a penicilina, e quem o descobriu?", "answers": ["Estreptomicina, por Selman Waksman", "Penicilina, por Alexander Fleming", "Sulfanilamida, por Gerhard Domagk", "Tetraciclina, por Benjamin Minge Duggar"], "correct": 1 },
      { "question": "Quem formulou as três leis do movimento e a lei da gravitação universal, fundamentando a mecânica clássica?", "answers": ["Galileu Galilei", "Isaac Newton", "Albert Einstein", "Johannes Kepler"], "correct": 1 },
      { "question": "Qual físico descobriu o elétron em 1897?", "answers": ["Ernest Rutherford", "Niels Bohr", "J.J. Thomson", "James Chadwick"], "correct": 2 },
      { "question": "A invenção da prensa de tipos móveis, que permitiu a impressão em massa, é atribuída a quem?", "answers": ["Leonardo da Vinci", "Johannes Gutenberg", "William Caxton", "Bi Sheng"], "correct": 1 },
      { "question": "Qual o nome do médico inglês que descreveu pela primeira vez, com precisão, a circulação sanguínea?", "answers": ["Hipócrates", "Galeno", "Andreas Vesalius", "William Harvey"], "correct": 3 },
      { "question": "Qual matemático é considerado o 'pai da computação' por seu trabalho na Máquina Analítica?", "answers": ["Alan Turing", "John von Neumann", "Charles Babbage", "Blaise Pascal"], "correct": 2 },
      { "question": "A invenção do telefone, embora patenteada por Bell, teve contribuições de qual outro inventor?", "answers": ["Thomas Edison", "Elisha Gray", "Nikola Tesla", "Guglielmo Marconi"], "correct": 1 },
      { "question": "A descoberta da fissão nuclear em 1938 por Otto Hahn e Fritz Strassmann foi crucial para o desenvolvimento de quê?", "answers": ["Reatores de fusão", "Armas nucleares e energia nuclear", "Computadores quânticos", "Aceleradores de partículas"], "correct": 1 },
      { "question": "Qual o nome do sistema de escrita para cegos, baseado em pontos em relevo, inventado por Louis Braille?", "answers": ["Código Morse", "Escrita Cuneiforme", "Sistema Braille", "Alfabeto Fonético"], "correct": 2 },
      { "question": "A invenção do algodão-doce, em 1897, foi surpreendentemente feita por um dentista. Qual era o seu nome?", "answers": ["William Morrison", "John Kellogg", "Charles C. Cretors", "James Dewar"], "correct": 0 },
      { "question": "Quem descobriu que o universo está em expansão ao observar o desvio para o vermelho na luz de galáxias distantes?", "answers": ["Carl Sagan", "Edwin Hubble", "Albert Einstein", "Stephen Hawking"], "correct": 1 },
      { "question": "A vulcanização, processo que torna a borracha mais resistente e útil, foi descoberta por quem?", "answers": ["John Dunlop", "Charles Goodyear", "Thomas Hancock", "Robert William Thomson"], "correct": 1 },
      { "question": "Qual matemático grego é conhecido como o 'pai da geometria' por sua obra 'Os Elementos'?", "answers": ["Pitágoras", "Arquimedes", "Euclides", "Tales de Mileto"], "correct": 2 },
      { "question": "A invenção do 'daguerreótipo', um dos primeiros processos fotográficos, foi desenvolvida por quem?", "answers": ["Nicéphore Niépce", "Louis Daguerre", "Henry Fox Talbot", "George Eastman"], "correct": 1 },
      { "question": "Qual o nome do biólogo que, junto com Francis Crick, propôs a estrutura de dupla hélice para o DNA?", "answers": ["Rosalind Franklin", "Maurice Wilkins", "Linus Pauling", "James Watson"], "correct": 3 }
    ]
  },
"curiosidades_hard": {
    "name": "Curiosidades (Hard)",
    "questions": [
      { "question": "Qual animal, na verdade, aparece no logotipo do navegador Firefox?", "answers": ["Uma raposa-vermelha", "Um panda-vermelho", "Um lobo-guará", "Um cachorro-vinagre"], "correct": 1 },
      { "question": "Qual é a única parte do corpo humano que não recebe suprimento de sangue?", "answers": ["A cartilagem da orelha", "As unhas", "O cristalino do olho", "A córnea"], "correct": 3 },
      { "question": "Originalmente, qual era a cor da roupa do Papai Noel antes da popularização pela campanha da Coca-Cola?", "answers": ["Azul", "Branco", "Verde", "Marrom"], "correct": 2 },
      { "question": "Qual o nome do fenômeno que faz com que, após um incêndio florestal, certas sementes germinem?", "answers": ["Pirofitismo", "Hidrofitismo", "Xerofitismo", "Fototropismo"], "correct": 0 },
      { "question": "Qual o nome do medo irracional de palhaços?", "answers": ["Acrofobia", "Agorafobia", "Coulrofobia", "Aracnofobia"], "correct": 2 },
      { "question": "Quantos narizes tem uma lesma?", "answers": ["Nenhum", "Um", "Dois", "Quatro"], "correct": 3 },
      { "question": "Qual era a principal função das pirâmides do Egito?", "answers": ["Observatórios astronômicos", "Templos de adoração", "Tumbas para os faraós", "Palácios do governo"], "correct": 2 },
      { "question": "O mel é um dos poucos alimentos que não estraga. Qual sua principal propriedade conservante?", "answers": ["Alta acidez e baixo teor de água", "Presença de pólen", "Textura viscosa", "Cor escura"], "correct": 0 },
      { "question": "Qual país tem o maior número de fusos horários (incluindo territórios ultramarinos)?", "answers": ["Rússia", "Canadá", "Estados Unidos", "França"], "correct": 3 },
      { "question": "Onde, no corpo humano, se localiza o menor osso, o estribo?", "answers": ["No nariz", "Na mão", "No pé", "No ouvido"], "correct": 3 },
      { "question": "Qual o nome da cor específica do céu em um dia sem nuvens?", "answers": ["Azul celeste", "Azul ciano", "Azul Rayleigh", "Não tem nome específico"], "correct": 2 },
      { "question": "Qual é o animal nacional oficial da Escócia?", "answers": ["Águia-real", "Cervo-vermelho", "Gato-escocês", "Unicórnio"], "correct": 3 },
      { "question": "Qual era o nome original da cidade de Nova Iorque?", "answers": ["Nova Amsterdã", "Nova Londres", "Nova Paris", "Nova Roma"], "correct": 0 },
      { "question": "De qual material os Óscares da Academia são feitos principalmente?", "answers": ["Ouro maciço", "Prata banhada a ouro", "Bronze banhado a ouro", "Latão banhado a ouro"], "correct": 2 },
      { "question": "Qual fruta é, na verdade, uma baga (berry) do ponto de vista botânico?", "answers": ["Morango", "Amora", "Framboesa", "Banana"], "correct": 3 },
      { "question": "Quanto tempo a luz do Sol leva para chegar à Terra?", "answers": ["Exatamente 8 minutos", "Aproximadamente 8 minutos e 20 segundos", "9 minutos e 10 segundos", "7 minutos e 40 segundos"], "correct": 1 },
      { "question": "A palavra 'abacate' vem de uma língua asteca (Nahuatl) e significava originalmente...", "answers": ["Fruto verde", "Pera-de-jacaré", "Testículo", "Manteiga da floresta"], "correct": 2 },
      { "question": "Qual a única bandeira nacional do mundo que não é retangular ou quadrada?", "answers": ["Suíça", "Vaticano", "Nepal", "Catar"], "correct": 2 },
      { "question": "O que os camelos armazenam em suas corcovas?", "answers": ["Água", "Gordura", "Músculo", "Areia"], "correct": 1 },
      { "question": "Qual o nome da unidade de medida para o nível de picância das pimentas?", "answers": ["Escala Richter", "Escala Scoville", "Escala Saffir-Simpson", "Escala Mercalli"], "correct": 1 }
    ]
  },
  "curiosidades_expert": {
    "name": "Curiosidades (Expert)",
    "questions": [
      { "question": "Qual é a única letra do alfabeto que não aparece no nome de nenhum estado dos EUA?", "answers": ["Z", "J", "X", "Q"], "correct": 3 },
      { "question": "Qual era a profissão de Guichard, o inventor da guilhotina?", "answers": ["Médico", "Carpinteiro", "Ferreiro", "Nobre"], "correct": 0 },
      { "question": "O som que um pato faz (quack) é um dos únicos sons da natureza que, por uma peculiaridade acústica, não produz o quê?", "answers": ["Eco", "Vibração", "Ressonância", "Onda de choque"], "correct": 0 },
      { "question": "Qual o nome da condição médica que faz com que a urina de uma pessoa cheire a xarope de ácer (maple syrup)?", "answers": ["Fenilcetonúria", "Doença de Wilson", "Doença da Urina de Xarope de Bordo", "Alcaptonúria"], "correct": 2 },
      { "question": "A hidrante de incêndio, como a conhecemos, teve sua patente perdida em qual evento histórico?", "answers": ["Grande Incêndio de Londres", "Incêndio no escritório de patentes de Washington em 1836", "Guerra Civil Americana", "Terremoto de São Francisco de 1906"], "correct": 1 },
      { "question": "No baralho de cartas padrão, qual rei é o único que não tem bigode?", "answers": ["Rei de Espadas", "Rei de Copas", "Rei de Ouros", "Rei de Paus"], "correct": 1 },
      { "question": "Como se chama o ponto no céu diretamente abaixo de um observador (o oposto do zênite)?", "answers": ["Nadir", "Ápice", "Azimute", "Perigeu"], "correct": 0 },
      { "question": "Qual animal tem o maior cérebro em proporção ao seu corpo?", "answers": ["Ser humano", "Golfinho", "Elefante", "Formiga"], "correct": 3 },
      { "question": "O efeito de 'olhos vermelhos' em fotografias é causado pelo flash refletindo em qual parte do olho?", "answers": ["Na retina", "Na íris", "Nos vasos sanguíneos da coroide", "No nervo óptico"], "correct": 2 },
      { "question": "Quantos anos durou a 'Guerra dos Cem Anos' entre a Inglaterra e a França?", "answers": ["99 anos", "100 anos", "116 anos", "125 anos"], "correct": 2 },
      { "question": "Qual o nome do primeiro filme a usar um som de descarga de vaso sanitário?", "answers": ["O Iluminado", "Psicose", "Laranja Mecânica", "O Poderoso Chefão"], "correct": 1 },
      { "question": "Qual animal produz o som mais alto do planeta Terra?", "answers": ["Baleia-azul", "Elefante", "Bugio", "Camarão-pistola"], "correct": 3 },
      { "question": "Como se chama o estudo de bandeiras?", "answers": ["Numismática", "Filatelia", "Vexilologia", "Heráldica"], "correct": 2 },
      { "question": "No manuscrito original de 'Alice no País das Maravilhas', o que o Chapeleiro Maluco vende?", "answers": ["Chá", "Relógios", "Chapéus", "Tempo"], "correct": 3 },
      { "question": "Qual era o nome do supercontinente que existiu antes da Pangeia?", "answers": ["Gondwana", "Laurásia", "Rodínia", "Pannotia"], "correct": 2 },
      { "question": "Qual planeta do nosso sistema solar tem o dia mais longo?", "answers": ["Júpiter", "Marte", "Vênus", "Mercúrio"], "correct": 2 },
      { "question": "O sinal de S.O.S. tornou-se padrão em 1908. O que as letras significam?", "answers": ["Save Our Ship (Salvem Nosso Navio)", "Save Our Souls (Salvem Nossas Almas)", "Send Out Succour (Enviem Socorro)", "Absolutamente nada"], "correct": 3 },
      { "question": "Qual o nome do espaço entre as sobrancelhas?", "answers": ["Filtro", "Glabela", "Trago", "Hélix"], "correct": 1 },
      { "question": "O primeiro produto a ter um código de barras escaneado em um supermercado foi o quê?", "answers": ["Uma lata de sopa", "Um pacote de chicletes", "Uma garrafa de Coca-Cola", "Uma caixa de cereais"], "correct": 1 },
      { "question": "O cérebro de Albert Einstein foi preservado após sua morte. Uma característica notável era a ausência de uma fissura em qual lobo cerebral?", "answers": ["Lobo frontal", "Lobo temporal", "Lobo parietal", "Lobo occipital"], "correct": 2 }
    ]
  },
  "citacoes_famosas": {
    "name": "Citações Famosas",
    "questions": [
      { "question": "Quem é o autor da famosa citação filosófica 'Penso, logo existo'?", "answers": ["Sócrates", "Platão", "Aristóteles", "René Descartes"], "correct": 3 },
      { "question": "A frase 'Olho por olho, e o mundo acabará cego' é frequentemente atribuída a qual líder pacifista?", "answers": ["Nelson Mandela", "Martin Luther King Jr.", "Mahatma Gandhi", "Dalai Lama"], "correct": 2 },
      { "question": "Qual dramaturgo inglês escreveu a célebre frase 'Ser ou não ser, eis a questão'?", "answers": ["Christopher Marlowe", "William Shakespeare", "Ben Jonson", "George Bernard Shaw"], "correct": 1 },
      { "question": "A afirmação 'O homem nasce livre, e por toda a parte encontra-se a ferros' é de qual filósofo iluminista?", "answers": ["Voltaire", "Montesquieu", "Jean-Jacques Rousseau", "Diderot"], "correct": 2 },
      { "question": "Qual cientista disse 'Se eu vi mais longe, foi por estar sobre ombros de gigantes'?", "answers": ["Albert Einstein", "Galileu Galilei", "Isaac Newton", "Nikola Tesla"], "correct": 2 },
      { "question": "A citação 'A única coisa que devemos temer é o próprio medo' foi dita por qual presidente americano em seu discurso de posse?", "answers": ["Abraham Lincoln", "Theodore Roosevelt", "Franklin D. Roosevelt", "John F. Kennedy"], "correct": 2 },
      { "question": "'Vim, vi, venci' (Veni, vidi, vici) é uma frase célebre atribuída a qual líder militar romano?", "answers": ["Augusto", "Júlio César", "Marco Antônio", "Nero"], "correct": 1 },
      { "question": "Quem disse 'A religião é o ópio do povo'?", "answers": ["Friedrich Nietzsche", "Sigmund Freud", "Karl Marx", "Vladimir Lenin"], "correct": 2 },
      { "question": "A frase 'Navegar é preciso, viver não é preciso' foi popularizada na língua portuguesa por qual poeta?", "answers": ["Luís de Camões", "Carlos Drummond de Andrade", "Fernando Pessoa", "Vinicius de Moraes"], "correct": 2 },
      { "question": "Qual figura histórica declarou 'O Estado sou eu' (L'État, c'est moi)?", "answers": ["Napoleão Bonaparte", "Luís XIV", "Cardeal Richelieu", "Carlos Magno"], "correct": 1 },
      { "question": "A frase 'Que a Força esteja com você' é um bordão de qual famosa franquia de filmes?", "answers": ["Star Trek", "O Senhor dos Anéis", "Harry Potter", "Star Wars"], "correct": 3 },
      { "question": "Quem é o autor da citação 'Tudo o que sei é que nada sei'?", "answers": ["Platão", "Aristóteles", "Sócrates", "Diógenes"], "correct": 2 },
      { "question": "A frase 'Os fins justificam os meios' é uma interpretação popular das ideias de qual pensador renascentista?", "answers": ["Erasmo de Roterdã", "Thomas More", "Nicolau Maquiavel", "Dante Alighieri"], "correct": 2 },
      { "question": "Qual escritor irlandês é famoso pela citação 'Posso resistir a tudo, exceto à tentação'?", "answers": ["James Joyce", "George Bernard Shaw", "Oscar Wilde", "Samuel Beckett"], "correct": 2 },
      { "question": "A frase 'Um pequeno passo para um homem, um salto gigante para a humanidade' foi dita por quem?", "answers": ["Yuri Gagarin", "John Glenn", "Buzz Aldrin", "Neil Armstrong"], "correct": 3 },
      { "question": "Quem disse a famosa frase 'Loucura é querer resultados diferentes fazendo tudo exatamente igual'?", "answers": ["Benjamin Franklin", "Thomas Edison", "Albert Einstein", "Sigmund Freud"], "correct": 2 },
      { "question": "A qual primeiro-ministro britânico é atribuída a frase 'Nunca, no campo dos conflitos humanos, tantos deveram tanto a tão poucos'?", "answers": ["Neville Chamberlain", "Clement Attlee", "Winston Churchill", "Margaret Thatcher"], "correct": 2 },
      { "question": "Qual escritor brasileiro é o autor da frase 'A saudade é a prova de que o passado valeu a pena'?", "answers": ["Machado de Assis", "Carlos Drummond de Andrade", "Clarice Lispector", "Padre Fábio de Melo"], "correct": 3 },
      { "question": "A frase 'A imaginação é mais importante que o conhecimento' é de autoria de qual gênio da física?", "answers": ["Stephen Hawking", "Richard Feynman", "Niels Bohr", "Albert Einstein"], "correct": 3 },
      { "question": "Qual pintor é famoso por ter dito 'Eu pinto os meus sonhos, não a minha realidade'?", "answers": ["Salvador Dalí", "Frida Kahlo", "Pablo Picasso", "Vincent van Gogh"], "correct": 1 }
    ]
  },
  "historia_brasil_2": {
    "name": "História do Brasil 2",
    "questions": [
      { "question": "Qual foi a maior revolta de escravizados do Brasil, ocorrida em Salvador em 1835 e liderada por muçulmanos?", "answers": ["Balaiada", "Sabinada", "Revolta dos Malês", "Cabanagem"], "correct": 2 },
      { "question": "Como ficou conhecida a expedição liderada por militares que percorreu mais de 25.000 km pelo interior do Brasil entre 1925 e 1927?", "answers": ["Guerra de Canudos", "Coluna Prestes", "Guerra do Contestado", "Revolta da Chibata"], "correct": 1 },
      { "question": "Qual o nome do plano econômico do governo de José Sarney que congelou preços e criou uma nova moeda, o Cruzado?", "answers": ["Plano Real", "Plano Bresser", "Plano Verão", "Plano Cruzado"], "correct": 3 },
      { "question": "A Guerra dos Farrapos, a mais longa revolta brasileira, resultou na proclamação de qual república independente?", "answers": ["República de Canudos", "República de Palmares", "República Rio-Grandense", "República de Piratini"], "correct": 2 },
      { "question": "O lema 'É proibido proibir' marcou qual movimento cultural e de contracultura no Brasil no final dos anos 60?", "answers": ["Bossa Nova", "Jovem Guarda", "Tropicália", "Cinema Novo"], "correct": 2 },
      { "question": "Qual era o principal objetivo das Entradas e Bandeiras no período colonial?", "answers": ["Catequizar indígenas", "Demarcar fronteiras", "Capturar indígenas e procurar metais preciosos", "Fundar cidades no litoral"], "correct": 2 },
      { "question": "Quem foi o líder da Revolta da Chibata, que lutou contra os castigos corporais na Marinha do Brasil?", "answers": ["Zumbi dos Palmares", "Tiradentes", "Antônio Conselheiro", "João Cândido"], "correct": 3 },
      { "question": "A criação de qual empresa estatal, em 1953, foi resultado da campanha nacional 'O petróleo é nosso'?", "answers": ["Vale do Rio Doce", "Eletrobras", "Petrobras", "Embraer"], "correct": 2 },
      { "question": "Qual batalha, em 1648 e 1649, foi decisiva para a expulsão dos holandeses de Pernambuco?", "answers": ["Batalha Naval do Riachuelo", "Batalha dos Guararapes", "Batalha do Monte das Tabordas", "Batalha do Jenipapo"], "correct": 1 },
      { "question": "O que foi o 'Ciclo da Borracha' no final do século XIX e início do XX?", "answers": ["A produção de pneus para a indústria automobilística", "A extração de látex na Amazônia para exportação", "A invenção da borracha sintética", "Uma crise na indústria calçadista"], "correct": 1 },
      { "question": "Qual o nome da primeira constituição republicana do Brasil, promulgada em 1891?", "answers": ["Constituição da Mandioca", "Constituição Cidadã", "Constituição de 1891", "Constituição do Império"], "correct": 2 },
      { "question": "A 'Noite da Agonia' foi o evento em que Dom Pedro I dissolveu qual órgão político em 1823?", "answers": ["O Senado", "A Câmara dos Deputados", "O Conselho de Estado", "A Assembleia Constituinte"], "correct": 3 },
      { "question": "Qual o nome do movimento messiânico liderado por Antônio Conselheiro no sertão da Bahia?", "answers": ["Guerra do Contestado", "Cangaço", "Guerra de Canudos", "Sedição de Juazeiro"], "correct": 2 },
      { "question": "O que foi a 'Política do Café com Leite' durante a República Velha?", "answers": ["Um incentivo à produção de laticínios", "Um acordo entre as oligarquias de SP e MG para alternar na presidência", "A criação de cafeterias estatais", "Uma lei que proibia a exportação de café e leite"], "correct": 1 },
      { "question": "A Bossa Nova, movimento musical brasileiro, surgiu em qual cidade no final da década de 1950?", "answers": ["São Paulo", "Salvador", "Belo Horizonte", "Rio de Janeiro"], "correct": 3 },
      { "question": "Qual o nome do Tratado de 1750 que redefiniu as fronteiras entre as colônias portuguesas e espanholas na América do Sul?", "answers": ["Tratado de Tordesilhas", "Tratado de Madri", "Tratado de Santo Ildefonso", "Tratado de Utrecht"], "correct": 1 },
      { "question": "A Inconfidência Carioca de 1794, diferentemente da Mineira, foi inspirada por qual evento internacional?", "answers": ["Independência dos EUA", "Revolução Francesa", "Revolução Industrial", "Iluminismo"], "correct": 1 },
      { "question": "Quem foi o único presidente a renunciar na história do Brasil?", "answers": ["Getúlio Vargas", "Fernando Collor", "Jânio Quadros", "João Goulart"], "correct": 2 },
      { "question": "Qual foi a 'Lei de Terras' de 1850?", "answers": ["Uma lei que distribuía terras aos imigrantes", "A primeira lei de reforma agrária", "Uma lei que proibia a posse de terra por posse, exigindo a compra", "Uma lei que demarcava terras indígenas"], "correct": 2 },
      { "question": "O que foi o 'Milagre Econômico' (1968-1973)?", "answers": ["Um período de inflação zero", "Um período de alto crescimento do PIB durante a ditadura militar", "A descoberta de grandes reservas de petróleo", "O perdão da dívida externa brasileira"], "correct": 1 }
    ]
  },
  "ciencias_tecnologia_2": {
    "name": "Ciências e Tecnologia 2",
    "questions": [
      { "question": "O que é CRISPR-Cas9, uma das mais importantes descobertas da biologia molecular recente?", "answers": ["Um tipo de telescópio espacial", "Uma técnica de edição de genes", "Um novo supercondutor", "Um algoritmo de inteligência artificial"], "correct": 1 },
      { "question": "No Modelo Padrão da física de partículas, qual bóson é responsável por dar massa às outras partículas?", "answers": ["Fóton", "Glúon", "Bóson de Higgs", "Bóson W"], "correct": 2 },
      { "question": "Como se chama a camada mais externa da atmosfera do Sol, visível durante um eclipse total?", "answers": ["Cromosfera", "Fotosfera", "Corona", "Zona de convecção"], "correct": 2 },
      { "question": "O que o Teste de Turing, proposto por Alan Turing, se propõe a avaliar?", "answers": ["A velocidade de um computador", "A capacidade de uma máquina de exibir comportamento inteligente equivalente a um humano", "A segurança de um sistema criptográfico", "A eficiência de um algoritmo"], "correct": 1 },
      { "question": "Grafeno, um material revolucionário, é uma estrutura bidimensional composta por átomos de qual elemento?", "answers": ["Silício", "Carbono", "Titânio", "Ouro"], "correct": 1 },
      { "question": "Qual é a função principal do ARN Mensageiro (mRNA) na célula?", "answers": ["Armazenar informação genética no núcleo", "Transportar aminoácidos para o ribossomo", "Levar uma cópia de um gene do DNA para o ribossomo para a síntese de proteínas", "Catalisar reações bioquímicas"], "correct": 2 },
      { "question": "O que é um 'exoplaneta'?", "answers": ["Um planeta anão no nosso sistema solar", "Um planeta que orbita uma estrela fora do nosso sistema solar", "Uma lua com potencial para abrigar vida", "Um planeta composto inteiramente de gás"], "correct": 1 },
      { "question": "Na química orgânica, qual é o nome do grupo funcional -COOH?", "answers": ["Álcool", "Aldeído", "Cetona", "Ácido carboxílico"], "correct": 3 },
      { "question": "O que é a 'computação quântica'?", "answers": ["Um tipo de supercomputador mais rápido", "Uma forma de computação que usa os princípios da mecânica quântica, como sobreposição e emaranhamento", "Um computador que funciona na velocidade da luz", "Uma rede neural que imita o cérebro humano"], "correct": 1 },
      { "question": "Qual o nome do processo pelo qual um buraco negro emite radiação e perde massa ao longo do tempo, teorizado por Stephen Hawking?", "answers": ["Radiação de corpo negro", "Radiação Hawking", "Efeito de lente gravitacional", "Singularidade nua"], "correct": 1 },
      { "question": "O que é o 'Efeito Peltier' na termodinâmica?", "answers": ["A emissão de luz por um material aquecido", "A geração de eletricidade a partir do calor", "O aquecimento ou resfriamento de uma junção de dois materiais diferentes quando uma corrente elétrica passa por ela", "A resistência elétrica de um material a zero"], "correct": 2 },
      { "question": "Qual o nome da sonda espacial lançada em 1977 que carrega o 'Golden Record', um disco com sons e imagens da Terra?", "answers": ["Pioneer 10", "Hubble", "Voyager 1", "Cassini"], "correct": 2 },
      { "question": "O que são 'extremófilos'?", "answers": ["Organismos que vivem em condições ambientais extremas, como altas temperaturas ou pressão", "Partículas subatômicas instáveis", "Um tipo de alga tóxica", "Espécies de plantas carnívoras raras"], "correct": 0 },
      { "question": "Qual o princípio por trás da tecnologia de Fibra Óptica para transmissão de dados?", "answers": ["Refração total da luz", "Difração da luz", "Reflexão interna total da luz", "Polarização da luz"], "correct": 2 },
      { "question": "Na geologia, como é chamada a camada semi-líquida do manto superior da Terra sobre a qual as placas tectônicas se movem?", "answers": ["Litosfera", "Astenosfera", "Mesosfera", "Endosfera"], "correct": 1 },
      { "question": "O que é o 'Grande Colisor de Hádrons' (LHC)?", "answers": ["O maior telescópio do mundo", "Uma estação espacial internacional", "O maior acelerador de partículas do mundo, localizado no CERN", "Um reator de fusão nuclear experimental"], "correct": 2 },
      { "question": "Qual o nome do paradoxo da mecânica quântica que envolve um gato que pode estar simultaneamente vivo e morto?", "answers": ["Paradoxo de Fermi", "Paradoxo dos Gêmeos", "Paradoxo de Schrödinger", "Paradoxo EPR"], "correct": 2 },
      { "question": "O que a Lei de Moore, observada na indústria de semicondutores, previa?", "answers": ["Que a velocidade da internet dobraria a cada dois anos", "Que o número de transistores em um chip dobraria aproximadamente a cada dois anos", "Que o custo de armazenamento de dados cairia pela metade a cada ano", "Que a inteligência artificial superaria a humana até 2029"], "correct": 1 },
      { "question": "O que são 'micorrizas'?", "answers": ["Um tipo de bactéria do solo", "Uma associação simbiótica entre fungos e raízes de plantas", "Organelas celulares responsáveis pela respiração", "Vírus que infectam fungos"], "correct": 1 },
      { "question": "Qual tecnologia, base do Bitcoin e outras criptomoedas, é um registro de dados distribuído e imutável?", "answers": ["Inteligência Artificial", "Computação em Nuvem", "Internet das Coisas", "Blockchain"], "correct": 3 }
    ]
  },
"esportes_2": {
    "name": "Esportes 2",
    "questions": [
      { "question": "No rugby, qual o nome da formação em que os jogadores se juntam para disputar a posse de bola após uma infração menor?", "answers": ["Try", "Scrum", "Ruck", "Maul"], "correct": 1 },
      { "question": "Qual o único país a ter participado de todas as edições da Copa do Mundo de Futebol Masculino?", "answers": ["Alemanha", "Itália", "Argentina", "Brasil"], "correct": 3 },
      { "question": "No beisebol, o que é um 'grand slam'?", "answers": ["Eliminar três jogadores em uma única jogada", "Um arremesso que ultrapassa 160 km/h", "Um home run com as três bases ocupadas, marcando 4 pontos", "Vencer a World Series por 4 jogos a 0"], "correct": 2 },
      { "question": "Qual é a prova mais longa do atletismo olímpico, com 50km de distância?", "answers": ["Maratona", "Marcha atlética", "Decatlo", "Triatlo"], "correct": 1 },
      { "question": "Em que esporte o Brasil ganhou sua primeira medalha de ouro em Olimpíadas de Inverno?", "answers": ["Bobsled", "Curling", "Snowboard", "O Brasil nunca ganhou ouro em Olimpíadas de Inverno"], "correct": 3 },
      { "question": "No críquete, como se chama o jogador que arremessa a bola?", "answers": ["Batsman", "Wicket-keeper", "Bowler", "Fielder"], "correct": 2 },
      { "question": "Qual é o nome da principal liga de hóquei no gelo da América do Norte?", "answers": ["AHL", "KHL", "NHL", "IIHF"], "correct": 2 },
      { "question": "Quantos buracos tem um campo de golfe oficial padrão?", "answers": ["9", "12", "18", "21"], "correct": 2 },
      { "question": "No MMA, qual a duração padrão de um round em uma luta que não vale cinturão?", "answers": ["3 minutos", "4 minutos", "5 minutos", "10 minutos"], "correct": 2 },
      { "question": "Qual o nome da manobra no skate em que o atleta salta com a prancha sem usar as mãos?", "answers": ["Kickflip", "Grind", "Manual", "Ollie"], "correct": 3 },
      { "question": "Qual país é considerado o berço do Judô?", "answers": ["China", "Coreia do Sul", "Japão", "Tailândia"], "correct": 2 },
      { "question": "No xadrez, qual a única peça que pode pular sobre outras peças?", "answers": ["Bispo", "Torre", "Rainha", "Cavalo"], "correct": 3 },
      { "question": "Qual é a pontuação máxima que um único dardo pode marcar em um alvo de dardos padrão?", "answers": ["20", "50", "60", "180"], "correct": 2 },
      { "question": "Qual o nome do evento de ciclismo de estrada mais famoso da Itália, análogo ao Tour de France?", "answers": ["Vuelta a España", "Giro d'Italia", "Milão-Sanremo", "Tour de Suisse"], "correct": 1 },
      { "question": "No handebol, por quantos segundos um jogador pode segurar a bola sem driblar, andar ou arremessar?", "answers": ["3 segundos", "5 segundos", "7 segundos", "Não há limite"], "correct": 0 },
      { "question": "Qual a cor da bola usada em uma partida oficial de sinuca para iniciar o jogo (bola de saída)?", "answers": ["Preta", "Branca", "Vermelha", "Amarela"], "correct": 1 },
      { "question": "Qual o nome da competição de iatismo mais antiga e prestigiada do mundo?", "answers": ["Volvo Ocean Race", "Vendée Globe", "America's Cup", "The Ocean Race"], "correct": 2 },
      { "question": "No badminton, o objeto que é rebatido sobre a rede é chamado de:", "answers": ["Puck", "Bola", "Peteca", "Frisbee"], "correct": 2 },
      { "question": "Qual o esporte em que Michael Jordan se aventurou profissionalmente durante sua primeira aposentadoria do basquete?", "answers": ["Golfe", "Beisebol", "Futebol Americano", "Hóquei"], "correct": 1 },
      { "question": "Em qual década o voleibol de praia se tornou um esporte olímpico?", "answers": ["1980", "1990", "2000", "2010"], "correct": 1 }
    ]
  },
  "futebol_brasileiro_hard": {
    "name": "Futebol Brasileiro (Hard)",
    "questions": [
      { "question": "Qual jogador detém o recorde de mais gols em uma única edição do Campeonato Brasileiro de pontos corridos?", "answers": ["Romário", "Dimba", "Fred", "Washington"], "correct": 3 },
      { "question": "Qual foi o primeiro clube brasileiro a conquistar a Copa Libertadores da América?", "answers": ["Flamengo", "Grêmio", "Santos", "Palmeiras"], "correct": 2 },
      { "question": "Em 1999, qual clube pequeno de Caxias do Sul surpreendeu o Brasil ao vencer a Copa do Brasil?", "answers": ["Caxias", "Brasil de Pelotas", "Juventude", "Esportivo"], "correct": 2 },
      { "question": "Quem foi o técnico da Seleção Brasileira na conquista da Copa do Mundo de 1994?", "answers": ["Zagallo", "Telê Santana", "Carlos Alberto Parreira", "Vanderlei Luxemburgo"], "correct": 2 },
      { "question": "Qual clube é conhecido pela alcunha de 'Pó de Arroz', devido a um episódio histórico envolvendo um de seus jogadores?", "answers": ["Vasco da Gama", "Botafogo", "Fluminense", "America-RJ"], "correct": 2 },
      { "question": "Qual foi o placar da final da Copa do Mundo de 1950, o 'Maracanazo', entre Brasil e Uruguai?", "answers": ["1 a 0 para o Uruguai", "2 a 1 para o Uruguai", "3 a 2 para o Uruguai", "2 a 0 para o Uruguai"], "correct": 1 },
      { "question": "Qual o único clube a ter conquistado o Campeonato Brasileiro da Série A por quatro vezes consecutivas?", "answers": ["Flamengo", "Palmeiras", "Santos", "São Paulo"], "correct": 3 },
      { "question": "Quem é o maior artilheiro da história do clássico 'Majestoso' (Corinthians vs São Paulo)?", "answers": ["Pelé", "Sócrates", "Raí", "Telêmaco"], "correct": 0 },
      { "question": "Qual jogador, ídolo do Vasco, é conhecido como 'Animal'?", "answers": ["Romário", "Edmundo", "Dinamite", "Juninho Pernambucano"], "correct": 1 },
      { "question": "Qual clube paranaense foi campeão brasileiro em 2001, liderado pela dupla Alex e Kléber Pereira?", "answers": ["Coritiba", "Paraná Clube", "Malutrom", "Athletico Paranaense"], "correct": 3 },
      { "question": "Qual o nome do estádio que foi palco do milésimo gol de Pelé?", "answers": ["Morumbi", "Pacaembu", "Maracanã", "Vila Belmiro"], "correct": 2 },
      { "question": "Qual jogador ficou conhecido como 'Doutor' por ser um dos líderes da 'Democracia Corinthiana'?", "answers": ["Casagrande", "Sócrates", "Wladimir", "Zenon"], "correct": 1 },
      { "question": "Qual clube mineiro foi o primeiro campeão da Taça Brasil, em 1959, considerado o primeiro campeonato nacional?", "answers": ["Atlético Mineiro", "Cruzeiro", "América Mineiro", "Bahia"], "correct": 3 },
      { "question": "Qual o único goleiro a ter marcado mais de 100 gols na carreira?", "answers": ["Marcos", "Taffarel", "Rogério Ceni", "Dida"], "correct": 2 },
      { "question": "O 'Quadrado Mágico' da Seleção de 2006 era formado por Ronaldo, Adriano, Kaká e quem mais?", "answers": ["Robinho", "Ronaldinho Gaúcho", "Juninho Pernambucano", "Zé Roberto"], "correct": 1 },
      { "question": "Qual time foi apelidado de 'Carrossel Caipira' nos anos 90 por seu estilo de jogo envolvente?", "answers": ["Guarani", "Bragantino", "Mogi Mirim", "Ponte Preta"], "correct": 2 },
      { "question": "Qual o primeiro clube-empresa a se tornar campeão brasileiro, em 2021?", "answers": ["Cuiabá", "Fortaleza", "Red Bull Bragantino", "Athletico Paranaense"], "correct": 2 },
      { "question": "Quem foi o autor do gol do título do Corinthians no Mundial de Clubes de 2012 contra o Chelsea?", "answers": ["Paulinho", "Danilo", "Jorge Henrique", "Paolo Guerrero"], "correct": 3 },
      { "question": "Qual o apelido do centro de treinamento do Flamengo?", "answers": ["Toca da Raposa", "Ninho do Urubu", "CT da Barra Funda", "Cidade do Galo"], "correct": 1 },
      { "question": "Qual o nome da taça original da Copa do Mundo, que foi roubada no Brasil e nunca recuperada?", "answers": ["Taça FIFA", "Taça Jules Rimet", "Taça da Vitória", "Taça da Independência"], "correct": 1 }
    ]
  },
  "capas_de_disco": {
    "name": "Capas de Disco",
    "questions": [
      { "question": "A que banda pertence esta icônica capa com um bebê submerso?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/b/b7/NirvanaNevermindalbumcover.jpg", "answers": ["Pearl Jam", "Soundgarden", "Nirvana", "Alice in Chains"], "correct": 2 },
      { "question": "Qual álbum dos Beatles apresenta os quatro membros atravessando uma faixa de pedestres?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/4/42/Beatles_-_Abbey_Road.jpg", "answers": ["Sgt. Pepper's Lonely Hearts Club Band", "The White Album", "Let It Be", "Abbey Road"], "correct": 3 },
      { "question": "Esta capa, com um prisma dispersando a luz, é de qual álbum clássico do Pink Floyd?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/3/3b/Dark_Side_of_the_Moon.png", "answers": ["The Wall", "Wish You Were Here", "Animals", "The Dark Side of the Moon"], "correct": 3 },
      { "question": "Esta capa, com um desenho de dois meninos sentados em um morro, pertence a qual marco da MPB?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/5/58/Clube_da_Esquina.jpg", "answers": ["Elis & Tom", "Tropicália", "Acabou Chorare", "Clube da Esquina"], "correct": 3 },
      { "question": "Qual álbum de Caetano Veloso tem esta capa com o cantor de perfil e cabelo black power?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/d/da/Caetano_Veloso_-_Transa.jpg", "answers": ["Araçá Azul", "Bicho", "Transa", "Cores, Nomes"], "correct": 2 },
      { "question": "Esta capa, que mostra um cemitério com uma cruz, é de qual álbum da Legião Urbana?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/c/c8/Legi%C3%A3o_Urbana_-_Dois.jpg", "answers": ["As Quatro Estações", "Dois", "Que País É Este", "V"], "correct": 1 },
      { "question": "Qual álbum de Michael Jackson tem esta famosa capa onde ele está deitado em um terno branco?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/3/30/Michael_Jackson_-_Thriller.jpg", "answers": ["Bad", "Off the Wall", "Dangerous", "Thriller"], "correct": 3 },
      { "question": "Esta capa dramática, com os rostos dos membros sobre um fundo preto, é de qual álbum do Queen?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/4/43/Queen_II.jpg", "answers": ["A Night at the Opera", "Sheer Heart Attack", "Queen II", "A Day at the Races"], "correct": 2 },
      { "question": "Qual artista aparece com um raio pintado no rosto nesta capa de álbum?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/d/d4/David_Bowie_-_Aladdin_Sane.jpg", "answers": ["Elton John", "David Bowie", "Lou Reed", "Iggy Pop"], "correct": 1 },
      { "question": "A famosa banana na capa, com a assinatura de Andy Warhol, é de qual banda?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/0/0c/TheVelvetUndergroundAndNico.jpg", "answers": ["The Doors", "The Rolling Stones", "The Velvet Underground & Nico", "The Stooges"], "correct": 2 },
      { "question": "Qual álbum do grupo Novos Baianos tem esta capa com os membros reunidos em um sofá?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/6/64/Acabou_chorare.jpg", "answers": ["Novos Baianos F.C.", "Acabou Chorare", "Vamos pro Mundo", "Farol da Barra"], "correct": 1 },
      { "question": "A capa deste álbum, um marco da Bossa Nova, mostra apenas os rostos dos dois intérpretes em preto e branco.", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/2/26/Elis_%26_Tom.jpg", "answers": ["Chega de Saudade", "Elis & Tom", "Falso Brilhante", "Construção"], "correct": 1 },
      { "question": "Qual álbum de Chico Buarque tem uma capa que simula a primeira página de um jornal?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/5/5f/Chico-buarque-construcao-1971.jpg", "answers": ["Meus Caros Amigos", "Chico Buarque de Hollanda", "Construção", "Ópera do Malandro"], "correct": 2 },
      { "question": "A capa deste álbum clássico do rap nacional mostra os quatro membros do grupo em um fundo de tijolos.", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/8/87/Sobrevivendo_no_Inferno.jpg", "answers": ["Facção Central - A Marcha Fúnebre", "Racionais MC's - Sobrevivendo no Inferno", "Sabotage - Rap é Compromisso", "Thaíde & DJ Hum - Preste Atenção"], "correct": 1 },
      { "question": "Esta capa minimalista, com manequins em trajes vermelhos, é de qual banda pioneira da música eletrônica?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/8/8a/Kraftwerk_-_The_Man-Machine.jpg", "answers": ["Daft Punk", "Kraftwerk", "New Order", "Depeche Mode"], "correct": 1 },
      { "question": "A imagem de ondas de rádio de um pulsar estampa a capa de qual álbum de estreia do Joy Division?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/7/70/Unknown_Pleasures.jpg", "answers": ["Closer", "Substance", "Unknown Pleasures", "Still"], "correct": 2 },
      { "question": "Qual álbum do The Clash tem esta capa com o baixista Paul Simonon quebrando seu instrumento no palco?", "imageUrl": "https://upload.wikimedia.org/wikipedia/en/9/95/The_Clash_-_London_Calling.jpg", "answers": ["The Clash (álbum)", "Sandinista!", "Combat Rock", "London Calling"], "correct": 3 },
      { "question": "Qual banda de rock brasileira tem esta capa com os quatro membros nus, fotografados por Tuca Reinés?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/e/e3/Carbono_capa.jpg", "answers": ["Titãs - Cabeça Dinossauro", "Sepultura - Roots", "RPM - Rádio Pirata ao Vivo", "Lenine - Carbono"], "correct": 3 },
      { "question": "Esta capa, com uma pintura surrealista de um rosto com um ovo, é do álbum de estreia de qual banda brasileira?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/a/a2/Secos_%26_Molhados_%281973%29.jpg", "answers": ["Mutantes", "O Terço", "Secos & Molhados", "A Barca do Sol"], "correct": 2 },
      { "question": "A que dupla de música eletrônica pertence esta capa com os dois membros usando capacetes espaciais?", "imageUrl": "https://upload.wikimedia.org/wikipedia/pt/8/82/Daft_Punk_-_Random_Access_Memories.jpg", "answers": ["Justice", "The Chemical Brothers", "Daft Punk", "Air"], "correct": 2 }
    ]
  },
  "criancas_mundo_magico": {
    "name": "Mundo Mágico e Divertido",
    "questions": [
      { "question": "Qual princesa da Disney perdeu seu sapatinho de cristal em um baile?", "answers": ["Bela", "Ariel", "Cinderela", "Jasmine"], "correct": 2 },
      { "question": "Na história dos Três Porquinhos, qual material o lobo não conseguiu derrubar com seu sopro?", "answers": ["Palha", "Madeira", "Tijolos", "Algodão"], "correct": 2 },
      { "question": "Qual é o nome do boneco de neve falante do filme 'Frozen'?", "answers": ["Sven", "Kristoff", "Hans", "Olaf"], "correct": 3 },
      { "question": "Qual super-herói usa um martelo mágico e vem de um lugar chamado Asgard?", "answers": ["Hulk", "Homem de Ferro", "Capitão América", "Thor"], "correct": 3 },
      { "question": "Em 'Procurando Nemo', qual é a cor principal do peixinho Nemo?", "answers": ["Azul e amarelo", "Laranja e branco", "Verde e roxo", "Todo preto"], "correct": 1 },
      { "question": "Qual personagem dos contos de fada tem um nariz que cresce quando ele mente?", "answers": ["Peter Pan", "Pinóquio", "Aladdin", "Gepeto"], "correct": 1 },
      { "question": "Qual o nome do melhor amigo de Woody em 'Toy Story'?", "answers": ["Buzz Lightyear", "Sr. Cabeça de Batata", "Slinky", "Rex"], "correct": 0 },
      { "question": "Na história da Chapeuzinho Vermelho, quem estava na cama da vovó disfarçado?", "answers": ["Um caçador", "Um urso", "O Lobo Mau", "Um lenhador"], "correct": 2 },
      { "question": "Qual o nome da equipe de cachorrinhos heróis que ajuda a cidade da Baía da Aventura?", "answers": ["Super Cães", "Patrulha Canina", "Esquadrão Pet", "Amigos da Aventura"], "correct": 1 },
      { "question": "Qual o nome da pequena sereia que sonhava em ter pernas para viver na terra?", "answers": ["Adella", "Aquata", "Andrina", "Ariel"], "correct": 3 },
      { "question": "Qual é a comida favorita do Garfield?", "answers": ["Pizza", "Hambúrguer", "Lasanha", "Peixe"], "correct": 2 },
      { "question": "Que animal mágico tem um chifre na testa e costuma ser branco?", "answers": ["Dragão", "Unicórnio", "Grifo", "Pégaso"], "correct": 1 },
      { "question": "No filme 'O Rei Leão', qual é o nome do filhote de leão que é o personagem principal?", "answers": ["Mufasa", "Scar", "Simba", "Zazu"], "correct": 2 },
      { "question": "Qual o nome do menino que nunca cresce e vive na Terra do Nunca?", "answers": ["Capitão Gancho", "Peter Pan", "Smee", "João"], "correct": 1 },
      { "question": "Qual o poder da super-heroína Mulher-Maravilha?", "answers": ["Soltar teias", "Super força e agilidade", "Ficar invisível", "Controlar o gelo"], "correct": 1 },
      { "question": "Qual o nome do ogro verde que vive em um pântano e é amigo de um burro falante?", "answers": ["Fiona", "Lord Farquaad", "Gato de Botas", "Shrek"], "correct": 3 },
      { "question": "Qual o nome da irmã de Elsa no filme 'Frozen'?", "answers": ["Moana", "Anna", "Merida", "Rapunzel"], "correct": 1 },
      { "question": "Qual o nome do herói que usa uma roupa de aranha e solta teias?", "answers": ["Batman", "Superman", "Homem-Aranha", "Flash"], "correct": 2 },
      { "question": "Na história 'João e o Pé de Feijão', o que João troca pelas sementes de feijão?", "answers": ["Um saco de moedas", "Sua casa", "Uma galinha", "Sua vaca"], "correct": 3 },
      { "question": "Qual o nome da porquinha rosa mais famosa dos desenhos animados?", "answers": ["Galinha Pintadinha", "Mônica", "Peppa Pig", "Pocoyo"], "correct": 2 }
    ]
  },
  "criancas_pequeno_curioso": {
    "name": "Descobertas do Pequeno Curioso",
    "questions": [
      { "question": "Qual é o animal mais alto do mundo?", "answers": ["Elefante", "Girafa", "Baleia", "Dinossauro"], "correct": 1 },
      { "question": "Qual é a estrela que aparece durante o dia e aquece o nosso planeta?", "answers": ["Lua", "Marte", "Sol", "Cometa"], "correct": 2 },
      { "question": "Quantos continentes existem no planeta Terra?", "answers": ["Três", "Cinco", "Sete", "Dez"], "correct": 2 },
      { "question": "Qual animal é conhecido como o 'Rei da Selva'?", "answers": ["Tigre", "Urso", "Macaco", "Leão"], "correct": 3 },
      { "question": "Qual a cor que obtemos quando misturamos o amarelo com o azul?", "answers": ["Vermelho", "Laranja", "Roxo", "Verde"], "correct": 3 },
      { "question": "Qual órgão do nosso corpo é responsável por bombear o sangue para todo lugar?", "answers": ["Cérebro", "Pulmão", "Coração", "Estômago"], "correct": 2 },
      { "question": "Como se chama a casa das abelhas?", "answers": ["Ninho", "Toca", "Colmeia", "Formigueiro"], "correct": 2 },
      { "question": "Qual dinossauro é famoso por ser um caçador gigante com braços bem curtinhos?", "answers": ["Tiranossauro Rex", "Tricerátops", "Braquiossauro", "Velociraptor"], "correct": 0 },
      { "question": "O que as plantas usam para produzir seu próprio alimento com a luz do sol?", "answers": ["Chuva", "Vento", "Fotossíntese", "Terra"], "correct": 2 },
      { "question": "Qual é o maior oceano do nosso planeta?", "answers": ["Atlântico", "Índico", "Pacífico", "Ártico"], "correct": 2 },
      { "question": "Quantas patas tem uma aranha?", "answers": ["Quatro", "Seis", "Oito", "Dez"], "correct": 2 },
      { "question": "Qual planeta é conhecido como 'Planeta Vermelho'?", "answers": ["Vênus", "Júpiter", "Saturno", "Marte"], "correct": 3 },
      { "question": "O que os ursos costumam fazer durante o inverno para economizar energia?", "answers": ["Fazem uma festa", "Viajam para o sul", "Hibernam", "Constroem casas de gelo"], "correct": 2 },
      { "question": "Como se chama o esqueleto de um peixe?", "answers": ["Osso", "Carapaça", "Espinha", "Crânio"], "correct": 2 },
      { "question": "Qual o nome do satélite natural que gira ao redor da Terra e brilha no céu à noite?", "answers": ["Sol", "Estrela Cadente", "Lua", "Cometa Halley"], "correct": 2 },
      { "question": "Qual o animal que late e é o melhor amigo do homem?", "answers": ["Gato", "Cavalo", "Cachorro", "Coelho"], "correct": 2 },
      { "question": "De qual animal obtemos o leite que bebemos?", "answers": ["Da galinha", "Da vaca", "Do porco", "Do pato"], "correct": 1 },
      { "question": "Como se chama a chuva bem fraquinha?", "answers": ["Tempestade", "Garoa", "Tornado", "Nevasca"], "correct": 1 },
      { "question": "Qual é o sentido que usamos para sentir o cheiro das coisas?", "answers": ["Visão", "Audição", "Paladar", "Olfato"], "correct": 3 },
      { "question": "Qual o nome do grande conjunto de árvores e animais que chamamos de 'pulmão do mundo' no Brasil?", "answers": ["Praia de Copacabana", "Floresta Amazônica", "Pão de Açúcar", "Cataratas do Iguaçu"], "correct": 1 }
    ]
  },
  "dinossauros_facil": {
    "name": "Dinossauros (Exploradores Jurássicos)",
    "questions": [
      { "question": "O que significa o nome 'Tiranossauro Rex'?", "answers": ["Lagarto Terrível", "Rei dos Lagartos Tiranos", "Ladrão Veloz", "Rosto de Três Chifres"], "correct": 1 },
      { "question": "Qual dinossauro é famoso por ter uma fileira de grandes placas nas costas e espigões na cauda?", "answers": ["Tricerátops", "Estegossauro", "Anquilossauro", "Braquiossauro"], "correct": 1 },
      { "question": "O Tricerátops era um dinossauro herbívoro. O que isso significa?", "answers": ["Que ele comia carne", "Que ele comia plantas", "Que ele comia peixes", "Que ele comia de tudo"], "correct": 1 },
      { "question": "Qual evento cataclísmico é a teoria mais aceita para a extinção dos dinossauros?", "answers": ["Uma era do gelo", "Uma grande inundação", "A queda de um asteroide", "Doenças"], "correct": 2 },
      { "question": "Qual dinossauro pescoçudo é um dos maiores animais que já andaram na Terra?", "answers": ["Tiranossauro Rex", "Velociraptor", "Braquiossauro", "Estegossauro"], "correct": 2 },
      { "question": "Como os cientistas que estudam fósseis de dinossauros são chamados?", "answers": ["Arqueólogos", "Geólogos", "Biólogos", "Paleontólogos"], "correct": 3 },
      { "question": "O Velociraptor era um caçador pequeno e ágil. Em que período ele viveu?", "answers": ["Triássico", "Jurássico", "Cretáceo", "Paleogeno"], "correct": 2 },
      { "question": "Qual dinossauro era conhecido como um 'tanque de guerra' vivo, por sua couraça óssea e clava na cauda?", "answers": ["Anquilossauro", "Tricerátops", "Parassaurolofo", "Iguanodonte"], "correct": 0 },
      { "question": "O que são fósseis?", "answers": ["Pedras preciosas", "Ossos de animais atuais", "Restos ou vestígios de seres vivos muito antigos", "Plantas raras"], "correct": 2 },
      { "question": "Qual dinossauro voador é o mais famoso, embora tecnicamente não fosse um dinossauro?", "answers": ["Arqueoptérix", "Pterodáctilo", "Dimorfodonte", "Quetzalcoatlus"], "correct": 1 },
      { "question": "Qual o nome do supercontinente que existia na época em que os primeiros dinossauros surgiram?", "answers": ["Gondwana", "Laurásia", "Pangeia", "Rodínia"], "correct": 2 },
      { "question": "Qual dinossauro tinha três chifres no rosto para se defender?", "answers": ["Estegossauro", "Tiranossauro Rex", "Tricerátops", "Alossauro"], "correct": 2 },
      { "question": "O Espinossauro é famoso por uma característica única nas costas. Qual é?", "answers": ["Placas ósseas", "Uma grande 'vela' ou barbatana", "Espinhos venenosos", "Uma concha"], "correct": 1 },
      { "question": "Os dinossauros botavam ovos. Por isso, são classificados como:", "answers": ["Mamíferos", "Anfíbios", "Ovíparos", "Marsupiais"], "correct": 2 },
      { "question": "Qual desses dinossauros era carnívoro?", "answers": ["Braquiossauro", "Parassaurolofo", "Tricerátops", "Alossauro"], "correct": 3 },
      { "question": "A Era Mesozoica é dividida em três períodos. Qual deles veio primeiro?", "answers": ["Triássico", "Jurássico", "Cretáceo", "Todos ao mesmo tempo"], "correct": 0 },
      { "question": "Qual dinossauro tinha um crânio em forma de cúpula, que possivelmente era usado em disputas de cabeçadas?", "answers": ["Paquicefalossauro", "Coritossauro", "Iguanodonte", "Diplodoco"], "correct": 0 },
      { "question": "Qual dinossauro marinho gigante, embora não fosse um dinossauro, dominou os mares no período Cretáceo?", "answers": ["Plesiossauro", "Ictiossauro", "Mosasassauro", "Tubarão Megalodon"], "correct": 2 },
      { "question": "De acordo com muitos cientistas, qual grupo de animais de hoje são os descendentes diretos dos dinossauros terópodes?", "answers": ["Lagartos", "Crocodilos", "Aves", "Cobras"], "correct": 2 },
      { "question": "Em qual continente foram encontrados os primeiros fósseis de Iguanodonte, um dos primeiros dinossauros a ser descrito?", "answers": ["América do Sul", "África", "Europa", "Ásia"], "correct": 2 }
    ]
  },
"dinossauros_dificil": {
    "name": "Dinossauros (Paleontólogo Expert)",
    "questions": [
      { "question": "Qual clado de dinossauros se caracteriza pela estrutura da pélvis 'semelhante à de um pássaro', com o osso púbico apontado para trás?", "answers": ["Saurísquios", "Terópodes", "Ornitísquios", "Sauropodomorfos"], "correct": 2 },
      { "question": "O que são 'gastrólitos', frequentemente encontrados em fósseis de dinossauros saurópodes?", "answers": ["Ovos fossilizados", "Pedras engolidas para ajudar na digestão", "Coprólitos (fezes fossilizadas)", "Marcas de dentes em ossos"], "correct": 1 },
      { "question": "A 'Guerra dos Ossos' foi uma rivalidade acirrada no final do século XIX entre quais dois paleontólogos americanos?", "answers": ["Barnum Brown e Charles Sternberg", "Othniel Marsh e Edward Cope", "Roy Chapman Andrews e Paul Sereno", "John Ostrom e Robert Bakker"], "correct": 1 },
      { "question": "O Archaeopteryx é um fóssil de transição crucial que demonstra a ligação evolutiva entre dinossauros terópodes e qual grupo de animais?", "answers": ["Mamíferos", "Pterossauros", "Aves", "Crocodilianos"], "correct": 2 },
      { "question": "Qual o nome da abertura (fenestra) extra no crânio, localizada entre a órbita ocular e a narina, que é uma característica dos arcossauros, incluindo os dinossauros?", "answers": ["Fenestra Mandibular", "Fenestra Supratemporal", "Fenestra Antorbital", "Forame Magno"], "correct": 2 },
      { "question": "O Therizinosaurus é um terópode bizarro conhecido por qual característica anatômica desproporcional?", "answers": ["Dentes minúsculos", "Pernas extremamente curtas", "Garras gigantes nas mãos", "Uma cabeça com chifres"], "correct": 2 },
      { "question": "Qual famosa formação geológica na América do Norte é uma das fontes mais ricas de fósseis do Jurássico Superior?", "answers": ["Formação Hell Creek", "Formação Morrison", "Folhelho de Burgess", "Formação Green River"], "correct": 1 },
      { "question": "O Argentinossauro, um dos maiores dinossauros conhecidos, pertence a qual grupo de saurópodes?", "answers": ["Diplodocídeos", "Braquiossaurídeos", "Titanossauros", "Camarasaurídeos"], "correct": 2 },
      { "question": "Como é chamada a clavícula fundida (osso da sorte) encontrada em dinossauros terópodes, uma característica compartilhada com as aves modernas?", "answers": ["Esterno", "Pigoestilo", "Fúrcula", "Coracoide"], "correct": 2 },
      { "question": "Qual o nome do primeiro dinossauro a ser descrito e nomeado cientificamente?", "answers": ["Iguanodonte", "Megalossauro", "Tricerátops", "Hadrossauro"], "correct": 1 },
      { "question": "O que é a tafonomia no campo da paleontologia?", "answers": ["O estudo da classificação dos dinossauros", "O estudo do processo de fossilização, desde a morte até a descoberta", "O estudo de ovos e ninhos de dinossauros", "O estudo da biomecânica do movimento dos dinossauros"], "correct": 1 },
      { "question": "O Giganotossauro, um carnívoro gigante, viveu na América do Sul e pertence a qual família de terópodes?", "answers": ["Tiranossaurídeos", "Abelissaurídeos", "Alossaurídeos", "Carcharodontossaurídeos"], "correct": 3 },
      { "question": "Qual dinossauro 'bico-de-pato' (hadrossaurídeo) é famoso por sua crista oca, que poderia ser usada para vocalização?", "answers": ["Edmontossauro", "Maiassaura", "Parassaurolofo", "Lambeossauro"], "correct": 2 },
      { "question": "Qual período da Era Mesozoica teve a maior diversidade de espécies de dinossauros?", "answers": ["Triássico Inferior", "Jurássico Médio", "Cretáceo Superior", "Todos tiveram diversidade similar"], "correct": 2 },
      { "question": "Mary Anning, uma das pioneiras da paleontologia, foi famosa por descobrir importantes fósseis de répteis marinhos em qual localidade?", "answers": ["Lyme Regis, Inglaterra", "Solnhofen, Alemanha", "Deserto de Gobi, Mongólia", "Patagônia, Argentina"], "correct": 0 },
      { "question": "O que significa o termo 'dinossauro saurísquio'?", "answers": ["'Lagarto com quadril de ave'", "'Lagarto com quadril de lagarto'", "'Lagarto com pés de ave'", "'Lagarto com chifres'"], "correct": 1 },
      { "question": "Qual pequeno dinossauro, cujo nome significa 'ladrão rápido', é frequentemente (e incorretamente) retratado em filmes como sendo muito maior do que era?", "answers": ["Deinonico", "Utahraptor", "Velociraptor", "Troodonte"], "correct": 2 },
      { "question": "Os fósseis de 'Sue', um dos mais completos T. rex já encontrados, estão em exibição em qual museu?", "answers": ["Museu Americano de História Natural (Nova York)", "Museu Field de História Natural (Chicago)", "Museu Nacional de História Natural (Washington, D.C.)", "Museu de História Natural (Londres)"], "correct": 1 },
      { "question": "O que são 'icnofósseis'?", "answers": ["Fósseis de plantas", "Fósseis de dentes", "Fósseis de vestígios, como pegadas e tocas", "Microfósseis"], "correct": 2 },
      { "question": "Qual o nome do impacto que marca a fronteira K-Pg, associado à extinção dos dinossauros?", "answers": ["Cratera de Vredefort", "Cratera da Baía de Chesapeake", "Cratera de Chicxulub", "Cratera de Manicouagan"], "correct": 2 }
    ]
  },
 "franquia_jurassica": {
    "name": "Franquia Jurássica",
    "questions": [
      { "question": "No primeiro 'Jurassic Park', qual tipo de DNA foi usado para preencher as lacunas nas sequências de genoma dos dinossauros?", "answers": ["DNA de Pássaro", "DNA de Lagarto", "DNA de Sapo", "DNA de Crocodilo"], "correct": 2 },
      { "question": "Qual é o nome do matemático especialista em Teoria do Caos que é convidado para avaliar o parque?", "answers": ["Dr. Alan Grant", "John Hammond", "Robert Muldoon", "Dr. Ian Malcolm"], "correct": 3 },
      { "question": "Em 'O Mundo Perdido: Jurassic Park', a expedição da InGen vai para qual ilha, conhecida como 'Sítio B'?", "answers": ["Ilha Nublar", "Ilha Sorna", "Ilha das Cinco Mortes", "Ilha Pena"], "correct": 1 },
      { "question": "Qual é o nome da principal velociraptor treinada por Owen Grady em 'Jurassic World'?", "answers": ["Delta", "Echo", "Charlie", "Blue"], "correct": 3 },
      { "question": "Qual o nome do primeiro dinossauro híbrido criado pela InGen, que escapa e causa o caos em 'Jurassic World'?", "answers": ["Indoraptor", "Indominus Rex", "Ultimasaurus", "Spinosaurus Rex"], "correct": 1 },
      { "question": "Qual a frase icônica dita pelo Dr. Ian Malcolm que resume a principal falha do parque?", "answers": ["'A vida não pode ser contida'", "'Bem-vindos ao Jurassic Park'", "'A vida encontra um meio'", "'Segurem seus traseiros'"], "correct": 2 },
      { "question": "Em 'Jurassic Park III', qual dinossauro carnívoro gigante derrota um Tiranossauro Rex em uma luta?", "answers": ["Giganotossauro", "Alossauro", "Carcharodontossauro", "Espinossauro"], "correct": 3 },
      { "question": "No filme original, qual dinossauro é o primeiro a ser visto em sua forma completa pelos visitantes do parque?", "answers": ["Tricerátops", "Braquiossauro", "Tiranossauro Rex", "Velociraptor"], "correct": 1 },
      { "question": "Em 'Jurassic World: Reino Ameaçado', o que ameaça destruir todos os dinossauros restantes na Ilha Nublar?", "answers": ["Um furacão", "Uma erupção vulcânica", "Uma nova era do gelo", "Uma doença"], "correct": 1 },
      { "question": "Qual o nome da empresa de bioengenharia fundada por John Hammond?", "answers": ["Masrani Global", "BioSyn", "InGen", "DinoTech"], "correct": 2 },
      { "question": "Qual item Dennis Nedry usa para contrabandear os embriões de dinossauro para fora do parque?", "answers": ["Uma garrafa térmica", "Uma lata de creme de barbear", "Uma maleta com fundo falso", "Um pacote de salgadinhos"], "correct": 1 },
      { "question": "Qual o nome do dinossauro híbrido e extremamente perigoso que é o principal antagonista em 'Jurassic World: Reino Ameaçado'?", "answers": ["Indominus Rex", "Scorpios Rex", "Indoraptor", "Allosinosaurus"], "correct": 2 },
      { "question": "Em 'Jurassic World: Domínio', os dinossauros estão vivendo ao lado dos humanos em qual continente principal?", "answers": ["África", "Austrália", "América do Sul", "Em todo o mundo"], "correct": 3 },
      { "question": "Qual personagem do filme original retorna para um papel principal em 'Jurassic Park III'?", "answers": ["Ian Malcolm", "Ellie Sattler", "Alan Grant", "John Hammond"], "correct": 2 },
      { "question": "No primeiro filme, o T. rex escapa de seu cercado após o quê?", "answers": ["Um terremoto", "Uma falha no sistema elétrico", "Uma tempestade tropical", "Os raptores o libertam"], "correct": 1 },
      { "question": "Em 'O Mundo Perdido', o que a equipe de Ian Malcolm tenta impedir que a InGen faça?", "answers": ["Clonar novos dinossauros", "Capturar dinossauros e levá-los para o continente", "Destruir a Ilha Sorna", "Criar dinossauros híbridos"], "correct": 1 },
      { "question": "Qual dinossauro pequeno e venenoso ataca Dennis Nedry em seu jipe?", "answers": ["Compsognato", "Dilofossauro", "Procompsognato", "Celófise"], "correct": 1 },
      { "question": "Qual é a profissão de Owen Grady em 'Jurassic World'?", "answers": ["Paleontólogo", "Geneticista", "Treinador de animais (Velociraptors)", "Chefe de segurança"], "correct": 2 },
      { "question": "Em qual local o Indoraptor é leiloado em 'Jurassic World: Reino Ameaçado'?", "answers": ["Em um cassino de Las Vegas", "Na Mansão Lockwood", "Na sede da Masrani Global", "Em uma base militar secreta"], "correct": 1 },
      { "question": "Qual empresa rival da InGen, liderada por Lewis Dodgson, desempenha um papel importante em 'Jurassic Park: Domínio'?", "answers": ["DinoGuard", "Apex Cybernetics", "BioSyn", "GenomaX"], "correct": 2 }
    ]
  },
        };

        const lobbyScreen = document.getElementById('lobby-screen');
        const waitingRoomScreen = document.getElementById('waiting-room-screen');
        const gameScreen = document.getElementById('game-screen');
        const endScreen = document.getElementById('end-screen');
        const screens = [lobbyScreen, waitingRoomScreen, gameScreen, endScreen];

        function showScreen(screenToShow) {
            screens.forEach(screen => screen.classList.add('hidden'));
            screenToShow.classList.remove('hidden');
        }

        async function setupAuth() {
            onAuthStateChanged(auth, (user) => {
                if (user) {
                    currentUser = user;
                    document.getElementById('user-id-display').textContent = user.uid.substring(0, 8);
                } else {
                    signInAnonymously(auth).catch(error => console.error("Anonymous sign-in failed", error));
                }
            });
             if (auth.currentUser) {
                currentUser = auth.currentUser;
             } else {
                await signInAnonymously(auth);
            }
        }
        
        function getValidPlayerName() {
            const playerName = document.getElementById('player-name').value.trim();
            if (!playerName) {
                document.getElementById('error-message').textContent = 'Por favor, digite seu nome!';
                return null;
            }
            if (playerName.length > 15) {
                document.getElementById('error-message').textContent = 'O nome deve ter no máximo 15 caracteres.';
                return null;
            }
            document.getElementById('error-message').textContent = '';
            return playerName;
        }

        document.getElementById('create-game-btn').addEventListener('click', async () => {
            if (!firebaseConfig.apiKey) {
                document.getElementById('error-message').textContent = 'A configuração do Firebase está em falta. Edite o ficheiro HTML.';
                return;
            }
            const playerName = getValidPlayerName();
            if (!playerName || !currentUser) return;
            
            const gameCode = Math.random().toString(36).substring(2, 7).toUpperCase();
            currentGameId = gameCode;
            
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", gameCode);
            const playerRef = doc(db, "artifacts", appId, "public/data/quiz_games", gameCode, "players", currentUser.uid);

            const batch = writeBatch(db);
            batch.set(gameRef, {
                hostId: currentUser.uid,
                gameState: 'lobby',
                currentQuestionIndex: -1,
                quizId: null,
                createdAt: serverTimestamp()
            });
            batch.set(playerRef, {
                name: playerName,
                score: 0,
                hasAnswered: false,
                answeredQuestionIndex: null,
                selectedAnswer: null
            });
            await batch.commit();

            await attachToGame(gameCode);
        });

        document.getElementById('join-game-btn').addEventListener('click', async () => {
            if (!firebaseConfig.apiKey) {
                document.getElementById('error-message').textContent = 'A configuração do Firebase está em falta. Fale com o criador da sala.';
                return;
            }
            const playerName = getValidPlayerName();
            const gameCode = document.getElementById('game-code-input').value.trim().toUpperCase();

            if (!playerName || !gameCode || !currentUser) {
                 document.getElementById('error-message').textContent = 'Nome e código da sala são obrigatórios.';
                 return;
            }
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", gameCode);
            const gameDoc = await getDoc(gameRef);

            if (gameDoc.exists() && gameDoc.data().gameState === 'lobby') {
                currentGameId = gameCode;
                const playerRef = doc(db, "artifacts", appId, "public/data/quiz_games", gameCode, "players", currentUser.uid);
                await setDoc(playerRef, { 
                    name: playerName, 
                    score: 0, 
                    hasAnswered: false,
                    answeredQuestionIndex: null,
                    selectedAnswer: null
                });
                await attachToGame(gameCode);
            } else {
                document.getElementById('error-message').textContent = 'Sala não encontrada ou o jogo já começou.';
            }
        });

        async function attachToGame(gameCode) {
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", gameCode);
            const playersRef = collection(db, "artifacts", appId, "public/data/quiz_games", gameCode, "players");
            
            unsubscribeGame = onSnapshot(gameRef, (doc) => {
                currentGameData = doc.data();
                if (!currentGameData) { resetGame(); return; }
                updateGameStateUI(currentGameData);
                checkIfAllAnswered();
            }, error => {
                console.error("Erro no listener do jogo: ", error);
                document.getElementById('error-message').textContent = 'Erro ao ligar à sala. Verifique a sua ligação.';
            });
            
            unsubscribePlayers = onSnapshot(playersRef, (snapshot) => {
                currentPlayers = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                updatePlayersUI(currentPlayers);
                checkIfAllAnswered();
            }, error => {
                console.error("Erro no listener dos jogadores: ", error);
            });
        }
        
        function updateGameStateUI(gameData) {
            switch(gameData.gameState) {
                case 'lobby':
                    showScreen(waitingRoomScreen);
                    document.getElementById('game-code-text').textContent = currentGameId;
                    if (currentUser.uid === gameData.hostId) {
                        document.getElementById('host-controls').classList.remove('hidden');
                        document.getElementById('waiting-for-host').classList.add('hidden');
                        renderQuizOptions();
                    } else {
                        document.getElementById('host-controls').classList.add('hidden');
                        document.getElementById('waiting-for-host').classList.remove('hidden');
                    }
                    break;
                case 'in-progress':
                    showScreen(gameScreen);
                    renderQuestion(gameData);
                    break;
                case 'finished':
                    showScreen(endScreen);
                    break;
            }
        }
        
        function checkIfAllAnswered() {
            const nextButtonControls = document.getElementById('next-question-controls');
            if (!currentGameData || currentPlayers.length === 0 || currentGameData.gameState !== 'in-progress') {
                nextButtonControls.classList.add('hidden');
                return;
            }

            const allAnsweredThisQuestion = currentPlayers.every(p => p.hasAnswered && p.answeredQuestionIndex === currentGameData.currentQuestionIndex);
            
            if (allAnsweredThisQuestion) {
                showVoteCounts();
            }

            const isHost = currentUser && currentUser.uid === currentGameData.hostId;

            if (allAnsweredThisQuestion && isHost) {
                nextButtonControls.classList.remove('hidden');
            } else {
                nextButtonControls.classList.add('hidden');
            }
        }

        function renderQuizOptions() {
            const container = document.getElementById('quiz-options-container');
            container.innerHTML = '';
            Object.keys(quizzes).forEach((quizId, index) => {
                const quiz = quizzes[quizId];
                const button = document.createElement('button');
                button.className = 'btn btn-secondary quiz-option';
                button.innerHTML = `<span class="mr-3">${index + 1}.</span> ${quiz.name}`;
                button.dataset.quizId = quizId;
                button.onclick = () => {
                    selectedQuizId = quizId;
                    document.querySelectorAll('.quiz-option').forEach(btn => btn.classList.remove('selected'));
                    button.classList.add('selected');
                    document.getElementById('start-game-btn').disabled = false;
                };
                container.appendChild(button);
            });
        }

        function updatePlayersUI(players) {
            const playerList = document.getElementById('player-list');
            playerList.innerHTML = '';
            players.forEach(player => {
                const li = document.createElement('li');
                li.className = 'bg-indigo-100 text-indigo-800 font-semibold p-3 rounded-lg flex items-center justify-between';
                const answeredCheck = (player.hasAnswered && currentGameData && player.answeredQuestionIndex === currentGameData.currentQuestionIndex) ? '✔️' : '';
                li.innerHTML = `${player.name} ${answeredCheck}`;
                playerList.appendChild(li);
            });

            const sortedPlayers = [...players].sort((a, b) => b.score - a.score);
            const scoreboardList = document.getElementById('scoreboard-list');
            scoreboardList.innerHTML = '';
            sortedPlayers.forEach(player => {
                const li = document.createElement('li');
                li.className = 'flex justify-between items-center p-2 bg-gray-50 rounded-md';
                li.innerHTML = `<span class="font-medium text-gray-700">${player.name}</span> <span class="font-bold text-indigo-600">${player.score} pontos</span>`;
                scoreboardList.appendChild(li);
            });
             
            const finalScoreboardList = document.getElementById('final-scoreboard-list');
            finalScoreboardList.innerHTML = '';
            sortedPlayers.forEach((player, index) => {
                const medal = ['🥇', '🥈', '🥉'][index] || '';
                const li = document.createElement('li');
                li.className = `text-lg p-3 rounded-lg flex justify-between items-center ${index === 0 ? 'bg-yellow-100' : 'bg-gray-100'}`;
                li.innerHTML = `<span class="font-semibold">${medal} ${player.name}</span> <span class="font-bold text-gray-800">${player.score} pontos</span>`;
                finalScoreboardList.appendChild(li);
            });

            if (sortedPlayers.length > 0) {
                 document.getElementById('winner-name').textContent = sortedPlayers[0].name;
            }
        }
        
        function renderQuestion(gameData) {
            const quiz = quizzes[gameData.quizId];
            if (!quiz) return;
            const questionIndex = gameData.currentQuestionIndex;
            if (questionIndex < 0 || questionIndex >= quiz.questions.length) return;
            
            const questionObj = quiz.questions[questionIndex];
            
            document.getElementById('quiz-name-display').textContent = quiz.name;
            document.getElementById('question-number').textContent = questionIndex + 1;
            document.getElementById('total-questions').textContent = quiz.questions.length;
            
            const questionContainer = document.getElementById('question-container');
            questionContainer.innerHTML = ''; // Limpa conteúdo anterior

            if (questionObj.imageUrl) {
                const image = document.createElement('img');
                image.src = questionObj.imageUrl;
                image.alt = 'Imagem da pergunta';
                image.className = 'question-image';
                image.onerror = () => { 
                    image.src = `https://placehold.co/160x107/eee/ccc?text=Imagem`; 
                    image.alt = 'Erro ao carregar imagem';
                };
                questionContainer.appendChild(image);
            }

            const questionText = document.createElement('p');
            questionText.className = 'text-2xl font-semibold text-gray-800 text-center';
            questionText.textContent = questionObj.question;
            questionContainer.appendChild(questionText);
            
            const answersContainer = document.getElementById('answers-container');
            answersContainer.innerHTML = '';
            document.getElementById('feedback-container').innerHTML = '';
            
            questionObj.answers.forEach((answer, index) => {
                const button = document.createElement('button');
                button.className = 'btn btn-secondary';
                
                const answerText = document.createElement('span');
                answerText.textContent = answer;
                button.appendChild(answerText);

                button.dataset.index = index;
                button.onclick = handleAnswerSelection;
                answersContainer.appendChild(button);
            });
        }

        async function handleAnswerSelection(event) {
            const button = event.currentTarget;
            const selectedIndex = parseInt(button.dataset.index);
            const playerRef = doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId, "players", currentUser.uid);
            
            const playerDoc = await getDoc(playerRef);
            if(playerDoc.data().hasAnswered && playerDoc.data().answeredQuestionIndex === currentGameData.currentQuestionIndex) return;

            const gameDoc = await getDoc(doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId));
            const gameData = gameDoc.data();
            const question = quizzes[gameData.quizId].questions[gameData.currentQuestionIndex];
            
            const updates = {
                hasAnswered: true,
                selectedAnswer: selectedIndex,
                answeredQuestionIndex: gameData.currentQuestionIndex
            };
            if(selectedIndex === question.correct) {
                updates.score = playerDoc.data().score + 10;
            }

            await updateDoc(playerRef, updates);

            const answerButtons = document.querySelectorAll('#answers-container button');
            answerButtons.forEach(btn => btn.disabled = true);
            button.classList.add('ring-4', 'ring-indigo-500');
        }

        function showVoteCounts() {
            const allHaveAnsweredCurrentQuestion = currentPlayers.every(p => p.hasAnswered && p.answeredQuestionIndex === currentGameData.currentQuestionIndex);
            if (!allHaveAnsweredCurrentQuestion) {
                return;
            }

            const answerButtons = document.querySelectorAll('#answers-container button');
            if(answerButtons.length === 0 || answerButtons[0].querySelector('.vote-count-badge')) return; // Already shown

            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId);
            
            getDoc(gameRef).then(gameDoc => {
                const gameData = gameDoc.data();
                const question = quizzes[gameData.quizId].questions[gameData.currentQuestionIndex];
                
                const voteCounts = [0, 0, 0, 0];
                currentPlayers.forEach(p => {
                    if (p.answeredQuestionIndex === gameData.currentQuestionIndex && p.selectedAnswer !== null) {
                        voteCounts[p.selectedAnswer]++;
                    }
                });

                answerButtons.forEach((button, index) => {
                    const badge = document.createElement('span');
                    badge.className = 'vote-count-badge bg-gray-300 text-gray-700';
                    badge.textContent = voteCounts[index];
                    button.appendChild(badge);
                    
                    if (index === question.correct) {
                        button.classList.remove('btn-secondary', 'ring-indigo-500');
                        button.classList.add('bg-green-500', 'text-white');
                        badge.classList.remove('bg-gray-300', 'text-gray-700');
                        badge.classList.add('bg-green-700', 'text-white');
                    } else if (button.classList.contains('ring-4')) {
                         button.classList.remove('btn-secondary', 'ring-indigo-500');
                         button.classList.add('bg-red-500', 'text-white');
                         badge.classList.remove('bg-gray-300', 'text-gray-700');
                         badge.classList.add('bg-red-700', 'text-white');
                    } else {
                         button.classList.add('opacity-60');
                    }
                });

                const myPlayer = currentPlayers.find(p => p.id === currentUser.uid);
                const feedbackEl = document.getElementById('feedback-container');
                if (myPlayer && myPlayer.answeredQuestionIndex === gameData.currentQuestionIndex) {
                    if (myPlayer.selectedAnswer === question.correct) {
                         feedbackEl.textContent = 'Resposta Certa! +10 pontos';
                         feedbackEl.className = 'mt-6 text-center text-xl font-bold text-green-600';
                    } else {
                         feedbackEl.textContent = 'Resposta Errada!';
                         feedbackEl.className = 'mt-6 text-center text-xl font-bold text-red-600';
                    }
                }
            });
        }


        document.getElementById('start-game-btn').addEventListener('click', async () => {
            if (!selectedQuizId) return;
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId);
            await updateDoc(gameRef, {
                gameState: 'in-progress',
                currentQuestionIndex: 0,
                quizId: selectedQuizId
            });
        });

        document.getElementById('next-question-btn').addEventListener('click', async () => {
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId);
            const gameDoc = await getDoc(gameRef);
            const gameData = gameDoc.data();
            const nextIndex = gameData.currentQuestionIndex + 1;

            const playersRef = collection(db, "artifacts", appId, "public/data/quiz_games", currentGameId, "players");
            const batch = writeBatch(db);
            const playersSnapshot = await getDocs(playersRef);
            playersSnapshot.forEach(playerDoc => {
                batch.update(playerDoc.ref, { hasAnswered: false, selectedAnswer: null, answeredQuestionIndex: null });
            });

            const quiz = quizzes[gameData.quizId];
            if (nextIndex < quiz.questions.length) {
                batch.update(gameRef, { currentQuestionIndex: nextIndex });
            } else {
                batch.update(gameRef, { gameState: 'finished' });
            }
            await batch.commit();
        });
        
         document.getElementById('play-again-btn').addEventListener('click', async () => {
            if (!currentGameId || !currentUser) return;
            const gameRef = doc(db, "artifacts", appId, "public/data/quiz_games", currentGameId);
            const gameDoc = await getDoc(gameRef);
            if (gameDoc.data().hostId !== currentUser.uid) {
                resetGame();
                return;
            }
            
            const playersRef = collection(db, "artifacts", appId, "public/data/quiz_games", currentGameId, "players");
            const playersSnapshot = await getDocs(playersRef);
            const batch = writeBatch(db);

            batch.update(gameRef, { gameState: 'lobby', currentQuestionIndex: -1, quizId: null });
            playersSnapshot.forEach(playerDoc => {
                batch.update(playerDoc.ref, { score: 0, hasAnswered: false, selectedAnswer: null, answeredQuestionIndex: null });
            });
            await batch.commit();
        });
        
        function resetGame() {
            if (unsubscribeGame) unsubscribeGame();
            if (unsubscribePlayers) unsubscribePlayers();
            unsubscribeGame = null;
            unsubscribePlayers = null;
            currentGameId = null;
            selectedQuizId = null;
            currentPlayers = [];
            currentGameData = null;

            document.getElementById('game-code-input').value = '';
            document.getElementById('error-message').textContent = '';
            document.getElementById('player-name').value = '';

            showScreen(lobbyScreen);
        }

        document.getElementById('game-code-display').addEventListener('click', () => {
            const code = document.getElementById('game-code-text').textContent;
            navigator.clipboard.writeText(code).then(() => {
                const display = document.getElementById('game-code-display');
                const originalContent = display.innerHTML;
                display.innerHTML = 'Copiado!';
                setTimeout(() => { display.innerHTML = originalContent; }, 1500);
            });
        });

        const scoreboardModal = document.getElementById('scoreboard-modal');
        document.getElementById('scoreboard-toggle').addEventListener('click', () => scoreboardModal.classList.remove('hidden'));
        document.getElementById('close-scoreboard-btn').addEventListener('click', () => scoreboardModal.classList.add('hidden'));
        scoreboardModal.addEventListener('click', (e) => {
            if (e.target === scoreboardModal) {
                 scoreboardModal.classList.add('hidden');
            }
        });

        window.onload = () => {
            showScreen(lobbyScreen);
            setupAuth();
        };

    </script>
</body>
</html>
