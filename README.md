# Sistema de Gestión de Inventario de Herramientas

Este proyecto es un sistema de gestión de inventario para herramientas, permitiendo el control de préstamos, devoluciones, y la administración de empleados y herramientas. Incluye la generación y escaneo (futuro) de códigos QR para agilizar los procesos.

## ⚙️ Tecnologías Utilizadas

* **Lenguaje:** Java 11+
* **Interfaz Gráfica:** Swing (Java Swing)
* **Base de Datos:** MySQL
* **Conector DB:** MySQL Connector/J
* **Generación/Lectura QR:** ZXing Library

## ✨ Características Principales

* **Gestión de Empleados:** Altas, bajas, modificaciones de datos de empleados.
* **Gestión de Herramientas:** Altas, bajas, modificaciones de herramientas y su estado (disponible/prestada).
* **Historial de Movimientos:** Registro detallado de cada préstamo y devolución de herramientas.
* **Generación de Códigos QR:** Creación automática de códigos QR para herramientas y empleados (vinculados a su ID).
* **Sistema de Login:** Diferenciación de acceso para administradores y usuarios (implementación simple sin seguridad avanzada por el momento).
* **Funcionalidad de Usuario:** Préstamo y devolución de herramientas a través de escaneo de QR.

## 🚀 Guía de Ejecución

Sigue estos pasos para configurar y ejecutar el proyecto en tu entorno local.

### 1. Requisitos Previos

Asegúrate de tener instalado lo siguiente:

* **Java Development Kit (JDK) 11 o superior:**
    * Puedes descargarlo desde el sitio oficial de Oracle o usar una distribución OpenJDK (como Adoptium Temurin).
* 
* **Un IDE (Entorno de Desarrollo Integrado):**
    * IntelliJ IDEA (recomendado)
    * Eclipse
    * NetBeans

### 2. Configuración de la Base de Datos

1.  **Crear la Base de Datos:**
    Abre tu cliente MySQL (MySQL Workbench, línea de comandos `mysql`, etc.) y ejecuta el siguiente comando para crear la base de datos:
    ```sql
    CREATE DATABASE IF NOT EXISTS inventario_db;
    USE inventario_db;
    ```
2.  **Crear Tablas:**
    Ejecuta el script SQL que contiene la definición de tus tablas. Deberías tener un archivo `.sql` con las sentencias `CREATE TABLE`. Si lo tienes, puedes importarlo directamente o copiar y pegar las sentencias.

    **Asegúrate de que la tabla `Empleados` tenga la columna `id_empleado` como `AUTO_INCREMENT`:**
    ```sql
    -- Ejemplo (adapta esto a la estructura real de tus tablas)
    CREATE TABLE Empleados (
        id_empleado INT AUTO_INCREMENT PRIMARY KEY,
        dni VARCHAR(20) UNIQUE NOT NULL,
        nombres VARCHAR(100) NOT NULL,
        apellidos VARCHAR(100) NOT NULL,
        labor VARCHAR(100),
        ruta_qr VARCHAR(255)
    );

    CREATE TABLE Herramientas (
        id_herramienta INT AUTO_INCREMENT PRIMARY KEY,
        nombre VARCHAR(100) NOT NULL,
        marca VARCHAR(50),
        modelo VARCHAR(50),
        numero_serie VARCHAR(100) UNIQUE,
        estado ENUM('Disponible', 'Prestada', 'Mantenimiento') NOT NULL DEFAULT 'Disponible',
        asignado_a INT, -- FK a Empleados
        ruta_qr VARCHAR(255),
        FOREIGN KEY (asignado_a) REFERENCES Empleados(id_empleado) ON DELETE SET NULL
    );

    CREATE TABLE HistorialMovimientos (
        id_movimiento INT AUTO_INCREMENT PRIMARY KEY,
        id_herramienta INT,
        id_empleado INT,
        tipo_accion ENUM('Prestacion', 'Devolucion') NOT NULL, -- Renombrado de 'tipo_movimiento'
        fecha_hora DATETIME NOT NULL,
        comentarios TEXT, -- Renombrado de 'observaciones'
        FOREIGN KEY (id_herramienta) REFERENCES Herramientas(id_herramienta) ON DELETE SET NULL,
        FOREIGN KEY (id_empleado) REFERENCES Empleados(id_empleado) ON DELETE SET NULL
    );

    -- Tabla de Usuarios (para el login simplificado)
    CREATE TABLE Usuarios (
        id_usuario INT AUTO_INCREMENT PRIMARY KEY,
        username VARCHAR(50) NOT NULL UNIQUE,
        password_hash VARCHAR(255) NOT NULL, -- En este proyecto, se usará "admin" / "admin123" y "usuario" / "usuario123" hardcodeado
        rol ENUM('admin', 'usuario') NOT NULL
    );

    -- Insertar usuarios de prueba (Solo si usas la tabla Usuarios)
    INSERT INTO Usuarios (username, password_hash, rol) VALUES ('admin', 'admin123', 'admin');
    INSERT INTO Usuarios (username, password_hash, rol) VALUES ('usuario', 'usuario123', 'usuario');
    ```
3.  **Configurar Credenciales de la Base de Datos en Java:**
    Asegúrate de que el archivo `com.fys.inventario.util.DatabaseConnection` (o su equivalente) contenga las credenciales correctas para tu base de datos MySQL (host, puerto, nombre de usuario, contraseña).

    ```java
    // Ejemplo de DatabaseConnection.java (asegúrate de que esto sea correcto)
    public class DatabaseConnection {
        private static final String URL = "jdbc:mysql://localhost:3306/inventario_db";
        private static final String USER = "your_mysql_username"; // <-- Cambia esto
        private static final String PASSWORD = "your_mysql_password"; // <-- Cambia esto
        // ...
    }
    ```

### 3. Configuración del Proyecto en el IDE

1.  **Abrir el Proyecto:**
    Abre el directorio raíz del proyecto (`inventario` o como lo hayas nombrado) en tu IDE.
2.  **Compilar el Proyecto:**
    Realiza una **limpieza y reconstrucción completa** del proyecto en tu IDE para asegurar que todos los cambios y dependencias se apliquen correctamente.
    * **IntelliJ IDEA:** `Build` > `Rebuild Project`
    * **Eclipse:** `Project` > `Clean...` seguido de `Project` > `Build Project`

### 4. Ejecutar la Aplicación

1.  **Buscar el Método `main`:**
    La aplicación se inicia desde el método `main` en la clase `com.fys.inventario.main.MainApp`.
2.  **Ejecutar:**
    Haz clic derecho en `MainApp.java` en tu IDE y selecciona `Run 'MainApp.main()'`.

    La aplicación debería iniciar mostrando la ventana de inicio de sesión.

## 🔒 Credenciales de Inicio de Sesión

* **Administrador:**
    * **Usuario:** `admin`
    * **Contraseña:** `admin123`
* **Usuario Estándar:**
    * **Escaneo QR**
