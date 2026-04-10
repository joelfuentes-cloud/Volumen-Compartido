## 🏗️ Arquitectura del Proyecto

El sistema está compuesto por tres contenedores interconectados dentro de una red privada de Docker:

1.  **Nginx (Proxy & Load Balancer):** * Escucha peticiones externas en el puerto `8080`.
    * Reparte el tráfico entre los servidores Apache (Balanceo).
    * Gestiona una zona de caché para reducir la latencia en la carga de imágenes.
2.  **Apache 1 & Apache 2 (Backends):** * Dos instancias independientes que procesan las peticiones HTTP.
3.  **Volumen Compartido:** * Una carpeta local (`web_compartida`) vinculada a ambos servidores, garantizando que el contenido (HTML e imágenes) sea consistente en todo el clúster.

---

## 🛠️ Configuraciones Implementadas

### 🔹 Balanceo de Carga (Round Robin)
Se ha configurado un bloque `upstream` en Nginx. Esto permite que, si un servidor se satura o cae, el otro pueda seguir dando servicio, garantizando la **Alta Disponibilidad**.

### 🔹 Volumen Compartido (Persistencia)
Gracias al uso de **bind mounts** en el archivo `docker-compose.yml`, cualquier cambio realizado en el archivo `index.html` de nuestra carpeta local se refleja instantáneamente en ambos servidores Apache sin necesidad de reiniciar o reconstruir los contenedores.

### 🔹 Optimización con Caché
Se ha habilitado una zona de caché en disco dentro de Nginx. 
* Se ha configurado la cabecera personalizada `X-Cache-Status`.
* Esto permite verificar si una imagen ha sido servida por el Apache (**MISS**) o directamente por la memoria de Nginx (**HIT**).

---

## 📋 Requisitos y Ejecución

1.  Tener instalado **Docker Desktop**.
2.  Clonar este repositorio o descargar los archivos de configuración.
3.  Desde una terminal en la carpeta del proyecto, levantar los servicios:
    ```bash
    docker compose up -d
    ```
4.  Acceder a la web en: `http://localhost:8080`

---

## ✅ Verificación del Funcionamiento

### 1. Logs de Balanceo
Ejecutando `docker compose logs -f`, se puede observar cómo los registros de acceso se alternan entre `apache1` y `apache2` al refrescar la página.

### 2. Prueba de Volumen
Al modificar el archivo `index.html` en el host, el cambio es visible inmediatamente en la web, confirmando que ambos contenedores comparten la misma fuente de datos.

### 3. Prueba de Caché
En las herramientas de desarrollador del navegador (F12 -> Network), las imágenes deben mostrar la cabecera:
`X-Cache-Status: HIT` tras la segunda carga.

---

**Desarrollado como parte de la práctica de despliegue de infraestructuras.**
