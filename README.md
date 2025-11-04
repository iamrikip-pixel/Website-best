# -------------------------------
# YOUR LOCAL SERVICE - Built by Rikip
# Professional Flask Web App (Deployable on Render/GitHub)
# -------------------------------

from flask import Flask, render_template_string, request, redirect, url_for, flash
import sqlite3, os, datetime

app = Flask(__name__)
app.secret_key = "rikip_secret"

DB_NAME = "local_service.db"

# ---------- DATABASE SETUP ----------
def init_db():
    if not os.path.exists(DB_NAME):
        conn = sqlite3.connect(DB_NAME)
        c = conn.cursor()
        c.execute('''CREATE TABLE providers (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT, service TEXT, location TEXT,
            phone TEXT, price INTEGER, rating REAL,
            description TEXT, image_url TEXT)''')
        c.execute('''CREATE TABLE bookings (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT, phone TEXT, message TEXT,
            provider_id INTEGER, created_at TEXT)''')
        demo = [
            ("Ravi Das","Electrician","Cooch Behar","9876543210",500,4.7,"Home wiring and repair","https://i.ibb.co/kGvHd4m/avatar2.png"),
            ("Anita Roy","Tutor","Siliguri","9998844221",600,4.9,"English & Maths tutor","https://i.ibb.co/y8RXbDv/avatar3.png"),
            ("Nisha Kapoor","Beautician","Jalpaiguri","9811223344",900,4.8,"Salon-quality services","https://i.ibb.co/bv8YYsF/avatar1.png")
        ]
        c.executemany("INSERT INTO providers (name, service, location, phone, price, rating, description, image_url) VALUES (?, ?, ?, ?, ?, ?, ?, ?)", demo)
        conn.commit(); conn.close()
init_db()

# ---------- ROUTES ----------
@app.route("/")
def home():
    q = request.args.get("q","")
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    if q:
        c.execute("SELECT * FROM providers WHERE name LIKE ? OR service LIKE ? OR location LIKE ?",
                  ('%'+q+'%','%'+q+'%','%'+q+'%'))
    else:
        c.execute("SELECT * FROM providers")
    data = c.fetchall(); conn.close()

    html = """
    <!DOCTYPE html><html><head>
    <title>Your Local Service - Built by Rikip</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
    body{font-family:'Poppins',sans-serif;background:#eef3f9;}
    .card{border-radius:15px;box-shadow:0 5px 15px rgba(0,0,0,0.1);transition:transform .3s;}
    .card:hover{transform:translateY(-5px);}
    .btn{transition:0.2s;}
    .btn:hover{opacity:0.8;}
    footer{margin-top:40px;color:#555;}
    </style></head>
    <body class="text-center">
    <nav class="navbar navbar-dark bg-primary shadow">
      <div class="container-fluid">
        <span class="navbar-brand mb-0 h1">💼 Your Local Service</span>
      </div>
    </nav>

    <div class="container mt-4">
      <h3>Find Trusted Local Experts</h3>
      <form class="d-flex justify-content-center mb-4" method="get">
        <input name="q" value="{{q}}" class="form-control w-50 me-2" placeholder="Search by name, service or location">
        <button class="btn btn-success" onclick="clickSound()">Search</button>
      </form>
      {% with messages = get_flashed_messages() %}
        {% if messages %}
        <div class="alert alert-info">{{ messages[0] }}</div>
        {% endif %}
      {% endwith %}
      <div class="row">
      {% for p in data %}
        <div class="col-md-4 mb-4">
          <div class="card p-2">
            <img src="{{p[8]}}" class="card-img-top rounded" style="height:200px;object-fit:cover;">
            <div class="card-body">
              <h5>{{p[1]}}</h5>
              <p>{{p[2]}} • {{p[3]}}</p>
              <p class="text-muted small">{{p[6]}}</p>
              <p><b>₹{{p[5]}}</b> ⭐ {{p[6]}}</p>
              <form method="post" action="/book/{{p[0]}}">
                <input name="name" class="form-control mb-1" placeholder="Your name" required>
                <input name="phone" class="form-control mb-1" placeholder="Phone" required>
                <textarea name="message" class="form-control mb-2" placeholder="Message"></textarea>
                <button class="btn btn-primary w-100" onclick="clickSound()">Book Now</button>
              </form>
            </div>
          </div>
        </div>
      {% endfor %}
      </div>
      <footer>
        © 2025 Your Local Service • Built by <b>Rikip</b><br>
        📧 iamrikip22llb@gmail.com | 📞 7810804100
      </footer>
    </div>

    <audio id="click" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_8b8be7b3f6.mp3?filename=interface-click-124467.mp3"></audio>
    <script>
    function clickSound(){var s=document.getElementById("click");s.currentTime=0;s.play();}
    </script>
    </body></html>
    """
    return render_template_string(html, data=data, q=q)

@app.route("/book/<int:pid>", methods=["POST"])
def book(pid):
    n=request.form["name"]; p=request.form["phone"]; m=request.form["message"]
    conn=sqlite3.connect(DB_NAME); c=conn.cursor()
    c.execute("INSERT INTO bookings (name, phone, message, provider_id, created_at) VALUES (?,?,?,?,?)",
              (n,p,m,pid,datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")))
    conn.commit(); conn.close()
    flash("✅ Booking submitted successfully!")
    return redirect(url_for("home"))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
