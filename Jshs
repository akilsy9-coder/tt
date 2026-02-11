# namos.py - Complete Restaurant Management System
from flask import Flask, render_template_string, request, redirect, url_for, flash, session, jsonify
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime, timedelta
import os

app = Flask(__name__)
app.secret_key = 'namos-secret-key-2024'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///namos.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# ============ MODELS ============
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(100), unique=True, nullable=False)
    username = db.Column(db.String(50), unique=True, nullable=False)
    password = db.Column(db.String(200), nullable=False)
    full_name = db.Column(db.String(100))
    department = db.Column(db.String(20))  # bar, kitchen, service, admin
    role = db.Column(db.String(20), default='staff')
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Room(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), unique=True, nullable=False)
    password = db.Column(db.String(200))
    is_locked = db.Column(db.Boolean, default=True)

class RoomMember(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    room_id = db.Column(db.Integer, db.ForeignKey('room.id'), nullable=False)
    joined_at = db.Column(db.DateTime, default=datetime.utcnow)

class Message(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    room_id = db.Column(db.Integer, db.ForeignKey('room.id'))
    content = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Reservation(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    customer_name = db.Column(db.String(100), nullable=False)
    customer_phone = db.Column(db.String(20), nullable=False)
    table_number = db.Column(db.String(10), nullable=False)
    persons = db.Column(db.Integer, nullable=False)
    reservation_date = db.Column(db.Date, nullable=False)
    reservation_time = db.Column(db.String(10), nullable=False)
    status = db.Column(db.String(20), default='confirmed')
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

class Order(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    table_number = db.Column(db.String(10), nullable=False)
    items = db.Column(db.Text, nullable=False)
    department = db.Column(db.String(20), nullable=False)  # bar, kitchen
    status = db.Column(db.String(20), default='pending')
    priority = db.Column(db.String(10), default='normal')
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

# ============ HTML TEMPLATES ============
BASE_HTML = '''
<!DOCTYPE html>
<html>
<head>
    <title>Namos Restaurant</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: #f5f5f5; color: #333; }
        .navbar { background: white; padding: 15px 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .nav-content { max-width: 1200px; margin: 0 auto; display: flex; justify-content: space-between; align-items: center; }
        .logo { font-size: 24px; font-weight: bold; color: #4a6ee0; text-decoration: none; }
        .nav-links { display: flex; gap: 20px; }
        .nav-link { color: #555; text-decoration: none; padding: 8px 16px; border-radius: 5px; }
        .nav-link:hover { background: #f0f0f0; }
        .container { max-width: 1200px; margin: 30px auto; padding: 0 20px; }
        .card { background: white; padding: 25px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; }
        .btn { padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; text-decoration: none; display: inline-block; }
        .btn-primary { background: #4a6ee0; color: white; }
        .btn-success { background: #28a745; color: white; }
        .btn-danger { background: #dc3545; color: white; }
        .form-group { margin-bottom: 15px; }
        .form-control { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 5px; }
        .rooms-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; margin-top: 20px; }
        .room-card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); text-decoration: none; color: inherit; }
        .room-card:hover { transform: translateY(-2px); box-shadow: 0 4px 15px rgba(0,0,0,0.2); }
        .message-box { max-height: 400px; overflow-y: auto; padding: 15px; background: #f9f9f9; border-radius: 5px; margin-bottom: 15px; }
        .message { margin-bottom: 10px; padding: 10px; background: white; border-radius: 5px; border-left: 4px solid #4a6ee0; }
        .alert { padding: 10px 15px; border-radius: 5px; margin-bottom: 15px; }
        .alert-success { background: #d4edda; color: #155724; }
        .alert-danger { background: #f8d7da; color: #721c24; }
        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
        .stat-card { background: white; padding: 20px; border-radius: 10px; text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .stat-value { font-size: 32px; font-weight: bold; color: #4a6ee0; }
        table { width: 100%; border-collapse: collapse; background: white; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
        th { background: #f8f9fa; font-weight: bold; }
    </style>
</head>
<body>
    <nav class="navbar">
        <div class="nav-content">
            <a href="/" class="logo">🍽️ NAMOS</a>
            <div class="nav-links">
                {% if session.user_id %}
                    <a href="/dashboard" class="nav-link">Dashboard</a>
                    <a href="/rooms" class="nav-link">Rooms</a>
                    <a href="/reservations" class="nav-link">Reservations</a>
                    <a href="/orders" class="nav-link">Orders</a>
                    <a href="/logout" class="nav-link">Logout</a>
                {% else %}
                    <a href="/login" class="nav-link">Login</a>
                    <a href="/register" class="nav-link">Register</a>
                {% endif %}
            </div>
        </div>
    </nav>
    <div class="container">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </div>
</body>
</html>
'''

# ============ ROUTES ============
@app.route('/')
def index():
    if 'user_id' in session:
        return redirect('/dashboard')
    return render_template_string(BASE_HTML.replace('{% block content %}', '''
        <div class="card text-center">
            <h1 style="margin-bottom: 20px;">Welcome to Namos Restaurant</h1>
            <p style="margin-bottom: 20px; color: #666;">Complete restaurant management system</p>
            <div style="display: flex; gap: 15px; justify-content: center;">
                <a href="/login" class="btn btn-primary">Login</a>
                <a href="/register" class="btn btn-success">Register</a>
            </div>
        </div>
    '''))

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        email = request.form['email']
        username = request.form['username']
        password = request.form['password']
        full_name = request.form['full_name']
        department = request.form['department']
        
        if User.query.filter_by(email=email).first():
            flash('Email already exists', 'danger')
            return redirect('/register')
        
        if User.query.filter_by(username=username).first():
            flash('Username already exists', 'danger')
            return redirect('/register')
        
        user = User(
            email=email,
            username=username,
            password=generate_password_hash(password),
            full_name=full_name,
            department=department
        )
        
        db.session.add(user)
        db.session.commit()
        
        flash('Registration successful! Please login.', 'success')
        return redirect('/login')
    
    return render_template_string(BASE_HTML.replace('{% block content %}', '''
        <div class="card">
            <h2 style="margin-bottom: 20px;">Register</h2>
            <form method="POST">
                <div class="form-group">
                    <input type="text" name="full_name" class="form-control" placeholder="Full Name" required>
                </div>
                <div class="form-group">
                    <input type="text" name="username" class="form-control" placeholder="Username" required>
                </div>
                <div class="form-group">
                    <input type="email" name="email" class="form-control" placeholder="Email" required>
                </div>
                <div class="form-group">
                    <input type="password" name="password" class="form-control" placeholder="Password" required>
                </div>
                <div class="form-group">
                    <select name="department" class="form-control" required>
                        <option value="">Select Department</option>
                        <option value="bar">Bar</option>
                        <option value="kitchen">Kitchen</option>
                        <option value="service">Service</option>
                        <option value="admin">Admin</option>
                    </select>
                </div>
                <button type="submit" class="btn btn-primary">Register</button>
                <a href="/login" class="btn">Already have account? Login</a>
            </form>
        </div>
    '''))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        email = request.form['email']
        password = request.form['password']
        
        user = User.query.filter_by(email=email).first()
        
        if user and check_password_hash(user.password, password):
            session['user_id'] = user.id
            session['username'] = user.username
            session['department'] = user.department
            session['role'] = user.role
            flash('Login successful!', 'success')
            return redirect('/dashboard')
        else:
            flash('Invalid email or password', 'danger')
    
    return render_template_string(BASE_HTML.replace('{% block content %}', '''
        <div class="card">
            <h2 style="margin-bottom: 20px;">Login</h2>
            <form method="POST">
                <div class="form-group">
                    <input type="email" name="email" class="form-control" placeholder="Email" required>
                </div>
                <div class="form-group">
                    <input type="password" name="password" class="form-control" placeholder="Password" required>
                </div>
                <button type="submit" class="btn btn-primary">Login</button>
                <a href="/register" class="btn">Don't have account? Register</a>
            </form>
            <hr style="margin: 20px 0;">
            <div style="background: #f8f9fa; padding: 15px; border-radius: 5px;">
                <h4>Test Accounts:</h4>
                <p><strong>Admin:</strong> admin@namos.com / admin123</p>
                <p><strong>Kitchen:</strong> kitchen@namos.com / kitchen123</p>
                <p><strong>Bar:</strong> bar@namos.com / bar123</p>
            </div>
        </div>
    '''))

@app.route('/dashboard')
def dashboard():
    if 'user_id' not in session:
        return redirect('/login')
    
    user = User.query.get(session['user_id'])
    
    # Stats
    today = datetime.now().date()
    reservations_today = Reservation.query.filter_by(reservation_date=today).count()
    active_orders = Order.query.filter(Order.status.in_(['pending', 'preparing'])).count()
    
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <h1 style="margin-bottom: 20px;">Welcome, {user.full_name or user.username}!</h1>
        
        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-value">{reservations_today}</div>
                <div>Today's Reservations</div>
            </div>
            <div class="stat-card">
                <div class="stat-value">{active_orders}</div>
                <div>Active Orders</div>
            </div>
            <div class="stat-card">
                <div class="stat-value">{User.query.count()}</div>
                <div>Total Staff</div>
            </div>
            <div class="stat-card">
                <div class="stat-value">{Room.query.count()}</div>
                <div>Active Rooms</div>
            </div>
        </div>
        
        <div class="card">
            <h2 style="margin-bottom: 15px;">Quick Actions</h2>
            <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px;">
                <a href="/rooms" class="btn btn-primary">Join Room</a>
                <a href="/reservations/new" class="btn btn-success">New Reservation</a>
                <a href="/orders/new" class="btn btn-primary">Create Order</a>
                <a href="/reservations" class="btn">View All Reservations</a>
            </div>
        </div>
        
        <div class="card">
            <h2 style="margin-bottom: 15px;">Today's Reservations</h2>
            <table>
                <thead>
                    <tr>
                        <th>Customer</th>
                        <th>Table</th>
                        <th>Time</th>
                        <th>Persons</th>
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody>
                    {"".join([f'''
                    <tr>
                        <td>{r.customer_name}</td>
                        <td>{r.table_number}</td>
                        <td>{r.reservation_time}</td>
                        <td>{r.persons}</td>
                        <td><span style="padding: 3px 8px; border-radius: 3px; background: {'#d4edda' if r.status == 'confirmed' else '#f8d7da'}">{r.status}</span></td>
                    </tr>
                    ''' for r in Reservation.query.filter_by(reservation_date=today).limit(5).all()])}
                </tbody>
            </table>
        </div>
    '''))

@app.route('/rooms')
def rooms():
    if 'user_id' not in session:
        return redirect('/login')
    
    rooms = Room.query.all()
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <div class="card">
            <h2 style="margin-bottom: 20px;">Rooms</h2>
            <p style="margin-bottom: 20px; color: #666;">Join department rooms to collaborate</p>
            
            <div class="rooms-grid">
                {"".join([f'''
                <a href="/room/{room.name.lower()}" class="room-card">
                    <h3>{room.name} Room</h3>
                    <p style="color: #666; margin: 10px 0;">{room.name} department communication</p>
                    <div style="display: flex; justify-content: space-between; margin-top: 15px;">
                        <span>{"🔒 Locked" if room.is_locked else "🔓 Open"}</span>
                        <span>👥 {RoomMember.query.filter_by(room_id=room.id).count()} members</span>
                    </div>
                </a>
                ''' for room in rooms])}
            </div>
        </div>
        
        {f'''
        <div class="card">
            <h3 style="margin-bottom: 15px;">Create New Room (Admin Only)</h3>
            <form method="POST" action="/rooms/create">
                <div style="display: grid; grid-template-columns: 1fr 1fr 1fr auto; gap: 10px;">
                    <input type="text" name="name" class="form-control" placeholder="Room Name" required>
                    <input type="password" name="password" class="form-control" placeholder="Password (optional)">
                    <select name="locked" class="form-control">
                        <option value="true">Locked</option>
                        <option value="false">Open</option>
                    </select>
                    <button type="submit" class="btn btn-primary">Create</button>
                </div>
            </form>
        </div>
        ''' if session.get('role') == 'admin' else ''}
    '''))

@app.route('/rooms/create', methods=['POST'])
def create_room():
    if 'user_id' not in session or session.get('role') != 'admin':
        flash('Admin access required', 'danger')
        return redirect('/rooms')
    
    name = request.form['name']
    password = request.form.get('password', '')
    is_locked = request.form.get('locked') == 'true'
    
    if Room.query.filter_by(name=name).first():
        flash('Room already exists', 'danger')
        return redirect('/rooms')
    
    room = Room(
        name=name,
        password=generate_password_hash(password) if password else None,
        is_locked=is_locked
    )
    
    db.session.add(room)
    db.session.commit()
    
    flash('Room created successfully', 'success')
    return redirect('/rooms')

@app.route('/room/<room_name>')
def room_chat(room_name):
    if 'user_id' not in session:
        return redirect('/login')
    
    room = Room.query.filter_by(name=room_name.capitalize()).first()
    if not room:
        flash('Room not found', 'danger')
        return redirect('/rooms')
    
    # Check if user is member
    is_member = RoomMember.query.filter_by(user_id=session['user_id'], room_id=room.id).first()
    
    if not is_member and room.is_locked:
        flash('Room is locked. Please request access.', 'danger')
        return redirect('/rooms')
    
    # Add user to room if not member
    if not is_member:
        member = RoomMember(user_id=session['user_id'], room_id=room.id)
        db.session.add(member)
        db.session.commit()
    
    # Get messages
    messages = Message.query.filter_by(room_id=room.id).order_by(Message.created_at.desc()).limit(50).all()
    
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <div class="card">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
                <h2>{room.name} Room</h2>
                <a href="/rooms" class="btn">← Back to Rooms</a>
            </div>
            
            <div class="message-box">
                {"".join([f'''
                <div class="message">
                    <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                        <strong>{User.query.get(m.user_id).username}</strong>
                        <small style="color: #666;">{m.created_at.strftime("%H:%M")}</small>
                    </div>
                    <div>{m.content}</div>
                </div>
                ''' for m in reversed(messages)])}
            </div>
            
            <form method="POST" action="/room/{room_name}/message">
                <div style="display: flex; gap: 10px;">
                    <input type="text" name="message" class="form-control" placeholder="Type your message..." required>
                    <button type="submit" class="btn btn-primary">Send</button>
                </div>
            </form>
        </div>
    '''))

@app.route('/room/<room_name>/message', methods=['POST'])
def send_message(room_name):
    if 'user_id' not in session:
        return redirect('/login')
    
    room = Room.query.filter_by(name=room_name.capitalize()).first()
    if not room:
        flash('Room not found', 'danger')
        return redirect('/rooms')
    
    message = request.form['message']
    if message:
        msg = Message(
            user_id=session['user_id'],
            room_id=room.id,
            content=message
        )
        db.session.add(msg)
        db.session.commit()
    
    return redirect(f'/room/{room_name}')

@app.route('/reservations')
def reservations():
    if 'user_id' not in session:
        return redirect('/login')
    
    all_reservations = Reservation.query.order_by(Reservation.reservation_date.desc()).all()
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
            <h1>Reservations</h1>
            <a href="/reservations/new" class="btn btn-success">+ New Reservation</a>
        </div>
        
        <div class="card">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Customer</th>
                        <th>Phone</th>
                        <th>Table</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Persons</th>
                        <th>Status</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {"".join([f'''
                    <tr>
                        <td>#{r.id}</td>
                        <td>{r.customer_name}</td>
                        <td>{r.customer_phone}</td>
                        <td>{r.table_number}</td>
                        <td>{r.reservation_date}</td>
                        <td>{r.reservation_time}</td>
                        <td>{r.persons}</td>
                        <td>
                            <span style="padding: 3px 8px; border-radius: 3px; background: 
                                {'#d4edda' if r.status == 'confirmed' else 
                                 '#f8d7da' if r.status == 'cancelled' else 
                                 '#fff3cd'}; color: 
                                {'#155724' if r.status == 'confirmed' else 
                                 '#721c24' if r.status == 'cancelled' else 
                                 '#856404'}">
                                {r.status}
                            </span>
                        </td>
                        <td>
                            <a href="/reservations/{r.id}/cancel" class="btn btn-danger" style="padding: 5px 10px; font-size: 12px;">Cancel</a>
                        </td>
                    </tr>
                    ''' for r in all_reservations])}
                </tbody>
            </table>
        </div>
    '''))

