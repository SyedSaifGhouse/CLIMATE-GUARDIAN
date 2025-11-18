CODES 

#admin.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Dashboard</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <nav>
        <div class="logo">🌍 Climate Guardians (ADMIN)</div>
        <ul>
            <li><a href="index.html">Home</a></li>
            <li><a href="dashboard.html">Student Portal</a></li>
        </ul>
    </nav>

    <main class="dashboard-main">
        <div id="admin-login-container" class="dashboard-card">
            <h2>Admin Login</h2>
            <p>Please enter your credentials.</p>
            <input type="text" id="admin-username" placeholder="Admin Username">
            <input type="password" id="admin-password" placeholder="Admin Password">
            <button id="admin-login-button" class="cta-button">Login</button>
            <p id="admin-error" class="error-msg" style="display: none;">Invalid credentials.</p>
        </div>

        <div id="admin-dashboard-container" class="dashboard-card" style="display: none;">
            <h2>Admin Dashboard</h2>
            
            <div class="admin-section">
                <h3>Award Green Points</h3>
                <input type="text" id="student-username-input" placeholder="Enter Student's Username">
                <input type="number" id="points-to-add-input" placeholder="Points to Award">
                <button id="award-points-button" class="cta-button">Award</button>
            </div>

            <div class="admin-section">
                <h3>All Student Points</h3>
                <button id="refresh-list-button" class="secondary-btn">Refresh List</button>
                <div id="student-list-container">
                    </div>
            </div>
            
            <button id="admin-logout-button" class="logout-btn">Log Out</button>
        </div>
    </main>

    <script src="admin.js"></script>
</body>
</html>
```


#admin.js
```
// === THIS IS THE NEW, UPDATED admin.js CONNECTED TO FLASK BACKEND ===

