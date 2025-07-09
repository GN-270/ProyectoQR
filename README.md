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
  
* **Un IDE (Entorno de Desarrollo Integrado):**
    * IntelliJ IDEA (recomendado)
    * Eclipse
    * NetBeans

### 2. Configuración de la Base de Datos

1.  **Crear la Base de Datos:**
    Abre tu cliente MySQL (MySQL Workbench, línea de comandos `mysql`, etc.) y ejecuta el siguiente comando para crear la base de datos:
    ```sql
    CREATE DATABASE IF NOT EXISTS inventario_fys;
    USE inventario_fyd;
    ```
2.  **Crear Tablas:**
    Ejecuta el script SQL que contiene la definición de tus tablas. Deberías tener un archivo `.sql` con las sentencias `CREATE TABLE`. Si lo tienes, puedes importarlo directamente o copiar y pegar las sentencias.

   
   ```sql
CREATE DATABASE IF NOT EXISTS `inventario_fys` 
USE `inventario_fys`;
CREATE TABLE `Empleados` (
  `id_empleado` int(8) unsigned zerofill NOT NULL AUTO_INCREMENT COMMENT 'ID único del empleado (8 números, rellenado con ceros a la izquierda)',
  `dni` varchar(10) NOT NULL UNIQUE COMMENT 'Número de DNI del empleado (Único)',
  `nombres` varchar(100) NOT NULL COMMENT 'Nombres del empleado',
  `apellidos` varchar(100) NOT NULL COMMENT 'Apellidos del empleado',
  `labor` enum('Electricista','Mecánico','Ayudante','Operario','Personal de mantenimiento','Personal de Limpieza') NOT NULL COMMENT 'Rol del empleado',
  `qr_code_path` varchar(255) DEFAULT NULL COMMENT 'Ruta donde se guarda la imagen del código QR del empleado',
  PRIMARY KEY (`id_empleado`),
  UNIQUE KEY `dni` (`dni`),
  KEY `idx_empleados_dni` (`dni`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='Información de los empleados de la empresa FYS';

CREATE TABLE `Herramientas` (
  `id_herramienta` int NOT NULL AUTO_INCREMENT COMMENT 'ID único de la herramienta (autoincremental)',
  `nombre` varchar(100) NOT NULL COMMENT 'Nombre de la herramienta (ej. Llave Inglesa)',
  `marca` varchar(50) DEFAULT NULL COMMENT 'Marca de la herramienta',
  `descripcion` text COMMENT 'Descripción detallada de la herramienta',
  `estado` enum('pesimo','regular','excelente') NOT NULL COMMENT 'Estado físico actual de la herramienta',
  `disponibilidad` enum('En uso','Libre') NOT NULL DEFAULT 'Libre' COMMENT 'Indica si la herramienta está disponible o en uso',
  `asignado_a` int(8) unsigned zerofill DEFAULT NULL COMMENT 'ID del empleado al que está asignada la herramienta si está En uso (FK a Empleados)',
  `qr_code_path` varchar(255) DEFAULT NULL COMMENT 'Ruta donde se guarda la imagen del código QR de la herramienta',
  PRIMARY KEY (`id_herramienta`),
  KEY `asignado_a` (`asignado_a`),
  KEY `idx_herramientas_nombre` (`nombre`),
  CONSTRAINT `Herramientas_ibfk_1` FOREIGN KEY (`asignado_a`) REFERENCES `Empleados` (`id_empleado`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='Información del inventario de herramientas';

CREATE TABLE `HistorialMovimientos` (
  `id_movimiento` int NOT NULL AUTO_INCREMENT COMMENT 'ID único del registro de movimiento',
  `tipo_accion` varchar(50) NOT NULL COMMENT 'Tipo de acción registrada (ej. Prestación, Devolución)',
  `fecha_hora` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Fecha y hora exactas del movimiento',
  `id_empleado` int(8) unsigned zerofill DEFAULT NULL COMMENT 'ID del empleado que realizó la acción (FK a Empleados)',
  `id_herramienta` int DEFAULT NULL COMMENT 'ID de la herramienta involucrada en el movimiento (FK a Herramientas)',
  `comentarios` text COMMENT 'Comentarios adicionales del administrador o usuario sobre el movimiento',
  PRIMARY KEY (`id_movimiento`),
  KEY `id_herramienta` (`id_herramienta`),
  KEY `idx_historial_fecha` (`fecha_hora`),
  KEY `idx_historial_empleado_herramienta` (`id_empleado`,`id_herramienta`),
  CONSTRAINT `HistorialMovimientos_ibfk_1` FOREIGN KEY (`id_empleado`) REFERENCES `Empleados` (`id_empleado`) ON DELETE SET NULL ON UPDATE CASCADE,
  CONSTRAINT `HistorialMovimientos_ibfk_2` FOREIGN KEY (`id_herramienta`) REFERENCES `Herramientas` (`id_herramienta`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='Registro de todas las prestaciones y devoluciones de herramientas';
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
