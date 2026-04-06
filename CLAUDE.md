# NovaTech - Sistema Inteligente de Tickets con IA

## Contexto del Proyecto

Este es un MVP de un sistema de gestión de tickets de soporte TI para **NovaTech Solutions**, una consultora de TI con 200 empleados en México (Monterrey y Guadalajara). El sistema usa IA para clasificar, priorizar y resolver automáticamente tickets de soporte.

### Cliente
- **Empresa:** NovaTech Solutions S.A. de C.V.
- **Giro:** Consultoría de TI, desarrollo de software, infraestructura cloud
- **Empleados:** ~200 (120 Monterrey, 80 Guadalajara)
- **Directora General:** Patricia Vega
- **Gerente de Operaciones:** Ricardo Méndez (contacto principal)
- **Presupuesto:** $500 USD/mes para herramientas (puede subir a $800 si demuestra valor)

### Equipo de TI
- **Marcos Solís** (líder, 8 años) — single point of failure, tiene wiki personal en Notion con ~200 artículos
- **Ana García** (2 años) — backup de Marcos, sin acceso completo
- **Luis Ramírez** (1 año)
- **Sofía Torres** (6 meses, medio tiempo)

### Herramientas del cliente
- Google Workspace (Gmail, Drive, Sheets) — estándar corporativo
- Slack — solo ~80 personas lo usan (desarrollo, QA, diseño)
- Google SSO para autenticación
- NO tienen sistema de tickets (Freshdesk se canceló hace 2 años por baja adopción)

## Arquitectura del MVP

### Frontend (`index.html`)
- Single-page app con HTML/JS/Tailwind CSS (CDN)
- Chart.js para gráficas del dashboard
- 3 tabs: Nuevo Ticket, Tickets, Dashboard
- Clasificación por keywords en JavaScript (funciona sin backend)
- Puede opcionalmente llamar a `/api/classify` si existe el endpoint

### API Serverless (`api/classify.py`)
- Vercel serverless function (Python)
- Clasifica tickets usando OpenAI GPT-4o-mini si `OPENAI_API_KEY` está configurada
- Fallback a clasificación por keywords si no hay API key
- Detección de contenido sensible SIEMPRE local (nunca se envía a APIs externas)

### App Streamlit (`app.py`)
- Versión alternativa del MVP en Streamlit
- Misma funcionalidad que el frontend web
- Para correr localmente: `pip install -r requirements.txt && streamlit run app.py`

## Sistema de Clasificación

### Categorías y Keywords

| Categoría | Keywords | Prioridad | Auto-resolvable |
|-----------|----------|-----------|-----------------|
| Reset de Contraseñas | contraseña, password, clave, no puedo entrar, acceso bloqueado, login | Baja | Sí |
| VPN / Conectividad | vpn, conectividad, red, internet, wifi, conexion | Media | Sí |
| Software | instalar, software, programa, aplicacion, actualizar | Media | Sí |
| Permisos de Acceso | permiso, acceso, carpeta, compartir, drive, folder | Media | No (requiere aprobación de jefe) |
| Hardware | mouse, teclado, monitor, pantalla, hardware, cable | Baja | No (intervención física) |
| Impresoras | impresora, imprimir, impresion, escaner | Baja | Sí |
| Servidor/Infraestructura | servidor caido, servidor de produccion, se cayo, brecha de seguridad, hackearon, ransomware | Crítica | No (siempre humano) |
| RRHH - Confidencial | acoso, hostigamiento, discriminacion, denuncia, abuso, discapacidad, embarazo, violencia, sexual, intimidacion, mobbing, represalia | Crítica | NUNCA (routing directo a RRHH) |

### Base de Conocimiento (Wiki de Marcos)
Las respuestas automáticas están en el objeto `KB` del frontend y `KNOWLEDGE_BASE` del Streamlit. Simulan los ~200 artículos de la wiki de Marcos en Notion. Cubren:
- Reset de contraseña (Google Workspace)
- Configuración/troubleshooting VPN
- Instalación de software aprobado (Chrome, Slack, Zoom, VS Code, Postman, Figma, Office 365, Notion, Docker)
- Problemas de impresora

### Flujo de Decisión
```
Email llega → ¿Contenido sensible?
  → SÍ → Derivar a RRHH (nunca toca IA externa)
  → NO → Clasificar (IA o keywords)
    → ¿Auto-resolvable?
      → SÍ → Respuesta automática de KB + nota "si no funciona, responde"
      → NO → ¿Crítico?
        → SÍ → Escalado inmediato a Marcos + Ricardo
        → NO → En cola para agente humano
```

### Flujos de Aprobación
- **Reset contraseña:** Sin aprobación (solo verificar identidad)
- **Permisos de acceso:** Aprobación del jefe directo
- **Software en lista aprobada:** Sin aprobación
- **Software fuera de lista:** Aprobación de Patricia Vega (directora)

## Restricciones Importantes

1. **Privacidad:** Patricia no quiere datos sensibles en servidores externos. Datos de RRHH NUNCA van a APIs de IA.
2. **Legal:** Cumplimiento con Ley Federal de Protección de Datos Personales (México). No datos personales en servicios de terceros sin consentimiento.
3. **Idioma:** Todo en español. Los empleados no hablan inglés.
4. **Adopción:** La herramienta DEBE adaptarse al flujo actual (email). La gente no va a usar otra plataforma (lección de Freshdesk).
5. **Marcos:** Excelente técnico pero resistente al cambio. Posicionar como "liberador de su tiempo", no reemplazo.

## SLAs Objetivo
- Crítico (servidor caído): 15 minutos
- Alto (no puede trabajar): 1 hora
- Medio (trabaja parcialmente): 4 horas
- Bajo (solicitudes): 24 horas

## Métricas de Éxito
- 30-50% resolución automática sin humano
- < 5 min tiempo de primera respuesta (IA)
- Clasificación correcta > 85%
- Presupuesto < $500 USD/mes

## Deploy
- **Vercel:** `index.html` + `vercel.json` (sitio estático). Framework: Other, Build: vacío, Output: `.`
- **Streamlit Cloud:** Alternativa para `app.py`
- **Local:** `pip install -r requirements.txt && streamlit run app.py`

## Roadmap
1. Integración real con Gmail API (recepción automática)
2. RAG con wiki de Marcos exportada de Notion
3. Notificaciones Slack para tickets críticos
4. Flujos de aprobación automatizados
5. Reportes mensuales automáticos para Patricia

## Stack
- Frontend: HTML + Tailwind CSS + Chart.js
- Alternativa: Python + Streamlit + Plotly
- IA: OpenAI GPT-4o-mini (opcional, funciona sin API key con keywords)
- Deploy: Vercel (estático) / Streamlit Cloud