@app.route('/reservations/new', methods=['GET', 'POST'])
def new_reservation():
    if 'user_id' not in session:
        return redirect('/login')
    
    if request.method == 'POST':
        reservation = Reservation(
            customer_name=request.form['customer_name'],
            customer_phone=request.form['customer_phone'],
            table_number=request.form['table_number'],
            persons=int(request.form['persons']),
            reservation_date=datetime.strptime(request.form['date'], '%Y-%m-%d').date(),
            reservation_time=request.form['time']
        )
        
        db.session.add(reservation)
        db.session.commit()
        
        flash('Reservation created successfully!', 'success')
        return redirect('/reservations')
    
    return render_template_string(BASE_HTML.replace('{% block content %}', '''
        <div class="card">
            <h2 style="margin-bottom: 20px;">New Reservation</h2>
            <form method="POST">
                <div class="form-group">
                    <input type="text" name="customer_name" class="form-control" placeholder="Customer Name" required>
                </div>
                <div class="form-group">
                    <input type="tel" name="customer_phone" class="form-control" placeholder="Phone Number" required>
                </div>
                <div class="form-group">
                    <input type="text" name="table_number" class="form-control" placeholder="Table Number" required>
                </div>
                <div class="form-group">
                    <input type="number" name="persons" class="form-control" placeholder="Number of Persons" min="1" required>
                </div>
                <div class="form-group">
                    <input type="date" name="date" class="form-control" required>
                </div>
                <div class="form-group">
                    <input type="time" name="time" class="form-control" required>
                </div>
                <button type="submit" class="btn btn-primary">Create Reservation</button>
                <a href="/reservations" class="btn">Cancel</a>
            </form>
        </div>
    '''))

