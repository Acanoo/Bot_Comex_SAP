# Plantilla de Configuración

Este archivo documenta la plantilla sugerida para `Config/Config.xlsx`.

| Key               | Value                                                                  | Description                                  |
|-------------------|------------------------------------------------------------------------|----------------------------------------------|
| SAPExecutablePath | C:\Program Files (x86)\SAP\SAP Business One\SAP Business One.exe   | Ruta del ejecutable de SAP Business One     |
| SAPCompanyName    | Megapaca GT                                                            | Nombre de sociedad que debe validar el login|
| LoginTimeout      | 120000                                                                 | Tiempo de espera para login en ms           |
| ScreenshotPath    | Screenshots                                                            | Carpeta donde se guardan evidencias         |
| LogPath           | Logs                                                                   | Carpeta donde se guardan logs               |

> Nota: la implementación del bot debe leer estos valores desde un archivo de configuración o desde assets si se extiende el proyecto más adelante. Para el flujo inicial, se recomienda cargar estos valores en `InitConfig.xaml`.
