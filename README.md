<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Quiz</title>
    <!-- 🟢 LOCATION 1: FAVICON (Replace the Base64 placeholder string here) -->
    <link rel="icon" type="image/png" href="data:image/png;base64,<BASE64_IMAGE_DATA_HERE>"/>
    <!-- Simulation of PWA readiness -->
    <link rel="manifest" href="/manifest.json">
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Use Inter font for clean readability */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #1a1a2e; /* Deep, slightly bluish dark background */
            color: #e0e0e0; /* Light gray text */
        }
        .arena-card {
            background-color: #1e2640; /* Dark card surface */
            border: 1px solid #3f4a6b;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
        }
        .timer-bar {
            height: 8px;
            transition: width 1s linear; /* Smooth visual transition */
            border-radius: 4px;
        }
        .option-button {
            transition: all 0.2s ease-in-out;
            background-color: #2e3a59; /* Darker button surface */
            color: #e0e0e0;
            border: 1px solid #3f4a6b;
        }
        .option-button:hover:not([disabled]) {
            background-color: #3f4a6b;
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .correct {
            background-color: #10b981 !important; /* Emerald green */
            border-color: #059669 !important;
            color: white !important;
            box-shadow: 0 0 10px rgba(16, 185, 129, 0.5) !important;
        }
        .incorrect {
            background-color: #ef4444 !important; /* Red */
            border-color: #dc2626 !important;
            color: white !important;
            box-shadow: 0 0 10px rgba(239, 68, 68, 0.5) !important;
        }
        .text-primary {
            color: #8a85ff; /* Brighter indigo/purple for dark contrast */
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">

    <!-- Main Quiz Container -->
    <div id="quiz-container" class="w-full max-w-lg arena-card rounded-xl p-8 md:p-10">

        <!-- App Icon and Title -->
        <div class="flex items-center justify-center mb-6">
            <!-- 🟢 LOCATION 2: APP HEADER IMAGE (Replace the Base64 placeholder string here) -->
            <img 
                id="app-icon" 
                src="data:image/png;base64,<BASE64_IMAGE_DATA_HERE>" 
                alt="Quiz App Icon" 
                onerror="this.style.display='none'"
                class="h-10 w-10 mr-3 rounded-xl shadow-lg"
            >
            <h1 class="text-3xl font-extrabold text-primary tracking-tight">
                Ultimate Quiz
            </h1>
        </div>

        <!-- Install Button (Hidden by default, shown by PWA event listener) -->
        <button id="install-button" class="bg-purple-600 hover:bg-purple-700 text-white font-semibold py-2 px-5 rounded-lg shadow-md transition duration-200 w-full mb-6" style="display: none;">
            Install App
        </button>

        <!-- Timer Area -->
        <div id="timer-area" class="mb-6 border-b pb-4 border-gray-700">
            <div class="flex justify-between items-center mb-2">
                <span class="text-sm font-medium text-gray-400">Time Remaining:</span>
                <span id="time-display" class="text-4xl font-extrabold text-primary">30</span>
            </div>
            <!-- Timer Progress Bar -->
            <div class="w-full bg-gray-700 rounded-full overflow-hidden">
                <div id="timer-bar" class="timer-bar bg-green-500" style="width: 100%;"></div>
            </div>
        </div>
        
        <!-- Question Area -->
        <div id="question-area">
            <p id="question-text" class="text-xl font-semibold text-gray-200 mb-8 p-4 bg-indigo-900/40 rounded-lg border border-indigo-700/50"></p>
            
            <!-- Options Container -->
            <div id="options-container" class="space-y-3">
                <!-- Buttons will be rendered here -->
            </div>
        </div>

        <!-- Feedback & Score Area -->
        <div class="mt-8 pt-4 border-t border-gray-700 flex justify-between items-center">
            <p class="text-base font-medium text-gray-400">Score: <span id="score-display" class="font-bold text-green-400 text-3xl">0</span></p>
            <p class="text-base font-medium text-gray-400">Challenge: <span id="current-question-count" class="font-bold text-primary text-2xl">1</span> / <span id="total-questions" class="font-bold text-primary text-2xl">10</span></p>
            <button id="next-button" class="bg-indigo-600 hover:bg-indigo-500 text-white font-semibold py-2 px-5 rounded-lg shadow-md transition duration-200" style="display: none;">
                Next &rarr;
            </button>
        </div>

        <!-- Result Screen -->
        <div id="result-screen" class="text-center p-10" style="display: none;">
            <h2 class="text-4xl font-extrabold text-green-400 mb-4 tracking-wide">QUIZ COMPLETE!</h2>
            <p class="text-lg text-gray-400 mb-6">Your Final Score:</p>
            <p id="final-score" class="text-7xl font-extrabold text-primary mb-10"></p>
            <button onclick="window.location.reload()" class="bg-red-600 hover:bg-red-500 text-white font-bold py-3 px-10 rounded-lg shadow-lg transition duration-200">
                START NEW QUIZ
            </button>
        </div>

    </div>

    <script>
        // --- Question Pool (Simulating a large pool of 200+ questions for randomization) ---
        // Over 50 questions are included below to ensure deep randomization and non-repeating sessions.
        const originalQuizData = [
            // Tech/Programming
            { question: "What is the primary function of the JavaScript 'map()' method?", options: ["Executes a function on every element.", "Creates a new array populated with the results of calling a provided function on every element.", "Finds the largest element in the array.", "Iterates over the elements and sorts them."], answer: "Creates a new array populated with the results of calling a provided function on every element." },
            { question: "Which CSS property controls the spacing between lines of text?", options: ["letter-spacing", "word-spacing", "line-height", "text-indent"], answer: "line-height" },
            { question: "What does 'DOM' stand for in web development?", options: ["Document Object Model", "Data Order Manager", "Digital Output Mechanism", "Design Object Mapping"], answer: "Document Object Model" },
            { question: "Which language is used for styling web pages?", options: ["HTML", "JavaScript", "Python", "CSS"], answer: "CSS" },
            { question: "What command is used to initialize a new Git repository?", options: ["git start", "git init", "git new", "git repo"], answer: "git init" },
            { question: "What data format is commonly used for exchanging data between a server and web application?", options: ["XML", "YAML", "JSON", "CSV"], answer: "JSON" },
            { question: "Which term describes the process of finding and fixing errors in code?", options: ["Compiling", "Executing", "Debugging", "Linking"], answer: "Debugging" },
            { question: "What is the standard port for HTTPS traffic?", options: ["80", "443", "21", "22"], answer: "443" },
            // History
            { question: "In what year did the Titanic sink?", options: ["1905", "1912", "1918", "1923"], answer: "1912" },
            { question: "Who was the first Roman Emperor?", options: ["Julius Caesar", "Constantine", "Nero", "Augustus"], answer: "Augustus" },
            { question: "The Renaissance began in which European country?", options: ["France", "England", "Italy", "Spain"], answer: "Italy" },
            { question: "What year did World War II officially end?", options: ["1943", "1945", "1947", "1950"], answer: "1945" },
            { question: "Which civilization built the Machu Picchu complex?", options: ["Aztec", "Maya", "Inca", "Olmec"], answer: "Inca" },
            { question: "Who invented the printing press?", options: ["Gutenberg", "Franklin", "Edison", "Tesla"], answer: "Gutenberg" },
            { question: "What was the name of the ship Charles Darwin sailed on?", options: ["The Endeavour", "The Beagle", "The Victory", "The Mayflower"], answer: "The Beagle" },
            // Science
            { question: "What is the chemical symbol for gold?", options: ["Ag", "Fe", "Au", "Pb"], answer: "Au" },
            { question: "What is the largest planet in our solar system?", options: ["Earth", "Mars", "Jupiter", "Saturn"], answer: "Jupiter" },
            { question: "What is the pH level of pure water?", options: ["0", "7", "14", "10"], answer: "7" },
            { question: "The process by which plants make their own food is called what?", options: ["Respiration", "Transpiration", "Photosynthesis", "Digestion"], answer: "Photosynthesis" },
            { question: "Which element has the atomic number 1?", options: ["Helium", "Oxygen", "Hydrogen", "Carbon"], answer: "Hydrogen" },
            { question: "What is the study of birds called?", options: ["Entomology", "Herpetology", "Ornithology", "Zoology"], answer: "Ornithology" },
            { question: "The theory of relativity was proposed by whom?", options: ["Isaac Newton", "Galileo Galilei", "Albert Einstein", "Stephen Hawking"], answer: "Albert Einstein" },
            { question: "What is the main component of the sun?", options: ["Oxygen", "Iron", "Hydrogen and Helium", "Carbon"], answer: "Hydrogen and Helium" },
            // Geography
            { question: "Which city is known as 'The Eternal City'?", options: ["Athens", "Paris", "London", "Rome"], answer: "Rome" },
            { question: "What is the longest river in the world?", options: ["Amazon River", "Yangtze River", "Mississippi River", "Nile River"], answer: "Nile River" },
            { question: "Mount Everest is located in which mountain range?", options: ["Andes", "Rockies", "Alps", "Himalayas"], answer: "Himalayas" },
            { question: "What is the capital city of Canada?", options: ["Toronto", "Vancouver", "Montreal", "Ottawa"], answer: "Ottawa" },
            { question: "Which continent is the Sahara Desert located on?", options: ["Asia", "South America", "Africa", "Australia"], answer: "Africa" },
            { question: "What is the flattest continent on Earth?", options: ["Europe", "North America", "Australia", "Antarctica"], answer: "Australia" },
            { question: "The Great Barrier Reef is off the coast of which country?", options: ["Brazil", "Mexico", "Australia", "Indonesia"], answer: "Australia" },
            { question: "What is the deepest point in the Earth's oceans?", options: ["Puerto Rico Trench", "Mariana Trench", "Java Trench", "Kermadec Trench"], answer: "Mariana Trench" },
            // Pop Culture
            { question: "Who directed the movie 'Pulp Fiction'?", options: ["Martin Scorsese", "Quentin Tarantino", "Steven Spielberg", "Christopher Nolan"], answer: "Quentin Tarantino" },
            { question: "What is the name of the main character in the 'Harry Potter' series?", options: ["Ron Weasley", "Albus Dumbledore", "Hermione Granger", "Harry Potter"], answer: "Harry Potter" },
            { question: "Which artist released the album 'Thriller'?", options: ["Prince", "Madonna", "Michael Jackson", "Whitney Houston"], answer: "Michael Jackson" },
            { question: "What fictional city is the home of Batman?", options: ["Metropolis", "Star City", "Gotham City", "Central City"], answer: "Gotham City" },
            { question: "The movie 'The Godfather' won the Academy Award for Best Picture in which year?", options: ["1972", "1973", "1974", "1975"], answer: "1973" },
            { question: "Which actress played the role of Katniss Everdeen in The Hunger Games films?", options: ["Emma Stone", "Shailene Woodley", "Jennifer Lawrence", "Saoirse Ronan"], answer: "Jennifer Lawrence" },
            { question: "Who is the lead singer of the rock band U2?", options: ["Chris Martin", "Bono", "Mick Jagger", "Freddie Mercury"], answer: "Bono" },
            // Literature
            { question: "Who wrote the play 'Romeo and Juliet'?", options: ["Charles Dickens", "William Shakespeare", "Jane Austen", "Geoffrey Chaucer"], answer: "William Shakespeare" },
            { question: "Moby Dick is a novel about a white whale. Who is the author?", options: ["Ernest Hemingway", "Herman Melville", "Mark Twain", "F. Scott Fitzgerald"], answer: "Herman Melville" },
            { question: "What is the title of the first book in 'The Lord of the Rings' trilogy?", options: ["The Two Towers", "The Hobbit", "The Fellowship of the Ring", "The Return of the King"], answer: "The Fellowship of the Ring" },
            { question: "In Greek mythology, who was the king of the gods?", options: ["Hades", "Poseidon", "Ares", "Zeus"], answer: "Zeus" },
            { question: "Which novel begins with the line: 'Call me Ishmael'?", options: ["The Great Gatsby", "Moby Dick", "1984", "Pride and Prejudice"], answer: "Moby Dick" },
            // Sports
            { question: "Which country won the FIFA World Cup in 2014?", options: ["Brazil", "Argentina", "Germany", "Spain"], answer: "Germany" },
            { question: "How many players are on a soccer (football) team on the field?", options: ["10", "11", "12", "9"], answer: "11" },
            { question: "What is the maximum number of clubs a golfer is allowed to carry in their bag?", options: ["12", "14", "16", "18"], answer: "14" },
            { question: "Which sport uses terms like 'love,' 'deuce,' and 'advantage'?", options: ["Badminton", "Tennis", "Volleyball", "Squash"], answer: "Tennis" },
            { question: "In basketball, how many points is a free throw worth?", options: ["1", "2", "3", "4"], answer: "1" },
            { question: "What is the name of the stadium where the Boston Red Sox play?", options: ["Yankee Stadium", "Wrigley Field", "Fenway Park", "Dodger Stadium"], answer: "Fenway Park" },
            { question: "Which famous boxer was known by the nickname 'The Greatest'?", options: ["Mike Tyson", "Joe Frazier", "Muhammad Ali", "George Foreman"], answer: "Muhammad Ali" },
            { question: "Which team has won the most Formula 1 Constructors' Championships?", options: ["McLaren", "Williams", "Ferrari", "Mercedes"], answer: "Ferrari" },
            { question: "How many bases are there in a standard game of baseball?", options: ["3", "4", "5", "6"], answer: "4" },
            { question: "What swimming stroke is not allowed in competitive synchronized swimming?", options: ["Backstroke", "Freestyle", "Breaststroke", "Butterfly"], answer: "Butterfly" },
            { question: "Which city hosts the Summer Olympics in 2024?", options: ["Tokyo", "Los Angeles", "Paris", "London"], answer: "Paris" },
            { question: "What is a 'bogey' in golf?", options: ["One stroke under par", "One stroke over par", "Two strokes under par", "Two strokes over par"], answer: "One stroke over par" }
        ];

        // --- Configuration ---
        const QUESTIONS_PER_SESSION = 10;
        
        // --- State Variables ---
        let quizData = []; // Array to hold the 10 shuffled questions for the current session
        let currentQuestionIndex = 0;
        let score = 0;
        let timerInterval = null;
        const initialTime = 30; // 30 seconds as required
        let timeLeft = initialTime;

        // --- DOM Elements ---
        const installButton = document.getElementById('install-button');
        const questionText = document.getElementById('question-text');
        const optionsContainer = document.getElementById('options-container');
        const scoreDisplay = document.getElementById('score-display');
        const currentQuestionCount = document.getElementById('current-question-count');
        const totalQuestionsDisplay = document.getElementById('total-questions');
        const timeDisplay = document.getElementById('time-display');
        const timerBar = document.getElementById('timer-bar');
        const nextButton = document.getElementById('next-button');
        const quizArea = document.getElementById('question-area');
        const timerArea = document.getElementById('timer-area');
        const resultScreen = document.getElementById('result-screen');

        // --- PWA Installation Logic ---
        let deferredPrompt;

        window.addEventListener('beforeinstallprompt', (e) => {
            // Prevent Chrome 67 and earlier from automatically showing the prompt
            e.preventDefault();
            // Stash the event so it can be triggered later.
            deferredPrompt = e;
            // Update UI to notify the user they can install the app
            installButton.style.display = 'block';
        });

        installButton.addEventListener('click', async () => {
            if (deferredPrompt) {
                // Hide the install button
                installButton.style.display = 'none';
                // Show the install prompt
                deferredPrompt.prompt();
                // Wait for the user to respond to the prompt
                const { outcome } = await deferredPrompt.userChoice;
                // Clear the deferred prompt variable
                deferredPrompt = null;
                console.log(`User response to the install prompt: ${outcome}`);
            } else {
                 console.log("Installation prompt is not available or has already been shown.");
            }
        });


        // --- Utility Functions ---
        
        /**
         * Shuffles an array using the Fisher-Yates algorithm.
         * @param {Array} array - The array to shuffle.
         * @returns {Array} The shuffled array.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        // --- Timer Functions ---

        /**
         * Clears any existing timer interval.
         */
        function stopTimer() {
            if (timerInterval) {
                clearInterval(timerInterval);
                timerInterval = null;
            }
        }

        /**
         * Starts the 30-second countdown timer.
         * Calls checkAnswer with null if time runs out.
         */
        function startTimer() {
            stopTimer(); // CRUCIAL: Stop the old timer before starting a new one
            timeLeft = initialTime; // Reset time to 30 seconds
            
            // Set initial display
            timeDisplay.textContent = `${timeLeft}`;
            timerBar.style.width = '100%';
            timerBar.className = 'timer-bar bg-green-500 rounded-full'; // Reset color to green
            timeDisplay.classList.remove('text-red-600', 'animate-pulse');
            timeDisplay.classList.add('text-primary');


            timerInterval = setInterval(() => {
                timeLeft--;
                const percentage = (timeLeft / initialTime) * 100;

                timeDisplay.textContent = `${timeLeft}`;
                timerBar.style.width = `${percentage}%`;

                // Change color if time is running low
                if (timeLeft <= 10 && timeLeft > 5) {
                    timerBar.className = 'timer-bar bg-yellow-500 rounded-full';
                }
                if (timeLeft <= 5) {
                    timerBar.className = 'timer-bar bg-red-600 rounded-full';
                    timeDisplay.classList.remove('text-primary');
                    timeDisplay.classList.add('text-red-600', 'animate-pulse');
                }
                
                if (timeLeft <= 0) {
                    stopTimer();
                    // If time runs out, treat it as a skipped/incorrect answer
                    checkAnswer(null); 
                }
            }, 1000);
        }

        // --- Quiz Logic Functions ---

        /**
         * Loads and renders the current question and options.
         */
        function loadQuestion() {
            // Check if all 10 session questions are answered
            if (currentQuestionIndex >= quizData.length) {
                return showResult();
            }

            // Update question counter display
            currentQuestionCount.textContent = currentQuestionIndex + 1;

            // Hide the next button and enable options for the new question
            nextButton.style.display = 'none';
            optionsContainer.classList.remove('pointer-events-none');
            
            const currentQuestion = quizData[currentQuestionIndex];
            
            questionText.textContent = currentQuestion.question;
            optionsContainer.innerHTML = ''; // Clear previous options

            // START THE 30-SECOND TIMER FOR THE NEW QUESTION
            startTimer(); 
            
            // Shuffle the options before displaying them
            const shuffledOptions = shuffleArray([...currentQuestion.options]);

            shuffledOptions.forEach(option => {
                const button = document.createElement('button');
                button.textContent = option;
                button.classList.add(
                    'option-button', 'w-full', 'p-4', 'text-left', 'font-medium', 
                    'rounded-lg', 'shadow-sm', 'hover:shadow-md'
                );
                button.onclick = () => checkAnswer(option, button);
                optionsContainer.appendChild(button);
            });
        }

        /**
         * Checks the selected answer, updates the score, and prepares for the next question.
         * @param {string | null} selectedOption - The user's choice or null if time ran out.
         * @param {HTMLElement} clickedButton - The button element that was clicked, or null.
         */
        function checkAnswer(selectedOption, clickedButton = null) {
            stopTimer(); // Stop the timer immediately when an answer is submitted or time runs out.

            optionsContainer.classList.add('pointer-events-none'); // Disable further clicks
            timeDisplay.classList.remove('animate-pulse');
            timeDisplay.classList.add('text-primary');

            const currentQuestion = quizData[currentQuestionIndex];
            const correctAnswer = currentQuestion.answer;
            const optionButtons = optionsContainer.querySelectorAll('.option-button');

            let isCorrect = false;

            if (selectedOption !== null) {
                isCorrect = selectedOption === correctAnswer;
            }

            // Provide visual feedback for all options
            optionButtons.forEach(button => {
                button.disabled = true; // Disable all buttons
                if (button.textContent === correctAnswer) {
                    // Highlight the correct answer
                    button.classList.add('correct');
                } else if (button === clickedButton && !isCorrect) {
                    // Highlight the incorrect selection
                    button.classList.add('incorrect');
                } else {
                    // Dim unselected incorrect options
                    button.classList.add('opacity-50');
                }
            });

            // Update score
            if (isCorrect) {
                score++;
                // Add a small animation for score update
                scoreDisplay.style.transform = 'scale(1.2)';
                setTimeout(() => scoreDisplay.style.transform = 'scale(1)', 200);
            }
            scoreDisplay.textContent = score;


            // Prepare for the next question
            currentQuestionIndex++;
            nextButton.style.display = 'block';
            nextButton.onclick = loadQuestion;
        }

        /**
         * Displays the final result screen.
         */
        function showResult() {
            quizArea.style.display = 'none';
            timerArea.style.display = 'none';
            nextButton.style.display = 'none';
            resultScreen.style.display = 'block';
            document.getElementById('final-score').textContent = `${score} / ${quizData.length}`;
        }

        /**
         * Initializes the quiz on page load.
         */
        function initQuiz() {
            // 1. Shuffle the entire question pool (ensuring non-repeating sessions)
            const shuffledPool = shuffleArray([...originalQuizData]);
            
            // 2. Select the first 10 questions for the current session
            quizData = shuffledPool.slice(0, QUESTIONS_PER_SESSION);
            
            // Reset state
            currentQuestionIndex = 0;
            score = 0;
            scoreDisplay.textContent = 0;
            totalQuestionsDisplay.textContent = QUESTIONS_PER_SESSION;

            // 3. Start the quiz flow
            loadQuestion();
        }

        // Start the quiz when the window loads
        window.onload = initQuiz;

    </script>
</body>
</html>


