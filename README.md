# Análisis Exhaustivo del Proyecto "ApiGestorTareaWeb"

## Introducción

El proyecto **ApiGestorTareaWeb** es una **aplicación web moderna** que funciona como el *frontend* (la interfaz que ves en el navegador) para gestionar proyectos y tareas. Consume la API REST que fue desarrollada en el proyecto anterior **ApiGestorTareas**. En otras palabras, si **ApiGestorTareas** es el "cerebro" que almacena y procesa los datos, **ApiGestorTareaWeb** es la "cara" que permite a los usuarios interactuar con esos datos de forma visual e intuitiva.

**¿Qué significa "consume la API"?** Significa que esta aplicación web realiza peticiones HTTP (solicitudes) a la API para obtener datos, crear proyectos, actualizar tareas, etc. Es como si un cliente (la web) pidiera información a un servidor (la API).

---

## 1. Estructura General del Proyecto

El proyecto está construido sobre **Laravel** [1], pero con un enfoque especial: es una **Single Page Application (SPA)** que se ejecuta principalmente en el navegador del usuario. Aunque usa Laravel como servidor, la mayor parte de la lógica se ejecuta en JavaScript en el cliente.

| Carpeta/Archivo | Propósito | ¿Por qué existe? |
| :--- | :--- | :--- |
| `resources/` | Contiene los archivos de la interfaz de usuario (HTML, CSS, JavaScript). | Es donde se define cómo se ve la aplicación y cómo interactúa el usuario con ella. |
| `resources/views/` | Contiene las vistas Blade (plantillas HTML de Laravel). | Define la estructura HTML de la página. |
| `resources/js/` | Contiene los archivos JavaScript que controlan la lógica de la aplicación. | Ejecuta la lógica del cliente: validaciones, peticiones a la API, actualizaciones de la interfaz. |
| `resources/css/` | Contiene los estilos CSS (aunque en este proyecto, los estilos están incrustados en el HTML). | Define cómo se ve la aplicación (colores, fuentes, espacios, etc.). |
| `routes/` | Define las rutas web que Laravel sirve. | En una SPA, generalmente hay una única ruta que sirve la aplicación. |
| `public/` | Archivos públicos accesibles desde la web (imágenes, JavaScript compilado, etc.). | Punto de acceso para los recursos estáticos. |
| `config/` | Archivos de configuración de Laravel. | Define cómo se comporta Laravel (base de datos, servicios, etc.). |
| `bootstrap/` | Archivos de inicialización de Laravel. | Prepara el entorno para que Laravel funcione. |
| `vendor/` | Librerías de terceros instaladas por Composer. | Proporciona funcionalidades adicionales necesarias para el proyecto. |
| `package.json` | Define las dependencias de Node.js y los scripts de compilación. | Permite compilar los archivos JavaScript y CSS con herramientas modernas como Vite. |
| `composer.json` | Define las dependencias de PHP. | Especifica qué librerías PHP necesita el proyecto. |

---

## 2. Archivos de Configuración Clave

### 2.1. `package.json` (Dependencias de Node.js)

```json
{
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
    "devDependencies": {
        "axios": "^1.6.4",
        "laravel-vite-plugin": "^1.0.0",
        "vite": "^5.0.0"
    }
}
```

**Explicación línea por línea:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `"private": true` | Indica que este proyecto es privado (no se publicará en npm). |
| 3 | `"type": "module"` | Especifica que el proyecto usa módulos ES6 de JavaScript (sintaxis moderna con `import`/`export`). |
| 5-7 | `"scripts": { ... }` | Define comandos que se ejecutan con `npm run`. `npm run dev` inicia el servidor de desarrollo, `npm run build` compila los archivos. |
| 9 | `"axios": "^1.6.4"` | **Axios** es una librería para hacer peticiones HTTP. Se usa para comunicarse con la API. |
| 10 | `"laravel-vite-plugin": "^1.0.0"` | Plugin que integra **Vite** (herramienta de compilación) con Laravel. |
| 11 | `"vite": "^5.0.0"` | **Vite** es un compilador de módulos que transforma el código JavaScript y CSS moderno en código que los navegadores entienden. |

### 2.2. `composer.json` (Dependencias de PHP)

