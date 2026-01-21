<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 1. Changed title -->
    <title>Class Countdown</title>
    <!-- 1. Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 2. REMOVED Tone.js script -->
    <style>
        /* Custom font for the countdown */
        @import url('https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@700&display=swap');
        
        .font-mono {
            font-family: 'Roboto Mono', monospace;
        }

        /* NEW: Animation for the splash screen gradient */
        @keyframes gradient-animation {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }
        .animated-gradient {
            background-size: 200% 200%;
            animation: gradient-animation 5s ease infinite;
        }
    </style>
</head>
<body class="bg-gray-900 text-white min-h-screen flex items-center justify-center p-4 font-sans">

    <!-- Audio Element (hidden) -->
    <audio id="custom-audio" loop></audio>

    <!-- Main Application View -->
    <div id="main-app-view" class="w-full max-w-md bg-gray-800 rounded-2xl shadow-2xl p-6 transition-all duration-300">
        <!-- 2. Changed h1 title -->
        <h1 class="text-3xl font-bold text-center mb-6 text-indigo-400">Class Countdown</h1>

        <!-- UPDATED: Custom Music Section -->
        <div class="mb-6 p-4 bg-gray-700 rounded-lg">
            <label for="music-file-input" class="block text-sm font-medium text-gray-300 mb-2">
                Custom Music File (Optional)
            </label>
            <!-- Replaced URL input and Set button with a file input -->
            <input type="file" id="music-file-input" accept="audio/*" class="block w-full text-sm text-gray-300
                file:mr-4 file:py-2 file:px-4
                file:rounded-lg file:border-0
                file:text-sm file:font-semibold
                file:bg-indigo-600 file:text-white
                hover:file:bg-indigo-700
                file:cursor-pointer
            ">
        </div>

        <!-- Form to Add Timer -->
        <form id="add-timer-form" class="mb-6">
            <div class="mb-4">
                <label for="timer-label" class="block text-sm font-medium text-gray-300 mb-1">Label</label>
                <input type="text" id="timer-label" placeholder="e.g., Lunch Break" class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500">
            </div>
            <div class="mb-4">
                <label for="timer-time" class="block text-sm font-medium text-gray-300 mb-1">Time (24h)</label>
                <input type="time" id="timer-time" required class="w-full px-3 py-2 bg-gray-700 border border-gray-600 rounded-lg text-white focus:outline-none focus:ring-2 focus:ring-indigo-500" style="color-scheme: dark;">
            </div>
            <button id="add-timer-btn" type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg shadow-lg transition duration-200 ease-in-out">
                Add Timer
            </button>
        </form>

        <!-- List of Timers -->
        <div>
            <h2 class="text-xl font-semibold text-gray-200 mb-3">Your Timers</h2>
            <div id="timer-list" class="space-y-3 max-h-60 overflow-y-auto pr-2">
                <!-- Timers will be injected here by JS -->
                <p id="empty-state" class="text-gray-500 text-center py-4">No timers set yet.</p>
            </div>
        </div>
    </div>

    <!-- Fullscreen Timer View (Initially Hidden) -->
    <div id="fullscreen-timer-view" class="hidden fixed inset-0 bg-gray-900 flex-col items-center justify-center z-50 p-8 transition-colors duration-500">
        <p id="fullscreen-label" class="text-4xl md:text-5xl font-bold text-gray-300 mb-8 text-center break-words"></p>
        <div id="fullscreen-countdown" class="text-8xl md:text-9xl font-bold text-indigo-300 font-mono tracking-tighter">
            00:00:00
        </div>
        <button id="exit-fullscreen-btn" class="mt-12 bg-gray-700 hover:bg-gray-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition duration-200 ease-in-out">
            Exit
        </button>
    </div>

    <!-- NEW: Splash Screen (Initially Hidden) -->
    <div id="splash-screen" class="hidden fixed inset-0 z-60 flex-col items-center justify-center p-8 bg-gradient-to-r from-pink-500 via-red-500 to-yellow-500 animated-gradient">
        <p id="splash-timer-label" class="text-3xl md:text-4xl font-bold text-white mb-8 text-center break-words" style="text-shadow: 1px 1px 2px rgba(0,0,0,0.2);"></p>
        <h1 class="text-6xl md:text-8xl font-bold text-white mb-12 text-center" style="text-shadow: 2px 2px 4px rgba(0,0,0,0.3);">
            ¡Bienvenidos a la clase de español!
        </h1>
        <button id="close-splash-btn" class="mt-12 bg-white/30 backdrop-blur-sm hover:bg-white/50 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition duration-200 ease-in-out">
            Close
        </button>
    </div>


    <script type="module">
        // --- State ---
        let timers = [];
        let focusedTimerId = null; // ID of the timer in fullscreen
        let fadeOutInterval = null; // NEW: To manage the audio fade-out interval
        // REMOVED all Tone.js state variables
        // let audioContextStarted = false;
        // let synth;
        // let musicSequence;
        // let distortion;

        // --- DOM Elements ---
        const addTimerForm = document.getElementById('add-timer-form');
        const labelInput = document.getElementById('timer-label');
        const timeInput = document.getElementById('timer-time');
        const timerList = document.getElementById('timer-list');
        const addTimerBtn = document.getElementById('add-timer-btn');
        const emptyState = document.getElementById('empty-state');
        const fullscreenView = document.getElementById('fullscreen-timer-view');
        const fullscreenLabel = document.getElementById('fullscreen-label');
        const fullscreenCountdown = document.getElementById('fullscreen-countdown');
        const exitFullscreenBtn = document.getElementById('exit-fullscreen-btn');
        const mainAppView = document.getElementById('main-app-view');

        // NEW: Audio DOM Elements
        // Removed URL input and Set button variables
        const musicFileInput = document.getElementById('music-file-input'); // Added file input
        const customAudioEl = document.getElementById('custom-audio');

        // NEW: Splash Screen DOM Elements
        const splashScreen = document.getElementById('splash-screen');
        const splashTimerLabel = document.getElementById('splash-timer-label');
        const closeSplashBtn = document.getElementById('close-splash-btn');


        // --- Audio Initialization ---
        // REMOVED entire initAudio() function
        // (Also removed the buggy, duplicated initAudio that was left behind)

        // --- Helper Functions ---

        /**
         * Formats remaining seconds into HH:MM:SS string.
         * @param {number} totalSeconds - The total number of seconds remaining.
         * @returns {string} The formatted time string.
         */
        // (Removed the buggy, duplicated formatTime function)
        function formatTime(totalSeconds) {
            if (totalSeconds < 0) totalSeconds = 0;

            const h = Math.floor(totalSeconds / 3600);
            const m = Math.floor((totalSeconds % 3600) / 60);
            const s = Math.floor(totalSeconds % 60);
            
            const pad = (num) => num.toString().padStart(2, '0');
            
            return `${pad(h)}:${pad(m)}:${pad(s)}`;
        }

        /**
         * Renders the current list of timers to the DOM.
         */
        function renderTimers() {
            timerList.innerHTML = ''; // Clear the list
            if (timers.length === 0) {
                timerList.appendChild(emptyState);
                return;
            }

            // Sort timers by their targetDate
            const sortedTimers = timers.sort((a, b) => a.targetDate - b.targetDate);

            sortedTimers.forEach(timer => {
                const timerEl = document.createElement('div');
                timerEl.id = `timer-card-${timer.id}`;
                timerEl.className = 'timer-card flex items-center justify-between p-4 bg-gray-700 rounded-lg shadow border-2 border-transparent transition-all duration-300';
                timerEl.dataset.timerId = timer.id;

                timerEl.innerHTML = `
                    <div class="flex-1 min-w-0">
                        <p class="text-lg font-semibold text-white truncate">${timer.label || 'Timer'}</p>
                        <p class="text-sm text-gray-400">${new Date(timer.targetDate).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</p>
                    </div>
                    <div class="ml-4 text-right">
                        <div id="countdown-${timer.id}" class="text-2xl font-bold text-indigo-300 font-mono">
                            00:00:00
                        </div>
                    </div>
                    <button class="focus-btn ml-4 text-gray-500 hover:text-indigo-400" data-id="${timer.id}" title="Focus timer">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 8V4m0 0h4M4 4l5 5m11-1V4m0 0h-4m4 0l-5 5M4 16v4m0 0h4m-4 0l5-5m11 1v4m0 0h-4m4 0l-5-5" />
                        </svg>
                    </button>
                    <button class="delete-btn ml-2 text-gray-500 hover:text-red-400" data-id="${timer.id}" title="Delete timer">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                    </button>
                `;
                timerList.appendChild(timerEl);
            });
        }

        /**
         * Main update loop. Runs every second to update all timer displays and check audio triggers.
         */
        function updateTimers() {
            const now = new Date();
            let anyTimerInMusicWindow = false;
            let minSecondsRemaining = Infinity; // Track the closest timer
            let focusedTimerInfo = null; // Track info for the focused timer

            timers.forEach(timer => {
                const display = document.getElementById(`countdown-${timer.id}`);
                const card = document.getElementById(`timer-card-${timer.id}`);
                
                if (!display || !card) return; // Skip if element not found

                let secondsRemaining = (timer.targetDate.getTime() - now.getTime()) / 1000;
                let timerState = 'normal'; // default

                // --- Timer State Management ---
                
                // 1. Timer has passed
                if (secondsRemaining <= 0) {
                    display.textContent = '00:00:00';
                    card.classList.add('bg-green-800', 'border-green-500');
                    card.classList.remove('bg-yellow-700', 'border-yellow-500', 'bg-gray-700');
                    timerState = 'finished';

                    // NEW: Check if this is the first time finishing
                    if (!timer.isFinished) {
                        timer.isFinished = true;
                        showSplashScreen(timer);
                    }

                    // If timer finished more than 10 seconds ago, reset it for the next day
                    // This prevents the "green" state from sticking forever
                    if (secondsRemaining < -10) {
                        timer.targetDate.setDate(timer.targetDate.getDate() + 1);
                        timer.isFinished = false; // Reset finished state for next day
                    }
                } 
                // 2. Timer is in the 1-minute "exciting music" window
                else if (secondsRemaining <= 60) { // 60 seconds = 1 minute
                    display.textContent = formatTime(secondsRemaining);
                    anyTimerInMusicWindow = true;
                    minSecondsRemaining = Math.min(minSecondsRemaining, secondsRemaining); // Track the closest timer
                    card.classList.add('bg-yellow-700', 'border-yellow-500');
                    card.classList.remove('bg-green-800', 'border-green-500', 'bg-gray-700');
                    timerState = 'warning';
                }
                // 3. Timer is active and more than 1 minute away
                else {
                    display.textContent = formatTime(secondsRemaining);
                    card.classList.remove('bg-green-800', 'border-green-500', 'bg-yellow-700', 'border-yellow-500');
                    card.classList.add('bg-gray-700');
                    timerState = 'normal';
                }

                // Store info if this is the focused timer
                if (timer.id === focusedTimerId) {
                    focusedTimerInfo = {
                        seconds: secondsRemaining,
                        state: timerState,
                    };
                }
            });

            // --- Fullscreen View Update ---
            if (focusedTimerInfo) {
                fullscreenCountdown.textContent = formatTime(focusedTimerInfo.seconds <= 0 ? 0 : focusedTimerInfo.seconds);
                
                fullscreenView.classList.remove('bg-gray-900', 'bg-green-800', 'bg-yellow-700');
                if (focusedTimerInfo.state === 'finished') {
                    fullscreenView.classList.add('bg-green-800');
                } else if (focusedTimerInfo.state === 'warning') {
                    fullscreenView.classList.add('bg-yellow-700');
                } else {
                    fullscreenView.classList.add('bg-gray-900');
                }
            }

            // --- Audio State Management ---
            // REPLACED Tone.js logic with simple HTML5 audio logic
            if (customAudioEl.src) { // Only play if a source is set
                // NEW: Don't start music if splash screen is visible
                const isSplashVisible = !splashScreen.classList.contains('hidden');

                if (anyTimerInMusicWindow && !isSplashVisible) {
                    // NEW: If music should play, stop any fade-out
                    stopFadeOut();

                    // Check if it's not already playing
                    if (customAudioEl.paused) {
                        customAudioEl.volume = 1; // Ensure volume is 1
                        customAudioEl.play().catch(e => console.error("Audio play failed:", e));
                    }
                } else {
                    // Stop music if it's playing (and not already fading)
                    if (!customAudioEl.paused && !fadeOutInterval) {
                        customAudioEl.pause();
                        customAudioEl.currentTime = 0; // Rewind
                    }
                }
            }
        }

        // --- Event Handlers ---

        /**
         * Handles the "Add Timer" form submission.
         * @param {Event} e - The submit event.
         */
        async function handleAddTimer(e) {
            e.preventDefault();

            // 1. Get form values
            const label = labelInput.value; // Can be empty
            const time = timeInput.value;
            
            if (!time) {
                // Using console.warn instead of alert as alert() is forbidden
                console.warn('Please select a time.');
                return;
            }

            // 2. REMOVED audio initialization logic
            // addTimerBtn.disabled = true;
            // addTimerBtn.textContent = 'Starting Audio...';
            // await initAudio(); 
            // addTimerBtn.disabled = false;
            // addTimerBtn.textContent = 'Add Timer';


            // 3. Get time and create date
            const [hours, minutes] = time.split(':').map(Number);
            const now = new Date();
            const targetDate = new Date();
            targetDate.setHours(hours, minutes, 0, 0);

            // If time is already past, set it for tomorrow
            if (targetDate < now) {
                targetDate.setDate(targetDate.getDate() + 1);
            }

            // 4. Create and add new timer
            const newTimer = {
                id: Date.now(),
                label: label || `Timer (${time})`,
                targetDate: targetDate,
                isFinished: false, // NEW: Add finished state
            };
            timers.push(newTimer);

            // 5. Update UI
            renderTimers();
            updateTimers(); // Run one update immediately

            // 6. Clear form
            labelInput.value = '';
            timeInput.value = '';
        }

        /**
         * Handles delete button clicks using event delegation.
         * @param {Event} e - The click event.
         */
        function handleTimerListClick(e) {
            // Check for Delete
            const deleteBtn = e.target.closest('.delete-btn');
            if (deleteBtn) {
                const idToDelete = parseInt(deleteBtn.dataset.id, 10);
                timers = timers.filter(timer => timer.id !== idToDelete);
                renderTimers();
                return; // Stop further processing
            }
            
            // Check for Focus
            const focusBtn = e.target.closest('.focus-btn');
            if (focusBtn) {
                const idToFocus = parseInt(focusBtn.dataset.id, 10);
                enterFullscreen(idToFocus);
                return; // Stop further processing
            }
        }

        /**
         * Enters the fullscreen (focus) view for a specific timer.
         * @param {number} timerId - The ID of the timer to focus.
         */
        function enterFullscreen(timerId) {
            const timer = timers.find(t => t.id === timerId);
            if (!timer) return;

            focusedTimerId = timerId;
            fullscreenLabel.textContent = timer.label || 'Timer';
            
            // Immediately update countdown text on enter
            const now = new Date();
            let secondsRemaining = (timer.targetDate.getTime() - now.getTime()) / 1000;
            fullscreenCountdown.textContent = formatTime(secondsRemaining <= 0 ? 0 : secondsRemaining);

            fullscreenView.classList.remove('hidden');
            fullscreenView.classList.add('flex');
            mainAppView.classList.add('blur-sm', 'scale-95'); // Optional: nice effect for background
        }

        /**
         * Exits the fullscreen (focus) view.
         */
        function exitFullscreen() {
            focusedTimerId = null;
            fullscreenView.classList.add('hidden');
            fullscreenView.classList.remove('flex');
            mainAppView.classList.remove('blur-sm', 'scale-95');
        }

        // UPDATED: Event listener for setting music from a file
        function handleMusicFileChange(e) {
            const file = e.target.files[0];
            if (file && file.type.startsWith('audio/')) {
                // Create a temporary URL for the local file
                const fileURL = URL.createObjectURL(file);
                customAudioEl.src = fileURL;
                console.log('Custom music file loaded:', file.name);

                // NEW: Stop any fade-out and reset volume
                stopFadeOut();
                customAudioEl.volume = 1;

                // Optional: Play a short snippet to confirm
                customAudioEl.play().catch(err => console.error("Audio test failed:", err));
                setTimeout(() => {
                    if (!customAudioEl.paused) {
                        customAudioEl.pause();
                        customAudioEl.currentTime = 0;
                    }
                }, 1000); // Play for 1 second
            }
        }

        // --- NEW: Splash Screen Functions ---

        /**
         * Shows the "Time's Up" splash screen.
         * @param {object} timer - The timer object that just finished.
         */
        function showSplashScreen(timer) {
            // UPDATED: Changed the text to "¡Es hora!"
            splashTimerLabel.textContent = "¡Es hora!";
            splashScreen.classList.remove('hidden');
            splashScreen.classList.add('flex');
            
            // UPDATED: Start fading out the music instead of stopping it
            fadeOutAudio();

            // NEW: If the focus view is visible, hide it.
            // This fixes the bug where the focus view would render on top
            // of the splash screen.
            if (focusedTimerId) {
                exitFullscreen();
            }
        }

        /**
         * Hides the "Time's Up" splash screen.
         */
        function hideSplashScreen() {
            splashScreen.classList.add('hidden');
            splashScreen.classList.remove('flex');
        }


        // --- NEW: Audio Fade Functions ---

        /**
         * Stops any active audio fade-out and resets the volume.
         */
        function stopFadeOut() {
            if (fadeOutInterval) {
                clearInterval(fadeOutInterval);
                fadeOutInterval = null;
            }
            // Always reset volume in case it was half-faded
            customAudioEl.volume = 1; 
        }

        /**
         * Smoothly fades out the custom audio over 5 seconds.
         */
        function fadeOutAudio() {
            // If already fading, or not playing, do nothing
            if (fadeOutInterval || customAudioEl.paused) {
                return;
            }

            const fadeDuration = 5000; // 5 seconds
            const fadeSteps = 50;
            const stepDuration = fadeDuration / fadeSteps;
            
            // Start fade from the *current* volume
            const startVolume = customAudioEl.volume;
            const volumeStep = startVolume / fadeSteps;

            fadeOutInterval = setInterval(() => {
                let newVolume = customAudioEl.volume - volumeStep;
                
                if (newVolume <= 0) {
                    newVolume = 0;
                    clearInterval(fadeOutInterval);
                    fadeOutInterval = null;
                    customAudioEl.pause();
                    customAudioEl.currentTime = 0;
                    customAudioEl.volume = 1; // Reset for next time
                }
                
                customAudioEl.volume = newVolume;

            }, stepDuration);
        }


        // --- Event Listeners ---
        window.addEventListener('DOMContentLoaded', () => {
            addTimerForm.addEventListener('submit', handleAddTimer);
            timerList.addEventListener('click', handleTimerListClick); // Updated listener
            exitFullscreenBtn.addEventListener('click', exitFullscreen); // New listener
            // Replaced setMusicBtn listener with file input listener
            musicFileInput.addEventListener('change', handleMusicFileChange); // NEW listener
            closeSplashBtn.addEventListener('click', hideSplashScreen); // NEW listener

            // Start the main update loop
            setInterval(updateTimers, 1000);
        });
    </script>
</body>
</html>
