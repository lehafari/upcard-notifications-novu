# Novu Events API Documentation

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Autenticación](#autenticación)
3. [URLs Base](#urls-base)
4. [Tipos de Eventos](#tipos-de-eventos)
   - [Trigger Simple](#trigger-simple)
   - [Trigger Bulk](#trigger-bulk)
   - [Trigger Broadcast](#trigger-broadcast)
   - [Cancelar Eventos](#cancelar-eventos)
5. [Estructura de Datos](#estructura-de-datos)
6. [Destinatarios y Topics](#destinatarios-y-topics)
7. [Payloads y Overrides](#payloads-y-overrides)
8. [Ejemplos Completos](#ejemplos-completos)
9. [Rate Limiting](#rate-limiting)
10. [Manejo de Errores](#manejo-de-errores)
11. [Mejores Prácticas](#mejores-prácticas)

## Introducción

La API de Events es el **punto de entrada principal** para disparar notificaciones en Novu. Permite enviar notificaciones individuales, masivas o de broadcast a través de workflows configurados previamente. Soporta múltiples canales (email, SMS, push, chat, in-app) de forma simultánea.

### Características Principales

- **Trigger Único**: Envío a destinatarios específicos
- **Trigger Bulk**: Hasta 100 eventos en una sola request
- **Broadcast**: Envío a todos los suscriptores
- **Cancelación**: Cancelar eventos pendientes o en proceso
- **Multi-canal**: Email, SMS, Push, Chat, In-App simultáneamente
- **Overrides**: Personalización por canal y proveedor
- **Topics**: Agrupación de suscriptores
- **Multi-tenant**: Soporte para múltiples tenants
- **Validación**: Validación automática de payloads

## Autenticación

Todas las requests requieren autenticación:

```bash
# Bearer Token
Authorization: Bearer <your-access-token>

# API Key
Authorization: ApiKey <your-api-key>
```

## URLs Base

- **API Base**: `https://api.novu.co/v1/events`

## Tipos de Eventos

### Trigger Simple

Envía una notificación a destinatarios específicos.

```http
POST /v1/events/trigger
```

**Body Parameters:**

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| `name` | string | Identificador del workflow | ✅ |
| `to` | object/array | Destinatario(s) | ✅ |
| `payload` | object | Datos para el template | ❌ |
| `overrides` | object | Configuraciones específicas | ❌ |
| `actor` | object | Actor que dispara la notificación | ❌ |
| `tenant` | object | Contexto de tenant | ❌ |
| `transactionId` | string | ID único de transacción | ❌ |

**Ejemplo básico:**

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "welcome-email",
    "to": {
      "subscriberId": "user_123",
      "email": "user@example.com",
      "firstName": "Juan",
      "lastName": "Pérez"
    },
    "payload": {
      "companyName": "Mi Empresa",
      "welcomeMessage": "¡Bienvenido a nuestra plataforma!"
    }
  }'
```

**Response:**

```json
{
  "data": {
    "acknowledged": true,
    "status": "processed",
    "transactionId": "txn_abc123def456"
  }
}
```

### Trigger Bulk

Envía múltiples notificaciones en una sola request (máximo 100).

```http
POST /v1/events/trigger/bulk
```

**Ejemplo:**

```bash
curl -X POST "https://api.novu.co/v1/events/trigger/bulk" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "events": [
      {
        "name": "user-notification",
        "to": "user_001",
        "payload": {
          "message": "Mensaje para usuario 1"
        }
      },
      {
        "name": "user-notification", 
        "to": "user_002",
        "payload": {
          "message": "Mensaje para usuario 2"
        }
      },
      {
        "name": "admin-alert",
        "to": "admin_001",
        "payload": {
          "alertType": "security",
          "severity": "high"
        }
      }
    ]
  }'
```

**Response:**

```json
{
  "data": [
    {
      "acknowledged": true,
      "status": "processed",
      "transactionId": "txn_bulk_001"
    },
    {
      "acknowledged": true,
      "status": "processed", 
      "transactionId": "txn_bulk_002"
    },
    {
      "acknowledged": false,
      "status": "error",
      "error": ["Workflow 'admin-alert' not found"]
    }
  ]
}
```

### Trigger Broadcast

Envía una notificación a **todos** los suscriptores existentes.

```http
POST /v1/events/trigger/broadcast
```

**Ejemplo:**

```bash
curl -X POST "https://api.novu.co/v1/events/trigger/broadcast" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "system-announcement",
    "payload": {
      "title": "Mantenimiento Programado",
      "message": "El sistema estará en mantenimiento el próximo domingo de 2:00 AM a 6:00 AM",
      "maintenanceDate": "2024-02-15",
      "duration": "4 horas"
    },
    "overrides": {
      "email": {
        "from": "system@company.com"
      }
    }
  }'
```

### Cancelar Eventos

Cancela eventos pendientes o en proceso usando el transaction ID.

```http
DELETE /v1/events/trigger/{transactionId}
```

**Ejemplo:**

```bash
curl -X DELETE "https://api.novu.co/v1/events/trigger/txn_abc123def456" \
  -H "Authorization: Bearer <your-token>"
```

**Response:**

```json
{
  "data": {
    "cancelled": true,
    "affectedJobs": 3
  }
}
```

## Estructura de Datos

### Destinatario Simple

```json
{
  "to": {
    "subscriberId": "user_123",
    "email": "user@example.com",
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "+34600123456",
    "avatar": "https://example.com/avatar.jpg",
    "locale": "es-ES",
    "data": {
      "customField": "customValue"
    }
  }
}
```

### Múltiples Destinatarios

```json
{
  "to": [
    {
      "subscriberId": "user_001",
      "email": "user1@example.com"
    },
    {
      "subscriberId": "user_002", 
      "email": "user2@example.com"
    },
    {
      "type": "Topic",
      "topicKey": "premium-users"
    }
  ]
}
```

### Actor

```json
{
  "actor": {
    "subscriberId": "admin_001",
    "firstName": "María",
    "lastName": "García",
    "avatar": "https://example.com/admin-avatar.jpg"
  }
}
```

### Tenant Context

```json
{
  "tenant": {
    "identifier": "company_abc",
    "name": "Empresa ABC",
    "data": {
      "industry": "technology",
      "region": "europe"
    }
  }
}
```

## Destinatarios y Topics

### Destinatarios Individuales

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "order-confirmation",
    "to": {
      "subscriberId": "customer_456",
      "email": "customer@example.com",
      "firstName": "Ana",
      "phone": "+34600987654"
    },
    "payload": {
      "orderNumber": "ORD-2024-001",
      "total": 89.99,
      "items": [
        {"name": "Producto A", "quantity": 2, "price": 44.99}
      ]
    }
  }'
```

### Usando Topics

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "newsletter-weekly",
    "to": [
      {
        "type": "Topic",
        "topicKey": "newsletter-subscribers"
      },
      {
        "type": "Topic", 
        "topicKey": "premium-users"
      }
    ],
    "payload": {
      "subject": "Newsletter Semanal - Febrero 2024",
      "articles": [
        {
          "title": "Nuevas Funcionalidades",
          "summary": "Descubre las últimas mejoras..."
        }
      ]
    }
  }'
```

### Destinatarios Mixtos

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "security-alert",
    "to": [
      {
        "subscriberId": "admin_001",
        "email": "admin@company.com"
      },
      {
        "subscriberId": "admin_002",
        "email": "admin2@company.com"
      },
      {
        "type": "Topic",
        "topicKey": "security-team"
      }
    ],
    "payload": {
      "alertType": "suspicious-login",
      "ipAddress": "192.168.1.100",
      "timestamp": "2024-02-10T14:30:00Z"
    }
  }'
```

## Payloads y Overrides

### Payload con Validación

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "invoice-generated",
    "to": "customer_789",
    "payload": {
      "invoiceNumber": "INV-2024-0025",
      "customerName": "Carlos Ruiz",
      "amount": 250.00,
      "currency": "EUR",
      "dueDate": "2024-03-01",
      "items": [
        {
          "description": "Servicio de consultoría",
          "quantity": 10,
          "unitPrice": 25.00,
          "total": 250.00
        }
      ],
      "companyLogo": "https://company.com/logo.png",
      "supportEmail": "support@company.com"
    }
  }'
```

### Overrides por Canal

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "multi-channel-alert",
    "to": "user_premium_001",
    "payload": {
      "alertMessage": "Tu cuenta premium expira pronto",
      "expiryDate": "2024-02-20",
      "renewalUrl": "https://app.com/renew"
    },
    "overrides": {
      "email": {
        "from": "premium@company.com",
        "replyTo": "premium-support@company.com",
        "subject": "🎯 Acción Requerida: Renovación Premium"
      },
      "sms": {
        "from": "+34600000000",
        "body": "Tu cuenta premium expira el 20/02. Renueva en: {{payload.renewalUrl}}"
      },
      "push": {
        "title": "Renovación Premium",
        "body": "Tu cuenta expira pronto. ¡Renueva ahora!",
        "data": {
          "action": "renewal",
          "urgency": "high"
        },
        "sound": "premium_alert.wav"
      },
      "in_app": {
        "primaryAction": {
          "label": "Renovar Ahora",
          "redirect": {
            "url": "{{payload.renewalUrl}}",
            "target": "_blank"
          }
        },
        "secondaryAction": {
          "label": "Recordar Más Tarde",
          "redirect": {
            "url": "/dashboard",
            "target": "_self"
          }
        }
      }
    }
  }'
```

### Overrides por Proveedor

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "transactional-email",
    "to": "customer_456",
    "payload": {
      "customerName": "Laura Martín",
      "transactionId": "TXN-456789"
    },
    "overrides": {
      "providers": {
        "sendgrid": {
          "ipPoolName": "transactional-pool",
          "customHeaders": {
            "X-Priority": "1",
            "X-Category": "transactional"
          }
        },
        "twilio": {
          "messagingServiceSid": "MG1234567890abcdef"
        }
      }
    }
  }'
```

### Overrides por Steps

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "complex-workflow",
    "to": "user_001",
    "payload": {
      "userName": "Pedro González",
      "actionRequired": true
    },
    "overrides": {
      "steps": {
        "welcome-email-step": {
          "subject": "Personalización específica para este envío",
          "from": "custom@company.com"
        },
        "follow-up-sms-step": {
          "skip": false,
          "body": "Mensaje SMS personalizado para {{payload.userName}}"
        },
        "delay-step": {
          "amount": 2,
          "unit": "hours"
        }
      }
    }
  }'
```

## Ejemplos Completos

### E-commerce: Flujo de Pedido Completo

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "order-complete-flow",
    "to": {
      "subscriberId": "customer_12345",
      "email": "cliente@example.com",
      "firstName": "María",
      "lastName": "López",
      "phone": "+34600123456"
    },
    "payload": {
      "orderNumber": "ORD-2024-0156",
      "customerName": "María López",
      "orderDate": "2024-02-10",
      "estimatedDelivery": "2024-02-15",
      "total": 129.99,
      "currency": "EUR",
      "shippingAddress": {
        "street": "Calle Mayor 123",
        "city": "Madrid",
        "postalCode": "28001",
        "country": "España"
      },
      "items": [
        {
          "name": "Zapatillas Running",
          "sku": "ZAP-RUN-001",
          "quantity": 1,
          "price": 89.99,
          "image": "https://shop.com/images/zapatillas.jpg"
        },
        {
          "name": "Calcetines Deportivos",
          "sku": "CALC-DEP-002", 
          "quantity": 2,
          "price": 20.00,
          "image": "https://shop.com/images/calcetines.jpg"
        }
      ],
      "trackingUrl": "https://tracking.com/ORD-2024-0156",
      "supportUrl": "https://shop.com/support",
      "returnPolicy": "30 días para devoluciones"
    },
    "actor": {
      "subscriberId": "system",
      "firstName": "Sistema",
      "lastName": "Tienda Online"
    },
    "overrides": {
      "email": {
        "from": "pedidos@tienda.com",
        "replyTo": "soporte@tienda.com"
      },
      "sms": {
        "from": "+34900123456"
      },
      "push": {
        "data": {
          "orderNumber": "ORD-2024-0156",
          "action": "view_order"
        }
      }
    },
    "transactionId": "order_confirmation_12345"
  }'
```

### SaaS: Onboarding de Usuario

```bash
curl -X POST "https://api.novu.co/v1/events/trigger" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "user-onboarding-sequence",
    "to": {
      "subscriberId": "new_user_789",
      "email": "nuevo@empresa.com",
      "firstName": "Carlos",
      "lastName": "Ruiz",
      "phone": "+34600987654"
    },
    "payload": {
      "userName": "Carlos",
      "companyName": "Empresa XYZ",
      "planType": "premium",
      "trialEndDate": "2024-03-10",
      "onboardingSteps": [
        {
          "step": 1,
          "title": "Completa tu perfil",
          "url": "/profile/complete",
          "completed": false
        },
        {
          "step": 2,
          "title": "Invita a tu equipo",
          "url": "/team/invite",
          "completed": false
        },
        {
          "step": 3,
          "title": "Crea tu primer proyecto",
          "url": "/projects/new",
          "completed": false
        }
      ],
      "featuresUnlocked": [
        "Proyectos ilimitados",
        "Colaboración en equipo",
        "Integraciones avanzadas",
        "Soporte prioritario"
      ],
      "welcomeVideoUrl": "https://app.com/videos/welcome",
      "documentationUrl": "https://docs.app.com/getting-started",
      "supportEmail": "support@app.com"
    },
    "tenant": {
      "identifier": "empresa_xyz",
      "name": "Empresa XYZ",
      "data": {
        "industry": "technology",
        "size": "startup",
        "region": "spain"
      }
    },
    "overrides": {
      "email": {
        "from": "onboarding@app.com",
        "replyTo": "success@app.com",
        "customHeaders": {
          "X-Campaign": "user-onboarding",
          "X-Segment": "new-premium-users"
        }
      },
      "in_app": {
        "primaryAction": {
          "label": "Comenzar Tour",
          "redirect": {
            "url": "/onboarding/tour",
            "target": "_self"
          }
        },
        "data": {
          "onboardingProgress": 0,
          "planType": "premium"
        }
      }
    }
  }'
```

### Bulk: Notificaciones Masivas de Marketing

```bash
curl -X POST "https://api.novu.co/v1/events/trigger/bulk" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "events": [
      {
        "name": "promotional-campaign",
        "to": {
          "type": "Topic",
          "topicKey": "premium-customers"
        },
        "payload": {
          "campaignName": "Black Friday 2024",
          "discountPercentage": 50,
          "validUntil": "2024-11-30",
          "featuredProducts": [
            {
              "name": "Producto Premium A",
              "originalPrice": 199.99,
              "discountedPrice": 99.99,
              "image": "https://shop.com/products/a.jpg"
            }
          ]
        }
      },
      {
        "name": "promotional-campaign",
        "to": {
          "type": "Topic", 
          "topicKey": "regular-customers"
        },
        "payload": {
          "campaignName": "Black Friday 2024",
          "discountPercentage": 30,
          "validUntil": "2024-11-30",
          "featuredProducts": [
            {
              "name": "Producto Regular B",
              "originalPrice": 99.99,
              "discountedPrice": 69.99,
              "image": "https://shop.com/products/b.jpg"
            }
          ]
        }
      },
      {
        "name": "vip-exclusive-offer",
        "to": {
          "type": "Topic",
          "topicKey": "vip-customers"
        },
        "payload": {
          "offerTitle": "Acceso Exclusivo VIP",
          "discountPercentage": 70,
          "earlyAccess": true,
          "personalShopper": true,
          "freeShipping": true
        }
      }
    ]
  }'
```

### Broadcast: Anuncio del Sistema

```bash
curl -X POST "https://api.novu.co/v1/events/trigger/broadcast" \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "system-maintenance-announcement",
    "payload": {
      "maintenanceTitle": "Mantenimiento Programado del Sistema",
      "maintenanceDate": "2024-02-18",
      "startTime": "02:00 AM CET",
      "endTime": "06:00 AM CET",
      "duration": "4 horas",
      "affectedServices": [
        "API Principal",
        "Dashboard Web",
        "Aplicación Móvil"
      ],
      "unaffectedServices": [
        "Notificaciones de emergencia",
        "Soporte técnico"
      ],
      "reason": "Actualización de seguridad y mejoras de rendimiento",
      "preparationSteps": [
        "Guarda tu trabajo actual",
        "Descarga reportes importantes",
        "Planifica actividades offline"
      ],
      "emergencyContact": "+34900123456",
      "statusPageUrl": "https://status.app.com",
      "estimatedImpact": "Servicios no disponibles durante la ventana de mantenimiento"
    },
    "overrides": {
      "email": {
        "from": "system@app.com",
        "replyTo": "support@app.com",
        "subject": "🔧 Mantenimiento Programado - 18 Feb 2024",
        "customHeaders": {
          "X-Priority": "1",
          "X-Category": "system-announcement"
        }
      },
      "push": {
        "title": "Mantenimiento del Sistema",
        "body": "Mantenimiento programado el 18/02 de 2:00-6:00 AM",
        "data": {
          "category": "maintenance",
          "priority": "high",
          "action": "view_details"
        },
        "sound": "system_alert.wav"
      },
      "in_app": {
        "primaryAction": {
          "label": "Ver Detalles",
          "redirect": {
            "url": "/maintenance-info",
            "target": "_self"
          }
        },
        "secondaryAction": {
          "label": "Estado del Sistema",
          "redirect": {
            "url": "https://status.app.com",
            "target": "_blank"
          }
        }
      }
    }
  }'
```

## Rate Limiting

### Límites por Endpoint

| Endpoint | Límite Base | Costo por Request | Límite Efectivo |
|----------|-------------|-------------------|-----------------|
| `POST /trigger` | 1000/min | 1 | 1000 requests/min |
| `POST /trigger/bulk` | 1000/min | 10 | 100 requests/min (1000 eventos) |
| `POST /trigger/broadcast` | 100/min | 5 | 100 requests/min |
| `DELETE /trigger/:id` | 500/min | 1 | 500 requests/min |

### Headers de Rate Limiting

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1642678800
X-RateLimit-Policy: 1000;w=60
```

### Manejo de Rate Limiting

```javascript
const triggerEvent = async (eventData) => {
  try {
    const response = await fetch('/v1/events/trigger', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer your-token',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(eventData)
    });

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After');
      console.log(`Rate limited. Retry after ${retryAfter} seconds`);
      // Implementar backoff exponencial
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      return triggerEvent(eventData); // Reintentar
    }

    return await response.json();
  } catch (error) {
    console.error('Error triggering event:', error);
    throw error;
  }
};
```

## Manejo de Errores

### Códigos de Estado

| Código | Descripción | Solución |
|--------|-------------|----------|
| `200` | Success - Evento procesado | - |
| `400` | Bad Request - Datos inválidos | Verificar formato del payload |
| `401` | Unauthorized - Token inválido | Verificar autenticación |
| `403` | Forbidden - Sin permisos | Verificar permisos de la API key |
| `404` | Not Found - Workflow no encontrado | Verificar que el workflow existe y está activo |
| `422` | Validation Error - Payload inválido | Revisar validación de schema |
| `429` | Rate Limit Exceeded | Implementar backoff, revisar límites |
| `500` | Internal Server Error | Contactar soporte |

### Estados de Trigger

```typescript
enum TriggerEventStatusEnum {
  PROCESSED = 'processed',                    // Evento procesado correctamente
  ERROR = 'error',                           // Error en el procesamiento
  NOT_ACTIVE = 'trigger_not_active',         // Workflow no activo
  NO_WORKFLOW_STEPS = 'no_workflow_steps_defined',      // Sin pasos definidos
  NO_WORKFLOW_ACTIVE_STEPS = 'no_workflow_active_steps_defined', // Sin pasos activos
  TENANT_MISSING = 'no_tenant_found',        // Tenant requerido no encontrado
  INVALID_RECIPIENTS = 'invalid_recipients',  // Destinatarios inválidos
}
```

### Ejemplos de Errores

#### Error de Validación de Payload

```json
{
  "message": "Payload validation failed",
  "error": "Unprocessable Entity",
  "statusCode": 422,
  "data": [
    {
      "field": "payload.email",
      "message": "must be a valid email address",
      "value": "invalid-email",
      "schemaPath": "#/properties/email/format"
    },
    {
      "field": "payload.amount",
      "message": "must be a positive number",
      "value": -10,
      "schemaPath": "#/properties/amount/minimum"
    }
  ]
}
```

#### Error de Workflow No Encontrado

```json
{
  "statusCode": 404,
  "message": "Workflow with identifier 'invalid-workflow' not found",
  "error": "Not Found"
}
```

#### Error de Destinatarios Inválidos

```json
{
  "data": {
    "acknowledged": false,
    "status": "invalid_recipients",
    "error": [
      "subscriberId 'invalid@subscriber@id' contains invalid characters",
      "topicKey 'invalid topic key' must not contain spaces"
    ]
  }
}
```

### Manejo de Errores en JavaScript

```javascript
const handleTriggerResponse = (response) => {
  const { data } = response;
  
  switch (data.status) {
    case 'processed':
      console.log(`Event triggered successfully: ${data.transactionId}`);
      break;
      
    case 'trigger_not_active':
      console.warn('Workflow is not active');
      // Activar workflow o usar uno alternativo
      break;
      
    case 'no_workflow_steps_defined':
      console.error('Workflow has no steps defined');
      // Configurar steps en el workflow
      break;
      
    case 'no_tenant_found':
      console.error('Tenant context required but not found');
      // Proporcionar contexto de tenant
      break;
      
    case 'invalid_recipients':
      console.error('Invalid recipients:', data.error);
      // Corregir formato de destinatarios
      break;
      
    case 'error':
      console.error('Processing error:', data.error);
      // Manejar errores específicos
      break;
      
    default:
      console.warn('Unknown status:', data.status);
  }
};

// Ejemplo de uso con manejo completo
const triggerWithErrorHandling = async (eventData) => {
  try {
    const response = await fetch('/v1/events/trigger', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer your-token',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(eventData)
    });

    if (!response.ok) {
      if (response.status === 422) {
        const errorData = await response.json();
        console.error('Validation errors:', errorData.data);
        return { success: false, errors: errorData.data };
      }
      
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        return { success: false, retryAfter };
      }
      
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const result = await response.json();
    handleTriggerResponse(result);
    
    return { 
      success: result.data.acknowledged, 
      transactionId: result.data.transactionId,
      status: result.data.status
    };
    
  } catch (error) {
    console.error('Network or parsing error:', error);
    return { success: false, error: error.message };
  }
};
```

## Mejores Prácticas

### 1. Transaction IDs Únicos

```javascript
// Usar UUIDs para evitar duplicados
import { v4 as uuidv4 } from 'uuid';

const triggerEvent = async (eventData) => {
  const transactionId = `${eventData.name}_${Date.now()}_${uuidv4()}`;
  
  return await fetch('/v1/events/trigger', {
    method: 'POST',
    body: JSON.stringify({
      ...eventData,
      transactionId
    })
  });
};
```

### 2. Retry Logic con Backoff Exponencial

```javascript
const triggerWithRetry = async (eventData, maxRetries = 3) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await triggerEvent(eventData);
      
      if (response.data.acknowledged) {
        return response;
      }
      
      // Si no fue exitoso pero no es un error de red, no reintentar
      if (response.data.status !== 'error') {
        return response;
      }
      
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      
      // Backoff exponencial: 1s, 2s, 4s
      const delay = Math.pow(2, attempt - 1) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};
```

### 3. Batch Processing para Bulk

```javascript
const triggerBulkInBatches = async (events, batchSize = 100) => {
  const results = [];
  
  for (let i = 0; i < events.length; i += batchSize) {
    const batch = events.slice(i, i + batchSize);
    
    try {
      const response = await fetch('/v1/events/trigger/bulk', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer your-token',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ events: batch })
      });
      
      const batchResults = await response.json();
      results.push(...batchResults.data);
      
      // Pequeña pausa entre batches para evitar rate limiting
      if (i + batchSize < events.length) {
        await new Promise(resolve => setTimeout(resolve, 100));
      }
      
    } catch (error) {
      console.error(`Error in batch ${i / batchSize + 1}:`, error);
      // Marcar el batch como fallido pero continuar
      results.push(...batch.map(() => ({ 
        acknowledged: false, 
        status: 'error', 
        error: ['Batch processing failed'] 
      })));
    }
  }
  
  return results;
};
```

### 4. Validación de Payload

```javascript
const validatePayload = (payload, schema) => {
  // Usar una librería como Joi o Yup para validación
  const { error } = schema.validate(payload);
  
  if (error) {
    throw new Error(`Payload validation failed: ${error.details.map(d => d.message).join(', ')}`);
  }
};

// Ejemplo de uso
const orderSchema = Joi.object({
  orderNumber: Joi.string().required(),
  customerName: Joi.string().required(),
  total: Joi.number().positive().required(),
  items: Joi.array().items(Joi.object({
    name: Joi.string().required(),
    quantity: Joi.number().integer().positive().required(),
    price: Joi.number().positive().required()
  })).min(1).required()
});

const triggerOrderConfirmation = async (orderData) => {
  validatePayload(orderData, orderSchema);
  
  return await triggerEvent({
    name: 'order-confirmation',
    to: orderData.customerId,
    payload: orderData
  });
};
```

### 5. Monitoreo y Logging

```javascript
const triggerWithMonitoring = async (eventData) => {
  const startTime = Date.now();
  
  try {
    console.log(`Triggering event: ${eventData.name}`, {
      recipients: Array.isArray(eventData.to) ? eventData.to.length : 1,
      hasPayload: !!eventData.payload,
      hasOverrides: !!eventData.overrides
    });
    
    const response = await triggerEvent(eventData);
    const duration = Date.now() - startTime;
    
    console.log(`Event triggered successfully`, {
      name: eventData.name,
      transactionId: response.data.transactionId,
      status: response.data.status,
      duration: `${duration}ms`
    });
    
    // Enviar métricas a sistema de monitoreo
    sendMetric('event.trigger.success', {
      workflow: eventData.name,
      duration,
      status: response.data.status
    });
    
    return response;
    
  } catch (error) {
    const duration = Date.now() - startTime;
    
    console.error(`Event trigger failed`, {
      name: eventData.name,
      error: error.message,
      duration: `${duration}ms`
    });
    
    sendMetric('event.trigger.error', {
      workflow: eventData.name,
      duration,
      error: error.message
    });
    
    throw error;
  }
};
```

### 6. Gestión de Cancelaciones

```javascript
const triggerWithCancellation = async (eventData) => {
  // Generar transaction ID único
  const transactionId = `txn_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  
  try {
    const response = await triggerEvent({
      ...eventData,
      transactionId
    });
    
    // Guardar transaction ID para posible cancelación
    saveTransactionId(eventData.userId, transactionId, eventData.name);
    
    return {
      ...response,
      cancel: () => cancelEvent(transactionId)
    };
    
  } catch (error) {
    console.error('Failed to trigger event:', error);
    throw error;
  }
};

const cancelEvent = async (transactionId) => {
  try {
    const response = await fetch(`/v1/events/trigger/${transactionId}`, {
      method: 'DELETE',
      headers: {
        'Authorization': 'Bearer your-token'
      }
    });
    
    if (response.ok) {
      const result = await response.json();
      console.log(`Cancelled event: ${transactionId}`, result.data);
      return result.data;
    } else {
      console.warn(`Failed to cancel event: ${transactionId}`);
    }
    
  } catch (error) {
    console.error(`Error cancelling event ${transactionId}:`, error);
  }
};
```

Esta documentación cubre completamente el Events API de Novu, proporcionando ejemplos prácticos, manejo de errores robusto y mejores prácticas para implementaciones en producción.