Este archivo es idéntico al del proyecto anterior, ya que ambos usan Laravel. Define que el proyecto necesita PHP 8.1+, Laravel 10.10+, y otras librerías esenciales como Sanctum (para autenticación).

### 2.3. `.env.example` (Variables de Entorno)

Este archivo contiene la configuración por defecto. En producción, se copia a `.env` y se modifican los valores según sea necesario. **Nota importante:** En este proyecto, el archivo `.env` debe incluir una línea adicional:

```
VITE_API_URL=http://127.0.0.1:8000/api
```

Esta variable le dice a la aplicación web dónde está la API REST. Si la API está en otro servidor, se cambiaría esta URL.

---

## 3. Rutas Web (`routes/web.php`)

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/{any}', function () {
    return view('app');
})->where('any', '.*');
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 5-7 | `Route::get('/{any}', function () { ... })->where('any', '.*');` | **Ruta Comodín (Wildcard).** Esta es la clave de una SPA. Cualquier URL que el usuario visite (por ejemplo, `/proyectos`, `/tareas`, `/perfil`) devuelve la misma vista `app.blade.php`. El JavaScript en el navegador se encarga de mostrar el contenido correcto según la URL. El `{any}` captura cualquier segmento de URL, y `where('any', '.*')` permite que capture múltiples segmentos (como `/proyectos/123/tareas`). |

**¿Por qué funciona así?** En una SPA, no hay múltiples páginas HTML. Hay una única página HTML que se carga una vez, y el JavaScript actualiza dinámicamente el contenido según lo que el usuario hace.

---

## 4. Vista Principal (`resources/views/app.blade.php`)

Este archivo es el corazón de la interfaz. Tiene 950 líneas, así que lo desglosaremos en secciones.

### 4.1. Estructura HTML Básica (Líneas 1-30)

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #0d101bff 0%, #271935ff 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1 | `<!DOCTYPE html>` | Declara que este es un documento HTML5. |
| 2 | `<html lang="es">` | Abre el documento HTML e indica que el idioma es español. |
| 4-5 | `<meta charset="UTF-8">` y `<meta name="viewport">` | Metadatos: el primero especifica la codificación de caracteres, el segundo hace que la página sea responsiva (se adapte a dispositivos móviles). |
| 7-12 | `* { margin: 0; padding: 0; box-sizing: border-box; }` | **Reset CSS Global.** Elimina los márgenes y rellenos por defecto de todos los elementos HTML, y establece que el tamaño de los elementos incluya el borde y el relleno. |
| 14-21 | `body { ... }` | Estilos del cuerpo de la página: fuente, fondo con gradiente (de azul oscuro a púrpura), altura mínima de 100% de la pantalla, y centrado con flexbox. |

### 4.2. Estilos CSS Principales (Líneas 23-314)

El archivo contiene estilos CSS incrustados que definen la apariencia de todos los elementos. Aquí están los más importantes:

| Línea(s) | Clase/Selector | Propósito |
| :--- | :--- | :--- |
| 23-30 | `.container` | Contenedor principal: ancho máximo de 1200px, fondo blanco, sombra, y esquinas redondeadas. |
| 32-43 | `.navbar` | Barra de navegación: fondo oscuro, texto blanco, flexbox para alinear elementos. |
| 62-65 | `.auth-form` | Formulario de autenticación: ancho máximo de 400px, centrado. |
| 95-105 | `.btn` | Botones: fondo azul, texto blanco, transiciones suaves. |
| 136-141 | `.projects-container` | Grid responsivo para mostrar tarjetas de proyectos. |
| 143-173 | `.project-card` | Tarjetas de proyectos: fondo gris claro, borde, sombra al pasar el ratón. |
| 175-228 | `.task-item` | Elementos de tareas: borde izquierdo coloreado, flexbox para alinear contenido y botones. |
| 225-249 | `.modal` | Modales (ventanas emergentes): posición fija, fondo oscuro semi-transparente. |
| 272-288 | `.error` y `.success` | Mensajes de error (rojo) y éxito (verde). |

### 4.3. Estructura HTML de la Interfaz (Líneas 316-456)

