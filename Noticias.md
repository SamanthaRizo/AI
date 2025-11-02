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
```


## Configuración del entorno Docker + MySQL

Crea un archivo docker-compose.yml en el mismo directorio con el siguiente contenido:

```version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-news-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: contrasena
      MYSQL_DATABASE: testdb
    ports:
      - "3306:3306"
    volumes:
      - ./mysql_data:/var/lib/mysql
```

### Instalar dependencias

```
pip3 install flask requests
```

### Correr el programa

```
python app.py
```

Esto iniciará un contenedor con MySQL y creará la base testdb.

Luego, ingresa al contenedor y crea la tabla:

```
CREATE TABLE Noticias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(255),
    resumen TEXT,
    fecha_publicacion DATE,
    empresa VARCHAR(100),
    fuente VARCHAR(100)
);
```

## Endpoints Disponibles
### GET /noticias

Obtiene todas las noticias registradas en la base de datos.

Ejemplo de uso:

```
curl -X GET http://localhost:8000/noticias
```


Respuesta esperada:

```
[
  {
    "id": 1,
    "titulo": "Microsoft presenta resultados récord",
    "resumen": "La compañía supera las expectativas del mercado...",
    "fecha_publicacion": "2025-10-30",
    "empresa": "Microsoft",
    "fuente": "Reuters"
  }
]
```

### GET /noticias/{id}

Consulta una noticia específica por su ID.

```
curl -X GET http://localhost:8000/noticias/1
```


Respuesta esperada:

```
{
  "id": 1,
  "titulo": "Microsoft presenta resultados récord",
  "resumen": "La compañía supera las expectativas del mercado...",
  "fecha_publicacion": "2025-10-30",
  "empresa": "Microsoft",
  "fuente": "Reuters"
}
```

### POST /noticias

Crea una nueva noticia.

```
curl -X POST http://localhost:8000/noticias \
-H "Content-Type: application/json" \
-d '{
  "titulo": "Apple lanza nuevo iPhone",
  "resumen": "El nuevo modelo introduce mejoras de IA y batería.",
  "fecha_publicacion": "2025-11-02",
  "empresa": "Apple",
  "fuente": "Bloomberg"
}'
```


Respuesta esperada:

```
{
  "message": "Noticia creada",
  "id": 5
}
```

### PUT /noticias/{id}

Actualiza completamente una noticia existente.

```
curl -X PUT http://localhost:8000/noticias/5 \
-H "Content-Type: application/json" \
-d '{
  "titulo": "Apple lanza el iPhone 17",
  "resumen": "Ahora con chip A20 Bionic y mejoras en cámara.",
  "fecha_publicacion": "2025-11-02",
  "empresa": "Apple",
  "fuente": "Bloomberg"
}'
```


Respuesta esperada:

```
{
  "message": "Noticia actualizada"
}
```

### DELETE /noticias/{id}

Elimina una noticia del sistema.

```
curl -X DELETE http://localhost:8000/noticias/5
```


Respuesta esperada:

```
{
  "message": "Noticia eliminada"
}
```
