# Arquitectura del bot UiPath para SAP Business One

## Objetivo
Crear desde cero un bot independiente que automatice visualmente el inicio de sesión en SAP Business One, usando Modern Design Experience y credenciales almacenadas en Orchestrator Asset `SAP-Loggin`.

## Estructura de carpetas

- `Config/` - archivos de configuración, incluyendo el `Config.xlsx` real para valores del bot.
- `Logs/` - registros de ejecución.
- `Screenshots/` - evidencias de inicio de login, errores y login exitoso.
- `Workflows/` - archivos XAML de los workflows.
- `Data/` - datos adicionales o archivos de configuración de soporte.
- `Exceptions/` - workflows y definiciones de excepciones controladas.

## Workflows principales

1. `Workflows/Main.xaml`
   - Inicia el proceso y orquesta todo el flujo.
   - Invoca: `InitConfig.xaml`, `GetSAPCredentials.xaml`, `OpenSAP.xaml`, `LoginSAP.xaml`, `ValidateLogin.xaml`, `ProcesosPosterioresSAP.xaml`.

2. `Workflows/InitConfig.xaml`
   - Carga valores generales.
   - Define rutas, timeouts, nombre de sociedad, logs y screenshots.

3. `Workflows/GetSAPCredentials.xaml`
   - Obtiene credenciales desde Orchestrator Asset `SAP-Loggin`.
   - Almacena en `strUsuarioSAP` y `securePasswordSAP`.
   - No imprime la contraseña en logs.

4. `Workflows/OpenSAP.xaml`
   - Valida si SAP Business One ya está abierto.
   - Si no está abierto, lanza `C:\Program Files (x86)\SAP\SAP Business One\SAP Business One.exe`.
   - Espera la ventana de login.

5. `Workflows/LoginSAP.xaml`
   - Detecta visualmente la ventana de login.
   - Valida campos: Sociedad, Usuario, Contraseña, OK.
   - Ingresa usuario y contraseña.
   - Presiona OK.

6. `Workflows/ValidateLogin.xaml`
   - Verifica que el login fue exitoso.
   - Detecta el menú principal o barra superior de SAP.
   - Registra `[INFO] Login SAP Business One exitoso`.
   - En caso de error, toma screenshot y lanza excepción controlada.

7. `Workflows/ProcesosPosterioresSAP.xaml`
   - Workflow base vacío para futuros procesos.
   - Punto exacto para agregar descargas, consultas, reportes y exportaciones.

## Variables principales

- `strUsuarioSAP` (String)
- `securePasswordSAP` (SecureString)
- `strSAPExecutablePath` (String)
- `strSAPCompanyName` (String)
- `intLoginTimeout` (Int32)
- `strScreenshotPath` (String)
- `strLogPath` (String)
- `strLoginScreenshot` (String)
- `strErrorScreenshot` (String)
- `strSuccessScreenshot` (String)

## Argumentos recomendados

- `in_Environment` (String) - opcional para seleccionar entornos.
- `in_SAPCompanyName` (String) - Nombre de sociedad a validar.
- `out_LoginSuccessful` (Boolean) - Resultado de la validación.

## Flujo de actividades clave

### Main.xaml
- `Sequence`
- `Log Message` `[INFO] Inicio del proceso`
- `Invoke Workflow File` `InitConfig.xaml`
- `Invoke Workflow File` `GetSAPCredentials.xaml`
- `Invoke Workflow File` `OpenSAP.xaml`
- `Invoke Workflow File` `LoginSAP.xaml`
- `Invoke Workflow File` `ValidateLogin.xaml`
- `Invoke Workflow File` `ProcesosPosterioresSAP.xaml`
- `Log Message` `[INFO] Proceso listo para continuar con siguientes workflows`

### InitConfig.xaml
- `Try Catch`
- `Assign` de rutas y timeout configurables.
- `Log Message` `[INFO] Configuración cargada`
- `If` para validar valores obligatorios.