```html
<body>
    <div class="container">
        <div class="navbar">
            <h1>Gestor de Tareas</h1>
            <div id="userInfo" class="hidden">
                <span id="userName" style="margin-right: 20px;"></span>
                <button onclick="logout()">Cerrar Sesión</button>
            </div>
        </div>
        <div class="content">
            <!-- Auth Forms -->
            <div id="authSection">
                <!-- Formulario de Login -->
                <!-- Formulario de Registro -->
            </div>
            <!-- Main App -->
            <div id="appSection" class="hidden">
                <!-- Sección de Proyectos -->
                <!-- Sección de Detalle de Proyecto -->
            </div>
        </div>
    </div>
    <!-- Modales -->
</body>
```

**Explicación de la estructura:**

| Elemento | ID | Propósito |
| :--- | :--- | :--- |
| `navbar` | N/A | Barra superior con el título y botón de cerrar sesión. |
| `authSection` | `authSection` | Contiene los formularios de login y registro. Visible cuando el usuario NO está autenticado. |
| `appSection` | `appSection` | Contiene la aplicación principal (proyectos y tareas). Visible cuando el usuario SÍ está autenticado. |
| `projectsList` | `projectsList` | Contenedor donde se muestran las tarjetas de proyectos. |
| `projectDetail` | `projectDetail` | Sección que muestra los detalles de un proyecto y sus tareas. |
| `projectModal` | `projectModal` | Ventana emergente para crear/editar proyectos. |
| `taskModal` | `taskModal` | Ventana emergente para crear/editar tareas. |

### 4.4. Formularios de Autenticación (Líneas 328-373)

**Formulario de Login:**

```html
<div id="loginForm" class="auth-form">
    <h2>Iniciar Sesión</h2>
    <div id="loginError" class="error hidden"></div>
    <form onsubmit="handleLogin(event)">
        <div class="form-group">
            <label>Email</label>
            <input type="email" id="loginEmail" required>
        </div>
        <div class="form-group">
            <label>Contraseña</label>
            <input type="password" id="loginPassword" required>
        </div>
        <button type="submit" class="btn" style="width: 100%;">Iniciar Sesión</button>
    </form>
    <div class="form-toggle">
        ¿No tienes cuenta? <a onclick="toggleForms()">Registrarse</a>
    </div>
</div>
```

**Explicación:**

| Línea(s) | Elemento | Explicación |
| :--- | :--- | :--- |
| 1 | `<div id="loginForm">` | Contenedor del formulario de login. |
| 3 | `<div id="loginError">` | Div donde se mostrarán mensajes de error (inicialmente oculto con `hidden`). |
| 4 | `<form onsubmit="handleLogin(event)">` | Formulario que ejecuta la función `handleLogin()` cuando se envía. |
| 5-8 | `<input type="email">` | Campo de entrada para el email. `required` significa que es obligatorio. |
| 9-12 | `<input type="password">` | Campo de entrada para la contraseña. |
| 13 | `<button type="submit">` | Botón que envía el formulario. |
| 15-17 | `<a onclick="toggleForms()">` | Enlace para cambiar al formulario de registro. |

**Formulario de Registro:** Es muy similar, pero con campos adicionales para nombre y confirmación de contraseña.

### 4.5. Sección de Proyectos (Líneas 382-389)

```html
<div>
    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
        <h2>Mis Proyectos</h2>
        <button class="btn" onclick="openProjectModal()">+ Nuevo Proyecto</button>
    </div>
    <div id="projectsList" class="projects-container"></div>
</div>
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `display: flex; justify-content: space-between;` | Usa flexbox para alinear el título a la izquierda y el botón a la derecha. |
| 4 | `onclick="openProjectModal()"` | Cuando se hace clic en el botón, se ejecuta la función `openProjectModal()` que abre la ventana emergente para crear un proyecto. |
| 6 | `<div id="projectsList">` | Contenedor vacío que será rellenado dinámicamente con JavaScript con las tarjetas de proyectos. |

### 4.6. Modales (Líneas 411-455)

Los **modales** son ventanas emergentes que aparecen sobre el contenido principal. Hay dos: uno para proyectos y otro para tareas.

**Modal de Proyecto:**

```html
<div id="projectModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2 id="projectModalTitle">Nuevo Proyecto</h2>
            <button class="close-btn" onclick="closeProjectModal()">&times;</button>
        </div>
        <form onsubmit="handleSaveProject(event)">
            <div class="form-group">
                <label>Nombre del Proyecto</label>
                <input type="text" id="projectName" required>
            </div>
            <div class="form-group">
                <label>Descripción</label>
                <textarea id="projectDescriptionInput" rows="4"></textarea>
            </div>
            <div class="form-group">
                <label>Fecha Límite (Opcional)</label>
                <input type="datetime-local" id="projectDeadline">
            </div>
            <button type="submit" class="btn" style="width: 100%;">Guardar Proyecto</button>
        </form>
    </div>
