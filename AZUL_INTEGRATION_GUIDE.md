# Integración de Pasarela de Pago Azul - Guía de Migración

Esta documentación describe los archivos modificados y creados para la integración de la pasarela de pago Azul, diseñada para ser fácilmente migrable a versiones futuras del sistema.

## Archivos Creados (Copiar en nueva versión)

### 1. Configuración
- `config/custom_payments.php` - Configuración de rutas personalizadas

### 2. Rutas
- `routes/custom_payments.php` - Rutas específicas para pagos personalizados

### 3. Service Provider
- `app/Providers/CustomPaymentServiceProvider.php` - Provider para cargar configuraciones y rutas personalizadas

### 4. Controlador
- `app/Http/Controllers/AzulController.php` - Controlador principal para Azul

### 5. Vistas
- `resources/views/payment-views/azul-pay.blade.php` - Formulario de pago
- `resources/views/payment-views/azul-success.blade.php` - Página de éxito
- `resources/views/payment-views/azul-cancel.blade.php` - Página de cancelación

## Archivos Modificados (Aplicar cambios manualmente)

### 1. app/Traits/Payment.php
**Cambio realizado:**
```php
// Agregar después del array $routes existente:
// Agregar rutas personalizadas desde configuración
$custom_routes = config('custom_payments.custom_routes', []);
$routes = array_merge($routes, $custom_routes);
```

### 2. config/app.php
**Cambio realizado:**
```php
// Agregar en el array 'providers':
App\Providers\CustomPaymentServiceProvider::class,
```

### 3. app/Http/Controllers/Admin/BusinessSettingsController.php
**⚠️ CRÍTICO: Cambios realizados para que Azul aparezca en la interfaz de administración:**

En el método `payment_index()` (alrededor de la línea 852):
```php
// Buscar esta línea y agregar 'azul' al final:
$payment_published_status = addon_published_status('Gateways');
$payment_gateways = $this->addon_settings->whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

En el método `payment_config_update()` (alrededor de la línea 869):
```php
// Buscar la validación de gateway y agregar 'azul':
'gateway' => 'required|in:ssl_commerz,paypal,stripe,razor_pay,senang_pay,paystack,paymob_accept,flutterwave,paytm,liqpay,mercadopago,bkash,sixcash,esewa,foree,xendit,iyziPay,azul'
```

### 4. app/CentralLogics/helpers.php
**⚠️ CRÍTICO: Cambio realizado para que Azul aparezca en funciones helper del sistema:**

En la función `getActivePaymentGateways()` (alrededor de la línea 4272):
```php
// Buscar esta línea y agregar 'azul' al final del array:
$payment_gateways = $this->addon_settings->whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

### 5. app/Http/Controllers/Api/V1/ConfigController.php
**⚠️ CRÍTICO: Cambio realizado para que Azul aparezca en la API de configuración:**

En el método `getDefaultPaymentMethods()` (alrededor de la línea 795):
```php
// Buscar esta línea y agregar 'azul' al final del array:
$default_payment_methods = AddonSetting::whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

### 6. app/Http/Controllers/Vendor/WalletController.php
**⚠️ CRÍTICO: Cambio realizado para que Azul aparezca en el controlador de billeteras:**

En el método correspondiente (alrededor de la línea 267):
```php
// Buscar esta línea y agregar 'azul' al final del array:
$q->whereIn('key_name', ['ssl_commerz','paypal','stripe','razor_pay','senang_pay','paytabs','paystack','paymob_accept','paytm','flutterwave','liqpay','bkash','mercadopago','azul']);
```

## Migración de Base de Datos

### Archivo de Migración
- `database/migrations/2024_12_28_000000_add_azul_payment_to_addon_settings.php` - Migración para insertar Azul en addon_settings

## Pasos para Migración a Nueva Versión

### ⚠️ IMPORTANTE: Identificación del Problema Principal

Durante la implementación se identificó que el sistema tiene múltiples listas restrictivas que determinan qué métodos de pago se muestran en diferentes partes del sistema. **Sin estas modificaciones, Azul no aparecerá en la interfaz de administración aunque esté en la base de datos.**

Los 5 archivos críticos que deben modificarse son:
1. `app/Http/Controllers/Admin/BusinessSettingsController.php` - Para que aparezca en la interfaz de admin
2. `app/CentralLogics/helpers.php` - Para funciones helper del sistema
3. `app/Http/Controllers/Api/V1/ConfigController.php` - Para la API
4. `app/Http/Controllers/Vendor/WalletController.php` - Para el controlador de billeteras
5. `app/Traits/Payment.php` - Para soporte de rutas personalizadas

### 1. Copia de Archivos
```bash
# Crear directorios si no existen
mkdir -p config routes app/Providers app/Http/Controllers resources/views/payment-views

