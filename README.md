# GitHub Action: OWASP ZAP Full Scan

Esta acción de GitHub (`zaproxy/action-full-scan@v0.12.0`) permite ejecutar un escaneo completo de seguridad en aplicaciones web utilizando [OWASP ZAP](https://www.zaproxy.org/). El escaneo realizado por esta acción incluye:

- **Spider tradicional**: Rastreo básico del sitio web.
- **Spider AJAX (opcional)**: Rastreo adicional para aplicaciones dinámicas basadas en AJAX.
- **Escaneo activo completo**: Análisis de vulnerabilidades en la aplicación web.
- **Reporte de resultados**: Creación o actualización automática de un issue con los hallazgos.

## Uso

```yaml
name: OWASP ZAP Full Scan

on:
  push:
    branches:
      - main

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Ejecutar escaneo de seguridad con OWASP ZAP
        uses: zaproxy/action-full-scan@v0.12.0
        with:
          target: 'https://example.com'  # URL objetivo del escaneo
          docker_name: 'owasp/zap2docker-stable'  # (Opcional) Imagen Docker de ZAP
          rules_file_name: '.zap/rules.tsv'  # (Opcional) Archivo de reglas para ignorar alertas
          cmd_options: '-a'  # (Opcional) Opciones de línea de comandos adicionales
          allow_issue_writing: true  # (Opcional) Habilita la creación/actualización de issues
          issue_title: 'Resultados de escaneo de seguridad OWASP ZAP'  # (Opcional) Título del issue
          token: ${{ secrets.GITHUB_TOKEN }}  # (Opcional) Token para autenticar la creación de issues
          fail_action: false  # (Opcional) Si se establece en true, la acción fallará si se detectan alertas
          artifact_name: 'zap_scan_report'  # (Opcional) Nombre del artefacto del reporte
```
## Parámetros Disponibles

```yaml
- target (Requerido): 
    Descripción: URL de la aplicación web a escanear (puede ser pública o accesible localmente).
    Valor por defecto: N/A
    Ejemplo: 'https://example.com'

- docker_name (Opcional): 
    Descripción: Nombre de la imagen Docker de ZAP a utilizar. Puede ser la versión estable o semanal.
    Valor por defecto: 'owasp/zap2docker-stable'
    Ejemplo: 'owasp/zap2docker-weekly'

- rules_file_name (Opcional): 
    Descripción: Ruta al archivo de reglas para ignorar alertas específicas del escaneo. 
    Valor por defecto: N/A
    Ejemplo: '.zap/rules.tsv'

- cmd_options (Opcional): 
    Descripción: Opciones adicionales de línea de comandos para el escaneo completo.
    Valor por defecto: N/A
    Ejemplo: '-a -config alert.threshold=Low'

- allow_issue_writing (Opcional): 
    Descripción: Habilita la creación/actualización automática de un issue con los resultados del escaneo.
    Valor por defecto: true
    Ejemplo: false

- issue_title (Opcional): 
    Descripción: Título del issue de GitHub que se creará o actualizará con los resultados.
    Valor por defecto: 'OWASP ZAP Full Scan Results'
    Ejemplo: 'Reporte de Seguridad ZAP'

- token (Opcional): 
    Descripción: Token de GitHub para autenticar la creación/actualización del issue.
    Valor por defecto: '${{ secrets.GITHUB_TOKEN }}'
    Ejemplo: '${{ secrets.MY_CUSTOM_TOKEN }}'

- fail_action (Opcional): 
    Descripción: Si se establece en true, la acción fallará si se detectan alertas.
    Valor por defecto: false
    Ejemplo: true

- artifact_name (Opcional): 
    Descripción: Nombre del artefacto que contiene el reporte de ZAP.
    Valor por defecto: 'zap_scan'
    Ejemplo: 'reporte_seguridad'

```
## Ejemplo de Archivo de Reglas (`rules.tsv`)

Si deseas ignorar ciertas alertas durante el escaneo, puedes crear un archivo `.zap/rules.tsv` dentro de tu repositorio con el siguiente formato:

```plaintext
# Código de Alerta    Acción     Comentario
10020                 IGNORE     Ignorar alertas de cookies inseguras
90033                 IGNORE     Ignorar advertencias de seguridad de encabezados HTTP
40012                 FAIL       Tratar como fallo crítico si se detecta
50000                 WARN       Registrar advertencia si se encuentra, pero no fallar

```
## Explicación del Formato

- Código de Alerta: Número único que identifica el tipo de alerta en OWASP ZAP.
- Acción: Define cómo manejar la alerta:
    - IGNORE – Ignorar la alerta por completo
    - WARN – Registrar la alerta como advertencia sin fallar el escaneo.
    - FAIL – Tratar la alerta como crítica y hacer fallar la acción.
- Comentario: Descripción opcional para indicar el motivo de la regla.

Para encontrar los códigos de alerta de ZAP, consulta la [documentación oficial](https://github.com/marketplace/actions/zap-full-scan)

## Reporte de Resultados

Una vez finalizado el escaneo, los resultados estarán disponibles como artefactos de GitHub Actions. El artefacto predeterminado se denomina `zap_scan` (o el nombre personalizado definido en `artifact_name`). 

Los resultados incluirán los siguientes archivos:

```plaintext
- report.html        # Reporte detallado en formato HTML con todas las alertas identificadas
- report.xml         # Reporte en formato XML para procesamiento automatizado
- report.md          # Resumen del escaneo en formato Markdown
- zap.log            # Registro detallado del proceso de escaneo
```
Para acceder a los reportes:
 1. Ve a la pestaña "Actions" en tu repositorio de GitHub

 2. Selecciona la ejecución del workflow correspondiente.

 3. Descarga los artefactos desde la sección "Artifacts".

## Fallo en la Acción

Si el parámetro fail_action se establece en true, la acción de GitHub fallará si se detectan vulnerabilidades.

```yaml

fail_action: true

```
Esto es útil para entornos de producción donde es crítico garantizar que no existan vulnerabilidades antes del despliegue.

## Ejemplo completo del workflow

```yaml
name: OWASP ZAP Security Scan

on:
  schedule:
    - cron: '0 0 * * 1'  # Escaneo semanal los lunes a medianoche
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  security_scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Ejecutar escaneo de seguridad con OWASP ZAP
        uses: zaproxy/action-full-scan@v0.12.0
        with:
          target: 'https://example.com'
          docker_name: 'owasp/zap2docker-stable'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
          allow_issue_writing: true
          issue_title: 'Resultados de escaneo OWASP ZAP'
          token: ${{ secrets.GITHUB_TOKEN }}
          fail_action: true
          artifact_name: 'owasp_zap_report'

      - name: Cargar reportes de escaneo como artefacto
        uses: actions/upload-artifact@v3
        with:
          name: 'owasp_zap_report'
          path: |
            report.html
            report.xml
            report.md
            zap.log
```

## Consejos y Mejores Prácticas

- Uso de entornos de prueba: Realizar escaneos en entornos aislados para evitar posibles interrupciones en producción.

- Revisión de reglas: Ajustar el archivo rules.tsv para omitir falsos positivos y enfocarse en vulnerabilidades críticas.