</div>
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1 | `<div id="projectModal" class="modal">` | Contenedor del modal. Inicialmente oculto con CSS (`display: none`). |
| 5 | `&times;` | Símbolo "×" para cerrar el modal. |
| 7 | `onsubmit="handleSaveProject(event)"` | Cuando se envía el formulario, se ejecuta `handleSaveProject()`. |
| 10 | `<input type="text" id="projectName" required>` | Campo para el nombre del proyecto. |
| 14 | `<textarea>` | Área de texto para la descripción (permite múltiples líneas). |
| 18 | `<input type="datetime-local">` | Campo para seleccionar fecha y hora. |

---

## 5. JavaScript: La Lógica de la Aplicación

El JavaScript es lo que hace que la aplicación funcione. Está dividido en tres archivos principales: `api.js`, `bootstrap.js`, y `app.js`.

### 5.1. `resources/js/api.js` (Cliente API)

Este archivo define una clase `ApiClient` que simplifica las peticiones HTTP a la API.

```javascript
export class ApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
        this.token = null;
    }

    setToken(token) {
        this.token = token;
    }

    async request(method, endpoint, data = null) {
        const url = `${this.baseURL}${endpoint}`;
        const options = {
            method,
            headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
            },
        };

        if (this.token) {
            options.headers['Authorization'] = `Bearer ${this.token}`;
        }

        if (data) {
            options.body = JSON.stringify(data);
        }

        try {
            const response = await fetch(url, options);
            const responseData = await response.json();

            if (!response.ok) {
                throw new Error(
                    responseData.message || 
                    responseData.error || 
                    `HTTP Error: ${response.status}`
                );
            }

            return responseData;
        } catch (error) {
            throw error;
        }
    }

    get(endpoint) {
        return this.request('GET', endpoint);
    }

    post(endpoint, data) {
        return this.request('POST', endpoint, data);
    }

    put(endpoint, data) {
        return this.request('PUT', endpoint, data);
    }

    patch(endpoint, data) {
        return this.request('PATCH', endpoint, data);
    }

    delete(endpoint) {
        return this.request('DELETE', endpoint);
    }
}
```

**Explicación línea por línea:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1-5 | `constructor(baseURL) { ... }` | **Constructor.** Se ejecuta cuando se crea una nueva instancia de `ApiClient`. Recibe la URL base de la API (ej. `http://127.0.0.1:8000/api`). |
| 7-9 | `setToken(token) { ... }` | Método para establecer el token de autenticación. El token se envía en cada petición para identificar al usuario. |
| 11-45 | `async request(method, endpoint, data = null) { ... }` | **Método Principal.** Realiza una petición HTTP genérica. `async` significa que es asincrónica (no bloquea el código). |
| 12 | `const url = \`${this.baseURL}${endpoint}\`;` | Construye la URL completa combinando la URL base con el endpoint (ej. `/login`). |
| 13-19 | `const options = { ... }` | Configura las opciones de la petición: método HTTP, encabezados. |
| 21-23 | `if (this.token) { ... }` | Si hay un token, lo añade al encabezado `Authorization` con el formato `Bearer {token}`. |
| 25-27 | `if (data) { ... }` | Si hay datos, los convierte a JSON y los añade al cuerpo de la petición. |
| 30 | `const response = await fetch(url, options);` | **Fetch API.** Realiza la petición HTTP. `await` espera a que se complete. |
| 31 | `const responseData = await response.json();` | Convierte la respuesta de JSON a un objeto JavaScript. |
| 33-39 | `if (!response.ok) { ... }` | Si la respuesta no es exitosa (código de estado 4xx o 5xx), lanza un error. |
| 47-65 | `get(endpoint) { ... }`, `post(endpoint, data) { ... }`, etc. | **Métodos de Conveniencia.** Simplifican las peticiones comunes (GET, POST, PUT, PATCH, DELETE). |

