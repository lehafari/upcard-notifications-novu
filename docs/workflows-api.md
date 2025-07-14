# Novu Workflows API Documentation

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Autenticación](#autenticación)
3. [URLs Base](#urls-base)
4. [Operaciones de Workflows](#operaciones-de-workflows)
   - [Obtener Workflows](#obtener-workflows)
   - [Crear Workflow](#crear-workflow)
   - [Actualizar Workflow](#actualizar-workflow)
   - [Eliminar Workflow](#eliminar-workflow)
   - [Disparar Workflow](#disparar-workflow)
5. [Tipos de Steps](#tipos-de-steps)
6. [Ejemplos Completos](#ejemplos-completos)
7. [Manejo de Errores](#manejo-de-errores)

## Introducción

La API de Workflows v2 de Novu permite crear, gestionar y ejecutar flujos de notificaciones multi-canal. Un workflow es una secuencia de pasos que pueden incluir emails, SMS, notificaciones push, in-app, chat, delays y digest.

### Características Principales

- **Multi-canal**: Email, SMS, Push, In-App, Chat
- **Templating**: Variables dinámicas con sintaxis Handlebars
- **Lógica condicional**: Condiciones de skip para pasos
- **Validación de payload**: JSON Schema validation
- **Sincronización de ambientes**: Deploy entre entornos
- **Duplicación**: Clonado de workflows existentes

## Autenticación

Todas las requests requieren autenticación mediante Bearer token o API Key:

```bash
# Bearer Token
Authorization: Bearer <your-access-token>

# API Key
Authorization: ApiKey <your-api-key>
```

## URLs Base

- **API v2 (Recomendada)**: `https://api.novu.co/v2/workflows`
- **API v1 (Deprecada)**: `https://api.novu.co/v1/workflows`

## Operaciones de Workflows

### Obtener Workflows

#### Listar todos los workflows

```http
GET /v2/workflows
```

**Parámetros de Query:**

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `limit` | number | Número de workflows por página (1-100) | ✅ |
| `offset` | number | Número de workflows a omitir | ✅ |
| `query` | string | Búsqueda por nombre o descripción | ✅ |
| `orderBy` | string | Campo para ordenar | ❌ |
| `orderDirection` | string | `ASC` o `DESC` | ❌ |
| `tags[]` | string[] | Filtrar por tags | ❌ |
| `status[]` | string[] | Filtrar por estado (`ACTIVE`, `INACTIVE`) | ❌ |

**Ejemplo de Request:**

```bash
curl -X GET "https://api.novu.co/v2/workflows?limit=10&offset=0&query=&orderBy=createdAt&orderDirection=DESC" \
  -H "Authorization: Bearer <your-token>"
```

**Ejemplo de Response:**

```json
{
  "data": {
    "workflows": [
      {
        "_id": "workflow_123",
        "workflowId": "welcome-email",
        "name": "Welcome Email Workflow",
        "description": "Send welcome email to new users",
        "active": true,
        "tags": ["onboarding", "email"],
        "status": "ACTIVE",
        "origin": "novu-cloud",
        "createdAt": "2024-01-15T10:30:00Z",
        "updatedAt": "2024-01-15T15:45:00Z",
        "steps": [
          {
            "_id": "step_456",
            "name": "Welcome Email",
            "type": "email",
            "controlValues": {
              "subject": "Welcome to {{payload.companyName}}!",
              "body": "Welcome {{payload.firstName}}!",
              "editorType": "block"
            }
          }
        ],
        "preferences": {
          "all": {
            "enabled": true,
            "channels": {
              "email": true,
              "in_app": true,
              "sms": false,
              "push": false,
              "chat": false
            }
          }
        }
      }
    ],
    "totalCount": 25,
    "hasMore": true
  }
}
```

#### Obtener un workflow específico

```http
GET /v2/workflows/{workflowSlug}
```

**Parámetros:**

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `workflowSlug` | string | ID o slug del workflow | ✅ |
| `environmentId` | string | ID del ambiente target | ❌ |

**Ejemplo:**

```bash
curl -X GET "https://api.novu.co/v2/workflows/welcome-email" \
  -H "Authorization: Bearer <your-token>"
```

### Crear Workflow

```http
POST /v2/workflows
```

**Body Parameters:**

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| `name` | string | Nombre del workflow | ✅ |
| `workflowId` | string | Identificador único | ✅ |
| `description` | string | Descripción del workflow | ❌ |
| `active` | boolean | Estado activo/inactivo | ❌ |
| `tags` | string[] | Tags para categorización | ❌ |
| `steps` | StepDto[] | Array de pasos del workflow | ✅ |
| `preferences` | object | Configuración de preferencias | ❌ |
| `payloadSchema` | object | JSON Schema para validación | ❌ |
| `validatePayload` | boolean | Habilitar validación de payload | ❌ |

**Ejemplo de Request:**

```bash
curl -X POST "https://api.novu.co/v2/workflows" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "User Onboarding",
    "workflowId": "user-onboarding-v2",
    "description": "Complete user onboarding workflow",
    "active": true,
    "tags": ["onboarding", "new-user"],
    "steps": [
      {
        "name": "Welcome Email",
        "type": "email",
        "controlValues": {
          "subject": "Welcome to {{payload.companyName}}, {{payload.firstName}}!",
          "body": "<h1>Welcome!</h1><p>Hi {{payload.firstName}}, thank you for joining {{payload.companyName}}.</p>",
          "editorType": "html"
        }
      },
      {
        "name": "Delay for 1 day",
        "type": "delay",
        "controlValues": {
          "amount": 1,
          "unit": "days"
        }
      },
      {
        "name": "Follow-up In-App",
        "type": "in_app",
        "controlValues": {
          "subject": "Complete your profile",
          "body": "Please complete your profile to get started",
          "primaryAction": {
            "label": "Complete Profile",
            "redirect": {
              "url": "/profile/complete",
              "target": "_self"
            }
          }
        }
      }
    ],
    "preferences": {
      "all": {
        "enabled": true,
        "channels": {
          "email": true,
          "in_app": true,
          "sms": false,
          "push": false,
          "chat": false
        }
      }
    },
    "payloadSchema": {
      "type": "object",
      "properties": {
        "firstName": { "type": "string" },
        "companyName": { "type": "string" },
        "userId": { "type": "string" }
      },
      "required": ["firstName", "companyName", "userId"]
    },
    "validatePayload": true
  }'
```

### Actualizar Workflow

```http
PUT /v2/workflows/{workflowSlug}
```

**Ejemplo de Request:**

```bash
curl -X PUT "https://api.novu.co/v2/workflows/user-onboarding-v2" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "User Onboarding - Updated",
    "description": "Updated user onboarding workflow with new steps",
    "active": true,
    "tags": ["onboarding", "new-user", "updated"],
    "steps": [
      {
        "name": "Welcome Email",
        "type": "email",
        "controlValues": {
          "subject": "Welcome to {{payload.companyName}}, {{payload.firstName}}!",
          "body": "<h1>Welcome!</h1><p>Updated welcome message for {{payload.firstName}}.</p>",
          "editorType": "html"
        }
      }
    ],
    "preferences": {
      "all": {
        "enabled": true,
        "channels": {
          "email": true,
          "in_app": true,
          "sms": true,
          "push": false,
          "chat": false
        }
      }
    },
    "origin": "novu-cloud"
  }'
```

#### Actualización Parcial (PATCH)

```http
PATCH /v2/workflows/{workflowSlug}
```

**Ejemplo - Solo cambiar estado:**

```bash
curl -X PATCH "https://api.novu.co/v2/workflows/user-onboarding-v2" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "active": false,
    "description": "Temporarily disabled for maintenance"
  }'
```

### Eliminar Workflow

```http
DELETE /v2/workflows/{workflowSlug}
```

**Ejemplo:**

```bash
curl -X DELETE "https://api.novu.co/v2/workflows/user-onboarding-v2" \
  -H "Authorization: Bearer <your-token>"
```

### Disparar Workflow

```http
POST /v1/events/trigger
```

**Body Parameters:**

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| `name` | string | workflowId del workflow | ✅ |
| `to` | object/string | Destinatario(s) | ✅ |
| `payload` | object | Datos para el workflow | ❌ |
| `overrides` | object | Sobreescribir configuraciones | ❌ |
| `actor` | object | Información del actor | ❌ |

**Ejemplo:**

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "user-onboarding-v2",
    "to": {
      "subscriberId": "user_123",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe"
    },
    "payload": {
      "firstName": "John",
      "companyName": "Acme Corp",
      "userId": "user_123",
      "welcomeBonus": 100
    },
    "actor": {
      "subscriberId": "system",
      "name": "System"
    }
  }'
```

## Tipos de Steps

### Email Step

```json
{
  "name": "Email Step",
  "type": "email",
  "controlValues": {
    "subject": "Subject with {{variables}}",
    "body": "HTML content or Maily JSON",
    "editorType": "html|block",
    "disableOutputSanitization": false
  }
}
```

### In-App Step

```json
{
  "name": "In-App Notification",
  "type": "in_app",
  "controlValues": {
    "subject": "Notification title",
    "body": "Notification content",
    "avatar": "https://example.com/avatar.png",
    "primaryAction": {
      "label": "View Details",
      "redirect": {
        "url": "/details/{{payload.itemId}}",
        "target": "_self"
      }
    },
    "secondaryAction": {
      "label": "Dismiss",
      "redirect": {
        "url": "/dismiss",
        "target": "_self"
      }
    },
    "data": {
      "customField": "customValue"
    }
  }
}
```

### SMS Step

```json
{
  "name": "SMS Alert",
  "type": "sms",
  "controlValues": {
    "body": "SMS content with {{variables}}. Reply STOP to unsubscribe."
  }
}
```

### Push Step

```json
{
  "name": "Push Notification",
  "type": "push",
  "controlValues": {
    "title": "Push title",
    "body": "Push content",
    "data": {
      "category": "alert",
      "actionUrl": "/action"
    }
  }
}
```

### Chat Step

```json
{
  "name": "Slack Message",
  "type": "chat",
  "controlValues": {
    "body": "Chat message content"
  }
}
```

### Delay Step

```json
{
  "name": "Wait 30 minutes",
  "type": "delay",
  "controlValues": {
    "amount": 30,
    "unit": "minutes"
  }
}
```

**Unidades disponibles:** `seconds`, `minutes`, `hours`, `days`, `weeks`, `months`

### Digest Step

```json
{
  "name": "Daily Digest",
  "type": "digest",
  "controlValues": {
    "amount": 1,
    "unit": "days",
    "lookBackWindow": {
      "amount": 24,
      "unit": "hours"
    }
  }
}
```

## Ejemplos Completos

### Workflow de E-commerce

```json
{
  "name": "E-commerce Order Flow",
  "workflowId": "ecommerce-order-flow",
  "description": "Complete order processing workflow",
  "active": true,
  "tags": ["ecommerce", "orders"],
  "steps": [
    {
      "name": "Order Confirmation Email",
      "type": "email",
      "controlValues": {
        "subject": "Order Confirmed - #{{payload.orderNumber}}",
        "body": "<h2>Order Confirmed!</h2><p>Thank you {{payload.customerName}}, your order #{{payload.orderNumber}} has been confirmed.</p><p>Total: ${{payload.total}}</p>",
        "editorType": "html"
      }
    },
    {
      "name": "Order Confirmation Push",
      "type": "push",
      "controlValues": {
        "title": "Order Confirmed",
        "body": "Your order #{{payload.orderNumber}} is confirmed",
        "data": {
          "orderNumber": "{{payload.orderNumber}}",
          "category": "order"
        }
      }
    },
    {
      "name": "Wait for Processing",
      "type": "delay",
      "controlValues": {
        "amount": 2,
        "unit": "hours"
      }
    },
    {
      "name": "Shipping Notification",
      "type": "email",
      "controlValues": {
        "subject": "Your Order Has Shipped - #{{payload.orderNumber}}",
        "body": "<h2>Order Shipped!</h2><p>Your order #{{payload.orderNumber}} has been shipped.</p><p>Tracking: {{payload.trackingNumber}}</p>",
        "editorType": "html",
        "skip": {
          "type": "boolean",
          "expression": "{{payload.status}} !== 'shipped'"
        }
      }
    },
    {
      "name": "Delivery In-App",
      "type": "in_app",
      "controlValues": {
        "subject": "Package Delivered",
        "body": "Your order #{{payload.orderNumber}} has been delivered!",
        "primaryAction": {
          "label": "Rate Your Purchase",
          "redirect": {
            "url": "/orders/{{payload.orderNumber}}/review",
            "target": "_self"
          }
        }
      }
    }
  ],
  "payloadSchema": {
    "type": "object",
    "properties": {
      "orderNumber": { "type": "string" },
      "customerName": { "type": "string" },
      "total": { "type": "number" },
      "status": { "type": "string" },
      "trackingNumber": { "type": "string" }
    },
    "required": ["orderNumber", "customerName", "total"]
  },
  "validatePayload": true
}
```

### Workflow de Marketing con Digest

```json
{
  "name": "Weekly Newsletter Digest",
  "workflowId": "weekly-newsletter",
  "description": "Weekly aggregated newsletter",
  "active": true,
  "tags": ["marketing", "newsletter"],
  "steps": [
    {
      "name": "Collect Weekly Content",
      "type": "digest",
      "controlValues": {
        "amount": 7,
        "unit": "days",
        "lookBackWindow": {
          "amount": 7,
          "unit": "days"
        }
      }
    },
    {
      "name": "Weekly Newsletter Email",
      "type": "email",
      "controlValues": {
        "subject": "Your Weekly Update - {{steps.digest.events.length}} items",
        "body": "<h1>Weekly Newsletter</h1>{{#each steps.digest.events}}<div><h3>{{this.payload.title}}</h3><p>{{this.payload.summary}}</p></div>{{/each}}",
        "editorType": "html"
      }
    }
  ]
}
```

### Disparar Workflows

#### Trigger simple

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "ecommerce-order-flow",
    "to": "user_12345",
    "payload": {
      "orderNumber": "ORD-2024-001",
      "customerName": "María García",
      "total": 159.99,
      "status": "confirmed"
    }
  }'
```

#### Trigger con múltiples destinatarios

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "weekly-newsletter",
    "to": [
      { "subscriberId": "user_1", "email": "user1@example.com" },
      { "subscriberId": "user_2", "email": "user2@example.com" }
    ],
    "payload": {
      "weekNumber": 42,
      "year": 2024
    }
  }'
```

#### Trigger con overrides

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "user-onboarding-v2",
    "to": "user_12345",
    "payload": {
      "firstName": "Carlos",
      "companyName": "TechStart"
    },
    "overrides": {
      "email": {
        "from": "welcome@mycompany.com"
      },
      "in_app": {
        "primaryAction": {
          "label": "Comenzar Ahora"
        }
      }
    }
  }'
```

## Operaciones Adicionales

### Duplicar Workflow

```http
POST /v2/workflows/{workflowSlug}/duplicate
```

```bash
curl -X POST "https://api.novu.co/v2/workflows/user-onboarding-v2/duplicate" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "User Onboarding - Copy",
    "workflowId": "user-onboarding-copy"
  }'
```

### Sincronizar entre Ambientes

```http
PUT /v2/workflows/{workflowSlug}/sync
```

```bash
curl -X PUT "https://api.novu.co/v2/workflows/user-onboarding-v2/sync" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "targetEnvironmentId": "env_production_123"
  }'
```

### Obtener Test Data

```http
GET /v2/workflows/{workflowSlug}/test-data
```

```bash
curl -X GET "https://api.novu.co/v2/workflows/user-onboarding-v2/test-data" \
  -H "Authorization: Bearer <your-token>"
```

## Manejo de Errores

### Códigos de Error Comunes

| Código | Descripción | Solución |
|--------|-------------|----------|
| `400` | Bad Request - Datos inválidos | Verificar formato del payload |
| `401` | Unauthorized - Token inválido | Verificar token de autenticación |
| `403` | Forbidden - Sin permisos | Verificar permisos del usuario |
| `404` | Not Found - Workflow no encontrado | Verificar workflowSlug existe |
| `409` | Conflict - workflowId duplicado | Usar workflowId único |
| `422` | Validation Error - Datos no válidos | Revisar validaciones de schema |
| `429` | Rate Limit Exceeded | Reducir frecuencia de requests |

### Ejemplo de Response de Error

```json
{
  "message": "Validation failed",
  "error": "Bad Request",
  "statusCode": 400,
  "data": [
    {
      "field": "steps.0.controlValues.subject",
      "message": "Subject is required for email steps"
    }
  ]
}
```

### Manejo de Errores en JavaScript

```javascript
try {
  const workflow = await createWorkflow({
    environment,
    workflow: workflowData
  });
  console.log('Workflow created:', workflow.data);
} catch (error) {
  if (error.response?.status === 409) {
    console.error('Workflow ID already exists');
  } else if (error.response?.status === 422) {
    console.error('Validation errors:', error.response.data.data);
  } else {
    console.error('Unexpected error:', error.message);
  }
}
```

## Límites y Consideraciones

### Rate Limits

- **Creación de workflows**: 100 requests/hora
- **Trigger de eventos**: 1000 requests/minuto
- **Consultas (GET)**: 500 requests/minuto

### Límites de Tamaño

- **Nombre del workflow**: Máximo 100 caracteres
- **Descripción**: Máximo 500 caracteres
- **Número de steps**: Máximo 20 pasos por workflow
- **Payload size**: Máximo 64KB por trigger
- **Tags**: Máximo 10 tags por workflow

### Mejores Prácticas

1. **Naming Convention**: Usar nombres descriptivos y únicos para workflowId
2. **Testing**: Siempre probar workflows en ambiente de desarrollo
3. **Payload Validation**: Habilitar validación para workflows críticos
4. **Error Handling**: Implementar manejo robusto de errores
5. **Performance**: Usar paginación para consultas grandes
6. **Security**: Nunca incluir datos sensibles en payloads
7. **Versioning**: Usar sufijos de versión en workflowId para versionado

Esta documentación cubre todas las operaciones principales para trabajar con workflows en la API de Novu. Para más información, consultar la [documentación oficial](https://docs.novu.co) o el [Swagger Spec](https://api.novu.co/api) de la API.