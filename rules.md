# Reglas de Gobernanza del Workspace - Sistema Logístico IoT con IA Multi-Agente

## 1. Arquitectura Backend
- **Framework & Asincronismo**: Todo el backend debe estar construido utilizando Python asincrónico nativo (`asyncio`) con frameworks de alto rendimiento (`FastAPI`). Prohibido el uso de llamadas bloqueantes de I/O en el loop principal.
- **Estructuración Modular**: Separación estricta de responsabilidades: configuración, controladores API, servicios de red, servicios de correo y motores de decisión de inventario.

## 2. Hardware Edge
- **Raspberry Pi Pico**: Firmware implementado exclusivamente en **MicroPython**. El sensor infrarrojo (IR) debe operar en el pin GP15 (0 para objeto detectado / presencia, 1 para ausencia) con resistencia pull-up activada y control de rebote (debounce). LED de estado visual en pin GP14.
- **ESP32**: Firmware en **C++ Asíncrono** utilizando `ESPAsyncWebServer` y `ArduinoJson`. Estructuración modular en archivos de cabecera (`main.ino`, `Server.hpp`, `API.hpp`, `ESP32_Utils_APIREST.hpp`). Prohibido el uso de `delay()` bloqueante en el loop principal.

## 3. Seguridad Obligatoria y Manejo de Secretos
- **Cero Credenciales Hardcodeadas**: Prohibido estrictamente incluir credenciales, contraseñas, URLs o tokens en el código fuente.
- **Variables de Entorno**: Carga obligatoria mediante `os.getenv` o Pydantic `BaseSettings`:
  - `MAILTRAP_API_KEY`: Token de API para autenticación con el servicio de Mailtrap.
  - `ESP32_IP`: Dirección IP o hostname del microcontrolador ESP32.
  - `MAILTRAP_SENDER_EMAIL`: Correo remitente autorizado.
  - `ALERT_RECIPIENT_EMAIL`: Correo del encargado de almacén/operaciones.
  - `SUPABASE_URL` / `SUPABASE_KEY`: Credenciales de acceso a la base de datos persistente.
- **Validación al Inicio**: La aplicación debe fallar rápidamente (`fail-fast`) si alguna variable crítica requerida no se encuentra presente al inicializarse.

## 4. Manejo de Errores y Tolerancia a Fallos
- **Bloques Try-Except Explícitos**: Prohibidos los bloques `except: pass` o captura genérica no tipada sin registro.
- **Límites de Reintentos de Sensores**: Las lecturas de telemetría y consultas HTTP hacia el ESP32 deben incorporar políticas de reintento exponencial con un máximo estricto de 3 intentos (`MAX_SENSOR_RETRIES = 3`), acompañadas de jitter y timeouts claros (ej. 3.0s).
- **Logging Estructurado**: Emisión de logs en formato JSON estructurado con niveles de severidad (`INFO`, `WARNING`, `ERROR`, `CRITICAL`), incluyendo timestamp ISO-8601, contexto del dispositivo y traza de error.