**¿Cómo se usa?**

```javascript
const api = new ApiClient('http://127.0.0.1:8000/api');
api.setToken('token_del_usuario');
const projects = await api.get('/projects');
```

### 5.2. `resources/js/bootstrap.js` (Inicialización)

```javascript
import axios from 'axios';
window.axios = axios;

window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1 | `import axios from 'axios';` | Importa la librería **Axios** (cliente HTTP). |
| 2 | `window.axios = axios;` | Hace que Axios esté disponible globalmente en el navegador. |
| 4 | `window.axios.defaults.headers.common[...]` | Configura un encabezado por defecto en todas las peticiones de Axios. |

**Nota:** En este proyecto, se importa Axios pero no se usa mucho. La mayoría de peticiones se hacen con la clase `ApiClient` personalizada.

### 5.3. `resources/js/app.js` (Lógica Principal)

Este es el archivo más importante. Contiene toda la lógica de la aplicación. Lo desglosaremos en funciones clave:

#### 5.3.1. Inicialización (Líneas 1-13)

```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://127.0.0.1:8000/api';
let api = new ApiClient(API_URL);
let currentProjectId = null;
let currentEditingProjectId = null;
let currentEditingTaskId = null;

export function initApp() {
    checkAuth();
    setupEventListeners();
}
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1 | `const API_URL = import.meta.env.VITE_API_URL \|\| ...` | Lee la URL de la API desde las variables de entorno. Si no está definida, usa la URL local por defecto. `import.meta.env` es una característica de Vite. |
| 2 | `let api = new ApiClient(API_URL);` | Crea una instancia del cliente API. |
| 3-5 | `let currentProjectId = null;` | Variables globales para rastrear el proyecto y tareas actuales. |
| 7-9 | `export function initApp() { ... }` | Función que se ejecuta cuando la aplicación se carga. Verifica la autenticación y configura los escuchadores de eventos. |

#### 5.3.2. Verificación de Autenticación (Líneas 27-43)

```javascript
async function checkAuth() {
    const token = localStorage.getItem('token');

    if (token) {
        api.setToken(token);
        try {
            const response = await api.get('/user');
            showApp(response.user);
            loadProjects();
        } catch (error) {
            logout();
        }
    } else {
        showAuth();
    }
}
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `const token = localStorage.getItem('token');` | Obtiene el token del almacenamiento local del navegador. El token se guardó cuando el usuario inició sesión. |
| 4-11 | `if (token) { ... }` | Si hay un token, lo establece en el cliente API, intenta obtener los datos del usuario, y carga los proyectos. Si hay error, cierra la sesión. |
| 12-13 | `else { showAuth(); }` | Si no hay token, muestra el formulario de autenticación. |

**¿Qué es `localStorage`?** Es un almacenamiento en el navegador que persiste incluso después de cerrar la pestaña. Se usa para guardar el token de forma que el usuario no tenga que iniciar sesión cada vez que recarga la página.

#### 5.3.3. Manejo de Login (Líneas 68-86)

```javascript
window.handleLogin = async function(event) {
    event.preventDefault();

    const email = document.getElementById('loginEmail').value;
    const password = document.getElementById('loginPassword').value;
    const errorDiv = document.getElementById('loginError');

    try {
        const response = await api.post('/login', { email, password });
        localStorage.setItem('token', response.token);
        api.setToken(response.token);
        showApp(response.user);
        loadProjects();
    } catch (error) {
        errorDiv.textContent = error.message || 'Error al iniciar sesión';
        errorDiv.classList.remove('hidden');
    }
};
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 1 | `window.handleLogin = async function(event) { ... }` | Se asigna a `window` para que sea accesible desde el HTML (`onclick="handleLogin()"`). |
| 2 | `event.preventDefault();` | Previene el comportamiento por defecto del formulario (recargar la página). |
| 4-6 | `const email = ...; const password = ...;` | Obtiene los valores del email y contraseña del formulario. |
| 9 | `const response = await api.post('/login', { email, password });` | Realiza una petición POST a la API con las credenciales. |
| 10-11 | `localStorage.setItem('token', response.token);` y `api.setToken(response.token);` | Guarda el token en el almacenamiento local y lo establece en el cliente API. |
| 12-13 | `showApp(response.user);` y `loadProjects();` | Muestra la aplicación y carga los proyectos del usuario. |
| 14-16 | `catch (error) { ... }` | Si hay error, lo muestra en el div de error. |