@app.route('/reservations/<int:id>/cancel')
def cancel_reservation(id):
    if 'user_id' not in session:
        return redirect('/login')
    
    reservation = Reservation.query.get(id)
    if reservation:
        reservation.status = 'cancelled'
        db.session.commit()
        flash('Reservation cancelled', 'success')
    
    return redirect('/reservations')

@app.route('/orders')
def orders():
    if 'user_id' not in session:
        return redirect('/login')
    
    user_dept = session.get('department')
    if user_dept == 'admin':
        orders_list = Order.query.order_by(Order.created_at.desc()).all()
    else:
        orders_list = Order.query.filter_by(department=user_dept).order_by(Order.created_at.desc()).all()
    
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
            <h1>Orders ({session.get('department', '').capitalize()})</h1>
            <a href="/orders/new" class="btn btn-success">+ New Order</a>
        </div>
        
        <div class="card">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Table</th>
                        <th>Items</th>
                        <th>Department</th>
                        <th>Priority</th>
                        <th>Status</th>
                        <th>Created</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {"".join([f'''
                    <tr>
                        <td>#{o.id}</td>
                        <td>{o.table_number}</td>
                        <td>{o.items[:50]}...</td>
                        <td>{o.department}</td>
                        <td>
                            <span style="padding: 3px 8px; border-radius: 3px; background: 
                                {'#f8d7da' if o.priority == 'urgent' else 
                                 '#fff3cd' if o.priority == 'high' else 
                                 '#d4edda'}; color: 
                                {'#721c24' if o.priority == 'urgent' else 
                                 '#856404' if o.priority == 'high' else 
                                 '#155724'}">
                                {o.priority}
                            </span>
                        </td>
                        <td>
                            <span style="padding: 3px 8px; border-radius: 3px; background: 
                                {'#fff3cd' if o.status == 'pending' else 
                                 '#d1ecf1' if o.status == 'preparing' else 
                                 '#d4edda'}; color: 
                                {'#856404' if o.status == 'pending' else 
                                 '#0c5460' if o.status == 'preparing' else 
                                 '#155724'}">
                                {o.status}
                            </span>
                        </td>
                        <td>{o.created_at.strftime("%H:%M")}</td>
                        <td>
                            <a href="/orders/{o.id}/complete" class="btn btn-success" style="padding: 5px 10px; font-size: 12px;">Complete</a>
                        </td>
                    </tr>
                    ''' for o in orders_list])}
                </tbody>
            </table>
        </div>
    '''))

@app.route('/orders/new', methods=['GET', 'POST'])
def new_order():
    if 'user_id' not in session:
        return redirect('/login')
    
    if request.method == 'POST':
        order = Order(
            table_number=request.form['table_number'],
            items=request.form['items'],
            department=request.form['department'],
            priority=request.form['priority']
        )
        
        db.session.add(order)
        db.session.commit()
        
        flash('Order created successfully!', 'success')
        return redirect('/orders')
    
    return render_template_string(BASE_HTML.replace('{% block content %}', f'''
        <div class="card">
            <h2 style="margin-bottom: 20px;">New Order</h2>
            <form method="POST">
                <div class="form-group">
                    <input type="text" name="table_number" class="form-control" placeholder="Table Number" required>
                </div>
                <div class="form-group">
                    <textarea name="items" class="form-control" placeholder="Order items (one per line)" rows="4" required></textarea>
                </div>
                <div class="form-group">
                    <select name="department" class="form-control" required>
                        <option value="">Select Department</option>
                        <option value="bar">Bar</option>
                        <option value="kitchen">Kitchen</option>
                    </select>
                </div>
                <div class="form-group">
                    <select name="priority" class="form-control" required>
                        <option value="normal">Normal</option>
                        <option value="high">High</option>
                        <option value="urgent">Urgent</option>
                    </select>
                </div>
                <button type="submit" class="btn btn-primary">Create Order</button>
                <a href="/orders" class="btn">Cancel</a>
            </form>
        </div>
    '''))

@app.route('/orders/<int:id>/complete')
def complete_order(id):
    if 'user_id' not in session:
        return redirect('/login')
    
    order = Order.query.get(id)
    if order:
        order.status = 'completed'
        db.session.commit()
        flash('Order marked as completed', 'success')
    
    return redirect('/orders')

@app.route('/logout')
def logout():
    session.clear()
    flash('Logged out successfully', 'success')
    return redirect('/login')

# ============ INITIALIZATION ============
def init_database():
    with app.app_context():
        db.create_all()
        
        # Create default rooms
        rooms = [
            {'name': 'Bar', 'password': 'bar123', 'locked': True},
            {'name': 'Kitchen', 'password': 'kitchen123', 'locked': True},
            {'name': 'Service', 'password': 'service123', 'locked': True},
            {'name': 'Admin', 'password': 'admin123', 'locked': True},
            {'name': 'Public', 'password': '', 'locked': False}
        ]
        
        for room_data in rooms:
            if not Room.query.filter_by(name=room_data['name']).first():
                room = Room(
                    name=room_data['name'],
                    password=generate_password_hash(room_data['password']) if room_data['password'] else None,
                    is_locked=room_data['locked']
                )
                db.session.add(room)
        
        # Create default users
        users = [
            {'email': 'admin@namos.com', 'username': 'admin', 'password': 'admin123', 
             'full_name': 'Admin User', 'department': 'admin', 'role': 'admin'},
            {'email': 'kitchen@namos.com', 'username': 'chef', 'password': 'kitchen123',
             'full_name': 'Kitchen Staff', 'department': 'kitchen', 'role': 'staff'},
            {'email': 'bar@namos.com', 'username': 'bartender', 'password': 'bar123',
             'full_name': 'Bar Staff', 'department': 'bar', 'role': 'staff'},
            {'email': 'service@namos.com', 'username': 'waiter', 'password': 'service123',
             'full_name': 'Service Staff', 'department': 'service', 'role': 'staff'}
        ]
        
        for user_data in users:
            if not User.query.filter_by(email=user_data['email']).first():
                user = User(
                    email=user_data['email'],
                    username=user_data['username'],
                    password=generate_password_hash(user_data['password']),
                    full_name=user_data['full_name'],
                    department=user_data['department'],
                    role=user_data['role']
                )
                db.session.add(user)
        
        db.session.commit()
        print("✅ Database initialized successfully!")

# ============ RUN APP ============
if __name__ == '__main__':
    init_database()
    
    print("\n" + "="*60)
    print("🚀 NAMOS RESTAURANT MANAGEMENT SYSTEM")
    print("="*60)
    print("📡 Server: http://localhost:5000")
    print("\n🔑 TEST ACCOUNTS:")
    print("   👑 Admin:    admin@namos.com / admin123")
    print("   👨‍🍳 Kitchen: kitchen@namos.com / kitchen123")
    print("   🍸 Bar:      bar@namos.com / bar123")
    print("   💁 Service:  service@namos.com / service123")
    print("\n🔐 ROOM PASSWORDS:")
    print("   Bar Room: bar123")
    print("   Kitchen Room: kitchen123")
    print("   Service Room: service123")
    print("   Admin Room: admin123")
    print("   Public Room: no password")
    print("="*60)
    
    app.run(debug=True, host='0.0.0.0', port=5000)
