# Chatbot de WhatsApp para atención al cliente de ecommerce

Flujo de n8n que recibe mensajes de WhatsApp a través de WACRM, utiliza Gemini para generar respuestas y permite derivar conversaciones a un asesor.

## Problema que resuelve

En una tienda online, las preguntas sobre productos, pedidos y atención general pueden acumularse y retrasar la respuesta al cliente. Este flujo automatiza la recepción y clasificación inicial de mensajes, responde con ayuda de IA y facilita el paso a atención humana cuando hace falta.

## Cómo funciona

1. WACRM envía un evento entrante al webhook de n8n.
2. El flujo valida la firma del webhook y obtiene los datos del contacto y de la conversación.
3. Prepara el mensaje para Gemini. Si incluye una imagen, intenta obtenerla y añadirla al contexto.
4. El agente de IA genera una respuesta usando memoria de conversación.
5. n8n envía la respuesta por WACRM. Si se detecta una transferencia, marca el contacto para que lo atienda un asesor.
6. Para mensajes de audio o video, el flujo puede enviar un aviso y marcar la conversación para atención humana.

## Herramientas

- **n8n:** orquesta el webhook, las validaciones, el procesamiento con IA y las llamadas a la API de WACRM.
- **Gemini AI:** interpreta el mensaje y genera una respuesta. Requiere una credencial de Google Gemini configurada en n8n.
- **WACRM:** recibe los eventos de WhatsApp y envía mensajes y actualizaciones de contacto. La exportación usa su API.
- **Airtable:** puede incorporarse como catálogo editable de productos, preguntas frecuentes o políticas de la tienda. **No está conectado en el JSON incluido**; hay que añadir y configurar los nodos correspondientes para que Gemini consulte esos datos.
- **Supabase:** puede incorporarse para persistir conversaciones, pedidos u otros datos del ecommerce. **No está conectado en el JSON incluido**; se deben añadir los nodos y definir las tablas y operaciones necesarias.

> La exportación actual implementa n8n, Gemini y WACRM. Airtable y Supabase son integraciones pendientes: importarla no las configura ni las utiliza automáticamente.

## Requisitos

- Una instancia de n8n con acceso HTTPS público para recibir webhooks.
- Una cuenta y credenciales de Google Gemini configurables en n8n.
- Una instancia de WACRM conectada a WhatsApp y con acceso a su API.
- El secreto usado por WACRM para firmar los webhooks.
- Para integrar Airtable y Supabase: sus cuentas, credenciales y los nodos correspondientes en n8n.

## Instalación

1. Descarga o clona este repositorio y abre el archivo `Webhook entrada.json` en la carpeta `chatbot-eccomerce`.
2. En n8n, importa el archivo desde **Workflows → Import from File**.
3. Configura las credenciales de Google Gemini en el nodo **Google Gemini Chat Model**.
4. Configura las credenciales de API de WACRM en los nodos HTTP Request que consultan contactos y conversaciones, envían mensajes y actualizan contactos.
5. En el entorno de n8n, define `WACRM_WEBHOOK_SECRET` con el mismo secreto configurado para firmar los eventos en WACRM. No guardes este valor directamente en el workflow ni en el repositorio.
6. Guarda y activa el workflow. Copia la URL de producción del nodo **Webhook** y configúrala en WACRM para eventos entrantes. La ruta definida en el flujo es `wacrm-inbound` y el método es `POST`.
7. Envía un mensaje de prueba y revisa las ejecuciones en n8n. Confirma la recepción, la validación de firma y el envío de la respuesta antes de ponerlo en producción.
8. Si necesitas Airtable o Supabase, añade sus nodos y credenciales en n8n, conecta las operaciones de consulta o escritura al flujo y prueba esos caminos por separado. La exportación actual no incluye ese cableado.

## Seguridad y operación

- Mantén las claves de API y `WACRM_WEBHOOK_SECRET` en las credenciales o variables de entorno de n8n.
- Usa HTTPS y limita el acceso a la instancia de n8n.
- Revisa las ejecuciones fallidas y ajusta las respuestas y reglas de transferencia antes de habilitar el flujo para todos los clientes.