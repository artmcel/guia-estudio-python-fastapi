# 🐍 Python + FastAPI
### De cero a una API profesional lista para tu portfolio

> **Para quién es esta guía:** Desarrolladores backend que quieren aprender Python con FastAPI desde cero, con fundamentos teóricos sólidos y un proyecto real.

---

## 📌 Índice

1. [Introducción y contexto](#1-introducción-y-contexto)
2. [Configuración del entorno](#2-configuración-del-entorno)
3. [Python esencial para FastAPI](#3-python-esencial-para-fastapi)
4. [FastAPI: fundamentos](#4-fastapi-fundamentos)
5. [Base de datos con SQLAlchemy + Alembic](#5-base-de-datos-con-sqlalchemy--alembic)
6. [Autenticación con JWT](#6-autenticación-con-jwt)
7. [Arquitectura limpia y organización del proyecto](#7-arquitectura-limpia-y-organización-del-proyecto)
8. [Testing con Pytest](#8-testing-con-pytest)
9. [Dockerización y despliegue](#9-dockerización-y-despliegue)
10. [El Proyecto: JobTracker API](#10-el-proyecto-jobtracker-api)

---

## 1. Introducción y contexto

### ¿Por qué FastAPI?

FastAPI es un framework web moderno para construir APIs con Python 3.8+. Fue creado por Sebastián Ramírez en 2018 y en pocos años se convirtió en uno de los frameworks más populares del ecosistema Python.

**Razones técnicas para elegirlo:**

- **Rendimiento:** Está construido sobre **Starlette** (para el manejo de requests ASGI) y **Pydantic** (para validación de datos). Sus benchmarks lo ubican al mismo nivel que Node.js y Go en velocidad de respuesta.
- **Tipado estático:** Aprovecha los *type hints* de Python para generar validación automática, serialización y documentación interactiva (Swagger UI + ReDoc).
- **Async nativo:** Soporta `async/await` de forma nativa, lo que lo hace ideal para operaciones I/O intensivas como consultas a bases de datos o llamadas a APIs externas.
- **Documentación automática:** Cada endpoint que defines genera automáticamente documentación interactiva en `/docs` (Swagger) y `/redoc`.

**Comparación rápida:**

| Concepto        | Laravel          | Node.js (Express) | FastAPI              |
|-----------------|------------------|-------------------|----------------------|
| Router          | `Route::get()`   | `app.get()`       | `@app.get()`         |
| Middleware      | Middleware class | `app.use()`       | `Depends()`          |
| Validación      | Form Request      | Joi / Zod         | Pydantic BaseModel   |
| ORM             | Eloquent         | Sequelize/Prisma  | SQLAlchemy           |
| Migraciones     | Artisan migrate  | Sequelize CLI     | Alembic              |
| Contenedor DI   | Service Container | NestJS IoC        | `Depends()` nativo   |

---

## 2. Configuración del entorno

### Teoría: Entornos virtuales en Python

A diferencia de `node_modules` (que vive dentro del proyecto) o Composer (que instala globalmente o por proyecto), Python usa **entornos virtuales** para aislar dependencias. Esto evita conflictos entre proyectos que usen versiones distintas de la misma librería.

Un entorno virtual es simplemente una carpeta que contiene una copia del intérprete de Python y las librerías instaladas para ese proyecto. El estándar moderno es usar `venv` (incluido en Python) o herramientas como `pyenv` + `poetry`.

### Instalación paso a paso

**Requisitos previos:**
- Python 3.11+ → [python.org/downloads](https://python.org/downloads)
- Docker Desktop (para el proyecto)

```bash
# 1. Verificar versión de Python
python --version  # debe ser 3.11+

# 2. Crear carpeta del proyecto
mkdir jobtracker-api && cd jobtracker-api

# 3. Crear entorno virtual
python -m venv venv

# 4. Activar el entorno virtual
# En Linux/Mac:
source venv/bin/activate
# En Windows:
venv\Scripts\activate

# Sabrás que está activo porque el prompt cambia a: (venv) $

# 5. Instalar dependencias base
pip install fastapi uvicorn[standard]

# 6. Verificar instalación
python -c "import fastapi; print(fastapi.__version__)"
```

**¿Qué es Uvicorn?**

FastAPI por sí solo es solo un framework; necesita un servidor ASGI (*Asynchronous Server Gateway Interface*) para ejecutarse. Uvicorn es ese servidor, equivalente a lo que es `php-fpm` para PHP o `node` para Express. La flag `[standard]` instala extras como recarga automática en desarrollo.

### Archivo de dependencias

En Python, el equivalente al `package.json` es el archivo `requirements.txt`:

```bash
pip freeze > requirements.txt
```

Para el desarrollo moderno se recomienda **Poetry** (similar a Composer), pero `requirements.txt` es suficiente para empezar y es lo más universal.

---

## 3. Python esencial para FastAPI

> Esta sección cubre únicamente lo que necesitas para trabajar con FastAPI. No es un curso completo de Python.

### 3.1 Type Hints (Tipado estático)

**Teoría:** Python es dinámicamente tipado, pero desde la versión 3.5 permite *hints* de tipos que no son obligatorios en tiempo de ejecución pero son usados por herramientas como FastAPI, Pydantic y los IDEs para validación y autocompletado.

FastAPI **depende totalmente** de los type hints para funcionar. Sin ellos, no puede generar validación ni documentación automática.

```python
# Sin type hints (Python clásico)
def suma(a, b):
    return a + b

# Con type hints (Python moderno)
def suma(a: int, b: int) -> int:
    return a + b

# Tipos comunes
nombre: str = "Arturo"
edad: int = 28
activo: bool = True
precio: float = 99.99

# Tipos compuestos (importar de typing o usar sintaxis moderna 3.9+)
from typing import Optional, List, Dict

def buscar_usuario(id: int) -> Optional[dict]:
    # Optional[X] = puede ser X o None
    pass

def listar_items() -> List[str]:
    pass
```

### 3.2 Clases y herencia

**Teoría:** Pydantic (la librería de validación de FastAPI) funciona con clases que heredan de `BaseModel`. Entender la herencia en Python es fundamental.

```python
class Persona:
    def __init__(self, nombre: str, edad: int):
        self.nombre = nombre
        self.edad = edad
    
    def saludar(self) -> str:
        return f"Hola, soy {self.nombre}"

class Empleado(Persona):
    def __init__(self, nombre: str, edad: int, empresa: str):
        super().__init__(nombre, edad)  # llama al __init__ del padre
        self.empresa = empresa
    
    def saludar(self) -> str:
        base = super().saludar()
        return f"{base}, trabajo en {self.empresa}"
```

### 3.3 Decoradores

**Teoría:** Un decorador es una función que recibe otra función como argumento y retorna una versión modificada de ella. En FastAPI, los decoradores como `@app.get("/ruta")` son la forma de registrar endpoints.

Conceptualmente es similar a los atributos de C# o las anotaciones de Java. Si vienes de Laravel, es parecido a cuando defines rutas con `Route::get()`, pero en FastAPI la ruta "vive" directamente encima de la función.

```python
# Decorador simple
def log_llamada(func):
    def wrapper(*args, **kwargs):
        print(f"Llamando a {func.__name__}")
        resultado = func(*args, **kwargs)
        print(f"Terminó {func.__name__}")
        return resultado
    return wrapper

@log_llamada
def saludar(nombre: str) -> str:
    return f"Hola {nombre}"

# Equivale a: saludar = log_llamada(saludar)
saludar("Arturo")
# Output:
# Llamando a saludar
# Terminó saludar
```

### 3.4 Async / Await

**Teoría:** Python soporta programación asíncrona nativa con `asyncio`. La diferencia clave con el modelo sincrónico es que mientras una operación I/O (consulta a DB, llamada HTTP) está esperando respuesta, el hilo puede procesar otras requests en lugar de bloquearse.

FastAPI soporta tanto funciones `async def` como funciones normales `def`. Usa `async def` cuando hagas operaciones de I/O (BD, HTTP, archivos); usa `def` para cómputo puro.

```python
import asyncio
import httpx

# Función sincrónica (bloquea el hilo mientras espera)
def obtener_datos_sync():
    # si esto tarda 2 segundos, bloquea todo
    pass

# Función asíncrona (libera el hilo mientras espera)
async def obtener_datos_async():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.github.com/users/octocat")
        return response.json()

# Para llamar una función async desde otra async:
async def main():
    datos = await obtener_datos_async()
    print(datos)

# Para ejecutar desde código sincrónico:
asyncio.run(main())
```

### 3.5 Pydantic BaseModel

**Teoría:** Pydantic es la librería que FastAPI usa para validación de datos. Cuando defines un modelo que hereda de `BaseModel`, Pydantic valida automáticamente que los datos entrantes tengan los tipos correctos, y genera un error 422 si no cumplen.

Es conceptualmente similar a un Form Request de Laravel o a un DTO (Data Transfer Object) en arquitecturas más formales.

```python
from pydantic import BaseModel, EmailStr, validator
from typing import Optional
from datetime import datetime

class UsuarioCreate(BaseModel):
    nombre: str
    email: EmailStr          # valida formato de email automáticamente
    password: str
    edad: Optional[int] = None  # campo opcional, default None

    # Validador personalizado (como una regla custom en Laravel)
    @validator('nombre')
    def nombre_no_vacio(cls, v):
        if len(v.strip()) < 2:
            raise ValueError('El nombre debe tener al menos 2 caracteres')
        return v.strip()

class UsuarioResponse(BaseModel):
    id: int
    nombre: str
    email: str
    created_at: datetime

    class Config:
        # Permite que Pydantic lea desde objetos ORM (SQLAlchemy)
        from_attributes = True

# Uso:
usuario = UsuarioCreate(
    nombre="Arturo",
    email="arturo@example.com",
    password="secret123"
)
print(usuario.model_dump())  # convierte a dict
# {'nombre': 'Arturo', 'email': 'arturo@example.com', ...}
```

---

## 4. FastAPI: Fundamentos

### 4.1 Primera API

```python
# main.py
from fastapi import FastAPI

app = FastAPI(
    title="JobTracker API",
    description="API para rastrear aplicaciones de trabajo",
    version="1.0.0"
)

@app.get("/")
def raiz():
    return {"mensaje": "API funcionando 🚀"}

@app.get("/health")
def health_check():
    return {"status": "ok"}
```

```bash
# Ejecutar con recarga automática (modo desarrollo)
uvicorn main:app --reload

# Ahora visita:
# http://localhost:8000         → {"mensaje": "API funcionando 🚀"}
# http://localhost:8000/docs    → Swagger UI automático
# http://localhost:8000/redoc   → ReDoc automático
```

### 4.2 Path Parameters y Query Parameters

**Teoría:** FastAPI distingue automáticamente entre parámetros de ruta (definidos en la URL con `{}`) y parámetros de query (pasados como `?clave=valor`), basándose únicamente en si el parámetro de la función coincide con algo en la URL.

```python
from fastapi import FastAPI, HTTPException
from typing import Optional

app = FastAPI()

empleos_db = [
    {"id": 1, "empresa": "Google", "puesto": "Backend Dev", "activo": True},
    {"id": 2, "empresa": "Meta",   "puesto": "Full Stack",  "activo": False},
]

# Path parameter: {empleo_id} en la URL → parámetro en la función
@app.get("/empleos/{empleo_id}")
def obtener_empleo(empleo_id: int):
    # FastAPI ya valida que empleo_id sea int; si no, retorna 422
    empleo = next((e for e in empleos_db if e["id"] == empleo_id), None)
    if not empleo:
        raise HTTPException(status_code=404, detail="Empleo no encontrado")
    return empleo

# Query parameters: todo lo que NO está en la URL
# GET /empleos?activo=true&limite=10
@app.get("/empleos")
def listar_empleos(
    activo: Optional[bool] = None,   # ?activo=true
    limite: int = 10,                 # ?limite=5 (default 10)
    offset: int = 0                   # ?offset=20
):
    resultado = empleos_db
    if activo is not None:
        resultado = [e for e in resultado if e["activo"] == activo]
    return resultado[offset : offset + limite]
```

### 4.3 Request Body con Pydantic

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class EmpleoCreate(BaseModel):
    empresa: str
    puesto: str
    salario_min: Optional[float] = None
    salario_max: Optional[float] = None
    url_oferta: Optional[str] = None

class EmpleoResponse(EmpleoCreate):
    id: int
    activo: bool = True

@app.post("/empleos", response_model=EmpleoResponse, status_code=201)
def crear_empleo(empleo: EmpleoCreate):
    # FastAPI deserializa el JSON del body y lo valida con Pydantic
    nuevo = {"id": len(empleos_db) + 1, "activo": True, **empleo.model_dump()}
    empleos_db.append(nuevo)
    return nuevo
```

### 4.4 Dependency Injection con `Depends`

**Teoría:** `Depends` es el sistema de inyección de dependencias de FastAPI. Permite reutilizar lógica común (como obtener el usuario autenticado, abrir una sesión de BD, validar permisos) sin repetirla en cada endpoint.

Es funcionalmente equivalente al Service Container de Laravel, pero más explícito y testeable.

```python
from fastapi import FastAPI, Depends, Header, HTTPException

app = FastAPI()

# Dependencia: se ejecuta antes del endpoint
def verificar_api_key(x_api_key: str = Header(...)):
    if x_api_key != "mi-api-key-secreta":
        raise HTTPException(status_code=401, detail="API Key inválida")
    return x_api_key

# El endpoint "depende" de verificar_api_key
@app.get("/admin/stats", dependencies=[Depends(verificar_api_key)])
def obtener_estadisticas():
    return {"total_empleos": 100, "activos": 42}

# O inyectando el valor retornado por la dependencia
@app.get("/perfil")
def ver_perfil(api_key: str = Depends(verificar_api_key)):
    return {"api_key_usada": api_key}
```

---

## 5. Base de datos con SQLAlchemy + Alembic

### Teoría: ORM y el patrón Data Mapper

SQLAlchemy es el ORM más usado en Python. A diferencia de Eloquent (Active Record), SQLAlchemy implementa el patrón **Data Mapper**, donde los modelos son clases Python puras y la lógica de persistencia está separada en una `Session`.

Esto da más control pero requiere un poco más de código inicial.

**Alembic** es la herramienta de migraciones de SQLAlchemy, equivalente a `php artisan migrate`.

### 5.1 Instalación

```bash
pip install sqlalchemy alembic asyncpg psycopg2-binary python-dotenv
# asyncpg: driver async para PostgreSQL
# psycopg2-binary: driver sync para Alembic
```

### 5.2 Configuración de la conexión

```python
# app/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker, DeclarativeBase
from dotenv import load_dotenv
import os

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")
# Ejemplo: postgresql+asyncpg://user:password@localhost:5432/jobtracker

# Motor de base de datos (equivalente a la conexión en Laravel)
engine = create_async_engine(DATABASE_URL, echo=True)

# Fábrica de sesiones (equivalente al DB facade de Laravel)
AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

# Clase base para todos los modelos
class Base(DeclarativeBase):
    pass

# Dependencia para inyectar sesión en endpoints
async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

### 5.3 Definición de modelos

```python
# app/models/job_application.py
from sqlalchemy import Column, Integer, String, Float, Boolean, DateTime, ForeignKey, Enum
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
import enum
from app.database import Base

class StatusEnum(enum.Enum):
    APPLIED      = "applied"
    PHONE_SCREEN = "phone_screen"
    INTERVIEW    = "interview"
    OFFER        = "offer"
    REJECTED     = "rejected"
    WITHDRAWN    = "withdrawn"

class JobApplication(Base):
    __tablename__ = "job_applications"

    id          = Column(Integer, primary_key=True, index=True)
    user_id     = Column(Integer, ForeignKey("users.id"), nullable=False)
    company     = Column(String(200), nullable=False)
    position    = Column(String(200), nullable=False)
    salary_min  = Column(Float, nullable=True)
    salary_max  = Column(Float, nullable=True)
    job_url     = Column(String(500), nullable=True)
    status      = Column(Enum(StatusEnum), default=StatusEnum.APPLIED)
    is_active   = Column(Boolean, default=True)
    applied_at  = Column(DateTime(timezone=True), server_default=func.now())
    updated_at  = Column(DateTime(timezone=True), onupdate=func.now())

    # Relación (equivalente a hasMany/belongsTo de Eloquent)
    user        = relationship("User", back_populates="applications")
    notes       = relationship("ApplicationNote", back_populates="application")
```

### 5.4 Migraciones con Alembic

```bash
# Inicializar Alembic
alembic init alembic

# Generar migración automáticamente (detecta cambios en modelos)
alembic revision --autogenerate -m "crear tabla job_applications"

# Aplicar migraciones
alembic upgrade head

# Ver historial
alembic history

# Revertir última migración
alembic downgrade -1
```

### 5.5 Operaciones CRUD con SQLAlchemy async

```python
# app/repositories/job_application_repo.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update, delete
from app.models.job_application import JobApplication

class JobApplicationRepository:

    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, data: dict) -> JobApplication:
        application = JobApplication(**data)
        self.db.add(application)
        await self.db.flush()  # obtiene el ID sin hacer commit
        await self.db.refresh(application)
        return application

    async def get_by_id(self, id: int) -> JobApplication | None:
        result = await self.db.execute(
            select(JobApplication).where(JobApplication.id == id)
        )
        return result.scalar_one_or_none()

    async def list_by_user(self, user_id: int, status: str | None = None):
        query = select(JobApplication).where(JobApplication.user_id == user_id)
        if status:
            query = query.where(JobApplication.status == status)
        result = await self.db.execute(query)
        return result.scalars().all()

    async def update(self, id: int, data: dict) -> JobApplication | None:
        await self.db.execute(
            update(JobApplication)
            .where(JobApplication.id == id)
            .values(**data)
        )
        return await self.get_by_id(id)

    async def delete(self, id: int) -> bool:
        result = await self.db.execute(
            delete(JobApplication).where(JobApplication.id == id)
        )
        return result.rowcount > 0
```

---

## 6. Autenticación con JWT

### Teoría: JSON Web Tokens

JWT (JSON Web Token) es un estándar para transmitir información entre partes de forma segura y compacta. Un token JWT tiene tres partes separadas por puntos:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
[   HEADER base64   ].[      PAYLOAD base64      ].[         SIGNATURE         ]
```

- **Header:** algoritmo de firma (HS256, RS256)
- **Payload:** datos del usuario (sub=user_id, exp=expiration, etc.)
- **Signature:** HMAC del header+payload con una clave secreta

El servidor no guarda el token (stateless). Cuando llega un request con un token, lo valida con la clave secreta sin consultar la BD. Esto lo hace ideal para APIs horizontalmente escalables.

### 6.1 Instalación

```bash
pip install python-jose[cryptography] passlib[bcrypt] python-multipart
```

### 6.2 Utilidades de autenticación

```python
# app/auth/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
import os

SECRET_KEY = os.getenv("SECRET_KEY", "cambia-esto-en-produccion")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Contexto para hash de contraseñas (bcrypt)
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    payload = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    payload.update({"exp": expire})
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError:
        raise ValueError("Token inválido o expirado")
```

### 6.3 Dependencia de autenticación

```python
# app/auth/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.auth.security import decode_token
from app.models.user import User
from sqlalchemy import select

# OAuth2PasswordBearer extrae el token del header: Authorization: Bearer <token>
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="No autenticado",
        headers={"WWW-Authenticate": "Bearer"}
    )
    try:
        payload = decode_token(token)
        user_id: int = int(payload.get("sub"))
    except (ValueError, TypeError):
        raise credentials_exception

    result = await db.execute(select(User).where(User.id == user_id))
    user = result.scalar_one_or_none()

    if not user:
        raise credentials_exception
    return user
```

### 6.4 Endpoints de autenticación

```python
# app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.auth.security import verify_password, create_access_token, hash_password
from app.models.user import User
from sqlalchemy import select
from pydantic import BaseModel, EmailStr

router = APIRouter(prefix="/auth", tags=["Autenticación"])

class RegisterRequest(BaseModel):
    name: str
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

@router.post("/register", status_code=201)
async def register(data: RegisterRequest, db: AsyncSession = Depends(get_db)):
    # Verificar si el email ya existe
    result = await db.execute(select(User).where(User.email == data.email))
    if result.scalar_one_or_none():
        raise HTTPException(status_code=400, detail="Email ya registrado")

    user = User(
        name=data.name,
        email=data.email,
        hashed_password=hash_password(data.password)
    )
    db.add(user)
    await db.commit()
    await db.refresh(user)
    return {"id": user.id, "name": user.name, "email": user.email}

@router.post("/login", response_model=TokenResponse)
async def login(
    form: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(select(User).where(User.email == form.username))
    user = result.scalar_one_or_none()

    if not user or not verify_password(form.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas"
        )

    token = create_access_token({"sub": str(user.id)})
    return {"access_token": token}
```

---

## 7. Arquitectura limpia y organización del proyecto

### Teoría: Capas de una API bien estructurada

Una API profesional separa responsabilidades en capas. Esto mejora la mantenibilidad, testabilidad y escalabilidad:

```
┌─────────────────────────────────────────┐
│           Router / Controller            │  ← Define rutas, llama al servicio
├─────────────────────────────────────────┤
│              Service / Use Case          │  ← Lógica de negocio
├─────────────────────────────────────────┤
│              Repository                  │  ← Acceso a datos (consultas SQL)
├─────────────────────────────────────────┤
│              Model / Schema              │  ← Definición de la estructura de datos
└─────────────────────────────────────────┘
```

**Schemas (Pydantic):** definen qué datos entran y salen (request/response). Son los DTOs.

**Models (SQLAlchemy):** definen la estructura de la base de datos.

**Repository:** encapsulan las consultas a la BD. Si cambias de PostgreSQL a MongoDB, solo cambias el repositorio.

**Service:** orquestan la lógica de negocio. Usan repositorios, no acceden directamente a la BD.

**Router:** definen las rutas HTTP, validan con Pydantic, delegan al servicio.

### Estructura de carpetas del proyecto

```
jobtracker-api/
├── app/
│   ├── __init__.py
│   ├── main.py               # Instancia de FastAPI, inclusión de routers
│   ├── database.py           # Configuración DB
│   ├── config.py             # Variables de entorno con Pydantic Settings
│   │
│   ├── models/               # Modelos SQLAlchemy (tablas)
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── job_application.py
│   │
│   ├── schemas/              # Modelos Pydantic (request/response)
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── job_application.py
│   │
│   ├── repositories/         # Acceso a datos
│   │   ├── __init__.py
│   │   ├── user_repo.py
│   │   └── job_application_repo.py
│   │
│   ├── services/             # Lógica de negocio
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   └── job_application_service.py
│   │
│   ├── routers/              # Endpoints HTTP
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── job_applications.py
│   │
│   └── auth/                 # JWT, hashing, dependencias
│       ├── __init__.py
│       ├── security.py
│       └── dependencies.py
│
├── alembic/                  # Migraciones
├── tests/                    # Tests
├── .env
├── .env.example
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

### main.py: punto de entrada

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import auth, job_applications, analytics, notes

app = FastAPI(
    title="JobTracker API",
    description="Rastrea tus aplicaciones de trabajo y el progreso de cada proceso",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS (necesario si el frontend es un dominio diferente)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # tu frontend
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

# Incluir routers
app.include_router(auth.router)
app.include_router(job_applications.router)
app.include_router(analytics.router)
app.include_router(notes.router)

@app.get("/", tags=["Sistema"])
def health():
    return {"status": "ok", "version": "1.0.0"}
```

### config.py: variables de entorno con Pydantic Settings

**Teoría:** En lugar de leer `os.getenv()` disperso por todo el código, centralizamos la configuración en una clase con Pydantic Settings. Esto valida que las variables existan al iniciar la app y documenta qué necesita el proyecto.

```bash
pip install pydantic-settings
```

```python
# app/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    # Base de datos
    database_url: str

    # JWT
    secret_key: str = "cambia-esto-en-produccion"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # App
    app_name: str = "JobTracker API"
    debug: bool = False

    class Config:
        env_file = ".env"          # lee el archivo .env automáticamente
        env_file_encoding = "utf-8"

# lru_cache garantiza que Settings() solo se instancie una vez
@lru_cache()
def get_settings() -> Settings:
    return Settings()
```

```bash
# .env.example  (sube este al repo; agrega .env al .gitignore)
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/jobtracker
SECRET_KEY=cambia-esto-en-produccion
ACCESS_TOKEN_EXPIRE_MINUTES=30
DEBUG=false
```

### models/user.py: modelo SQLAlchemy del usuario

```python
# app/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class User(Base):
    __tablename__ = "users"

    id              = Column(Integer, primary_key=True, index=True)
    name            = Column(String(100), nullable=False)
    email           = Column(String(200), nullable=False, unique=True, index=True)
    hashed_password = Column(String(255), nullable=False)
    is_active       = Column(Boolean, default=True)
    created_at      = Column(DateTime(timezone=True), server_default=func.now())
    updated_at      = Column(DateTime(timezone=True), onupdate=func.now())

    # Relación inversa: un usuario tiene muchas aplicaciones
    applications = relationship("JobApplication", back_populates="user")
```

> **Nota:** Este modelo completa la FK `user_id` definida en `JobApplication`. SQLAlchemy necesita que ambos lados de la relación estén declarados para que `back_populates` funcione correctamente.

### models/application_note.py: modelo de notas por aplicación

La estructura de carpetas y los endpoints del proyecto incluyen notas. Este es el modelo que faltaba:

```python
# app/models/application_note.py
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.database import Base

class ApplicationNote(Base):
    __tablename__ = "application_notes"

    id             = Column(Integer, primary_key=True, index=True)
    application_id = Column(Integer, ForeignKey("job_applications.id", ondelete="CASCADE"), nullable=False)
    content        = Column(Text, nullable=False)
    created_at     = Column(DateTime(timezone=True), server_default=func.now())

    # Relación inversa
    application = relationship("JobApplication", back_populates="notes")
```

> **Importante:** Agrega `from app.models import application_note` en `alembic/env.py` para que Alembic detecte esta tabla al autogenerar migraciones.

### schemas/user.py: schemas Pydantic del usuario

```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr
from datetime import datetime
from typing import Optional

# --- Request ---

class UserCreate(BaseModel):
    name: str
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    email: EmailStr
    password: str

# --- Response ---

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    is_active: bool
    created_at: datetime

    class Config:
        from_attributes = True  # permite leer desde objetos SQLAlchemy

# --- Token ---

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

class TokenData(BaseModel):
    user_id: Optional[int] = None
```

### schemas/job_application.py: schemas Pydantic de aplicaciones

Estos son los schemas que el router de `job_applications.py` importa. Sin ellos el proyecto no inicia.

```python
# app/schemas/job_application.py
from pydantic import BaseModel, HttpUrl
from typing import Optional
from datetime import datetime
from app.models.job_application import StatusEnum

# --- Request ---

class JobApplicationCreate(BaseModel):
    company: str
    position: str
    salary_min: Optional[float] = None
    salary_max: Optional[float] = None
    job_url: Optional[str] = None

class JobApplicationUpdate(BaseModel):
    company: Optional[str] = None
    position: Optional[str] = None
    salary_min: Optional[float] = None
    salary_max: Optional[float] = None
    job_url: Optional[str] = None

class StatusUpdate(BaseModel):
    status: StatusEnum

# --- Response ---

class JobApplicationResponse(BaseModel):
    id: int
    user_id: int
    company: str
    position: str
    salary_min: Optional[float]
    salary_max: Optional[float]
    job_url: Optional[str]
    status: StatusEnum
    is_active: bool
    applied_at: datetime
    updated_at: Optional[datetime]

    class Config:
        from_attributes = True

# --- Notes (anidado) ---

class NoteCreate(BaseModel):
    content: str

class NoteResponse(BaseModel):
    id: int
    application_id: int
    content: str
    created_at: datetime

    class Config:
        from_attributes = True
```

### repositories/user_repo.py: repositorio del usuario

```python
# app/repositories/user_repo.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.user import User

class UserRepository:

    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_by_email(self, email: str) -> User | None:
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()

    async def get_by_id(self, user_id: int) -> User | None:
        result = await self.db.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalar_one_or_none()

    async def create(self, name: str, email: str, hashed_password: str) -> User:
        user = User(name=name, email=email, hashed_password=hashed_password)
        self.db.add(user)
        await self.db.flush()
        await self.db.refresh(user)
        return user
```

### services/auth_service.py: servicio de autenticación

```python
# app/services/auth_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from fastapi import HTTPException, status
from app.repositories.user_repo import UserRepository
from app.auth.security import hash_password, verify_password, create_access_token
from app.schemas.user import UserCreate, TokenResponse

class AuthService:

    def __init__(self, db: AsyncSession):
        self.repo = UserRepository(db)

    async def register(self, data: UserCreate):
        existing = await self.repo.get_by_email(data.email)
        if existing:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Email ya registrado"
            )
        user = await self.repo.create(
            name=data.name,
            email=data.email,
            hashed_password=hash_password(data.password)
        )
        return user

    async def login(self, email: str, password: str) -> TokenResponse:
        user = await self.repo.get_by_email(email)
        if not user or not verify_password(password, user.hashed_password):
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Credenciales incorrectas"
            )
        token = create_access_token({"sub": str(user.id)})
        return TokenResponse(access_token=token)
```

> **Nota de refactorización:** Con este servicio, el router `auth.py` (sección 6.4) puede simplificarse delegando a `AuthService` en lugar de tener lógica de BD directamente. Esto sigue el mismo patrón de capas descrito al inicio de esta sección.

### services/job_application_service.py: servicio de aplicaciones

Este servicio es importado directamente por el router del proyecto (sección 10). Es el archivo central de la lógica de negocio:

```python
# app/services/job_application_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from fastapi import HTTPException, status
from app.repositories.job_application_repo import JobApplicationRepository
from app.schemas.job_application import JobApplicationCreate, JobApplicationUpdate
from app.models.job_application import StatusEnum

class JobApplicationService:

    def __init__(self, db: AsyncSession):
        self.repo = JobApplicationRepository(db)

    async def list_applications(
        self,
        user_id: int,
        status: str | None = None,
        company: str | None = None,
        limit: int = 20,
        offset: int = 0
    ):
        return await self.repo.list_by_user(
            user_id=user_id,
            status=status,
            company=company,
            limit=limit,
            offset=offset
        )

    async def create_application(self, user_id: int, data: JobApplicationCreate):
        return await self.repo.create({
            "user_id": user_id,
            **data.model_dump()
        })

    async def get_application(self, id: int, user_id: int):
        app = await self.repo.get_by_id(id)
        if not app or app.user_id != user_id:
            return None
        return app

    async def update_application(self, id: int, user_id: int, data: JobApplicationUpdate):
        app = await self.get_application(id, user_id)
        if not app:
            return None
        return await self.repo.update(id, data.model_dump(exclude_unset=True))

    async def update_status(self, id: int, user_id: int, new_status: StatusEnum):
        app = await self.get_application(id, user_id)
        if not app:
            raise HTTPException(status_code=404, detail="Aplicación no encontrada")
        return await self.repo.update(id, {"status": new_status})

    async def delete_application(self, id: int, user_id: int) -> bool:
        app = await self.get_application(id, user_id)
        if not app:
            return False
        return await self.repo.delete(id)
```

> **Nota:** El repositorio `list_by_user` necesita soporte para los parámetros `company`, `limit` y `offset`. Actualiza el método en `repositories/job_application_repo.py`:

```python
# Versión actualizada de list_by_user en job_application_repo.py
async def list_by_user(
    self,
    user_id: int,
    status: str | None = None,
    company: str | None = None,
    limit: int = 20,
    offset: int = 0
):
    query = select(JobApplication).where(JobApplication.user_id == user_id)
    if status:
        query = query.where(JobApplication.status == status)
    if company:
        query = query.where(JobApplication.company.ilike(f"%{company}%"))
    query = query.limit(limit).offset(offset)
    result = await self.db.execute(query)
    return result.scalars().all()
```

### routers/notes.py: router de notas por aplicación

El árbol de endpoints del proyecto define `GET/POST /applications/{id}/notes` pero este router nunca aparece en la guía original:

```python
# app/routers/notes.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.auth.dependencies import get_current_user
from app.models.user import User
from app.models.job_application import JobApplication
from app.models.application_note import ApplicationNote
from app.schemas.job_application import NoteCreate, NoteResponse

router = APIRouter(prefix="/applications", tags=["Notas"])

@router.get("/{id}/notes", response_model=list[NoteResponse])
async def listar_notas(
    id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    # Verificar que la aplicación pertenece al usuario
    result = await db.execute(
        select(JobApplication).where(
            JobApplication.id == id,
            JobApplication.user_id == current_user.id
        )
    )
    if not result.scalar_one_or_none():
        raise HTTPException(status_code=404, detail="Aplicación no encontrada")

    notes_result = await db.execute(
        select(ApplicationNote).where(ApplicationNote.application_id == id)
    )
    return notes_result.scalars().all()

@router.post("/{id}/notes", response_model=NoteResponse, status_code=201)
async def agregar_nota(
    id: int,
    data: NoteCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    # Verificar que la aplicación pertenece al usuario
    result = await db.execute(
        select(JobApplication).where(
            JobApplication.id == id,
            JobApplication.user_id == current_user.id
        )
    )
    if not result.scalar_one_or_none():
        raise HTTPException(status_code=404, detail="Aplicación no encontrada")

    note = ApplicationNote(application_id=id, content=data.content)
    db.add(note)
    await db.commit()
    await db.refresh(note)
    return note
```

### Resumen de la sección 7: mapa de imports

Para que el proyecto arranque sin errores de importación, todos los archivos deben existir y conectarse así:

```
main.py
  ├── routers/auth.py          → services/auth_service.py
  │                                → repositories/user_repo.py → models/user.py
  ├── routers/job_applications.py → services/job_application_service.py
  │                                    → repositories/job_application_repo.py
  │                                        → models/job_application.py
  ├── routers/analytics.py     → models/job_application.py
  ├── routers/notes.py         → models/application_note.py
  │                              → schemas/job_application.py (NoteCreate, NoteResponse)
  └── auth/dependencies.py     → models/user.py
                                 → auth/security.py
```

Todo depende de `app/database.py` (Base, get_db) y `app/config.py` (get_settings).

---

## 8. Testing con Pytest

### Teoría: Por qué testear una API

El testing en APIs tiene tres niveles:

- **Unit tests:** prueban funciones aisladas (lógica de negocio, validaciones).
- **Integration tests:** prueban que varias capas funcionan juntas (servicio + repositorio + BD).
- **E2E / API tests:** hacen requests HTTP reales al servidor y validan respuestas.

FastAPI incluye un `TestClient` basado en `httpx` que permite hacer E2E tests sin necesidad de levantar un servidor real.

**Pytest** es el framework de testing estándar de Python. Equivalente a PHPUnit (Laravel) o Jest (Node.js).

### Instalación

```bash
pip install pytest pytest-asyncio httpx
```

### Configuración

```python
# tests/conftest.py
import pytest
import pytest_asyncio
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import Base, get_db

# BD en memoria para tests (SQLite)
TEST_DATABASE_URL = "sqlite+aiosqlite:///:memory:"

@pytest_asyncio.fixture
async def db_session():
    engine = create_async_engine(TEST_DATABASE_URL)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    TestSession = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
    async with TestSession() as session:
        yield session

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest_asyncio.fixture
async def client(db_session):
    # Sobreescribir la dependencia de BD con la de test
    app.dependency_overrides[get_db] = lambda: db_session
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    app.dependency_overrides.clear()
```

### Tests de endpoints

```python
# tests/test_auth.py
import pytest

@pytest.mark.asyncio
async def test_register_usuario(client):
    response = await client.post("/auth/register", json={
        "name": "Arturo Test",
        "email": "arturo@test.com",
        "password": "Password123!"
    })
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "arturo@test.com"
    assert "password" not in data  # nunca exponer contraseña

@pytest.mark.asyncio
async def test_login_exitoso(client):
    # Primero registrar
    await client.post("/auth/register", json={
        "name": "Test User",
        "email": "test@test.com",
        "password": "Password123!"
    })
    # Luego login
    response = await client.post("/auth/login", data={
        "username": "test@test.com",
        "password": "Password123!"
    })
    assert response.status_code == 200
    assert "access_token" in response.json()

@pytest.mark.asyncio
async def test_login_credenciales_invalidas(client):
    response = await client.post("/auth/login", data={
        "username": "noexiste@test.com",
        "password": "wrong"
    })
    assert response.status_code == 401

# Ejecutar tests:
# pytest tests/ -v
# pytest tests/ -v --cov=app  # con cobertura
```

---

## 9. Dockerización y despliegue

### Teoría: Por qué Docker para APIs Python

Docker garantiza que el entorno de desarrollo sea idéntico al de producción, eliminando el "en mi máquina funciona". Para una API FastAPI en portfolio, tener un `docker-compose.yml` funcional demuestra madurez en DevOps.

### Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Variables de entorno para Python en Docker
ENV PYTHONDONTWRITEBYTECODE=1  # No generar archivos .pyc
ENV PYTHONUNBUFFERED=1         # Logs inmediatos sin buffer

WORKDIR /app

# Instalar dependencias primero (mejor caché de Docker)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código
COPY . .

EXPOSE 8000

# Usar gunicorn con workers uvicorn para producción
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### docker-compose.yml

```yaml
# docker-compose.yml
version: "3.9"

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql+asyncpg://postgres:postgres@db:5432/jobtracker
      SECRET_KEY: super-secret-key-change-in-production
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - .:/app  # hot-reload en desarrollo
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: jobtracker
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

```bash
# Levantar todo el stack
docker-compose up --build

# Aplicar migraciones dentro del contenedor
docker-compose exec api alembic upgrade head

# Ver logs
docker-compose logs -f api

# Parar y limpiar
docker-compose down -v
```

---

## 10. El Proyecto: JobTracker API

### ¿Por qué este proyecto impacta tu portfolio?

**JobTracker** es una API para rastrear procesos de selección. Es un proyecto con:

- **Relevancia real:** Todo desarrollador en búsqueda activa necesita esto (como tú ahora mismo).
- **CRUD completo:** Crea, lee, actualiza y elimina aplicaciones de trabajo.
- **Autenticación JWT:** Cada usuario ve solo sus propios datos.
- **Estadísticas:** Endpoint de analytics que muestra métricas del proceso de búsqueda.
- **Stack profesional:** FastAPI + PostgreSQL + Docker + JWT + Alembic.
- **Documentación automática:** Swagger disponible en `/docs`.

Puedes presentarlo como "una herramienta que construí y uso activamente para mi búsqueda de trabajo". Eso genera conversación en entrevistas.

### Endpoints del proyecto

```
AUTH
POST   /auth/register           → Registrar usuario
POST   /auth/login              → Login, retorna JWT

JOB APPLICATIONS
GET    /applications            → Listar mis aplicaciones (con filtros)
POST   /applications            → Crear nueva aplicación
GET    /applications/{id}       → Ver detalle de una aplicación
PUT    /applications/{id}       → Actualizar aplicación
DELETE /applications/{id}       → Eliminar aplicación
PATCH  /applications/{id}/status → Cambiar status (applied → interview → offer...)

NOTES (notas por aplicación)
GET    /applications/{id}/notes  → Ver notas de una aplicación
POST   /applications/{id}/notes  → Agregar nota

ANALYTICS
GET    /analytics/summary        → Resumen: total, por status, tasa de respuesta
GET    /analytics/timeline       → Aplicaciones por semana/mes
```

### Implementación del router principal

```python
# app/routers/job_applications.py
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Optional
from app.database import get_db
from app.auth.dependencies import get_current_user
from app.models.user import User
from app.schemas.job_application import (
    JobApplicationCreate,
    JobApplicationUpdate,
    JobApplicationResponse,
    StatusUpdate
)
from app.services.job_application_service import JobApplicationService

router = APIRouter(prefix="/applications", tags=["Aplicaciones"])

@router.get("", response_model=list[JobApplicationResponse])
async def listar_aplicaciones(
    status: Optional[str] = Query(None, description="Filtrar por status"),
    company: Optional[str] = Query(None, description="Buscar por empresa"),
    limit: int = Query(20, ge=1, le=100),
    offset: int = Query(0, ge=0),
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    return await service.list_applications(
        user_id=current_user.id,
        status=status,
        company=company,
        limit=limit,
        offset=offset
    )

@router.post("", response_model=JobApplicationResponse, status_code=201)
async def crear_aplicacion(
    data: JobApplicationCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    return await service.create_application(user_id=current_user.id, data=data)

@router.get("/{id}", response_model=JobApplicationResponse)
async def ver_aplicacion(
    id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    application = await service.get_application(id=id, user_id=current_user.id)
    if not application:
        raise HTTPException(status_code=404, detail="Aplicación no encontrada")
    return application

@router.put("/{id}", response_model=JobApplicationResponse)
async def actualizar_aplicacion(
    id: int,
    data: JobApplicationUpdate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    application = await service.update_application(
        id=id, user_id=current_user.id, data=data
    )
    if not application:
        raise HTTPException(status_code=404, detail="Aplicación no encontrada")
    return application

@router.patch("/{id}/status", response_model=JobApplicationResponse)
async def cambiar_status(
    id: int,
    data: StatusUpdate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    return await service.update_status(
        id=id, user_id=current_user.id, new_status=data.status
    )

@router.delete("/{id}", status_code=204)
async def eliminar_aplicacion(
    id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    service = JobApplicationService(db)
    deleted = await service.delete_application(id=id, user_id=current_user.id)
    if not deleted:
        raise HTTPException(status_code=404, detail="Aplicación no encontrada")
```

### Analytics endpoint

```python
# app/routers/analytics.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from app.database import get_db
from app.auth.dependencies import get_current_user
from app.models.user import User
from app.models.job_application import JobApplication, StatusEnum

router = APIRouter(prefix="/analytics", tags=["Analytics"])

@router.get("/summary")
async def resumen(
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    # Contar por status
    result = await db.execute(
        select(JobApplication.status, func.count(JobApplication.id))
        .where(JobApplication.user_id == current_user.id)
        .group_by(JobApplication.status)
    )
    by_status = {row[0].value: row[1] for row in result.all()}

    total = sum(by_status.values())
    responses = total - by_status.get("applied", 0)
    response_rate = round((responses / total * 100), 1) if total > 0 else 0

    return {
        "total": total,
        "by_status": by_status,
        "response_rate_percent": response_rate,
        "offers": by_status.get("offer", 0),
        "rejections": by_status.get("rejected", 0)
    }
```

---

## 📋 Plan de estudio sugerido

| Semana | Tema | Meta |
|--------|------|------|
| 1 | Secciones 2 y 3 | Entorno listo, Python esencial dominado |
| 2 | Sección 4 | CRUD básico con FastAPI sin BD |
| 3 | Sección 5 | BD con SQLAlchemy + primeras migraciones |
| 4 | Sección 6 | Autenticación JWT completa |
| 5 | Secciones 7 y 8 | Arquitectura limpia + primeros tests |
| 6 | Secciones 9 y 10 | Docker + proyecto completo funcionando |

## 🔗 Recursos complementarios

- [Documentación oficial FastAPI](https://fastapi.tiangolo.com) — la mejor documentación de cualquier framework Python
- [SQLAlchemy 2.0 Docs](https://docs.sqlalchemy.org/en/20/) — especialmente la sección async
- [Pydantic v2 Docs](https://docs.pydantic.dev/latest/)
- [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
- [Awesome FastAPI](https://github.com/mjhea0/awesome-fastapi) — recursos y proyectos de ejemplo

---

> **Tip para el portfolio:** Una vez terminada la API, despliégala gratis en [Railway](https://railway.app) o [Render](https://render.com) para tener una URL pública. Agrega el link en tu CV y LinkedIn. Una API documentada con Swagger en producción vale más que diez proyectos locales.