#### 5.3.4. Carga de Proyectos (Líneas 137-169)

```javascript
async function loadProjects() {
    try {
        const response = await api.get('/projects');
        const projects = response.data || response;
        displayProjects(projects);
    } catch (error) {
        showError('Error al cargar proyectos: ' + error.message);
    }
}

function displayProjects(projects) {
    const container = document.getElementById('projectsList');

    if (projects.length === 0) {
        container.innerHTML = '<div class="empty-state"><p>No hay proyectos. ¡Crea uno para comenzar!</p></div>';
        return;
    }

    container.innerHTML = projects.map(project => `
        <div class="project-card">
            <h3>${escapeHtml(project.name)}</h3>
            <p>${escapeHtml(project.description || 'Sin descripción')}</p>
            ${project.deadline ? `<p><small>Vencimiento: ${project.deadline}</small></p>` : ''}
            <div class="project-actions">
                <button class="btn btn-success btn-small" onclick="openProject(${project.id})">Abrir</button>
                <button class="btn btn-small" onclick="editProject(${project.id})">Editar</button>
                <button class="btn btn-danger btn-small" onclick="deleteProject(${project.id})">Eliminar</button>
            </div>
        </div>
    `).join('');
}
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `const response = await api.get('/projects');` | Realiza una petición GET a la API para obtener todos los proyectos del usuario. |
| 3 | `const projects = response.data \|\| response;` | Extrae los proyectos. Algunos endpoints devuelven `{ data: [...] }`, otros devuelven directamente el array. |
| 4 | `displayProjects(projects);` | Llama a la función que renderiza los proyectos en la interfaz. |
| 12 | `const container = document.getElementById('projectsList');` | Obtiene el contenedor donde se mostrarán los proyectos. |
| 14-16 | `if (projects.length === 0) { ... }` | Si no hay proyectos, muestra un mensaje vacío. |
| 18-31 | `container.innerHTML = projects.map(...)` | **Template Literals y Map.** Transforma cada proyecto en HTML. `map()` itera sobre cada proyecto, `escapeHtml()` previene inyección de código. |
| 23 | `${project.deadline ? ... : ''}` | **Operador Ternario.** Si hay fecha límite, la muestra; si no, no muestra nada. |

#### 5.3.5. Manejo de Tareas (Líneas 195-228)

Las funciones para tareas son muy similares a las de proyectos:

```javascript
async function loadTasks(projectId) {
    try {
        const response = await api.get(`/projects/${projectId}/tasks`);
        const tasks = response.data || response;
        displayTasks(tasks);
    } catch (error) {
        showError('Error al cargar tareas: ' + error.message);
    }
}

function displayTasks(tasks) {
    const container = document.getElementById('tasksList');

    if (tasks.length === 0) {
        container.innerHTML = '<div class="empty-state"><p>No hay tareas en este proyecto.</p></div>';
        return;
    }

    container.innerHTML = tasks.map(task => `
        <div class="task-item ${task.completed ? 'completed' : ''}">
            <div class="task-info">
                <div class="task-title">${escapeHtml(task.title)}</div>
                ${task.description ? `<div class="task-description">${escapeHtml(task.description)}</div>` : ''}
            </div>
            <div class="task-actions">
                <button class="btn btn-success btn-small" onclick="toggleTask(${task.id})">${task.completed ? '↩️ Deshacer' : '✓ Completar'}</button>
                <button class="btn btn-small" onclick="editTask(${task.id})">Editar</button>
                <button class="btn btn-danger btn-small" onclick="deleteTask(${task.id})">Eliminar</button>
            </div>
        </div>
    `).join('');
}
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `const response = await api.get(\`/projects/${projectId}/tasks\`);` | Obtiene las tareas de un proyecto específico. Usa template literals (backticks) para insertar el ID del proyecto en la URL. |
| 20 | `<div class="task-item ${task.completed ? 'completed' : ''}">` | Si la tarea está completada, añade la clase `completed` que la hace aparecer tachada y más opaca. |
| 25 | `${task.completed ? '↩️ Deshacer' : '✓ Completar'}` | El botón cambia de texto según si la tarea está completada o no. |

