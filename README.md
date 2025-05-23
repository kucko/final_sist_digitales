# Historia de Usuario Mejorada  
**Título:** Registro, almacenamiento y alerta de variable de temperatura (Clean Architecture)

---

## 1. Historia de Usuario  
**Como**  
técnico de mantenimiento de un sistema de monitoreo,  

**Quiero**  
que el dispositivo Arduino R4 WiFi con sensor MAX6675 tome lecturas de temperatura a intervalos configurables,  

**Para**  
1. Almacenar las mediciones en un servidor local (XAMPP) siguiendo la separación de capas de **Clean Architecture**.  
2. Consultar las lecturas de las últimas 3 horas desde una interfaz web.  
3. Recibir una alerta visual vía MQTT (LED en ESP8266) cuando la temperatura exceda los rangos definidos.  

---

## 2. Criterios de Aceptación  

### 2.1 Dominio / Entidades  
- **Medición**  
  - `temperatura: Float`  
  - `timestamp: DateTime`  
  - `alarma: Boolean`  

- **RangoTemperatura**  
  - `minimo: Float`  
  - `maximo: Float`  

### 2.2 Casos de Uso (Application / Use Cases)  
1. **RegistrarMedicion**  
   - Input: valor de temperatura, timestamp  
   - Lógica: compara con rangos y determina `alarma`  
   - Output: persiste objeto `Medición`

2. **ConsultarMediciones**  
   - Input: periodo (`desde`, `hasta`)  
   - Output: lista de `Medición` en ese rango

3. **ConfigurarIntervaloLectura**  
   - Input: nuevo intervalo (segundos/minutos)  
   - Output: confirmación de actualización

### 2.3 Interfaces / Controladores  
- **HTTP API** (Infraestructura / API / Routes)  
  - `POST /api/mediciones` → `RegistrarMedicion`  
  - `GET  /api/mediciones?desde={t1}&hasta={t2}` → `ConsultarMediciones`  
  - `GET  /api/intervalo`  
  - `PUT  /api/intervalo` → `ConfigurarIntervaloLectura`

- **Web UI** (Interfaces / Presenters)  
  - Formulario para CRUD de rangos (mínimo/máximo)  
  - Tabla filtrable de mediciones (últimas 3 horas)  

### 2.4 Infraestructura / Persistencia  
- Repositorio MySQL (`infrastructure/database/mysql/repositories`) implementa interfaces de  
  `application/interfaces/repositories`.  
- Scripts de migración:  
  - Tabla `mediciones`  
  - Tabla `rangos`

### 2.5 Alertas MQTT  
- Si `alarma = true` en `RegistrarMedicion`, el caso de uso publica en tópico  
  `alerta/temperatura`.  
- **ESP8266** (Infraestructura / Messaging) suscrito a `alerta/temperatura`, enciende LED.

---

## 3. Requisitos Funcionales  

| ID   | Descripción                                                                                         |
|------|-----------------------------------------------------------------------------------------------------|
| RF1  | Configurar intervalo de lectura en Arduino vía HTTP GET `/api/intervalo`.                            |
| RF2  | Enviar medición al servidor vía HTTP POST `/api/mediciones` con payload `{ temperatura, timestamp }`. |
| RF3  | CRUD de rangos de temperatura en `/api/rangos`.                                                     |
| RF4  | Consultar mediciones con GET `/api/mediciones?desde={t1}&hasta={t2}`.                               |
| RF5  | Publicar alerta MQTT cuando una medición excede rango.                                              |

---

## 4. Requisitos No Funcionales  

- **RNF1:** Respuesta de API < 200 ms para hasta 10 000 registros.  
- **RNF2:** Disponibilidad del servidor ≥ 99 %.  
- **RNF3:** Código versionado en Git, con ramas por etapa.  
- **RNF4:** Documentación de API y diagramas de Clean Architecture.  

---

## 5. Plan de Desarrollo (Etapas)  

1. **Arquitectura y estructura de directorios**  
   - Generar carpeta raíz `sensor_system/` con subcarpetas por capa:  
     ```
     sensor_system/
       domain/entities
       domain/value_objects
       domain/exceptions
       application/interfaces/repositories
       application/use_cases
       infrastructure/database/mysql/repositories
       infrastructure/api/routes
       infrastructure/messaging
       interfaces/controllers
       interfaces/presenters
     ```  
   - Crear archivos `__init__.py` y README de proyecto.

2. **Servidor local (XAMPP) & Backend HTTP/CRUD**  
   - Definir modelos de datos y scripts de migración MySQL.  
   - Implementar repositorios e interfaces.  
   - Crear casos de uso y controladores REST.  
   - Probar con Postman o similar.

3. **Interfaz Web**  
   - Páginas HTML/JS (Cursor Generated) para:  
     - CRUD de rangos.  
     - Consulta de mediciones (tabla filtrable de últimas 3 horas).  
   - Consumo de API con `fetch`.

4. **Arduino R4 WiFi (Sensor)**  
   - Sketch con lectura periódica de MAX6675.  
   - Cliente HTTP para GET (intervalo) y POST (mediciones).  
   - Manejo de reintentos y registro de errores.

5. **ESP8266 & MQTT**  
   - Configurar broker MQTT en servidor.  
   - Servicio en Python (o Node.js) publica alertas.  
   - Firmware ESP8266 se suscribe y enciende LED de emergencia.  

---

Con esta estructura en **Markdown**, tienes la historia de usuario, criterios de aceptación, requisitos y plan de desarrollo totalmente alineados con **Clean Architecture**. Avísame cuando quieras comenzar la **Etapa 1** o si necesitas más aclaraciones.
