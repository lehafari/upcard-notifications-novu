# Novu Integrations API Documentation

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Autenticación](#autenticación)
3. [URLs Base](#urls-base)
4. [Operaciones de Integraciones](#operaciones-de-integraciones)
   - [Obtener Integraciones](#obtener-integraciones)
   - [Crear Integración](#crear-integración)
   - [Actualizar Integración](#actualizar-integración)
   - [Eliminar Integración](#eliminar-integración)
   - [Configurar como Primaria](#configurar-como-primaria)
5. [Providers por Tipo de Canal](#providers-por-tipo-de-canal)
6. [Configuraciones Específicas](#configuraciones-específicas)
7. [Ejemplos Completos](#ejemplos-completos)
8. [Validación y Testing](#validación-y-testing)
9. [Manejo de Errores](#manejo-de-errores)

## Introducción

La API de Integraciones de Novu permite conectar múltiples proveedores de comunicación para enviar notificaciones a través de diferentes canales: Email, SMS, Push, Chat e In-App. Soporta más de 70 proveedores diferentes con configuraciones flexibles y validación automática.

### Características Principales

- **Multi-proveedor**: Más de 70 proveedores soportados
- **Multi-canal**: Email, SMS, Push, Chat, In-App
- **Credenciales seguras**: Encriptación automática de credenciales
- **Validación automática**: Testing de conectividad
- **Routing condicional**: Enrutamiento basado en condiciones
- **Configuración primaria**: Proveedores principales por canal

## Autenticación

Todas las requests requieren autenticación:

```bash
# Bearer Token
Authorization: Bearer <your-access-token>

# API Key
Authorization: ApiKey <your-api-key>
```

## URLs Base

- **API Base**: `https://api.novu.co/v1/integrations`

## Operaciones de Integraciones

### Obtener Integraciones

#### Listar todas las integraciones

```http
GET /v1/integrations
```

**Ejemplo de Request:**

```bash
curl -X GET "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>"
```

**Ejemplo de Response:**

```json
{
  "data": [
    {
      "_id": "integration_123",
      "_environmentId": "env_456",
      "_organizationId": "org_789",
      "name": "SendGrid Production",
      "identifier": "sendgrid-prod",
      "providerId": "sendgrid",
      "channel": "email",
      "credentials": {
        "apiKey": "SG.encrypted_key",
        "from": "noreply@company.com",
        "senderName": "Company Name"
      },
      "active": true,
      "deleted": false,
      "primary": true,
      "conditions": []
    },
    {
      "_id": "integration_124",
      "name": "Twilio SMS",
      "identifier": "twilio-sms",
      "providerId": "twilio",
      "channel": "sms",
      "credentials": {
        "accountSid": "AC_encrypted_sid",
        "authToken": "encrypted_token",
        "from": "+1234567890"
      },
      "active": true,
      "primary": true
    }
  ]
}
```

#### Listar integraciones activas

```http
GET /v1/integrations/active
```

```bash
curl -X GET "https://api.novu.co/v1/integrations/active" \
  -H "Authorization: Bearer <your-token>"
```

#### Verificar soporte de webhook

```http
GET /v1/integrations/webhook/provider/{providerOrIntegrationId}/status
```

```bash
curl -X GET "https://api.novu.co/v1/integrations/webhook/provider/sendgrid/status" \
  -H "Authorization: Bearer <your-token>"
```

### Crear Integración

```http
POST /v1/integrations
```

**Body Parameters:**

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| `providerId` | string | ID del proveedor | ✅ |
| `channel` | string | Tipo de canal (`email`, `sms`, `push`, `chat`, `in_app`) | ✅ |
| `name` | string | Nombre de la integración | ❌ |
| `identifier` | string | Identificador único | ❌ |
| `credentials` | object | Credenciales del proveedor | ❌ |
| `active` | boolean | Estado activo/inactivo | ❌ |
| `check` | boolean | Validar credenciales | ❌ |
| `conditions` | array | Condiciones de enrutamiento | ❌ |

**Ejemplo de Request:**

```bash
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "sendgrid",
    "channel": "email",
    "name": "SendGrid Production",
    "credentials": {
      "apiKey": "SG.your_api_key_here",
      "from": "noreply@yourcompany.com",
      "senderName": "Your Company Name"
    },
    "active": true,
    "check": true
  }'
```

### Actualizar Integración

```http
PUT /v1/integrations/{integrationId}
```

**Ejemplo de Request:**

```bash
curl -X PUT "https://api.novu.co/v1/integrations/integration_123" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "credentials": {
      "apiKey": "SG.new_api_key",
      "from": "noreply@newdomain.com"
    },
    "active": false
  }'
```

### Eliminar Integración

```http
DELETE /v1/integrations/{integrationId}
```

**Ejemplo:**

```bash
curl -X DELETE "https://api.novu.co/v1/integrations/integration_123" \
  -H "Authorization: Bearer <your-token>"
```

### Configurar como Primaria

```http
POST /v1/integrations/{integrationId}/set-primary
```

**Ejemplo:**

```bash
curl -X POST "https://api.novu.co/v1/integrations/integration_123/set-primary" \
  -H "Authorization: Bearer <your-token>"
```

## Providers por Tipo de Canal

### Email Providers (19 disponibles)

| Proveedor | Provider ID | Credenciales Principales |
|-----------|-------------|-------------------------|
| **SendGrid** | `sendgrid` | `apiKey`, `from`, `senderName` |
| **Mailgun** | `mailgun` | `apiKey`, `user`, `domain`, `from` |
| **Mailjet** | `mailjet` | `apiKey`, `secretKey`, `from` |
| **Postmark** | `postmark` | `apiKey`, `from`, `senderName` |
| **AWS SES** | `ses` | `accessKey`, `secretKey`, `region`, `from` |
| **Resend** | `resend` | `apiKey`, `from`, `senderName` |
| **Brevo** | `brevo` | `apiKey`, `from`, `senderName` |
| **Custom SMTP** | `nodemailer` | `host`, `port`, `user`, `password` |
| **Mailtrap** | `mailtrap` | `apiKey`, `from`, `senderName` |
| **Mandrill** | `mandrill` | `apiKey`, `from`, `senderName` |
| **Outlook 365** | `outlook365` | `password`, `from`, `senderName` |
| **MailerSend** | `mailersend` | `apiKey`, `from`, `senderName` |
| **Infobip** | `infobip` | `apiKey`, `baseUrl`, `from` |
| **SparkPost** | `sparkpost` | `apiKey`, `region`, `from` |
| **Netcore** | `netcore` | `apiKey`, `from`, `senderName` |
| **Plunk** | `plunk` | `apiKey`, `from`, `senderName` |
| **Email Webhook** | `email-webhook` | `webhookUrl`, `hmacSecretKey` |
| **Braze** | `braze` | `apiKey`, `baseUrl`, `appId` |
| **Novu Email** | `novu-email` | Sin credenciales (default) |

### SMS Providers (30+ disponibles)

| Proveedor | Provider ID | Credenciales Principales |
|-----------|-------------|-------------------------|
| **Twilio** | `twilio` | `accountSid`, `authToken`, `from` |
| **Nexmo/Vonage** | `nexmo` | `apiKey`, `secretKey`, `from` |
| **AWS SNS** | `sns` | `accessKey`, `secretKey`, `region` |
| **Plivo** | `plivo` | `authId`, `authToken`, `from` |
| **MessageBird** | `messagebird` | `accessKey`, `from` |
| **sms77** | `sms77` | `apiKey`, `from` |
| **Telnyx** | `telnyx` | `apiKey`, `messageProfileId`, `from` |
| **Gupshup** | `gupshup` | `userid`, `password`, `from` |
| **Firetext** | `firetext` | `apiKey`, `from` |
| **Infobip** | `infobip` | `apiKey`, `baseUrl`, `from` |
| **BurstSMS** | `burst-sms` | `apiKey`, `secretKey`, `from` |
| **Africa's Talking** | `africas-talking` | `apiKey`, `username`, `from` |
| **Termii** | `termii` | `apiKey`, `from` |
| **46elks** | `forty-six-elks` | `username`, `password`, `from` |
| **Clickatell** | `clickatell` | `apiKey`, `from` |
| **SMS Central** | `sms-central` | `username`, `password`, `from` |
| **Maqsam** | `maqsam` | `accessKeyId`, `accessSecret`, `from` |
| **iSend SMS** | `isend-sms` | `apiToken`, `from` |
| **BulkSMS** | `bulksms` | `apiToken`, `from` |
| **Sendchamp** | `sendchamp` | `apiKey`, `from` |
| **Brevo SMS** | `brevo-sms` | `apiKey`, `from` |
| **Azure SMS** | `azure-sms` | `connectionString`, `from` |
| **RingCentral** | `ring-central` | `clientId`, `clientSecret`, `jwtToken` |
| **Generic SMS** | `generic-sms` | `baseUrl`, `apiKeyRequestHeader`, `apiKey` |
| **Novu SMS** | `novu-sms` | Sin credenciales (default) |

### Push Providers (7 disponibles)

| Proveedor | Provider ID | Credenciales Principales |
|-----------|-------------|-------------------------|
| **Firebase FCM** | `fcm` | `serviceAccount` (JSON completo) |
| **Apple APNs** | `apns` | `privateKey`, `keyId`, `teamId`, `bundleId` |
| **OneSignal** | `one-signal` | `applicationId`, `apiKey` |
| **Expo Push** | `expo` | `accessToken` |
| **Pushpad** | `pushpad` | `authToken`, `projectId` |
| **Pusher Beams** | `pusher-beams` | `instanceId`, `secretKey` |
| **Push Webhook** | `push-webhook` | `webhookUrl`, `hmacSecretKey` |

### Chat Providers (10 disponibles)

| Proveedor | Provider ID | Credenciales Principales |
|-----------|-------------|-------------------------|
| **Slack** | `slack` | `applicationId`, `clientId`, `clientSecret` |
| **Discord** | `discord` | `webhookUrl` |
| **Microsoft Teams** | `msteams` | `webhookUrl` |
| **Mattermost** | `mattermost` | `webhookUrl` |
| **WhatsApp Business** | `whatsapp-business` | `accessToken`, `phoneNumberId` |
| **Rocket.Chat** | `rocket-chat` | `token`, `userId` |
| **GetStream** | `getstream` | `apiKey` |
| **Zulip** | `zulip` | `webhookUrl` |
| **Ryver** | `ryver` | `webhookUrl` |
| **Chat Webhook** | `chat-webhook` | `webhookUrl`, `hmacSecretKey` |

### In-App Providers (1 disponible)

| Proveedor | Provider ID | Credenciales Principales |
|-----------|-------------|-------------------------|
| **Novu In-App** | `novu` | `hmac` (opcional, para seguridad) |

## Configuraciones Específicas

### SendGrid Email

```json
{
  "providerId": "sendgrid",
  "channel": "email",
  "name": "SendGrid Production",
  "credentials": {
    "apiKey": "SG.your_api_key_here",
    "from": "noreply@yourcompany.com",
    "senderName": "Your Company Name",
    "ipPoolName": "your_ip_pool_name"
  },
  "active": true
}
```

### Twilio SMS

```json
{
  "providerId": "twilio",
  "channel": "sms",
  "name": "Twilio SMS",
  "credentials": {
    "accountSid": "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "authToken": "your_auth_token_here",
    "from": "+1234567890"
  },
  "active": true
}
```

### Custom SMTP

```json
{
  "providerId": "nodemailer",
  "channel": "email",
  "name": "Custom SMTP Server",
  "credentials": {
    "host": "smtp.yourcompany.com",
    "port": 587,
    "secure": false,
    "requireTls": true,
    "user": "smtp_user",
    "password": "smtp_password",
    "from": "system@yourcompany.com",
    "senderName": "Your System"
  },
  "active": true
}
```

### Firebase FCM Push

```json
{
  "providerId": "fcm",
  "channel": "push",
  "name": "Firebase Push",
  "credentials": {
    "serviceAccount": "{\"type\":\"service_account\",\"project_id\":\"your-project\",\"private_key_id\":\"key-id\",\"private_key\":\"-----BEGIN PRIVATE KEY-----\\n...\\n-----END PRIVATE KEY-----\\n\",\"client_email\":\"firebase-adminsdk@your-project.iam.gserviceaccount.com\"}"
  },
  "active": true
}
```

### Slack Chat

```json
{
  "providerId": "slack",
  "channel": "chat",
  "name": "Slack Integration",
  "credentials": {
    "applicationId": "A1234567890",
    "clientId": "1234567890.1234567890",
    "clientSecret": "your_client_secret_here",
    "redirectUrl": "https://yourapp.com/auth/slack/callback"
  },
  "active": true
}
```

## Ejemplos Completos

### Configuración Multi-Proveedor para Email

```bash
# Crear SendGrid como proveedor primario
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "sendgrid",
    "channel": "email",
    "name": "SendGrid Primary",
    "credentials": {
      "apiKey": "SG.primary_key",
      "from": "noreply@company.com",
      "senderName": "Company"
    },
    "active": true
  }'

# Crear Mailgun como proveedor de respaldo
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "mailgun",
    "channel": "email",
    "name": "Mailgun Backup",
    "credentials": {
      "apiKey": "key-mailgun_api_key",
      "user": "api",
      "domain": "mg.company.com",
      "from": "backup@company.com"
    },
    "active": true
  }'
```

### Integración con Routing Condicional

```bash
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "postmark",
    "channel": "email",
    "name": "Postmark Enterprise",
    "credentials": {
      "apiKey": "your-postmark-api-key",
      "from": "enterprise@company.com",
      "senderName": "Enterprise Support"
    },
    "conditions": [{
      "children": [{
        "field": "identifier",
        "value": "enterprise",
        "operator": "EQUAL",
        "on": "tenant"
      }]
    }],
    "active": true
  }'
```

### Setup Multi-Canal Completo

```bash
# Email con SendGrid
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "sendgrid",
    "channel": "email",
    "credentials": {
      "apiKey": "SG.your_key",
      "from": "noreply@company.com",
      "senderName": "Company"
    },
    "active": true
  }'

# SMS con Twilio
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "twilio",
    "channel": "sms",
    "credentials": {
      "accountSid": "AC_your_sid",
      "authToken": "your_token",
      "from": "+1234567890"
    },
    "active": true
  }'

# Push con FCM
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "fcm",
    "channel": "push",
    "credentials": {
      "serviceAccount": "{\\"type\\":\\"service_account\\",\\"project_id\\":\\"your-project\\",...}"
    },
    "active": true
  }'

# Chat con Slack
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "slack",
    "channel": "chat",
    "credentials": {
      "applicationId": "A1234567890",
      "clientId": "1234567890.1234567890",
      "clientSecret": "your_secret"
    },
    "active": true
  }'
```

## Validación y Testing

### Validar Credenciales durante Creación

```bash
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "sendgrid",
    "channel": "email",
    "credentials": {
      "apiKey": "SG.test_key",
      "from": "test@company.com"
    },
    "check": true,
    "active": false
  }'
```

### Verificar Estado de Integración

```bash
# Verificar integraciones activas
curl -X GET "https://api.novu.co/v1/integrations/active" \
  -H "Authorization: Bearer <your-token>"

# Verificar soporte de webhook
curl -X GET "https://api.novu.co/v1/integrations/webhook/provider/sendgrid/status" \
  -H "Authorization: Bearer <your-token>"
```

### Testing de Conectividad

```bash
# Crear integración con validación automática
curl -X POST "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "providerId": "mailgun",
    "channel": "email",
    "credentials": {
      "apiKey": "key-test",
      "user": "api",
      "domain": "sandbox123.mailgun.org",
      "from": "test@sandbox123.mailgun.org"
    },
    "check": true,
    "active": true
  }'
```

## Operaciones Avanzadas

### Configurar Proveedor Primario

```bash
# Obtener ID de la integración
curl -X GET "https://api.novu.co/v1/integrations" \
  -H "Authorization: Bearer <your-token>"

# Configurar como primaria
curl -X POST "https://api.novu.co/v1/integrations/integration_123/set-primary" \
  -H "Authorization: Bearer <your-token>"
```

### Obtener Límites por Canal

```bash
curl -X GET "https://api.novu.co/v1/integrations/email/limit" \
  -H "Authorization: Bearer <your-token>"
```

### Verificar Estado In-App

```bash
curl -X GET "https://api.novu.co/v1/integrations/in-app/status" \
  -H "Authorization: Bearer <your-token>"
```

## Manejo de Errores

### Códigos de Error Comunes

| Código | Descripción | Solución |
|--------|-------------|----------|
| `400` | Bad Request - Credenciales inválidas | Verificar formato de credenciales |
| `401` | Unauthorized - Token inválido | Verificar token de autenticación |
| `403` | Forbidden - Sin permisos | Verificar permisos INTEGRATION_WRITE |
| `404` | Not Found - Integración no encontrada | Verificar ID de integración |
| `409` | Conflict - Identificador duplicado | Usar identificador único |
| `422` | Validation Error - Provider inválido | Verificar providerId y credenciales |

### Ejemplos de Errores de Validación

```json
{
  "message": "Provider validation failed",
  "error": "Unprocessable Entity",
  "statusCode": 422,
  "data": [
    {
      "field": "credentials.apiKey",
      "message": "API Key is required for SendGrid"
    },
    {
      "field": "credentials.from",
      "message": "From email address is required"
    }
  ]
}
```

### Error de Conectividad

```json
{
  "message": "Integration test failed",
  "error": "Service Unavailable",
  "statusCode": 503,
  "data": {
    "provider": "sendgrid",
    "error": "ENOTFOUND: hostname not found",
    "details": "Unable to connect to SendGrid API"
  }
}
```

### Manejo de Errores en JavaScript

```javascript
try {
  const integration = await createIntegration({
    providerId: 'sendgrid',
    channel: 'email',
    credentials: {
      apiKey: 'SG.invalid_key',
      from: 'test@company.com'
    },
    check: true
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation errors:', error.response.data.data);
  } else if (error.response?.status === 503) {
    console.error('Connectivity error:', error.response.data.details);
  } else {
    console.error('Unexpected error:', error.message);
  }
}
```

## Límites y Consideraciones

### Rate Limits

- **Creación de integraciones**: 50 requests/hora
- **Validación de credenciales**: 100 requests/hora
- **Consultas (GET)**: 500 requests/minuto

### Límites por Canal

- **Email**: Máximo 10 proveedores activos por ambiente
- **SMS**: Máximo 10 proveedores activos por ambiente
- **Push**: Máximo 5 proveedores activos por ambiente
- **Chat**: Máximo 20 proveedores activos por ambiente
- **In-App**: Máximo 1 proveedor por ambiente

### Consideraciones de Seguridad

1. **Credenciales**: Todas las credenciales se encriptan automáticamente
2. **Permisos**: Requiere permisos específicos para lectura/escritura
3. **HMAC**: Webhooks soportan verificación HMAC
4. **Testing**: Usar ambientes de desarrollo para testing
5. **Rotación**: Rotar credenciales periódicamente

### Mejores Prácticas

1. **Naming**: Usar nombres descriptivos para identificar fácilmente
2. **Testing**: Siempre validar credenciales con `check: true`
3. **Backup**: Configurar múltiples proveedores por canal crítico
4. **Monitoring**: Monitorear estado de integraciones activas
5. **Conditions**: Usar routing condicional para casos específicos
6. **Primary**: Configurar proveedores primarios para canales principales

Esta documentación cubre todas las operaciones principales para trabajar con integraciones en la API de Novu. Para más información sobre proveedores específicos, consultar la [documentación oficial](https://docs.novu.co/integrations/providers/) de cada proveedor.