#### 5.3.6. Crear/Editar Proyectos (Líneas 244-290)

```javascript
window.handleSaveProject = async function(event) {
    event.preventDefault();

    const name = document.getElementById('projectName').value;
    const description = document.getElementById('projectDescription').value;
    const deadline = document.getElementById('projectDeadline').value;

    try {
        if (currentEditingProjectId) {
            await api.put(`/projects/${currentEditingProjectId}`, {
                name,
                description,
                deadline: deadline || null
            });
            showSuccess('Proyecto actualizado correctamente');
        } else {
            await api.post('/projects', {
                name,
                description,
                deadline: deadline || null
            });
            showSuccess('Proyecto creado correctamente');
        }

        closeProjectModal();
        loadProjects();
    } catch (error) {
        showError('Error al guardar proyecto: ' + error.message);
    }
};
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 9-15 | `if (currentEditingProjectId) { ... }` | Si estamos editando un proyecto existente, realiza una petición PUT. |
| 16-22 | `else { ... }` | Si es un proyecto nuevo, realiza una petición POST. |
| 13 | `deadline: deadline || null` | Si no hay fecha límite, envía `null` en lugar de una cadena vacía. |
| 25 | `closeProjectModal();` | Cierra la ventana emergente. |
| 26 | `loadProjects();` | Recarga la lista de proyectos para mostrar los cambios. |

#### 5.3.7. Eliminar Proyectos (Líneas 292-307)

```javascript
window.deleteProject = async function(projectId) {
    if (!confirm('¿Estás seguro de que deseas eliminar este proyecto?')) return;

    try {
        await api.delete(`/projects/${projectId}`);
        showSuccess('Proyecto eliminado correctamente');
        if (currentProjectId === projectId) {
            closeProjectDetail();
        } else {
            loadProjects();
        }
    } catch (error) {
        showError('Error al eliminar proyecto: ' + error.message);
    }
};
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 2 | `if (!confirm(...)) return;` | Muestra un cuadro de confirmación. Si el usuario no confirma, la función retorna sin hacer nada. |
| 5 | `await api.delete(\`/projects/${projectId}\`);` | Realiza una petición DELETE a la API. |
| 7-10 | `if (currentProjectId === projectId) { ... } else { ... }` | Si el proyecto eliminado es el que estamos viendo, cierra la vista de detalles. Si no, solo recarga la lista. |

#### 5.3.8. Cambiar Estado de Tarea (Líneas 371-379)

```javascript
window.toggleTask = async function(taskId) {
    try {
        await api.patch(`/projects/${currentProjectId}/tasks/${taskId}/complete`, {});
        loadTasks(currentProjectId);
    } catch (error) {
        showError('Error al actualizar tarea: ' + error.message);
    }
};
```

**Explicación:**

| Línea(s) | Código | Explicación |
| :--- | :--- | :--- |
| 3 | `await api.patch(..., {});` | Realiza una petición PATCH (actualización parcial) al endpoint `/complete`. El cuerpo está vacío `{}` porque el servidor solo necesita el ID de la tarea. |
| 4 | `loadTasks(currentProjectId);` | Recarga las tareas para mostrar el cambio de estado. |

#### 5.3.9. Funciones Auxiliares (Líneas 394-424)

```javascript
function showSuccess(message) {
    const div = document.getElementById('successMessage');
    div.textContent = message;
    div.classList.remove('hidden');
    setTimeout(() => {
        div.classList.add('hidden');
    }, 3000);
}

function showError(message) {
    const div = document.getElementById('errorMessage');
    div.textContent = message;
    div.classList.remove('hidden');
    setTimeout(() => {
        div.classList.add('hidden');
    }, 3000);
}

function escapeHtml(text) {
    const map = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#039;'
    };
    return text.replace(/[&<>"']/g, m => map[m]);
}
```

**Explicación:**

| Función | Propósito |
| :--- | :--- |
| `showSuccess(message)` | Muestra un mensaje de éxito verde durante 3 segundos. |
| `showError(message)` | Muestra un mensaje de error rojo durante 3 segundos. |
| `escapeHtml(text)` | **Prevención de XSS.** Convierte caracteres especiales HTML en entidades, para que el contenido no sea interpretado como código. Por ejemplo, `<script>` se convierte en `&lt;script&gt;`. |