### GetSAPCredentials.xaml
- `Try Catch`
- `Get Credential` con Asset Name `SAP-Loggin`
- Asignar output a `strUsuarioSAP` y `securePasswordSAP`
- `Log Message` `[INFO] Credenciales obtenidas desde Asset SAP-Loggin`

### OpenSAP.xaml
- `Try Catch`
- `Check App State` para ventana SAP Business One
- `If` abierto -> continuar
- `Else` `Start Process` con la ruta ejecutable
- `Wait for Element` / `Element Exists` para la ventana login
- `Log Message` `[INFO] SAP Business One abierto`

### LoginSAP.xaml
- `Try Catch`
- `Wait for Element` en el campo Sociedad y validar texto `Megapaca GT`
- `If` sociedad incorrecta:
  - `Log Message` advertencia
  - `Take Screenshot`
  - `Throw` excepción controlada
- `Type Into` en campo ID de usuario con `strUsuarioSAP`
- `Type Secure Text` en campo clave con `securePasswordSAP`
- `Click` en botón OK
- `Log Message` `[INFO] Usuario ingresado`, `[INFO] Contraseña ingresada`, `[INFO] Botón OK presionado`

### ValidateLogin.xaml
- `Try Catch`
- `Retry Scope` con `Element Exists` del menú principal o barra SAP
- Si existe:
  - `Log Message` `[INFO] Login SAP Business One exitoso`
  - `Take Screenshot` de éxito
- Si no:
  - `Take Screenshot`
  - `Throw` excepción controlada

## Manejo de errores

- Asset no encontrado: Capturar excepción del `Get Credential`.
- SAP no abre: Capturar fallo en `Start Process` y `Check App State`.
- Ventana login no encontrada: Capturar en `Wait for Element`.
- Campo usuario no encontrado: validar con `Element Exists` antes de `Type Into`.
- Campo contraseña no encontrado: idem.
- Botón OK no encontrado: validar antes de `Click`.
- Sociedad incorrecta: registrar warning, screenshot, abortar.
- Usuario/contraseña incorrectos: detectar mensaje de error en SAP y abortar.
- Timeout de carga: usar timeouts configurables en `Wait for Element`.
- Ventana emergente inesperada: detectar con `Element Exists` y manejar con `Throw` o `Click` de cierre.

## Buenas prácticas

- Usar `Modern Design Experience` en todos los workflows.
- Centralizar configuración en `InitConfig.xaml`.
- Mantener el login independiente de procesos posteriores.
- No usar credenciales hardcodeadas ni archivos planos.
- Usar `Try Catch` para cada flujo crítico.
- Usar `Retry Scope` solo donde hay flakiness.
- Evitar `Delay` largos y preferir `Wait for Element`.
- Usar `Take Screenshot` en puntos críticos.
- Registrar logs claros y consistentes.

## Punto de extensión para futuros procesos

- `Workflows/ProcesosPosterioresSAP.xaml` es el `Invoke Workflow File` reservado para añadir:
  - Descarga de facturas
  - Consultas
  - Reportes
  - Exportaciones
  - Otros procesos SAP

## Evidencias esperadas

- `Screenshots/yyyyMMdd_HHmmss_LoginSAP.png` en:
  - inicio de login
  - error
  - login exitoso

## Configuración sugerida

Crear `Config/Config.xlsx` con columnas:
- `Key`
- `Value`
- `Description`

Valores iniciales:
- `SAPExecutablePath` = `C:\Program Files (x86)\SAP\SAP Business One\SAP Business One.exe`
- `SAPCompanyName` = `Megapaca GT`
- `LoginTimeout` = `120000`
- `ScreenshotPath` = `Screenshots`
- `LogPath` = `Logs`

> Se incluyó el archivo real `Config/Config.xlsx` con estos valores para que el workflow pueda ser extendido a lectura desde Excel en la siguiente iteración.

---

Este proyecto está diseñado para una implementación modular, con workflows independientes y una transición clara hacia procesos posteriores después de la autenticación en SAP Business One.
