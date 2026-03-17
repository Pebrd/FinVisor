# FinVisor

![Version](https://img.shields.io/badge/version-1.1.0%2B10-blue)
![Platform](https://img.shields.io/badge/platform-Android%20%7C%20Linux%20%7C%20Windows-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)
![Flutter](https://img.shields.io/badge/Flutter-3.22-orange)

**Personal finance organizer. Your APIs, your data, your privacy.**

FinVisor is a mobile-first finance dashboard that gives you complete control over your financial data. Unlike traditional finance apps that collect and monetize your information, FinVisor keeps everything local. You bring your own API keys, and your data stays on your device.

---

## Tabla de Contenidos

1. [Features](#features)
2. [Tech Stack](#tech-stack)
3. [Arquitectura](#arquitectura)
4. [APIs Utilizadas](#apis-utilizadas)
5. [Privacidad y Seguridad](#privacidad-y-seguridad)
6. [Instalación](#instalación)
7. [Configuración de API Keys](#configuración-de-api-keys)
8. [Pantallas](#pantallas)
9. [Personalidades IA](#personalidades-ia)
10. [Desarrollo](#desarrollo)
11. [Estructura del Proyecto](#estructura-del-proyecto)
12. [Resolución de Problemas](#resolución-de-problemas)
13. [Licencia](#licencia)

---

## Features

| Feature | Descripción |
|---------|-------------|
| **Dashboard** | Vista en tiempo real de cotizaciones del dólar argentino (Oficial, Blue, CCL, MEP, Bolsa, Tarjeta), índice de riesgo país, inflación mensual y Fear & Greed Index |
| **Mercados** | Seguimiento de acciones y criptomonedas con precios en tiempo real, gráficos sparkline de 7 días y watchlists personalizables |
| **Asistente IA** | Chat con asesor financiero IA. Elegí entre 3 personalidades: Analista, Trader o Profesor. Incluye resumen diario de mercados |
| **Noticias** | Noticias financieras curadas de Argentina y mercados globales con lector de artículos y soporte Markdown |
| **Alertas** | Configuración de alertas de precio personalizadas con notificaciones locales cuando se alcanza el objetivo |
| **Export/Import** | Exportá e importá tu configuración (watchlist, preferences, API keys) de forma segura con contraseña opcional |
| **Privacy First** | Todas las API keys almacenadas localmente. Sin telemetría. Sin recolección de datos. Sin backend propio |

### Features Adicionales

- **Gráficos interactivos**: Gráficos de velas (candlestick) para análisis técnico en pantalla de detalle de activo
- **Calendario de Ganancias**: Ver fechas de reportes de earnings próximos
- **Sentimiento Insider**: Información de operaciones de insiders de empresas
- **Recomendaciones de Analistas**: Tendencias de recomendaciones de analistas financieros
- **Commodities**: Seguimiento de precios de commodities (oro, petróleo, soja, etc.)
- **Indicadores Macroeconómicos**: Datos de FRED (tasas de interés, inflación USA, etc.)
- **PDF Export**: Exportar resúmenes de briefing IA a PDF

---

## Tech Stack

| Categoría | Tecnología | Versión |
|-----------|------------|---------|
| Framework | Flutter | 3.22+ |
| Lenguaje | Dart | 3.3+ |
| Estado | Riverpod | 2.5.1 |
| Anotaciones | riverpod_annotation | 2.3.5 |
| Navegación | go_router | 13.2.0 |
| HTTP Client | Dio | 5.4.3 |
| Cache HTTP | dio_cache_interceptor | 4.0.5 |
| Conectividad | connectivity_plus | 6.0.3 |
| Base de Datos | Drift (SQLite) | 2.18.0 |
| Storage Seguro | flutter_secure_storage | 9.2.2 |
| Preferencias | shared_preferences | 2.2.3 |
| Modelos | Freezed | 2.5.2 |
| JSON | json_serializable | 6.8.0 |
| UI | google_fonts | 6.2.1 |
| Gráficos | fl_chart | 0.68.0 |
| Notificaciones | flutter_local_notifications | 19.5.0 |
| Markdown | flutter_markdown | 0.7.7+1 |
| PDF | pdf | 3.11.3 |
| Shares | share_plus | 10.1.0 |
| WebView | flutter_inappwebview | 6.1.5 |
| XML/RSS | dart_rss, xml | 3.0.3, 6.3.0 |
| Similaridad | string_similarity | 2.0.0 |
| Iconos | phosphor_flutter | 2.1.0 |
| Slider | carousel_slider | 5.1.2 |
| Archivos | file_picker | 8.1.5 |
| Utils | intl | 0.19.0 |
| Test | mocktail | 1.0.4 |

---

## Arquitectura

FinVisor sigue **Clean Architecture** con separación clara de responsabilidades:

```
lib/
├── core/                    # Theme, constants, router, servicios, utils
│   ├── constants/           # Constantes de app, APIs, personalidades IA
│   ├── services/            # Servicios (config, notifications)
│   ├── theme/               # Tema, colores, estilos de texto
│   ├── router/              # Configuración de rutas (go_router)
│   └── utils/               # Utilidades (formateo, validación)
├── data/                    # Capa de datos
│   ├── datasources/
│   │   ├── api/             # APIs externas (Dio clients)
│   │   └── local/           # Base de datos (Drift), Secure Storage
│   ├── models/              # Modelos de datos (JSON parsing)
│   └── repositories/        # Implementación de repositorios
├── domain/                  # Capa de dominio
│   ├── entities/            # Entidades inmutables (Freezed)
│   ├── repositories/        # Interfaces de repositorios
│   └── states/              # Estados de datos (DataState)
└── presentation/            # Capa de presentación
    ├── providers/           # Providers Riverpod
    ├── screens/             # Pantallas/widgets de UI
    └── widgets/             # Componentes reutilizables
```

### Flujo de Datos

```
UI (Widgets)
    ↓
Providers (Riverpod) ←── Estado global
    ↓
Repositories ←── Lógica de negocio
    ↓
Data Sources (API / Local DB)
    ↓
DataState<T>
    ├── DataLoading<T>     // Cargando
    ├── DataReady<T>       // Datos listos
    ├── DataStale<T>       // Datos cacheados (API falló)
    ├── DataError<T>       // Error sin cache
    └── ApiKeyMissing<T>   // Falta API key
```

### Conceptos Clave

- **DataState**: Wrapper unificado que maneja todos los estados de operaciones de datos (cargando, listo, desactualizado, error, key faltante)
- **Offline-First**: La app funciona con datos cacheados cuando las APIs no están disponibles
- **Graceful Degradation**: Cada feature funciona independientemente según las API keys disponibles
- **Fallback System**: Múltiples proveedores de datos con caída automática (ej: Finnhub → FMP → AlphaVantage)

---

## APIs Utilizadas

| API | Propósito | Tier Gratuito | Endpoint Base |
|-----|-----------|---------------|---------------|
| **ArgentinaDatos** | Cotizaciones dólar argentino, riesgo país, inflación | Ilimitado | `https://api.argentinadatos.com` |
| **Finnhub** | Quotes de acciones, recomendaciones, calendario earnings, sentimiento insider | 60 calls/min | `https://finnhub.io/api/v1` |
| **NewsAPI** | Noticias financieras | 100 requests/día | `https://newsapi.org/v2` |
| **OpenRouter** | Chat completions IA | Pay-as-you-go (créditos gratis) | `https://openrouter.ai/api/v1` |
| **AlphaVantage** | Quotes globales, forex | 25 requests/día | `https://www.alphavantage.co/query` |
| **FRED** | Indicadores macroeconómicos USA | Ilimitado | `https://api.stlouisfed.org/fred` |
| **CoinGecko** | Precios de criptomonedas | 10-30 calls/min | `https://api.coingecko.com/api/v3` |
| **Alternative.me** | Fear & Greed Index | Ilimitado | `https://api.alternative.me/fng` |
| **Financial Modeling Prep (FMP)** | Quotes fallback, datos empresa | 250 requests/día | `https://financialmodelingprep.com/api/v3` |
| **Twelve Data** | Quotes fallback | 800 requests/día | `https://api.twelvedata.com` |
| **Google Gemini** | IA fallback (vision) | Pay-as-you-go | `https://generativelanguage.googleapis.com/v1` |
| **DuckDuckGo** | Búsqueda web para IA | Ilimitado | `https://duckduckgo.com` |
| **RSS Feeds** | Noticias alternativas | Ilimitado | Múltiples fuentes |

---

## Privacidad y Seguridad

### Principios Fundamentales

1. **Tus Keys, Tus Datos**: Todas las API keys se almacenan localmente usando storage nativo seguro de cada plataforma
2. **Sin Telemetría**: Cero analytics, cero crash reporting que envíe datos externamente
3. **Sin Backend**: La app opera 100% del lado del cliente
4. **Offline-First**: Las features core funcionan sin internet usando datos cacheados
5. **Tu Control**: Vos traés tus propias API keys y decidís qué datos compartir

### Medidas de Seguridad por Plataforma

| Plataforma | Mecanismo | Requerimientos |
|------------|-----------|----------------|
| Android | Keystore + EncryptedSharedPreferences | Ninguno |
| iOS | Keychain | Ninguno |
| Linux | libsecret (GNOME Keyring) | `sudo apt install libsecret-1-dev` |
| Windows | DPAPI | Parte del SO |
| macOS | Keychain | Ninguno |

### Prácticas de Seguridad Implementadas

- API keys nunca se loggean ni envían a servicios de terceros
- `LogInterceptor` de Dio con `requestHeader: false` siempre
- Uso de `debugPrint()` en lugar de `print()` para evitar leaks en producción
- Android `android:allowBackup="false"` y exclusion de FlutterSecureStorage en backups
- Builds de release siempre con obfuscation (`--obfuscate`)
- Validación de formato de API keys antes de almacenamiento

### Validación de API Keys

```
OpenRouter:   Empieza con "sk-or-", longitud > 20
NewsAPI:      Longitud >= 32, solo [a-zA-Z0-9]
Finnhub:      Longitud >= 20, solo [a-zA-Z0-9_]
AlphaVantage: Longitud >= 16, solo [a-zA-Z0-9]
FRED:         Longitud >= 20, solo [a-zA-Z0-9]
```

---

## Instalación

### Requisitos Previos

- Flutter 3.22+ SDK
- [FVM](https://fvm.app/) (Flutter Version Manager) - recomendado
- Dart 3.3+
- Git

### Pasos de Instalación

#### 1. Clonar el repositorio

```bash
git clone <repository-url>
cd finvisor
```

#### 2. Instalar versión de Flutter (opcional pero recomendado)

```bash
fvm install 3.22.0
fvm use 3.22.0
```

#### 3. Instalar dependencias

```bash
flutter pub get
# o con FVM
fvm flutter pub get
```

#### 4. Configurar archivo de entorno

Crear archivo `.env` en la raíz del proyecto:

```bash
# Copiar de ejemplo
cp .env.example .env

# Editar con tus API keys
nano .env
```

#### 5. Ejecutar generador de código

Si modificaste archivos con anotaciones (`@riverpod`, `@freezed`, `@DriftDatabase`):

```bash
dart run build_runner build --delete-conflicting-outputs
# o con FVM
fvm dart run build_runner build --delete-conflicting-outputs
```

#### 6. Ejecutar la app

```bash
flutter run
# o con FVM
fvm flutter run
```

### Builds

```bash
# Debug build (Android)
flutter build apk --debug

# Release build (Android)
flutter build apk --release

# Linux desktop
flutter build linux --release

# Windows
flutter build windows --release
```

---

## Configuración de API Keys

### Keys Requeridas vs Opcionales

| Feature | Key Requerida | Opción Gratis |
|---------|---------------|---------------|
| Dashboard (dólar, riesgo, inflación) | No | ArgentinaDatos (gratis) |
| Mercados (stocks) | Sí | Finnhub (recomendado) |
| Mercados (cripto) | No | CoinGecko (gratis) |
| IA Chat | Sí | OpenRouter (créditos gratis) |
| Noticias | Sí | NewsAPI (100/día gratis) |
| AI Briefing | Sí | OpenRouter |
| Commodities | Sí | AlphaVantage o FMP |
| Macro USA | Sí | FRED (gratis) |

### Obtención de API Keys

#### 1. OpenRouter (AI Chat)
- URL: [openrouter.ai](https://openrouter.ai)
- Registro: Gratis con crédito inicial
- Modelos Soportados: Mistral, Llama, GPT, Claude, Gemini

#### 2. Finnhub (Stocks)
- URL: [finnhub.io](https://finnhub.io)
- Registro: API key inmediata
- Límite: 60 calls/minuto en tier gratuito

#### 3. NewsAPI (Noticias)
- URL: [newsapi.org](https://newsapi.org)
- Registro: 100 requests/día gratis
- Nota: No funciona en localhost (usar device/emulator)

#### 4. AlphaVantage (Opcional - Stocks Global)
- URL: [alphavantage.co](https://www.alphavantage.co)
- Registro: 25 requests/día gratis

#### 5. FRED (Opcional - Macro USA)
- URL: [fred.stlouisfed.org](https://fred.stlouisfed.org)
- Registro: API key gratuita

#### 6. CoinGecko (Cripto)
- URL: [coingecko.com](https://www.coingecko.com)
- Registro: 10-30 calls/min (sin key básica)

### Variables de Entorno

```bash
# .env file - NO COMMITEAR A GIT
OPENROUTER_KEY=sk-or-v1-tu-key-aqui
NEWS_API_KEY=tu-newsapi-key
FINNHUB_KEY=tu-finnhub-key
ALPHA_VANTAGE_KEY=tu-alpha-vantage-key
FRED_KEY=tu-fred-key
COINGECKO_API_KEY=tu-coingecko-key
FMP_KEY=tu-fmp-key
TWELVEDATA_KEY=tu-twelvedata-key
ARG_MARKET_ID=tu-arg-market-id
```

---

## Pantallas

### 1. Dashboard (`/`)

Pantalla central que muestra:
- **Cotizaciones del Dólar**: Oficial, Blue, Bolsa, CCL, MEP, Tarjeta con precios de compra y venta
- **Riesgo País**: Índice actual con gráfico histórico
- **Inflación**: Tasa de inflación mensual Argentina
- **Fear & Greed Index**: Índice de sentimiento de mercado con clasificación visual
- **AI Daily Briefing**: Resumen diario generado por IA (si OpenRouter configurado)
- **Resumen de Commodities**: Precios de oro, petróleo, soja (si keys configuradas)

### 2. Mercados (`/markets`)

- **Watchlist**: Lista de símbolos seguidos con precios en tiempo real
- **Agregar Símbolo**: Búsqueda con validación en proveedor primario (Finnhub)
- **Gráficos Sparkline**: Tendencia de 7 días en lista
- **Detalle de Activo**: Pantalla completa con:
  - Gráfico de velas (TradingView-style)
  - Precio actual y cambio
  - Stats (high/low, volume, market cap, P/E)
  - Recomendaciones de analistas
  - Sentimiento insider
  - Calendario de earnings
  - Noticias de la empresa

### 3. Asistente IA (`/ai`)

- **Chat Interface**: Interfaz de chat estilo messenger
- **3 Personalidades**:
  - **Analista Financiero**: Objetivo, basado en datos, análisis serio
  - **Trader Directo**: Rápido, concreto, orientado a acción
  - **Profesor de Economía**: Educativo, explica desde cero
- **Contexto**: Puede analizar datos actuales de mercado
- **Historia Persistida**: Historial de chat guardado localmente
- **Comandos**: `/ayuda`, `/resumen`, `/analizar [símbolo]`

### 4. Noticias (`/news`)

- **Feed de Noticias**: Noticias financieras de Argentina y mundo
- **Filtros por Categoría**: Economía, Markets, Política, Empresas
- **Lector de Artículos**: Vista completa con Markdown
- **Compartir**: Compartir artículos externamente
- **RSS Feeds**: Fuentes alternativas (Infobae, Cronista, etc.)

### 5. Alertas (`/alerts`)

- **Crear Alerta**: Precio objetivo (arriba/abajo del actual)
- **Notificaciones Locales**: Aviso cuando se alcanza el objetivo
- **Gestión**: Habilitar/deshabilitar/editar/eliminar alertas
- **Historial**: Lista de alertas activadas

### 6. Configuración (`/settings`)

- **Gestión de API Keys**: Input seguro con validación de formato
- **Personalidad IA**: Selector de personalidad del asistente
- **Preferencias de Moneda**: Display ARS/USD
- **Export/Import Config**: Backup de preferencias, watchlist y keys
- **Información de App**: Versión, links, créditos

---

## Personalidades IA

### 1. Analista Financiero

- **Estilo**: Objetivo, preciso, basado en datos
- **Mejor para**: Análisis serio de mercado
- **Prompt**: Analista financiero senior especializado en mercados emergentes y economía argentina

### 2. Trader Directo

- **Estilo**: Directo, rápido, orientado a la acción
- **Mejor para**: Insights rápidos y señales
- **Prompt**: Trader con 10 años de experiencia en mercados latinoamericanos

### 3. Profesor de Economía

- **Estilo**: Educativo, explica desde cero
- **Mejor para**: Aprender sobre finanzas
- **Prompt**: Profesor de economía y finanzas personales con vocación didáctica

---

## Desarrollo

### Análisis de Código

```bash
# Analizar código
flutter analyze

# Analizar con FVM
fvm flutter analyze
```

### Tests

```bash
# Ejecutar tests
flutter test

# Ejecutar tests con coverage
flutter test --coverage
```

### Generador de Código

Después de modificar archivos con anotaciones:

```bash
# Freezed, Riverpod, Drift
dart run build_runner build --delete-conflicting-outputs

# Limpiar y regenerar si hay conflictos
flutter clean && flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

### Configuración de IDE

#### VS Code (.vscode/settings.json)

```json
{
  "dart.flutterSdkPath": ".fvm/flutter_sdk",
  "dart.lineLength": 100,
  "[dart]": {
    "editor.formatOnSave": true,
    "editor.rulers": [100],
    "editor.defaultFormatter": "Dart-Code.dart-code"
  }
}
```

### Reglas de Lint Activas

El proyecto usa `flutter_lints` con reglas adicionales:

- `prefer_const_constructors`: true
- `prefer_const_declarations`: true
- `prefer_final_fields`: true
- `prefer_final_locals`: true
- `avoid_print`: true (usar `debugPrint`)
- `use_super_parameters`: true
- `prefer_single_quotes`: true
- `require_trailing_commas`: true
- `always_use_package_imports`: true
- `avoid_dynamic_calls`: true

---

## Estructura del Proyecto

```
finvisor/
├── lib/
│   ├── main.dart                      # Entry point
│   ├── app.dart                       # Configuración de app
│   ├── core/
│   │   ├── constants/
│   │   │   ├── api_constants.dart     # URLs de APIs
│   │   │   ├── app_constants.dart     # Constantes de app
│   │   │   └── ai_personalities.dart  # Personalidades IA
│   │   ├── services/
│   │   │   ├── config_service.dart    # Export/Import config
│   │   │   └── notification_service.dart
│   │   ├── theme/
│   │   │   ├── app_theme.dart
│   │   │   ├── app_colors.dart
│   │   │   └── app_text_styles.dart
│   │   ├── router/
│   │   │   └── app_router.dart        # go_router config
│   │   └── utils/
│   │       ├── currency_formatter.dart
│   │       ├── date_formatter.dart
│   │       ├── number_formatter.dart
│   │       └── api_key_validators.dart
│   ├── data/
│   │   ├── datasources/
│   │   │   ├── api/
│   │   │   │   ├── core/
│   │   │   │   │   ├── dio_client.dart
│   │   │   │   │   └── stock_provider.dart
│   │   │   │   ├── argentina_datos_api.dart
│   │   │   │   ├── finnhub_api.dart
│   │   │   │   ├── fmp_api.dart
│   │   │   │   ├── alpha_vantage_api.dart
│   │   │   │   ├── coingecko_api.dart
│   │   │   │   ├── news_api.dart
│   │   │   │   ├── openrouter_api.dart
│   │   │   │   ├── fred_api.dart
│   │   │   │   ├── fear_greed_api.dart
│   │   │   │   ├── rss_feed_parser.dart
│   │   │   │   └── ...
│   │   │   └── local/
│   │   │       ├── database/
│   │   │       │   ├── app_database.dart
│   │   │       │   └── daos/
│   │   │       ├── secure_storage/
│   │   │       │   └── api_key_storage.dart
│   │   │       └── preferences/
│   │   │           └── app_preferences.dart
│   │   ├── models/
│   │   │   ├── dollar_quote_model.dart
│   │   │   ├── asset_price_model.dart
│   │   │   ├── news_article_model.dart
│   │   │   ├── macro_indicator_model.dart
│   │   │   ├── fear_greed_model.dart
│   │   │   └── ai_message_model.dart
│   │   └── repositories/
│   │       ├── argentina_repository_impl.dart
│   │       ├── market_repository_impl.dart
│   │       ├── stock_repository_impl.dart
│   │       ├── news_repository_impl.dart
│   │       ├── ai_repository_impl.dart
│   │       ├── fear_greed_repository_impl.dart
│   │       ├── commodity_repository_impl.dart
│   │       └── macro_repository_impl.dart
│   ├── domain/
│   │   ├── entities/
│   │   │   ├── dollar_quote.dart
│   │   │   ├── asset_price.dart
│   │   │   ├── news_article.dart
│   │   │   ├── price_alert.dart
│   │   │   ├── ai_message.dart
│   │   │   ├── fear_greed.dart
│   │   │   ├── earnings_calendar.dart
│   │   │   └── insider_sentiment.dart
│   │   ├── repositories/
│   │   │   ├── argentina_repository.dart
│   │   │   ├── market_repository.dart
│   │   │   ├── news_repository.dart
│   │   │   ├── ai_repository.dart
│   │   │   └── fear_greed_repository.dart
│   │   └── states/
│   │       └── data_state.dart
│   └── presentation/
│       ├── providers/
│       │   ├── core/
│       │   │   ├── dio_provider.dart
│       │   │   ├── database_provider.dart
│       │   │   └── api_key_storage_provider.dart
│       │   ├── argentina_providers.dart
│       │   ├── market_providers.dart
│       │   ├── stock_providers.dart
│       │   ├── news_providers.dart
│       │   ├── ai_providers.dart
│       │   ├── ai_briefing_provider.dart
│       │   ├── fear_greed_providers.dart
│       │   ├── commodity_providers.dart
│       │   ├── macro_providers.dart
│       │   └── preferences_provider.dart
│       ├── screens/
│       │   ├── splash/
│       │   ├── onboarding/
│       │   ├── dashboard/
│       │   ├── markets/
│       │   ├── ai/
│       │   ├── news/
│       │   ├── alerts/
│       │   └── settings/
│       └── widgets/
│           ├── dollar_quote_card.dart
│           ├── fear_greed_card.dart
│           ├── asset_price_card.dart
│           ├── news_card.dart
│           ├── commodity_carousel.dart
│           ├── tradingview_chart.dart
│           ├── price_target_widget.dart
│           ├── recommendation_trends_chart.dart
│           ├── earnings_calendar_widget.dart
│           ├── insider_sentiment_widget.dart
│           ├── loading_skeleton.dart
│           ├── empty_state.dart
│           ├── data_state_builder.dart
│           └── ...
├── assets/
│   ├── images/
│   ├── fonts/
│   └── js/
├── test/
│   ├── unit/
│   ├── widget/
│   └── helpers/
├── android/
├── ios/
├── linux/
├── windows/
├── web/
├── pubspec.yaml
├── analysis_options.yaml
├── .env
├── .gitignore
└── README.md
```

---

## Resolución de Problemas

### Issues Comunes

| Issue | Solución |
|-------|----------|
| NewsAPI no funciona | NewsAPI bloquea localhost. Usar device/emulator o debug APK |
| IA no responde | Verificar que OpenRouter tenga créditos en openrouter.ai |
| Símbolos no encontrados | Verificar que el símbolo exista en Finnhub (ej: AAPL, TSLA) |
| Build falla | `flutter clean && flutter pub get` |
| Errores de lints | `flutter analyze` para ver errores |
| Keys no se guardan | Verificar permisos de secure storage en Android |
| Error de Dio | Verificar conexión a internet y keys correctas |

### Debug Mode

Para habilitar logs de debug en desarrollo:

```dart
// Los debugPrint están activos por defecto en debug mode
// Para ver logs de Dio, los mensajes aparecen en debug console
```

### Errores Comunes de Build Runner

| Error | Causa | Solución |
|-------|-------|----------|
| `.g.dart` no generado | build_runner no corrió | `dart run build_runner build` |
| Conflictos de generación | Archivos corruptos | `flutter clean && flutter pub get` |
| DAO not found | DAO no incluido en database | Verificar `app_database.dart` |

---

## Licencia

MIT License

Copyright (c) 2024 FinVisor

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Disclaimer

FinVisor es solo una herramienta informativa. No proporciona asesoramiento financiero.
Todas las decisiones de inversión son responsabilidad del usuario. La app
agrega datos de APIs de terceros y no puede garantizar la exactitud o
integridad de la información mostrada.

---

## Contribuir

1. Fork el repositorio
2. Crear una rama (`git checkout -b feature/amazing-feature`)
3. Commitear cambios (`git commit -m 'Add amazing feature'`)
4. Push a la rama (`git push origin feature/amazing-feature`)
5. Abrir un Pull Request

---

## Contacto

- Website: [finvisor.app](https://finvisor.app)
- Issues: [GitHub Issues](https://github.com/your-repo/issues)

---

⭐️ Si te gusta el proyecto, no olvides dar una estrella en GitHub!
