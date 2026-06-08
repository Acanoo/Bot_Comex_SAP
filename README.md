# Bot_Comex_SAP
Tomar de la carpeta los consolidados correspondientes al primer y segundo corte del día y adjuntarlas a SAP.

## Nuevo bot SAP Business One
Se creó una arquitectura inicial para un bot independiente de UiPath que automatiza el login en SAP Business One.

- `Project.json` con configuración básica.
- `Workflows/Main.xaml` orquesta el login de SAP.
- `Workflows/InitConfig.xaml` inicializa configuración y timeouts.
- `Workflows/GetSAPCredentials.xaml` obtiene credenciales desde Orchestrator Asset `SAP-Loggin`.
- `Workflows/OpenSAP.xaml` abre SAP Business One o reutiliza la sesión existente.
- `Workflows/LoginSAP.xaml` ingresa usuario, contraseña y presiona OK.
- `Workflows/ValidateLogin.xaml` valida login exitoso y toma evidencia.
- `Workflows/ProcesosPosterioresSAP.xaml` placeholder para procesos posteriores.
- `Config/Config_Template.md` especifica la configuración recomendada.
- `Config/Config.xlsx` contiene los valores reales del proyecto.
- `BOT_ARCHITECTURE.md` describe el diseño completo, variables, pseudocódigo y manejo de errores.
