# 🪙 AlkeWallet — Proyecto de Base de Datos Relacional

> **Módulo 5 · Evaluación Integradora ABP**  
> Fundamentos de Bases de Datos Relacionales con MySQL

---

## 📋 Descripción

AlkeWallet es un sistema de monedero virtual diseñado como proyecto educativo para aplicar fundamentos de bases de datos relacionales. Implementa el ciclo completo: diseño de esquema, inserción de datos, consultas, modificaciones y control transaccional con propiedades ACID.

---

## 🗂️ Estructura del Repositorio

```
AlkeWallet/
├── AlkeWallet_DB.sql        # Script completo ejecutable
├── AlkeWalleT_SQL_2.docx    # Documento de entregable con explicaciones
└── README.md
```

---

## 🏗️ Modelo de Datos

### Diagrama de Entidades

```
┌──────────────┐        ┌──────────────────┐        ┌──────────────┐
│    moneda    │        │     usuario      │        │  transaccion │
│──────────────│        │──────────────────│        │──────────────│
│ currency_id  │◄───────│ currency_id (FK) │        │transaction_id│
│ currency_name│  N:1   │ user_id (PK)     │◄───────│sender_user_id│
│currency_symbol        │ nombre           │  1:N   │receiver_id   │
└──────────────┘        │ correo (UNIQUE)  │        │ importe      │
                        │ contrasena       │        │transaction_dt│
                        │ saldo            │        └──────────────┘
                        │ fecha_creacion   │
                        └──────────────────┘
```

### Tablas

| Tabla | Descripción | Campos clave |
|---|---|---|
| `moneda` | Monedas disponibles en el sistema | `currency_id`, `currency_name`, `currency_symbol` |
| `usuario` | Usuarios registrados con saldo | `user_id`, `correo` (UNIQUE), `saldo`, `currency_id` (FK) |
| `transaccion` | Historial de transferencias entre usuarios | `transaction_id`, `sender_user_id` (FK), `receiver_user_id` (FK), `importe` |

---

## ⚙️ Configuración y Ejecución

### Requisitos

- MySQL 8.0+ (o MariaDB 10.5+)
- Cliente MySQL: MySQL Workbench, DBeaver, o terminal

### Pasos para ejecutar

```sql
-- 1. Clonar o descargar el repositorio
-- 2. Abrir el archivo AlkeWallet_DB.sql en tu cliente MySQL
-- 3. Ejecutar el script completo, o por secciones según necesidad

-- También puedes ejecutarlo desde terminal:
mysql -u tu_usuario -p < AlkeWallet_DB.sql
```

El script es **idempotente**: usa `CREATE DATABASE IF NOT EXISTS` y `CREATE TABLE IF NOT EXISTS`, por lo que puede ejecutarse varias veces sin errores.

---

## 📦 Contenido del Script SQL

### 1. DDL — Creación de estructura

```sql
CREATE DATABASE IF NOT EXISTS AlkeWallet
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
```

- Base de datos con soporte `utf8mb4` (incluye emojis y caracteres especiales)
- Tres tablas con `PRIMARY KEY`, `FOREIGN KEY`, `INDEX` y restricciones `NOT NULL`
- `moneda` se crea antes que `usuario` para satisfacer la dependencia de FK

### 2. DML — Datos de prueba

- **5 monedas**: CLP, USD, EUR, ARS, PEN
- **5 usuarios** con saldos distintos y contraseñas hasheadas con `SHA2(..., 256)`
- **6 transacciones** entre distintos usuarios

### 3. SELECT — Consultas

| Consulta | Descripción |
|---|---|
| 4.1 | Moneda de un usuario específico (`INNER JOIN`) |
| 4.2 | Todas las transacciones con nombres de usuario (`INNER JOIN` doble) |
| 4.3 | Transacciones enviadas y recibidas por un usuario (`CASE WHEN`) |
| 4.4 ⭐ | Vista `vw_top5_saldo` con los 5 usuarios de mayor saldo |

### 4. UPDATE y DELETE

```sql
-- Actualizar correo
UPDATE usuario SET correo = 'nuevo@mail.com' WHERE user_id = 1;

-- Eliminar transacción
DELETE FROM transaccion WHERE transaction_id = 6;

-- Simular transferencia (sin transacción controlada)
UPDATE usuario SET saldo = saldo - 50000.00 WHERE user_id = 1;
UPDATE usuario SET saldo = saldo + 50000.00 WHERE user_id = 2;
```

### 5. Transaccionalidad ACID

```sql
-- Transacción exitosa
START TRANSACTION;
    INSERT INTO transaccion (sender_user_id, receiver_user_id, importe) VALUES (1, 3, 20000.00);
    UPDATE usuario SET saldo = saldo - 20000.00 WHERE user_id = 1;
    UPDATE usuario SET saldo = saldo + 20000.00 WHERE user_id = 3;
COMMIT;

-- Transacción con error → reversión total
START TRANSACTION;
    INSERT INTO transaccion (...) VALUES (1, 999, 5000.00); -- FK violation
ROLLBACK;
```

### 6. Agregaciones

```sql
-- COUNT + SUM + LEFT JOIN por usuario
-- Subconsulta correlacionada para transacciones enviadas
```

---

## 🔐 Propiedades ACID implementadas

| Propiedad | Significado | Implementación |
|---|---|---|
| **Atomicidad** | Todo o nada | `START TRANSACTION` + `ROLLBACK` |
| **Consistencia** | Estado válido → estado válido | `FOREIGN KEY` + `NOT NULL` |
| **Aislamiento** | Transacciones independientes | InnoDB con `REPEATABLE READ` (por defecto) |
| **Durabilidad** | Cambios confirmados persisten | `COMMIT` escribe en disco |

---

## 🔒 Notas de Seguridad

> ⚠️ Las contraseñas se almacenan con `SHA2('password', 256)`. Esto es válido para un contexto educativo, pero **en producción se debe usar bcrypt o Argon2** con salt, ya que SHA-2 sin salt es vulnerable a ataques de diccionario y rainbow tables.

---

## 💡 Conceptos aplicados

- DDL vs DML
- Claves primarias y foráneas
- Índices para optimización de búsquedas
- `INNER JOIN` y `LEFT JOIN`
- `CASE WHEN` para lógica condicional en consultas
- Vistas (`CREATE VIEW`)
- Transacciones con `START TRANSACTION / COMMIT / ROLLBACK`
- Funciones de agregación: `COUNT`, `SUM`
- Subconsultas correlacionadas

---

## 👤 Autor

Proyecto desarrollado como evaluación del Módulo 5 del programa **Alke — Desarrollo Fullstack**.

---

## 📄 Licencia

Proyecto educativo. Libre uso para fines de aprendizaje.