document.addEventListener("DOMContentLoaded", () => {
    const API_BASE_URL = 'http://127.0.0.1:5000/api/admin'; 

    // Get all elements
    const loginContainer = document.getElementById("admin-login-container");
    const dashboardContainer = document.getElementById("admin-dashboard-container");
    const loginButton = document.getElementById("admin-login-button");
    const logoutButton = document.getElementById("admin-logout-button");
    const adminUserInput = document.getElementById("admin-username");
    const adminPassInput = document.getElementById("admin-password");
    const errorMsg = document.getElementById("admin-error");
    
    const awardButton = document.getElementById("award-points-button");
    const studentUsernameInput = document.getElementById("student-username-input");
    const pointsInput = document.getElementById("points-to-add-input");
    
    const refreshListButton = document.getElementById("refresh-list-button");
    const studentListContainer = document.getElementById("student-list-container");

    // --- Core Admin Functions ---

    async function adminLogin() {
        const username = adminUserInput.value;
        const password = adminPassInput.value;

        try {
            const response = await fetch(`${API_BASE_URL}/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });

            const data = await response.json();

            if (response.ok) {
                // Store session for simplicity in this prototype
                sessionStorage.setItem("adminLoggedIn", "true"); 
                showDashboard();
            } else {
                errorMsg.textContent = data.message;
                errorMsg.style.display = "block";
            }
        } catch (error) {
            console.error('Admin login error:', error);
            errorMsg.textContent = "Could not connect to server.";
            errorMsg.style.display = "block";
        }
    }

    function adminLogout() {
        sessionStorage.removeItem("adminLoggedIn");
        showLogin();
    }

    function showDashboard() {
        errorMsg.style.display = "none";
        loginContainer.style.display = "none";
        dashboardContainer.style.display = "block";
        loadStudentData();
    }

    function showLogin() {
        loginContainer.style.display = "block";
        dashboardContainer.style.display = "none";
    }

    // This loads and displays all students from the API
    async function loadStudentData() {
        studentListContainer.innerHTML = "<p>Loading student data...</p>";

        try {
            const response = await fetch(`${API_BASE_URL}/users`);
            const users = await response.json();

            studentListContainer.innerHTML = ""; // Clear old list

            if (users.length === 0) {
                studentListContainer.innerHTML = "<p>No students have registered yet.</p>";
                return;
            }

            // Create a table to display data
            let table = "<table><tr><th>Student Username</th><th>Points</th></tr>";
            users.forEach(user => {
                table += `<tr><td>${user.username}</td><td>${user.points}</td></tr>`;
            });
            table += "</table>";
            studentListContainer.innerHTML = table;

        } catch (error) {
            console.error('Fetch student data error:', error);
            studentListContainer.innerHTML = "<p class='error-msg'>Error loading data. Is the Python server running?</p>";
        }
    }

    // Function to award points via API
    async function awardPoints() {
        const username = studentUsernameInput.value.trim();
        const pointsToAward = parseInt(pointsInput.value);

        if (!username || isNaN(pointsToAward)) {
            alert("Please enter a valid username and point amount.");
            return;
        }

        try {
            const response = await fetch(`${API_BASE_URL}/award_points`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username: username, points_to_award: pointsToAward })
            });

            const data = await response.json();

            if (response.ok) {
                alert(data.message);
                studentUsernameInput.value = "";
                pointsInput.value = "";
                loadStudentData(); // Refresh list to show new points
            } else {
                alert(`Award failed: ${data.message}`);
            }
        } catch (error) {
            console.error('Award points error:', error);
            alert('Could not connect to the server.');
        }
    }

    // --- Event Listeners ---
    loginButton.addEventListener("click", adminLogin);
    logoutButton.addEventListener("click", adminLogout);
    refreshListButton.addEventListener("click", loadStudentData);
    awardButton.addEventListener("click", awardPoints);

    // Check if admin is already logged in
    if (sessionStorage.getItem("adminLoggedIn") === "true") {
        showDashboard();
    } else {
        showLogin();
    }
});
```


#app.py
```
# app.py - The Python Flask Backend Server (NOW WITH ADMIN SUPPORT)

from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash
from flask_cors import CORS # Needed for cross-origin requests from Live Server

# --- 1. CONFIGURATION ---
app = Flask(__name__)
# IMPORTANT: Install flask-cors if you haven't: pip install flask-cors
CORS(app) 

# Configure SQLite database (saved as 'site.db' in the project folder)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///site.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'your_super_secret_key_change_me' # IMPORTANT: Change this!

db = SQLAlchemy(app)

# --- 2. DATABASE MODELS ---

# Model for Student Users
class User(db.Model):
    """Database model for storing student user data"""
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    points = db.Column(db.Integer, default=0)

    def to_dict(self):
        return {'username': self.username, 'points': self.points}

# Model for Admin User
class AdminUser(db.Model):
    """Database model for storing admin credentials"""
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)

# --- 3. HELPER FUNCTIONS & INITIAL SETUP ---

with app.app_context():
    db.create_all()
    
    # --- Initialize Admin Account ---
    # Create a default admin account if none exists
    if not AdminUser.query.first():
        admin_pass_hash = generate_password_hash("pass123") # Default password
        default_admin = AdminUser(username="admin", password_hash=admin_pass_hash)
        db.session.add(default_admin)
        db.session.commit()
        print("Default admin account created: U: admin, P: pass123")


# --- 4. STUDENT API ENDPOINTS ---

@app.route('/api/register', methods=['POST'])
def register():
    """Student registration."""
    # ... (code for student registration remains the same) ...
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    if not username or not password: return jsonify({'message': 'Missing username or password'}), 400
    if User.query.filter_by(username=username).first(): return jsonify({'message': 'Username already exists'}), 409
    
    hashed_password = generate_password_hash(password)
    new_user = User(username=username, password_hash=hashed_password, points=0)
    db.session.add(new_user)
    db.session.commit()
    return jsonify({'message': 'User registered successfully!', 'points': 0}), 201

@app.route('/api/login', methods=['POST'])
def login():
    """Student login."""
    # ... (code for student login remains the same) ...
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    user = User.query.filter_by(username=username).first()

    if user and check_password_hash(user.password_hash, password):
        return jsonify({'message': 'Login successful', 'username': user.username, 'points': user.points}), 200
    else:
        return jsonify({'message': 'Invalid username or password'}), 401

@app.route('/api/update_points', methods=['POST'])
def update_points():
    """Student point update (self-reporting actions)."""
    # ... (code for student point update remains the same) ...
    data = request.get_json()
    username = data.get('username')
    points_to_add = data.get('points_to_add')
    if not username or points_to_add is None: return jsonify({'message': 'Missing data'}), 400

    user = User.query.filter_by(username=username).first()
    if user:
        new_points = max(0, user.points + points_to_add)
        user.points = new_points
        db.session.commit()
        return jsonify({'message': 'Points updated', 'new_points': new_points}), 200
    else:
        return jsonify({'message': 'User not found'}), 404

# --- 5. ADMIN API ENDPOINTS (NEW) ---

@app.route('/api/admin/login', methods=['POST'])
def admin_login():
    """Admin login authentication."""
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')

    admin_user = AdminUser.query.filter_by(username=username).first()
    
    if admin_user and check_password_hash(admin_user.password_hash, password):
        return jsonify({'message': 'Admin login successful'}), 200
    else:
        return jsonify({'message': 'Invalid admin credentials'}), 401

@app.route('/api/admin/users', methods=['GET'])
def get_all_users():
    """Fetch all student users and their points (for Admin dashboard)."""
    # NOTE: You would typically add an authentication token check here for security!
    users = User.query.all()
    # Convert list of User objects to list of dictionaries
    user_list = [user.to_dict() for user in users] 
    
    return jsonify(user_list), 200

@app.route('/api/admin/award_points', methods=['POST'])
def admin_award_points():
    """Admin awards specific points to a specific student."""
    data = request.get_json()
    target_username = data.get('username')
    points_to_award = data.get('points_to_award')
    
    if not target_username or points_to_award is None:
        return jsonify({'message': 'Missing username or points amount'}), 400
        
    user = User.query.filter_by(username=target_username).first()
    
    if user:
        # Admin action: Add points directly
        new_points = max(0, user.points + points_to_award)
        user.points = new_points
        db.session.commit()
        return jsonify({'message': f'Successfully awarded {points_to_award} to {target_username}'}), 200
    else:
        return jsonify({'message': f'Student {target_username} not found'}), 404


# --- 6. RUN SERVER ---

if __name__ == '__main__':
    # Run the server on port 5000
    app.run(debug=True, port=5000)
```


#dashboard.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Green Points</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <nav>
        <div class="logo">🌍 Climate Guardians</div>
        <ul>
            <li><a href="index.html">Home</a></li>
            <li><a href="dashboard.html" class="active">My Green Points</a></li>
            <li><a href="admin.html">Admin</a></li>
        </ul>
    </nav>

    <main class="dashboard-main">
        <div id="login-container" class="dashboard-card">
            <h2>Climate Guardian Login</h2>
            <p>Enter your credentials to manage your points.</p>
            <input type="text" id="username-input" placeholder="Username (Unique ID)">
            <input type="password" id="password-input" placeholder="Password"> <div style="display: flex; gap: 10px; margin-top: 15px;">
                <button id="login-button" class="cta-button" style="flex-grow: 1;">Login</button>
                <button id="register-button" class="secondary-btn" style="flex-grow: 1;">Register</button> </div>
        </div>

        <div id="dashboard-container" class="dashboard-card" style="display: none;">
            <h2 id="welcome-message">Welcome, Climate Guardian!</h2>
            
            <div class="wallet">
                <h3>My Green Points Wallet</h3>
                <div id="points-display">0</div>
                <p>Points</p>
            </div>

            <h4>Log Your Eco-Actions!</h4>
            <p>The Admin will verify your actions and may award more points!</p>
            <div class="action-buttons">
                <button class="action-btn" data-points="5">♻️ Recycled Waste (+5)</button>
                <button class="action-btn" data-points="5">💡 Saved Energy (+5)</button>
                <button class="action-btn" data-points="10">🚲 Used Public Transport (+10)</button>
                <button class="action-btn" data-points="10">💧 Saved Water (+10)</button>
                <button class="action-btn" data-points="-1" style="background-color: #e74c3c;">Oops/Mistake (-1)</button>
            </div>
            
            <button id="logout-button" class="logout-btn">Log Out</button>
        </div>
    </main>

    <script src="dashboard.js"></script>
</body>
</html>
```



#dashboard.js
```
// === THIS IS THE NEW, UPDATED dashboard.js (Student) FILE CONNECTED TO FLASK BACKEND ===

document.addEventListener("DOMContentLoaded", () => {
    // Flask server runs on port 5000 by default
    const API_BASE_URL = 'http://127.0.0.1:5000/api'; 

    // Get all the elements
    const loginContainer = document.getElementById("login-container");
    const dashboardContainer = document.getElementById("dashboard-container");
    const loginButton = document.getElementById("login-button");
    const usernameInput = document.getElementById("username-input");
    const passwordInput = document.getElementById("password-input"); // New: Password input
    const welcomeMessage = document.getElementById("welcome-message");
    const pointsDisplay = document.getElementById("points-display");
    const logoutButton = document.getElementById("logout-button");
    const actionButtons = document.querySelectorAll(".action-btn");
    const registerButton = document.getElementById("register-button"); // New: Register button

    let currentUser = null; // Store current logged-in username

    // --- Core API Functions ---

    // Function to handle login via API
    async function handleLogin() {
        const username = usernameInput.value.trim();
        const password = passwordInput.value.trim();

        if (!username || !password) {
            alert("Please enter both username and password!");
            return;
        }

        try {
            const response = await fetch(`${API_BASE_URL}/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });

            const data = await response.json();

            if (response.ok) {
                // Successful login
                currentUser = data.username;
                sessionStorage.setItem('loggedInUser', currentUser); // Store session (not points)
                showDashboard(currentUser, data.points);
            } else {
                alert(`Login failed: ${data.message}`);
            }
        } catch (error) {
            console.error('Login error:', error);
            alert('Could not connect to the server.');
        }
    }

    // Function to handle registration via API
    async function handleRegister() {
        const username = usernameInput.value.trim();
        const password = passwordInput.value.trim();

        if (!username || !password) {
            alert("Please enter both username and password!");
            return;
        }

        try {
            const response = await fetch(`${API_BASE_URL}/register`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });

            const data = await response.json();

            if (response.ok) {
                alert(`Registration successful! You can now log in.`);
            } else {
                alert(`Registration failed: ${data.message}`);
            }
        } catch (error) {
            console.error('Registration error:', error);
            alert('Could not connect to the server.');
        }
    }

    // Function to update points via API
    async function addPoints(pointsToAdd) {
        if (!currentUser) return;

        try {
            const response = await fetch(`${API_BASE_URL}/update_points`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ 
                    username: currentUser, 
                    points_to_add: pointsToAdd 
                })
            });

            const data = await response.json();

            if (response.ok) {
                updatePointsDisplay(data.new_points);
            } else {
                alert(`Error logging action: ${data.message}`);
            }
        } catch (error) {
            console.error('Update points error:', error);
            alert('Could not connect to the server to log action.');
        }
    }
    
    // --- UI Functions ---
    
    function updatePointsDisplay(points) {
        pointsDisplay.textContent = points;
    }

    function showDashboard(username, points) {
        welcomeMessage.textContent = `Welcome, ${username}!`;
        updatePointsDisplay(points);
        loginContainer.style.display = "none";
        dashboardContainer.style.display = "block";
    }

    function showLogin() {
        currentUser = null;
        sessionStorage.removeItem('loggedInUser');
        loginContainer.style.display = "block";
        dashboardContainer.style.display = "none";
        usernameInput.value = "";
        passwordInput.value = "";
    }
    
    function logout() {
        showLogin();
    }
    
    // --- Initial Load & Event Listeners ---
    
    // Check for existing session
    const storedUser = sessionStorage.getItem('loggedInUser');
    if (storedUser) {
        // If session exists, we should ideally fetch the current points 
        // We'll rely on the user to re-login for simplicity in this prototype.
        // For a full app, you'd add a '/fetch_points' endpoint.
        // For now, prompt relogin or let the user login button handle it.
        showLogin(); 
        usernameInput.value = storedUser;
        alert("Welcome back! Please enter your password to load your current points.");
    }

    loginButton.addEventListener("click", handleLogin);
    registerButton.addEventListener("click", handleRegister);
    logoutButton.addEventListener("click", logout);

    actionButtons.forEach(button => {
        button.addEventListener("click", () => {
            if (!currentUser) {
                alert("Please log in first!");
                return;
            }
            const points = parseInt(button.dataset.points, 10);
            addPoints(points);
        });
    });
});
```



#index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Climate Guardians Hub</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <nav>
        <div class="logo">🌍 Climate Guardians</div>
        <ul>
            <li><a href="index.html" class="active">Home</a></li>
            <li><a href="#impact">Impacts</a></li>
            <li><a href="#solutions">Solutions</a></li>
            <li><a href="dashboard.html" class="cta-button">My Green Points</a></li>
            <li><a href="admin.html">Admin</a></li>
        </ul>
    </nav>

    <header>
        <h1>Our School's Guide to a Changing World</h1>
        <p>Understand the facts, see the impact, and be the solution.</p>
        
    </header>

    <main>
        <section id="basics" class="content-section">
            <h2>🔬 Section 1: The Climate Crisis Today - Understanding the Basics</h2>
            <p><strong>Climate change</strong> refers to long-term shifts in global temperatures and weather patterns. While Earth's climate has always changed, the changes we see today are happening much faster and are primarily driven by <strong>human activities</strong>.</p>
            
            <h3>What's Happening? The Warming Planet</h3>
            <p>Since the 1800s, activities like burning <strong>fossil fuels</strong> (coal, oil, and gas) for energy have released huge amounts of "greenhouse gases" into the atmosphere.</p>
            
            <div class="container">
                <div class="text-block">
                    <h4>The Greenhouse Effect Explained</h4>
                    <p>Gases like Carbon Dioxide ($\text{CO}_2$) naturally trap some of the sun's heat, keeping Earth warm. This is the <strong>natural greenhouse effect</strong>.</p>
                    <p>By burning fossil fuels, we add *extra* $\text{CO}_2$, making the blanket too thick. This <strong>enhanced greenhouse effect</strong> traps too much heat, causing the planet to warm up rapidly.</p>
                </div>
                <div class="image-block">
                    

                    <img src="greenhouse.png" alt="Diagram illustrating the enhanced greenhouse effect trapping heat.">

                </div>
            </div>

            <h3>The Evidence is Clear</h3>
            <ul>
                <li><strong>Rising Temperatures:</strong> The last decade (2011-2020) was the warmest on record.</li>
                <li><strong>Melting Ice:</strong> Glaciers and polar ice sheets are melting at an alarming rate.</li>
                <li><strong>Rising Sea Levels:</strong> As ice melts and ocean water warms (it expands!), sea levels rise, threatening coastal areas.</li>
            </ul>
            
        </section>

        <section id="impact" class="content-section gray-bg">
            <h2>🔥 Section 2: Its Impact on Our Planet - A World in Flux</h2>
            <p>This extra warmth disrupts the entire climate system, leading to severe consequences for natural environments and weather.</p>

            <div class="container reverse">
                <div class="text-block">
                    <h4>More Extreme Weather</h4>
                    <p>A warmer world means more energy in the atmosphere, fueling more intense weather events.</p>
                    <ul>
                        <li><strong>Intense Heatwaves:</strong> Causing health risks and stressing water supplies.</li>
                        <li><strong>Severe Storms:</strong> Warmer air holds more moisture, leading to heavier rainfall, floods, and more powerful hurricanes.</li>
                        <li><strong>Widespread Droughts:</strong> Changing patterns mean some areas get far too little water, harming crops and leading to wildfires.</li>
                    </ul>
                </div>
                <div class="image-block">
                   <img src="landbreak.png" alt="Diagram illustrating the enhanced greenhouse effect trapping heat."> 
                </div>
            </div>

            <h4>Impact on Oceans and Wildlife</h4>
            <p>The oceans have absorbed over 90% of the extra heat. This has major impacts:</p>
            <ul>
                <li><strong>Ocean Acidification:</strong> As the ocean absorbs $\text{CO}_2$, it becomes more acidic, harming marine life like coral.</li>
                <li><strong>Coral Bleaching:</strong> Warmer water causes coral reefs to turn white, threatening their survival.</li>
                <li><strong>Loss of Biodiversity:</strong> Many animals and plants are struggling to adapt as their habitats change, leading to an increased risk of extinction.</li>
            </ul>
            
        </section>

        <section id="solutions" class="content-section">
            <h2>💡 Section 3: How We Deal With It: Solutions</h2>
            <p>We have the solutions! From renewable energy like solar and wind to protecting our forests, we know what to do. At school, we can Reduce, Reuse, and Recycle, save energy, and use our voices to ask for change.</p>
            <a href="dashboard.html" class="cta-button">Join the Challenge & Earn Points!</a>
        </section>
    </main>

    <footer>
        <p>Climate Guardians Hub - An SDG 4 & 13 Project</p>
    </footer>
</body>
</html>
```

#style.css
```
/* Basic Setup */
body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f7f6;
    color: #333;
}

.container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    align-items: center;
    margin: 20px 0;
}
.container.reverse { flex-direction: row-reverse; }
.text-block { flex: 2; min-width: 300px; }
.image-block { flex: 1; min-width: 250px; }
.image-block img { width: 100%; border-radius: 8px; }

/* Navigation */
nav {
    background-color: #fff;
    padding: 10px 5%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    position: sticky;
    top: 0;
    z-index: 100;
}
nav .logo {
    font-size: 1.5em;
    font-weight: 700;
    color: #2c3e50;
}
nav ul {
    list-style: none;
    display: flex;
    gap: 20px;
    margin: 0;
    padding: 0;
}
nav ul li a {
    text-decoration: none;
    color: #555;
    font-weight: 600;
    transition: color 0.3s;
}
nav ul li a:hover, nav ul li a.active {
    color: #27ae60;
}

/* Call-to-Action Button */
.cta-button {
    background-color: #2ecc71;
    color: #fff !important;
    padding: 10px 20px;
    border-radius: 50px;
    transition: background-color 0.3s, transform 0.3s;
    border: none;
    cursor: pointer;
    font-size: 1em;
    font-weight: 600;
}
.cta-button:hover {
    background-color: rgb(39, 174, 96);
    transform: translateY(-2px);
}

/* Header */
header {
    background: linear-gradient(rgb(39, 174, 96), rgb(39, 174, 96)), url('https://source.unsplash.com/1600x600/?nature,earth') no-repeat center center/cover;
    color: #fff;
    text-align: center;
    padding: 100px 20px;
}
header h1 {
    font-size: 3em;
    margin-bottom: 10px;
}

/* Content Sections */
.content-section {
    max-width: 1000px;
    margin: 40px auto;
    padding: 20px;
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}
.content-section h2 {
    color: #2c3e50;
    border-bottom: 3px solid #2ecc71;
    display: inline-block;
    padding-bottom: 5px;
}
.content-section.gray-bg {
    background-color: #ecf0f1;
}

/* Footer */
footer {
    text-align: center;
    padding: 20px;
    background-color: #2c3e50;
    color: #fff;
    margin-top: 40px;
}

/* --- Dashboard Styles --- */
.dashboard-main {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding: 40px 20px;
    min-height: 70vh;
}
.dashboard-card {
    background-color: #fff;
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    width: 100%;
    max-width: 500px;
    text-align: center;
}
#login-container input {
    width: 80%;
    padding: 12px;
    margin: 20px 0;
    border: 1px solid #ccc;
    border-radius: 8px;
    font-size: 1em;
}

.wallet {
    background-color: #2ecc71;
    color: #fff;
    padding: 20px;
    border-radius: 10px;
    margin: 20px 0;
}
.wallet h3 {
    margin: 0 0 10px 0;
}
#points-display {
    font-size: 4em;
    font-weight: 700;
    line-height: 1;
}

.action-buttons {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
    margin: 20px 0;
}
.action-btn {
    padding: 15px;
    border: none;
    border-radius: 8px;
    background-color: #3498db;
    color: #fff;
    font-size: 1em;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s;
}
.action-btn:hover {
    transform: scale(1.05);
}
.action-btn[data-points="50"] { background-color: #27ae60; }
.action-btn[data-points="30"] { background-color: #f39c12; }

#logout-button {
    background: none;
    border: none;
    color: #e74c3c;
    cursor: pointer;
    font-size: 1em;
    margin-top: 20px;
    text-decoration: underline;
}
```
