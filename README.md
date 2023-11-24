# CODEPROYECT
import flask
import json
import sqlite3

app = flask.Flask(__name__)

# Conexión a la base de datos
conn = sqlite3.connect("delitos.db")

# Tabla de delitos
c = conn.cursor()
c.execute("""CREATE TABLE IF NOT EXISTS delitos (
  id INTEGER PRIMARY KEY,
  fecha DATETIME,
  hora TIME,
  tipo TEXT,
  ubicación TEXT,
  descripción TEXT
)""")

# Ruta principal
@app.route("/")
def index():
    return "Bienvenido a la aplicación de reporte de delitos"

# Ruta para reportar un delito
@app.route("/reportar", methods=["POST"])
def reportar():
    # Obtener los datos del delito
    delito = json.loads(flask.request.data)

    # Guardar el delito en la base de datos
    c.execute("""INSERT INTO delitos (fecha, hora, tipo, ubicación, descripción)
                 VALUES (?, ?, ?, ?, ?)""", (
                 delito["fecha"], delito["hora"], delito["tipo"], delito["ubicación"], delito["descripción"]
             ))
    conn.commit()

    # Enviar un correo electrónico a las autoridades
    asunto = "Reporte de delito"
    cuerpo = """
    Se ha reportado un delito de tipo {} en la ubicación {} a la hora {}.

    Descripción: {}
    """.format(
        delito["tipo"], delito["ubicación"], delito["hora"], delito["descripción"]
    )
    enviar_correo(asunto, cuerpo)

    return "El delito ha sido reportado"

# Función para enviar un correo electrónico
def enviar_correo(asunto, cuerpo):
    # Importar la librería smtplib
    import smtplib

    # Configurar el servidor de correo
    server = smtplib.SMTP("smtp.gmail.com", 587)
    server.starttls()
    server.login("tu_correo_electronico", "tu_contraseña")

    # Enviar el correo electrónico
    destinatarios = ["correo_de_las_autoridades"]
    server.sendmail("tu_correo_electronico", destinatarios, asunto + "\n\n" + cuerpo)

    server.quit()

if __name__ == "__main__":
    app.run(debug=True)
****