---

## 6. Flujo Completo de la Aplicación

Para entender cómo todo funciona junto, aquí está el flujo completo:

### 6.1. Cuando el usuario abre la aplicación

1. **Se carga `app.blade.php`** en el navegador.
2. **Se ejecuta `initApp()`** (línea 457 del HTML tiene `<script>` que llama a esta función).
3. **`initApp()` llama a `checkAuth()`:**
   - Si hay un token en `localStorage`, lo usa para obtener los datos del usuario.
   - Si la petición es exitosa, muestra la aplicación (`showApp()`).
   - Si falla, muestra el formulario de autenticación (`showAuth()`).

### 6.2. Cuando el usuario inicia sesión

1. El usuario ingresa email y contraseña en el formulario.
2. Se ejecuta `handleLogin()`.
3. Se realiza una petición POST a `/login` con las credenciales.
4. La API devuelve un token y los datos del usuario.
5. El token se guarda en `localStorage`.
6. Se muestra la aplicación y se cargan los proyectos.

### 6.3. Cuando el usuario crea un proyecto

1. Hace clic en "+ Nuevo Proyecto".
2. Se abre el modal (`openProjectModal()`).
3. Ingresa el nombre, descripción y fecha límite.
4. Hace clic en "Guardar Proyecto".
5. Se ejecuta `handleSaveProject()`.
6. Se realiza una petición POST a `/projects` con los datos.
7. La API crea el proyecto y devuelve los datos.
8. Se muestra un mensaje de éxito.
9. Se cierra el modal y se recarga la lista de proyectos.

### 6.4. Cuando el usuario abre un proyecto

1. Hace clic en "Abrir" en una tarjeta de proyecto.
2. Se ejecuta `openProject(projectId)`.
3. Se realiza una petición GET a `/projects/{id}` para obtener los detalles.
4. Se muestra la sección de detalles del proyecto.
5. Se cargan las tareas del proyecto (`loadTasks()`).
6. Las tareas se muestran en la interfaz.

---

## 7. Tecnologías Utilizadas

| Tecnología | Propósito |
| :--- | :--- |
| **Laravel** | Framework PHP que sirve la aplicación y maneja las rutas. |
| **Blade** | Motor de plantillas de Laravel que genera el HTML. |
| **JavaScript (ES6)** | Lógica de la aplicación que se ejecuta en el navegador. |
| **CSS** | Estilos visuales de la interfaz. |
| **Vite** | Herramienta de compilación que transforma el código moderno en código que los navegadores entienden. |
| **Axios** | Librería para hacer peticiones HTTP (importada pero no usada mucho en este proyecto). |
| **Fetch API** | API nativa del navegador para hacer peticiones HTTP (usada en `ApiClient`). |
| **localStorage** | API del navegador para almacenar datos persistentemente. |

---

## 8. Seguridad

La aplicación implementa varias medidas de seguridad:

1. **Autenticación con Tokens:** Los usuarios deben iniciar sesión para acceder a la aplicación. El token se envía en cada petición.
2. **Validación de Entrada:** La función `escapeHtml()` previene ataques XSS (inyección de código).
3. **Confirmación de Acciones Destructivas:** Antes de eliminar, se pide confirmación al usuario.
4. **HTTPS en Producción:** En producción, todas las comunicaciones deben ser cifradas con HTTPS.

---

## Conclusión

**ApiGestorTareaWeb** es una aplicación web moderna que demuestra cómo construir una interfaz interactiva que se comunica con una API REST. Utiliza un enfoque de **Single Page Application (SPA)**, donde toda la lógica se ejecuta en el navegador del usuario, y el servidor solo sirve la página HTML inicial y responde a las peticiones de la API.

La arquitectura es clara y modular: el archivo `api.js` maneja la comunicación con la API, `bootstrap.js` configura las librerías, y `app.js` contiene toda la lógica de la aplicación. El HTML define la estructura visual, y el CSS define el estilo.

---

### Referencias

[1] Laravel. *The PHP Framework For Web Artisans*. https://laravel.com/
