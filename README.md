POC – Detection as Code con Splunk
Arnau Muntané
Alex Bueno

────────────────────────────────────────

1.Objetivo de la POC

El objetivo de esta prueba ha sido demostrar cómo crear una detección en Splunk utilizando el enfoque Detection as Code, definiendo el contenido mediante archivos YAML y generando posteriormente una aplicación instalable en Splunk Enterprise.

La detección desarrollada identifica múltiples intentos fallidos de inicio de sesión en Windows (EventCode 4625) en un intervalo corto de tiempo, lo que puede indicar un posible ataque de fuerza bruta.

────────────────────────────────────────

2.Prerrequisitos

Para reproducir esta POC se necesita:

Ubuntu 22.04 o superior
Docker instalado
Python3 instalado
Git instalado
Acceso a navegador web
Puerto 8000 disponible

Instalación de dependencias en Ubuntu:

sudo apt update
sudo apt install git python3 python3-venv python3-pip docker.io -y
sudo usermod -aG docker $USER
newgrp docker

────────────────────────────────────────

3.Instalación y arranque de Splunk en Docker

Descargar la imagen oficial:

docker pull splunk/splunk:latest

Ejecutar Splunk:

docker run -d --name splunk-dac
-p 8000:8000
-p 8089:8089
-e SPLUNK_START_ARGS=--accept-license
-e SPLUNK_PASSWORD=Admin123!
splunk/splunk:latest

Verificar que el contenedor está activo:

docker ps

Acceso a la interfaz web:

http://localhost:8000

Usuario: admin
Contraseña: Admin123!

────────────────────────────────────────

4.Clonado del proyecto

git clone https://github.com/arnau/splunk-detection-as-code-poc.git

cd splunk-detection-as-code-poc

────────────────────────────────────────

5.Instalación de contentctl

Crear entorno virtual:

python3 -m venv .venv
source .venv/bin/activate

Instalar contentctl:

pip install contentctl

Verificar instalación:

contentctl --version

────────────────────────────────────────

6.Validación del contenido

Validar estructura del proyecto:

contentctl validate --path .

Si la validación no muestra errores, continuar.

────────────────────────────────────────

7.Construcción de la aplicación

Generar la app instalable:

contentctl build --path .

El archivo generado aparecerá en la carpeta dist en formato .tar.gz.

────────────────────────────────────────

8.Instalación de la app en Splunk (vía Web)

Entrar en:

http://localhost:8000

Ir a:

Apps → Manage Apps → Install app from file

Seleccionar el archivo generado en la carpeta dist:

DA-ESS-DAC-WINDOWS-0.0.1.tar.gz

Instalar la aplicación.

────────────────────────────────────────

9.Generación de eventos de prueba

Entrar en Splunk en la sección:

Search & Reporting

Ejecutar la siguiente búsqueda para generar eventos simulados:

| makeresults count=25
| eval _time=now(), EventCode=4625, user="test_user", src_ip="192.168.1.50"
| collect index=wineventlog

Esto genera 25 eventos con EventCode 4625 en el índice wineventlog.

────────────────────────────────────────

10.Ejecución de la detección

Ir a la sección Search y ejecutar la siguiente búsqueda:

index=wineventlog EventCode=4625
| bin _time span=5m
| stats count by _time, user, src_ip
| where count >= 20

El resultado esperado es:

Usuario: test_user
IP origen: 192.168.1.50
Count: 25

Esto confirma que la detección funciona correctamente.

────────────────────────────────────────

11.Activación de la alerta

Ir a:

Settings → Searches, Reports and Alerts

Editar la alerta asociada a la detección.

Configurar:

Trigger alert when number of results > 0
Enable alert

Guardar cambios.

────────────────────────────────────────

12.Validación con AppInspect

Crear entorno virtual independiente:

python3 -m venv appinspect-env
source appinspect-env/bin/activate

Instalar AppInspect:

pip install splunk-appinspect

Ejecutar validación:

splunk-appinspect inspect dist/DA-ESS-DAC-WINDOWS-0.0.1.tar.gz
--mode precert
--output-file appinspect-report.json

Mover el reporte generado a la carpeta appinspect.

────────────────────────────────────────

13.Estructura final del proyecto

appinspect/
contentctl.yml
data_sources/
detections/
dist/
evidence/
README.md
stories/

────────────────────────────────────────

14.Despliegue

En esta POC el despliegue se ha realizado manualmente mediante la interfaz web de Splunk Enterprise.

En un entorno real podría desplegarse mediante:

Instalación manual en Splunk Enterprise
Despliegue en Splunk Cloud mediante ACS
Automatización con CI/CD

────────────────────────────────────────

15.Conclusión

Esta POC demuestra el ciclo completo de Detection as Code:

Definición del contenido en YAML
Validación con contentctl
Construcción de la aplicación
Validación con AppInspect
Instalación en Splunk
Generación de eventos de prueba
Verificación funcional

El proyecto es completamente reproducible y funcional.
