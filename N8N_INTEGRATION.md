# Integración de GitHub con n8n para Automatización de Contenido

Este documento explica cómo usar este repositorio de GitHub con n8n para automatizar la publicación de contenido en múltiples canales de redes sociales.

## Estructura del Repositorio

```
content-automation-config/
├── config/
│   └── api-credentials.json     # Credenciales de APIs (NO subir con datos reales)
├── templates/
│   └── content-template.json    # Plantillas de contenido para los canales
└── N8N_INTEGRATION.md          # Esta documentación
```

## Configuración en n8n

### 1. Crear Token de GitHub

1. Ve a GitHub Settings > Developer settings > Personal access tokens
2. Genera un nuevo token con permisos:
   - `repo` (acceso completo a repositorios)
   - `read:org` (si usas organizaciones)
3. Guarda el token de forma segura

### 2. Nodo HTTP Request para leer archivos de GitHub

En n8n, usa el nodo **HTTP Request** para obtener archivos del repositorio:

**URL:** `https://api.github.com/repos/susanagreenbetanzos-maker/content-automation-config/contents/[RUTA_ARCHIVO]`

**Ejemplo para leer api-credentials.json:**
```
https://api.github.com/repos/susanagreenbetanzos-maker/content-automation-config/contents/config/api-credentials.json
```

**Headers:**
```json
{
  "Authorization": "Bearer TU_GITHUB_TOKEN",
  "Accept": "application/vnd.github.v3+json"
}
```

**Método:** GET

### 3. Decodificar contenido Base64

GitHub devuelve el contenido en Base64. Usa un nodo **Code** (JavaScript) para decodificarlo:

```javascript
const base64Content = $json.content;
const decodedContent = Buffer.from(base64Content, 'base64').toString('utf-8');
const jsonData = JSON.parse(decodedContent);

return { json: jsonData };
```

### 4. Workflow Ejemplo para Publicación Automática

**Paso 1: Schedule Trigger**
- Configura el horario según `posting_schedule` en las plantillas

**Paso 2: HTTP Request (GitHub)**
- Lee `templates/content-template.json`
- Decodifica el contenido Base64

**Paso 3: HTTP Request (GitHub)**
- Lee `config/api-credentials.json`
- Decodifica las credenciales

**Paso 4: Function Node**
- Combina los datos de plantilla con las credenciales
- Prepara el contenido para publicar

**Paso 5: Nodos de Redes Sociales**
- **Instagram:** Usa nodo HTTP Request con Instagram Graph API
- **TikTok:** Usa nodo HTTP Request con TikTok API
- **YouTube:** Usa nodo HTTP Request con YouTube Data API

## Ejemplo de Código n8n para Leer GitHub

```javascript
// Nodo Function: Preparar request a GitHub
const owner = 'susanagreenbetanzos-maker';
const repo = 'content-automation-config';
const path = 'templates/content-template.json';

return {
  json: {
    url: `https://api.github.com/repos/${owner}/${repo}/contents/${path}`,
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${$env.GITHUB_TOKEN}`,
      'Accept': 'application/vnd.github.v3+json'
    }
  }
};
```

## Configuración de Credenciales en n8n

1. En n8n, ve a **Credentials** > **Add Credential**
2. Crea credenciales para:
   - GitHub (token personal)
   - Instagram API
   - TikTok API
   - YouTube API

## Actualización de Contenido

Para actualizar las plantillas o configuraciones:

1. Edita los archivos en GitHub directamente
2. n8n leerá automáticamente la versión más reciente en la siguiente ejecución
3. No necesitas reiniciar el workflow

## Seguridad

⚠️ **IMPORTANTE:**
- **NUNCA** subas credenciales reales a GitHub
- Usa variables de entorno en n8n para datos sensibles
- El archivo `api-credentials.json` debe ser una plantilla, no contener datos reales
- Configura el repositorio como **privado** si almacenas información sensible

## Estructura de Datos Esperada

### Plantilla de Canal
```json
{
  "channel_id": "channel_1",
  "channel_name": "Nombre del Canal",
  "platforms": ["instagram", "tiktok"],
  "posting_schedule": {
    "frequency": "daily",
    "time": "10:00"
  }
}
```

### Post Template
```json
{
  "title": "Título del post",
  "description": "Descripción",
  "hashtags": ["#tag1", "#tag2"],
  "media_type": "image",
  "media_url": "https://..."
}
```

## Soporte

Para preguntas o problemas, crea un issue en este repositorio.