# Copiar archivos personalizados
cp config/custom_payments.php [nueva_version]/config/
cp routes/custom_payments.php [nueva_version]/routes/
cp app/Providers/CustomPaymentServiceProvider.php [nueva_version]/app/Providers/
cp app/Http/Controllers/AzulController.php [nueva_version]/app/Http/Controllers/
cp resources/views/payment-views/azul-*.blade.php [nueva_version]/resources/views/payment-views/
```

### 2. Modificaciones Manuales (⚠️ CRÍTICAS)

#### En app/Http/Controllers/Admin/BusinessSettingsController.php:
Buscar la línea que contiene `whereIn('key_name',` en el método `payment_index()` y agregar 'azul':
```php
$payment_gateways = $this->addon_settings->whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

Buscar la validación en el método `payment_config_update()` y agregar 'azul':
```php
'gateway' => 'required|in:ssl_commerz,paypal,stripe,razor_pay,senang_pay,paystack,paymob_accept,flutterwave,paytm,liqpay,mercadopago,bkash,sixcash,esewa,foree,xendit,iyziPay,azul'
```

#### En app/CentralLogics/helpers.php:
Buscar la función `getActivePaymentGateways()` y agregar 'azul' al whereIn:
```php
$payment_gateways = $this->addon_settings->whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

#### En app/Http/Controllers/Api/V1/ConfigController.php:
Buscar el método `getDefaultPaymentMethods()` y agregar 'azul' al whereIn:
```php
$default_payment_methods = AddonSetting::whereIn('key_name', ['ssl_commerz', 'paypal', 'stripe', 'razor_pay', 'senang_pay', 'paystack', 'flutterwave', 'paymob_accept', 'paytm', 'liqpay', 'mercadopago', 'bkash', 'sixcash', 'esewa', 'foree', 'xendit', 'iyziPay', 'azul'])->get();
```

#### En app/Http/Controllers/Vendor/WalletController.php:
Buscar la función que contiene `whereIn('key_name',` y agregar 'azul':
```php
$q->whereIn('key_name', ['ssl_commerz','paypal','stripe','razor_pay','senang_pay','paytabs','paystack','paymob_accept','paytm','flutterwave','liqpay','bkash','mercadopago','azul']);
```

#### En app/Traits/Payment.php:
Buscar el array `$routes` y agregar después de su cierre:
```php
// Agregar rutas personalizadas desde configuración
$custom_routes = config('custom_payments.custom_routes', []);
$routes = array_merge($routes, $custom_routes);
```

#### En config/app.php:
Agregar en el array 'providers':
```php
App\Providers\CustomPaymentServiceProvider::class,
```

### 3. Configuración de Base de Datos

Ejecutar la migración o insertar manualmente en la tabla `addon_settings`:
```sql
INSERT INTO addon_settings (id, key_name, live_values, test_values, settings_type, mode, is_active, created_at, updated_at) 
VALUES (
    NULL,
    'azul',
    '{"merchant_id": "", "api_key": "", "secret_key": "", "endpoint": "https://api.azul.com.do/v1/"}',
    '{"merchant_id": "", "api_key": "", "secret_key": "", "endpoint": "https://pruebas.azul.com.do/v1/"}',
    'payment_config',
    'test',
    1,
    NOW(),
    NOW()
);
```

### 4. Verificación
Después de completar todos los pasos:
1. Verificar que Azul aparece en `/admin/business-settings/payment-method`
2. Verificar que las rutas personalizadas funcionan correctamente
3. Probar el flujo completo de pago con Azul

## Configuración de Azul

### Variables de Configuración Requeridas
```php
// En la base de datos, tabla payment_config
'live_values' => {
    "merchant_id": "tu_merchant_id",
    "api_key": "tu_api_key_live",
    "secret_key": "tu_secret_key_live",
    "endpoint": "https://api.azul.com.do/v1/"
}

'test_values' => {
    "merchant_id": "test_merchant_id",
    "api_key": "tu_api_key_test", 
    "secret_key": "tu_secret_key_test",
    "endpoint": "https://pruebas.azul.com.do/v1/"
}
```

## Checklist de Verificación Post-Migración

### ✅ Lista de Verificación Obligatoria

**1. Archivos Copiados:**
- [ ] `config/custom_payments.php` - Existe y contiene configuración de rutas
- [ ] `routes/custom_payments.php` - Existe y contiene rutas de Azul
- [ ] `app/Providers/CustomPaymentServiceProvider.php` - Existe y está registrado
- [ ] `app/Http/Controllers/AzulController.php` - Existe y tiene todos los métodos
- [ ] `resources/views/payment-views/azul-*.blade.php` - Las 3 vistas existen

**5. Modificaciones Realizadas:**
- [ ] `app/Traits/Payment.php` - Contiene el código para rutas personalizadas
- [ ] `config/app.php` - CustomPaymentServiceProvider registrado en providers
- [ ] `app/Http/Controllers/Admin/BusinessSettingsController.php` - 'azul' agregado en ambos métodos
- [ ] `app/CentralLogics/helpers.php` - 'azul' agregado en getActivePaymentGateways()
- [ ] `app/Http/Controllers/Api/V1/ConfigController.php` - 'azul' agregado en getDefaultPaymentMethods()
- [ ] `app/Http/Controllers/Vendor/WalletController.php` - 'azul' agregado en whereIn query

**3. Base de Datos:**
- [ ] Registro de Azul existe en tabla `addon_settings`
- [ ] `key_name` = 'azul'
- [ ] `is_active` = 1
- [ ] `settings_type` = 'payment_config'

**4. Verificación Funcional:**
- [ ] Azul aparece en `/admin/business-settings/payment-method`
- [ ] Se puede configurar Azul desde la interfaz de administración
- [ ] Las rutas de Azul responden correctamente
- [ ] El flujo de pago funciona sin errores

### 🚨 Síntomas de Problemas Comunes

**Si Azul no aparece en la interfaz de administración:**
- Verificar que 'azul' esté en `BusinessSettingsController.php` (líneas 852 y 869)
- Verificar que la migración se ejecutó correctamente
- Verificar que `is_active = 1` en la base de datos

**Si las rutas de Azul no funcionan:**
- Verificar que `CustomPaymentServiceProvider` esté registrado en `config/app.php`
- Verificar que el trait `Payment.php` tenga el código de rutas personalizadas
- Ejecutar `php artisan route:list` para verificar que las rutas estén registradas

**Si Azul no aparece en la API:**
- Verificar que 'azul' esté en `ConfigController.php` (línea 795)
- Verificar que 'azul' esté en `helpers.php` (línea 4272)

## Ventajas de esta Implementación

1. **Modular**: Todos los archivos de Azul están separados del código base
2. **Fácil Migración**: Solo requiere copiar archivos y hacer 2 modificaciones menores
3. **Escalable**: Fácil agregar más pasarelas usando el mismo patrón
4. **No Invasivo**: Cambios mínimos al código original

## Notas Importantes

- La implementación actual incluye una simulación del procesamiento de Azul
- Para producción, implementar la integración real con la API de Azul
- Verificar las rutas y vistas con el layout específico del proyecto
- Adaptar la validación y procesamiento según documentación oficial de Azul

## Para Implementación Real de Azul

1. Obtener credenciales de Azul (merchant_id, api_key, secret_key)
2. Revisar documentación oficial de Azul
3. Implementar llamadas reales a la API en el método `verify_payment()`
4. Configurar webhooks si Azul los soporta
5. Implementar manejo de errores específicos de Azul

## Contacto y Soporte

Para modificaciones o mejoras a esta integración, consultar este documento y los archivos relacionados.
