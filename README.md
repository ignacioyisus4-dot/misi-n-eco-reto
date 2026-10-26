from flask import Flask, render_template_string, request, redirect, url_for, g
import sqlite3

app = Flask(__name__)
DATABASE = 'eco_reto_nuevo.db'

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
        db.row_factory = sqlite3.Row
    return db

@app.teardown_appcontext
def close_db(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

def init_db():
    with app.app_context():
        db = get_db()
        cursor = db.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS usuarios (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nombre TEXT NOT NULL,
                usuario TEXT NOT NULL UNIQUE,
                contrasena TEXT NOT NULL,
                fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS acciones (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                usuario_id INTEGER NOT NULL,
                tipo TEXT NOT NULL,
                descripcion TEXT NOT NULL,
                fecha TEXT NOT NULL,
                impacto TEXT,
                FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
            )
        ''')
        db.commit()
        print("✅ Base de datos creada correctamente")

# ==================================
# PLANTILLAS JINJA2
# ==================================

TEMPLATE_LOGIN = '''
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Misión Eco-Reto</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: linear-gradient(135deg, #E8F5E9, #C8E6C9); min-height: 100vh; display: flex; align-items: center; justify-content: center; }
        .caja { background: white; padding: 40px 30px; border-radius: 16px; box-shadow: 0 8px 32px rgba(0,0,0,0.1); width: 100%; max-width: 360px; text-align: center; }
        h1 { color: #00695C; font-size: 26px; margin-bottom: 8px; }
        .sub { color: #666; font-size: 14px; margin-bottom: 25px; }
        input { width: 100%; padding: 13px; margin: 8px 0; border: 2px solid #E0E0E0; border-radius: 8px; font-size: 15px; }
        input:focus { outline: none; border-color: #4CAF50; }
        button { width: 100%; background: #00695C; color: white; border: none; padding: 13px; border-radius: 8px; font-size: 16px; font-weight: bold; cursor: pointer; margin-top: 8px; }
        button:hover { background: #004D40; }
        .enlace { margin-top: 20px; font-size: 14px; color: #666; }
        .enlace a { color: #00695C; text-decoration: none; font-weight: bold; }
        .mensaje { margin-top: 12px; padding: 10px; border-radius: 6px; font-size: 14px; }
        .error { background: #FFEBEE; color: #C62828; }
        .frase { margin-top: 25px; font-style: italic; color: #666; font-size: 14px; }
    </style>
</head>
<body>
    <div class="caja">
        <h1>🌿 Misión Eco-Reto</h1>
        <p class="sub">Ingresa para registrar tus acciones ecológicas</p>
        
        <!-- ✅ JavaScript -->
        <script>
        window.onload = function() {
            console.log("Página de inicio cargada");
        };
        </script>
        
        <form method="POST">
            <input type="text" name="usuario" placeholder="Usuario" required>
            <input type="password" name="contrasena" placeholder="Contraseña" required>
            <button type="submit">Entrar</button>
        </form>
        
        <div class="enlace">¿No tienes cuenta? <a href="{{ url_for('registro') }}">Regístrate</a></div>
        
        <!-- ✅ Jinja2: Condicional -->
        {% if error %}
        <div class="mensaje error">{{ error }}</div>
        {% endif %}
        
        <p class="frase">"Pequeñas acciones, grandes cambios"</p>
    </div>
</body>
</html>
'''

TEMPLATE_REGISTRO = '''
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Registro - Eco-Reto</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: linear-gradient(135deg, #E8F5E9, #C8E6C9); min-height: 100vh; display: flex; align-items: center; justify-content: center; padding: 20px; }
        .caja { background: white; padding: 35px; border-radius: 16px; box-shadow: 0 8px 32px rgba(0,0,0,0.1); width: 100%; max-width: 360px; }
        h2 { text-align: center; color: #00695C; margin-bottom: 25px; }
        input { width: 100%; padding: 12px; margin: 7px 0; border: 2px solid #E0E0E0; border-radius: 8px; font-size: 15px; }
        input:focus { outline: none; border-color: #4CAF50; }
        button { width: 100%; background: #00695C; color: white; border: none; padding: 12px; border-radius: 8px; font-size: 16px; font-weight: bold; cursor: pointer; margin-top: 8px; }
        button:hover { background: #004D40; }
        .enlace { margin-top: 20px; text-align: center; font-size: 14px; }
        .enlace a { color: #00695C; text-decoration: none; font-weight: bold; }
        .mensaje { padding: 10px; border-radius: 6px; margin-bottom: 15px; text-align: center; font-size: 14px; }
        .error { background: #FFEBEE; color: #C62828; }
        .exito { background: #E8F5E9; color: #2E7D32; }
    </style>
</head>
<body>
    <div class="caja">
        <h2>🌍 Crear cuenta</h2>
        
        <!-- ✅ Jinja2: Condicionales -->
        {% if exito %}
        <div class="mensaje exito">{{ exito|safe }}</div>
        {% endif %}
        
        {% if error %}
        <div class="mensaje error">{{ error }}</div>
        {% endif %}
        
        <form method="POST" id="formRegistro">
            <input type="text" name="nombre" placeholder="Nombre completo" required id="nombre">
            <input type="text" name="usuario" placeholder="Nombre de usuario" required id="usuario">
            <input type="password" name="contrasena" placeholder="Contraseña" required id="contrasena">
            <input type="password" name="confirmar" placeholder="Confirmar contraseña" required id="confirmar">
            <button type="submit">Registrarse</button>
        </form>
        
        <div class="enlace">¿Ya tienes cuenta? <a href="{{ url_for('login') }}">Inicia sesión</a></div>
        
        <!-- ✅ JavaScript: Validación -->
        <script>
        document.getElementById('formRegistro').addEventListener('submit', function(e) {
            const pass1 = document.getElementById('contrasena').value;
            const pass2 = document.getElementById('confirmar').value;
            if (pass1 !== pass2) {
                alert("❌ Las contraseñas no coinciden");
                e.preventDefault();
            }
        });
        </script>
    </div>
</body>
</html>
'''

TEMPLATE_INICIO = '''
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Inicio - Eco-Reto</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: #E8F5E9; padding: 30px; }
        .caja { max-width: 500px; margin: 0 auto; background: white; padding: 35px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
        h1 { color: #00695C; text-align: center; margin-bottom: 10px; }
        .bienvenida { text-align: center; color: #555; margin-bottom: 25px; }
        .tarjeta { background: #E8F5E9; padding: 20px; border-radius: 8px; text-align: center; margin-bottom: 20px; }
        .numero { font-size: 28px; font-weight: bold; color: #00695C; }
        .accion { background: #F1F8E9; padding: 12px; margin: 8px 0; border-radius: 6px; }
        .btn { display: inline-block; padding: 10px 20px; background: #00695C; color: white; text-decoration: none; border-radius: 6px; margin-top: 15px; font-weight: bold; }
        .btn:hover { background: #004D40; }
        .btn-rojo { background: #D32F2F; margin-left: 10px; }
        .btn-rojo:hover { background: #B71C1C; }
        .vacio { color: #888; text-align: center; padding: 15px; }
    </style>
</head>
<body>
    <div class="caja">
        <!-- ✅ Jinja2: Variable -->
        <h1>🌍 ¡Bienvenido, {{ usuario.nombre }}!</h1>
        <p class="bienvenida">Gracias por cuidar el planeta 💚</p>
        
        <div class="tarjeta">
            <!-- ✅ Jinja2: Filtro longitud -->
            <div class="numero">{{ acciones|length }}</div>
            <div>Acciones registradas</div>
        </div>
        
        <h3 style="margin-bottom:15px;">Mis acciones:</h3>
        
        <!-- ✅ Jinja2: Condicional y Bucle -->
        {% if acciones %}
            {% for accion in acciones %}
            <div class="accion">
                <strong>{{ accion.tipo }}</strong><br>
                {{ accion.descripcion }}<br>
                <small>📅 {{ accion.fecha }} | 🌍 {{ accion.impacto or 'Sin dato' }}</small>
            </div>
            {% endfor %}
        {% else %}
            <p class="vacio">Aún no tienes acciones registradas. ¡Agrega una abajo! 🌱</p>
        {% endif %}
        
        <div style="text-align:center;margin-top:25px;">
            <!-- ✅ Jinja2: Enlace dinámico -->
            <a href="{{ url_for('nueva', usuario_id=usuario_id) }}" class="btn">➕ Agregar acción</a>
            <a href="{{ url_for('login') }}" class="btn btn-rojo">Cerrar sesión</a>
        </div>
    </div>
</body>
</html>
'''

TEMPLATE_NUEVA = '''
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Nueva Acción - Eco-Reto</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: #E8F5E9; padding: 30px; }
        .caja { max-width: 450px; margin: 0 auto; background: white; padding: 35px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
        h2 { color: #00695C; text-align: center; margin-bottom: 25px; }
        label { display: block; margin-bottom: 6px; font-weight: 600; }
        input, select, textarea { width: 100%; padding: 11px; margin-bottom: 15px; border: 2px solid #ddd; border-radius: 6px; font-size: 15px; }
        button { background: #00695C; color: white; border: none; padding: 12px 25px; border-radius: 6px; font-size: 16px; font-weight: bold; cursor: pointer; width: 100%; }
        button:hover { background: #004D40; }
        .enlace { margin-top: 20px; text-align: center; }
        .enlace a { color: #00695C; text-decoration: none; font-weight: bold; }
    </style>
</head>
<body>
    <div class="caja">
        <h2>🌿 Nueva Acción Ecológica</h2>
        <form method="POST" id="formAccion">
            <label>Tipo de acción:</label>
            <select name="tipo" required id="tipo">
                <option value="">Selecciona una opción</option>
                <option value="♻️ Reciclaje">♻️ Reciclaje</option>
                <option value="💧 Ahorro de agua">💧 Ahorro de agua</option>
                <option value="🚲 Transporte sostenible">🚲 Transporte sostenible</option>
                <option value="⚡ Ahorro de energía">⚡ Ahorro de energía</option>
            </select>
            
            <label>Descripción:</label>
            <textarea name="descripcion" rows="3" placeholder="Escribe qué hiciste..." required id="descripcion"></textarea>
            
            <label>Fecha (ej: 25/10/2026):</label>
            <input type="text" name="fecha" placeholder="25/10/2026" required id="fecha">
            
            <label>Impacto (ej: 5 kg, 50 litros):</label>
            <input type="text" name="impacto" placeholder="5 kg" id="impacto">
            
            <button type="submit">✅ Guardar</button>
        </form>
        
        <!-- ✅ JavaScript: Mensaje al guardar -->
        <script>
        document.getElementById('formAccion').addEventListener('submit', function(e) {
            const tipo = document.getElementById('tipo').value;
            if (!tipo) {
                alert("❌ Por favor selecciona un tipo de acción");
                e.preventDefault();
            }
        });
        </script>
        
        <div class="enlace">
            <a href="{{ url_for('inicio', usuario_id=usuario_id) }}">← Regresar</a>
        </div>
    </div>
</body>
</html>
'''

# ==================================
# RUTAS
# ==================================

@app.route('/', methods=['GET', 'POST'])
def login():
    error = ""
    if request.method == 'POST':
        u = request.form.get('usuario')
        p = request.form.get('contrasena')
        db = get_db()
        cursor = db.cursor()
        cursor.execute('SELECT id, nombre, contrasena FROM usuarios WHERE usuario = ?', (u,))
        usuario = cursor.fetchone()
        if usuario and usuario['contrasena'] == p:
            return redirect(url_for('inicio', usuario_id=usuario['id']))
        error = "Usuario o contraseña incorrectos"
    return render_template_string(TEMPLATE_LOGIN, error=error)

@app.route('/registro', methods=['GET', 'POST'])
def registro():
    error = ""
    exito = ""
    if request.method == 'POST':
        nombre = request.form.get('nombre')
        u = request.form.get('usuario')
        p = request.form.get('contrasena')
        c = request.form.get('confirmar')
        if p != c:
            error = "❌ Las contraseñas no coinciden"
        else:
            db = get_db()
            cursor = db.cursor()
            try:
                cursor.execute('INSERT INTO usuarios (nombre, usuario, contrasena) VALUES (?, ?, ?)', 
                             (nombre, u, p))
                db.commit()
                exito = "✅ ¡Registro exitoso! Ve a <a href='/'>iniciar sesión</a>"
            except sqlite3.IntegrityError:
                error = "❌ Ese usuario ya existe"
    return render_template_string(TEMPLATE_REGISTRO, error=error, exito=exito)

@app.route('/inicio/<int:usuario_id>')
def inicio(usuario_id):
    db = get_db()
    cursor = db.cursor()
    cursor.execute('SELECT nombre FROM usuarios WHERE id = ?', (usuario_id,))
    usuario = cursor.fetchone()
    cursor.execute('SELECT tipo, descripcion, fecha, impacto FROM acciones WHERE usuario_id = ? ORDER BY id DESC', (usuario_id,))
    acciones = cursor.fetchall()
    return render_template_string(TEMPLATE_INICIO, usuario=usuario, acciones=acciones, usuario_id=usuario_id)

@app.route('/nueva/<int:usuario_id>', methods=['GET', 'POST'])
def nueva(usuario_id):
    if request.method == 'POST':
        db = get_db()
        cursor = db.cursor()
        tipo = request.form.get('tipo')
        descripcion = request.form.get('descripcion')
        fecha = request.form.get('fecha')
        impacto = request.form.get('impacto')
        cursor.execute('INSERT INTO acciones (usuario_id, tipo, descripcion, fecha, impacto) VALUES (?, ?, ?, ?, ?)',
                     (usuario_id, tipo, descripcion, fecha, impacto))
        db.commit()
        return redirect(url_for('inicio', usuario_id=usuario_id))
    return render_template_string(TEMPLATE_NUEVA, usuario_id=usuario_id)

with app.app_context():
    init_db()

if __name__ == '__main__':
    app.run(debug=True)
