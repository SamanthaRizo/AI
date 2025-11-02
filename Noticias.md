# API de Noticias Financieras (Flask + MySQL + Docker)

Este proyecto implementa las operaciones CRUD (GET, POST, PUT, DELETE) para `/noticias`, tal como se definen en el archivo `openapi.yaml`.

La API permite gestionar noticias financieras relacionadas con empresas del S&P500, integrando un análisis técnico y sentimental para futuras etapas del sistema.

---

## Programa en Python (Flask)

Guarda el siguiente código como `app.py`:

```python
from flask import Flask, jsonify, request
from flask_cors import CORS
import mysql.connector
from mysql.connector import Error

app = Flask(__name__)
CORS(app)

# ------------------------------------------------
# Función para obtener conexión segura a MySQL
# ------------------------------------------------
def get_connection():
    try:
        conn = mysql.connector.connect(
            host="127.0.0.1",
            port=3306,
            user="root",
            password="contrasena",
            database="testdb"
        )
        print("Conexión a MySQL exitosa")
        return conn
    except Error as e:
        print("Error al conectar con la base de datos:", e)
        return None

# --------------------------
# CRUD - Noticias
# --------------------------

@app.route('/noticias', methods=['GET'])
def get_noticias():
    conn = get_connection()
    if conn is None:
        return jsonify({"error": "No se pudo conectar a la base de datos"}), 500

    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT * FROM Noticias")
    result = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify(result)

@app.route('/noticias/<int:id>', methods=['GET'])
def get_noticia(id):
    conn = get_connection()
    if conn is None:
        return jsonify({"error": "No se pudo conectar a la base de datos"}), 500

    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT * FROM Noticias WHERE id = %s", (id,))
    noticia = cursor.fetchone()
    cursor.close()
    conn.close()
    if noticia:
        return jsonify(noticia)
    return jsonify({"error": "Noticia no encontrada"}), 404

@app.route('/noticias', methods=['POST'])
def create_noticia():
    data = request.get_json()
    conn = get_connection()
    if conn is None:
        return jsonify({"error": "No se pudo conectar a la base de datos"}), 500

    cursor = conn.cursor()
    sql = """INSERT INTO Noticias (titulo, resumen, fecha_publicacion, empresa, fuente)
             VALUES (%s, %s, %s, %s, %s)"""
    values = (data['titulo'], data['resumen'], data['fecha_publicacion'], data['empresa'], data['fuente'])
    cursor.execute(sql, values)
    conn.commit()
    new_id = cursor.lastrowid
    cursor.close()
    conn.close()
    return jsonify({"message": "Noticia creada", "id": new_id}), 201

@app.route('/noticias/<int:id>', methods=['PUT'])
def update_noticia(id):
    data = request.get_json()
    conn = get_connection()
    if conn is None:
        return jsonify({"error": "No se pudo conectar a la base de datos"}), 500

    cursor = conn.cursor()
    sql = """UPDATE Noticias
             SET titulo=%s, resumen=%s, fecha_publicacion=%s, empresa=%s, fuente=%s
             WHERE id=%s"""
    values = (data['titulo'], data['resumen'], data['fecha_publicacion'], data['empresa'], data['fuente'], id)
    cursor.execute(sql, values)
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"message": "Noticia actualizada"}), 200

@app.route('/noticias/<int:id>', methods=['DELETE'])
def delete_noticia(id):
    conn = get_connection()
    if conn is None:
        return jsonify({"error": "No se pudo conectar a la base de datos"}), 500

    cursor = conn.cursor()
    cursor.execute("DELETE FROM Noticias WHERE id = %s", (id,))
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"message": "Noticia eliminada"}), 200

if __name__ == '__main__':
    print("Iniciando API de Noticias Financieras en http://localhost:8000")
    app.run(host='0.0.0.0', port=8000, debug=True)
