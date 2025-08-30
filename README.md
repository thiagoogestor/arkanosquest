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
