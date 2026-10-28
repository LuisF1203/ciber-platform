# iteso-bdnr-final-project-template

Plantilla (template) para el proyecto final de la materia de **Bases de Datos No Relacionales (BDNR)** en ITESO.

---

## Integrantes del Equipo

| Nombre | Expediente |
| :--- | :--- |
| *Ivanna Camerota Curiel* | *749096* |
| *Luis Armando Castro Arvizu* | *765826* |
| *Luis Fernando Montes Blanco* | *738801* |

---

## Descripción del Proyecto

**Plataforma Inteligente de Detección e Investigación de Amenazas de Ciberseguridad.** \
El sistema monitorea la red de una empresa ficticia, detecta comportamientos sospechosos (por ejemplo, un equipo que se conecta a 25 servidores distintos en 3 minutos) y permite investigar el incidente: de dónde viene, qué dispositivos, usuarios y servidores están involucrados, y qué información o documentación previa se relaciona con él. Funciona como una pequeña plataforma de *Threat Intelligence*, monitoreo de red e investigación de incidentes, pensada para que un analista de seguridad reconstruya un ataque de principio a fin.

Cada base de datos tiene un papel distinto:  
- **Cassandra** guarda el historial masivo de eventos en el tiempo (logs de firewall, inicios de sesión, conexiones, tráfico, alertas) y responde *"¿qué pasó y cuándo?"*.  
- **MongoDB** almacena la información de entidades y configuración (usuarios, dispositivos, servidores, aplicaciones, IPs, vulnerabilidades, malware e incidentes registrados) y responde *"¿qué sabemos de este dispositivo o usuario?"*.  
- **Dgraph** modela las relaciones dentro de la red (quién usa qué equipo, qué IP se comunica con cuál, qué servidor tiene acceso a otro) para analizar el movimiento lateral y responder *"¿qué pudo alcanzar un atacante a partir de este equipo?"*.  
- Como extra, **ChromaDB** guarda embeddings de reportes de incidentes, descripciones de malware y documentación de vulnerabilidades para encontrar casos semánticamente similares al incidente actual.

---

## Flujo de Trabajo (Workflow - Draft Inicial)

> Este flujo es un borrador inicial y se irá ajustando conforme avance el proyecto. Se asume que las instancias de Cassandra, MongoDB, Dgraph y ChromaDB ya están corriendo localmente (por ejemplo, con Docker) y que las dependencias están instaladas (ver [Requisitos y Configuración](#requisitos-y-configuración)).

1. **Inserción / Carga de Datos:**
   - *Comando / Script:* `python data/seeders/seed.py`
   - *Detalles:* `generator.py` genera datos sintéticos de una red corporativa ficticia, incluyendo un escenario de ataque completo (un equipo comprometido que hace movimiento lateral). `seed.py` orquesta la carga llamando a cada seeder:
     - `mongo_seeder.py`: inserta usuarios, dispositivos, servidores, aplicaciones, vulnerabilidades, malware e incidentes.
     - `cassandra_seeder.py`: inserta los eventos y logs (logins, conexiones, tráfico, alertas) con su timestamp.
     - `dgraph_seeder.py`: crea los nodos (usuarios, dispositivos, servidores, IPs) y sus relaciones (`utiliza`, `se_conecta`, `tiene_acceso`, `comunica_con`, `comparte_credencial`).
     - `chroma_seeder.py`: genera embeddings de reportes de incidentes, descripciones de malware y documentación de vulnerabilidades.

2. **Ejecución del Servidor / Backend:**
   - *Comando:* `python server/app.py`
   - *Detalles:* `app.py` levanta la API y abre las conexiones a las cuatro bases de datos (módulos en `server/db/`). Cada base tiene su propio servicio (`model.py` con las consultas en CQL, MQL, DQL o Chroma y `resources.py` con los endpoints). El `extra_service` combina las cuatro bases para armar el reporte completo de un incidente.

3. **Ejecución de Consultas y Cliente:**
   - *Comando / Interfaz:* `python client/menu.py` (menú por consola / CLI)
   - *Detalles:* El cliente muestra un menú con las consultas planeadas y llama a los endpoints del servidor. El flujo de investigación típico es:
     1. **Cassandra:** detectar actividad anómala y ver exactamente qué conexiones ocurrieron y cuándo.
     2. **MongoDB:** consultar quién usa el dispositivo afectado, su sistema operativo, software instalado y vulnerabilidades conocidas.
     3. **Dgraph:** reconstruir las conexiones del dispositivo y ver qué servidores, cuentas y equipos pudo alcanzar.
     4. **ChromaDB:** buscar incidentes, vulnerabilidades, malware o reportes similares.
     5. **Reporte de incidente:** opción que combina todo lo anterior en un resumen (dispositivo, usuario, actividad, servidores involucrados, posibles rutas de ataque e información relacionada) con su nivel de alerta.

---

## Estructura del Repositorio

```text
iteso-bdnr-final-project-template/
├── data/
│   ├── data1.csv
│   ├── data2.csv
│   ├── dataX.csv
│   └── seeders/                        # Se extraen datos de un csv o se generan datos sintéticos.
│       ├── generator.py             
│       ├── cassandra_seeder.py
│       ├── mongo_seeder.py
│       ├── dgraph_seeder.py
│       ├── chroma_seeder.py          
│       └── seed.py                   
│
├── client/
│   └── menu.py                         # UN SOLO CLIENTE (cli, página, app de escritorio, etc.)
│
├── server/
│   ├── app.py                          # Punto de entrada
│   └── db/
│       ├── cassandra_db.py             # Conexión a Cassandra
│       ├── mongo_db.py                 # Conexión a MongoDB
│       ├── dgraph_db.py                # Conexión a Dgraph
│       ├── chroma_db.py                # EXTRA: Conexión a ChromaDB
│       │
│       └── services/                   # Un servicio por cada base de datos.
│           ├── cassandra_service/
│           │   ├── model.py            # Consultas en CQL
│           │   └── resources.py        # Endpoints del servicio de Cassandra
│           ├── mongo_service/
│           │   ├── model.py            # Consultas en MQL
│           │   └── resources.py        # Endpoints del servicio de MongoDB
│           ├── dgraph_service/
│           │   ├── model.py            # Consultas en DQL
│           │   └── resources.py        # Endpoints del servicio de Dgraph
│           ├── chroma_service/         # EXTRA
│           │   ├── model.py         
│           │   └── resources.py      
│           └── extra_service/          # EXTRA: Servicio que utilice bases de datos combinadas.
│               ├── model.py          
│               └── resources.py
│
├── README.md                           # Descripción del proyecto.
└── requirements.txt                    # cassandra-driver, pymongo, pydgraph, ...
```

## Requisitos y Configuración

1. **Crear entorno virtual**:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

2. **Instalar dependencias**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Cargar datos (seeders)**:
   ```bash
   python data/seeders/seed.py
   ```

4. **Iniciar servidor**:
   ```bash
   python server/app.py
   ```

5. **Iniciar cliente**:
   ```bash
   python client/menu.py
   ```
