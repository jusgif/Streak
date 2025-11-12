<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daily Streak Tracker - Habit Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        * {
            font-family: 'Inter', sans-serif;
        }
        
        .habit-card {
            transition: all 0.3s ease;
        }
        
        .habit-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
        }
        
        .streak-fire {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }
        
        .calendar-day {
            transition: all 0.2s ease;
        }
        
        .calendar-day:hover {
            transform: scale(1.1);
        }
        
        .completed {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        
        .missed {
            background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
        }
        
        .future {
            background: #f3f4f6;
        }
        
        .today {
            border: 3px solid #3b82f6;
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    <!-- Header -->
    <header class="bg-gradient-to-r from-blue-600 to-purple-600 text-white py-8 px-4">
        <div class="max-w-6xl mx-auto">
            <div class="flex items-center justify-between">
                <div>
                    <h1 class="text-4xl font-bold flex items-center gap-3">
                        <i class="fas fa-fire streak-fire"></i>
                        Daily Streak Tracker
                    </h1>
                    <p class="text-blue-100 mt-2">Build habits, track progress, maintain streaks</p>
                </div>
                <div class="text-right">
                    <div class="text-2xl font-bold" id="currentDate"></div>
                    <div class="text-sm text-blue-100">Today</div>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-6xl mx-auto px-4 py-8">
        <!-- Stats Overview -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Total Habits</p>
                        <p class="text-3xl font-bold text-gray-800" id="totalHabits">0</p>
                    </div>
                    <div class="text-3xl text-blue-500">
                        <i class="fas fa-list-check"></i>
                    </div>
                </div>
            </div>
            
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Today's Progress</p>
                        <p class="text-3xl font-bold text-green-600" id="todayProgress">0%</p>
                    </div>
                    <div class="text-3xl text-green-500">
                        <i class="fas fa-chart-line"></i>
                    </div>
                </div>
            </div>
            
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-gray-500 text-sm">Longest Streak</p>
                        <p class="text-3xl font-bold text-orange-600" id="longestStreak">0 days</p>
                    </div>
                    <div class="text-3xl text-orange-500">
                        <i class="fas fa-trophy"></i>
                    </div>
                </div>
            </div>
        </div>

        <!-- Add New Habit -->
        <div class="bg-white rounded-xl shadow-lg p-6 mb-8">
            <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center gap-2">
                <i class="fas fa-plus-circle text-blue-500"></i>
                Add New Habit
            </h2>
            <div class="flex gap-4">
                <input 
                    type="text" 
                    id="habitInput" 
                    placeholder="Enter a new habit (e.g., Exercise, Read, Meditate)"
                    class="flex-1 px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                >
                <button 
                    onclick="addHabit()" 
                    class="px-6 py-3 bg-gradient-to-r from-blue-500 to-purple-500 text-white rounded-lg hover:from-blue-600 hover:to-purple-600 transition-all duration-200 font-medium"
                >
                    <i class="fas fa-plus mr-2"></i>
                    Add Habit
                </button>
            </div>
        </div>

        <!-- Habits List -->
        <div class="bg-white rounded-xl shadow-lg p-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-6 flex items-center gap-2">
                <i class="fas fa-fire text-orange-500"></i>
                Your Habits
            </h2>
            <div id="habitsList" class="space-y-4">
                <!-- Habits will be dynamically added here -->
            </div>
            <div id="emptyState" class="text-center py-12 hidden">
                <i class="fas fa-clipboard-list text-6xl text-gray-300 mb-4"></i>
                <h3 class="text-xl font-semibold text-gray-500 mb-2">No habits yet</h3>
                <p class="text-gray-400">Start building your daily streaks by adding your first habit!</p>
            </div>
        </div>
    </main>

    <script>
        // Initialize data
        let habits = JSON.parse(localStorage.getItem('habits')) || [];
        
        // Set current date
        document.getElementById('currentDate').textContent = new Date().toLocaleDateString('en-US', {
            weekday: 'long',
            month: 'short',
            day: 'numeric'
        });

        // Format date for storage
        function formatDate(date) {
            return date.toISOString().split('T')[0];
        }

        // Get today's date string
        function getToday() {
            return formatDate(new Date());
        }

        // Add new habit
        function addHabit() {
            const input = document.getElementById('habitInput');
            const habitName = input.value.trim();
            
            if (!habitName) {
                alert('Please enter a habit name');
                return;
            }

            const newHabit = {
                id: Date.now(),
                name: habitName,
                createdAt: new Date().toISOString(),
                records: {}
            };

            habits.push(newHabit);
            saveHabits();
            input.value = '';
            renderHabits();
            updateStats();
        }

        // Save habits to localStorage
        function saveHabits() {
            localStorage.setItem('habits', JSON.stringify(habits));
        }

        // Delete habit
        function deleteHabit(id) {
            if (confirm('Are you sure you want to delete this habit?')) {
                habits = habits.filter(habit => habit.id !== id);
                saveHabits();
                renderHabits();
                updateStats();
            }
        }

        // Toggle habit completion for today
        function toggleHabit(id) {
            const habit = habits.find(h => h.id === id);
            const today = getToday();
            
            if (habit.records[today]) {
                delete habit.records[today];
            } else {
                habit.records[today] = true;
            }
            
            saveHabits();
            renderHabits();
            updateStats();
        }

        // Calculate streak for a habit
        function calculateStreak(habit) {
            let streak = 0;
            const today = new Date();
            
            // Start from yesterday and go backwards
            for (let i = 1; i <= 365; i++) {
                const date = new Date(today);
                date.setDate(today.getDate() - i);
                const dateStr = formatDate(date);
                
                if (habit.records[dateStr]) {
                    streak++;
                } else {
                    // If yesterday wasn't completed, streak is broken
                    if (i === 1) {
                        streak = 0;
                    }
                    break;
                }
            }
            
            // If completed today, add to streak
            if (habit.records[getToday()]) {
                streak++;
            }
            
            return streak;
        }

        // Get last 30 days of completion data
        function getLast30Days(habit) {
            const days = [];
            const today = new Date();
            
            for (let i = 29; i >= 0; i--) {
                const date = new Date(today);
                date.setDate(today.getDate() - i);
                const dateStr = formatDate(date);
                const isToday = i === 0;
                
                days.push({
                    date: dateStr,
                    completed: habit.records[dateStr] || false,
                    isToday: isToday
                });
            }
            
            return days;
        }

        // Render habits
        function renderHabits() {
            const habitsList = document.getElementById('habitsList');
            const emptyState = document.getElementById('emptyState');
            
            if (habits.length === 0) {
                habitsList.innerHTML = '';
                emptyState.classList.remove('hidden');
                return;
            }
            
            emptyState.classList.add('hidden');
            
            habitsList.innerHTML = habits.map(habit => {
                const streak = calculateStreak(habit);
                const days = getLast30Days(habit);
                const todayCompleted = habit.records[getToday()];
                
                return `
                    <div class="habit-card bg-gradient-to-r from-gray-50 to-white rounded-lg p-6 border border-gray-200">
                        <div class="flex items-center justify-between mb-4">
                            <div class="flex items-center gap-4">
                                <button 
                                    onclick="toggleHabit(${habit.id})" 
                                    class="w-8 h-8 rounded-full border-2 border-gray-300 flex items-center justify-center transition-all duration-200 ${todayCompleted ? 'bg-green-500 border-green-500' : 'hover:border-green-400'}"
                                >
                                    ${todayCompleted ? '<i class="fas fa-check text-white text-sm"></i>' : ''}
                                </button>
                                <h3 class="text-lg font-semibold text-gray-800">${habit.name}</h3>
                            </div>
                            <div class="flex items-center gap-3">
                                <div class="flex items-center gap-2 bg-orange-50 px-3 py-1 rounded-full">
                                    <i class="fas fa-fire text-orange-500"></i>
                                    <span class="font-semibold text-orange-600">${streak} day${streak !== 1 ? 's' : ''}</span>
                                </div>
                                <button 
                                    onclick="deleteHabit(${habit.id})" 
                                    class="text-red-500 hover:text-red-700 transition-colors duration-200 p-2 rounded-lg hover:bg-red-50"
                                >
                                    <i class="fas fa-trash"></i>
                                </button>
                            </div>
                        </div>
                        
                        <div class="flex items-center gap-2">
                            <span class="text-sm text-gray-500 mr-2">Last 30 days:</span>
                            <div class="flex gap-1">
                                ${days.map(day => `
                                    <div 
                                        class="calendar-day w-6 h-6 rounded cursor-pointer flex items-center justify-center text-xs font-medium
                                               ${day.completed ? 'completed text-white' : day.isToday ? 'today bg-blue-100 text-blue-600' : 'future text-gray-400'}
                                               ${day.isToday ? 'ring-2 ring-blue-300' : ''}"
                                        title="${day.date} - ${day.completed ? 'Completed' : 'Not completed'}"
                                    >
                                        ${day.isToday ? 'T' : day.date.split('-')[2]}
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        // Update statistics
        function updateStats() {
            const totalHabits = habits.length;
            const today = getToday();
            
            // Calculate today's progress
            let completedToday = 0;
            habits.forEach(habit => {
                if (habit.records[today]) completedToday++;
            });
            const todayProgress = totalHabits > 0 ? Math.round((completedToday / totalHabits) * 100) : 0;
            
            // Calculate longest streak
            let longestStreak = 0;
            habits.forEach(habit => {
                const streak = calculateStreak(habit);
                longestStreak = Math.max(longestStreak, streak);
            });
            
            // Update UI
            document.getElementById('totalHabits').textContent = totalHabits;
            document.getElementById('todayProgress').textContent = todayProgress + '%';
            document.getElementById('longestStreak').textContent = longestStreak + ' days';
            
            // Update progress color
            const progressElement = document.getElementById('todayProgress');
            progressElement.className = `text-3xl font-bold ${
                todayProgress === 100 ? 'text-green-600' : 
                todayProgress >= 50 ? 'text-yellow-600' : 
                totalHabits > 0 ? 'text-red-600' : 'text-gray-600'
            }`;
        }

        // Allow Enter key to add habit
        document.getElementById('habitInput').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                addHabit();
            }
        });

        // Initial render
        renderHabits();
        updateStats();
    </script>
</body>
</html>
