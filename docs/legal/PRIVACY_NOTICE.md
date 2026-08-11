# Aviso de privacidad y tratamiento de datos personales (BORRADOR)

> **Estado: BORRADOR técnico, no vigente.** Este documento fue redactado por un asistente de IA a partir de las políticas técnicas del repositorio (`docs/SECURITY_AND_PRIVACY.md`, `docs/security/IDENTITY_VERIFICATION_POLICY.md`, `docs/security/SAFE_REUNIFICATION_POLICY.md`, `docs/security/SAFE_PETS_POLICY.md`) y de una lectura general de la Ley 1581 de 2012, su Decreto reglamentario 1377 de 2013 y el artículo 15 de la Constitución Política de Colombia. **No es concepto jurídico y no debe publicarse tal cual.** Antes de operar con datos reales, la entidad responsable debe hacerlo revisar, completar y aprobar por su oficina jurídica y su responsable de protección de datos, según exige `docs/SECURITY_AND_PRIVACY.md`. Todos los campos entre corchetes `[ ]` deben completarse antes de publicar.

## 1. Responsable del tratamiento

- **Responsable:** [NOMBRE DE LA ENTIDAD OPERADORA — p. ej. alcaldía, gobernación, organismo de socorro].
- **NIT / identificación:** [COMPLETAR].
- **Domicilio:** [COMPLETAR].
- **Contacto para ejercer derechos de datos:** [CORREO/CANAL DEL RESPONSABLE O ENCARGADO DE PROTECCIÓN DE DATOS].
- **Registro Nacional de Bases de Datos (RNBD):** [COMPLETAR ESTADO DE INSCRIPCIÓN ANTE LA SIC].

SOS Eje Cafetero es software; quien lo despliega para atender una emergencia real es quien asume el rol de Responsable del Tratamiento ante la Ley 1581 de 2012, no el proyecto de código abierto.

## 2. Principio rector: pedir ayuda nunca exige datos

Reportar una emergencia por GPS (`FEATURE_PUBLIC_SOS`) **nunca** requiere cuenta, documento de identidad ni biometría. El flujo reforzado de identidad (expediente de damnificado) es un proceso aparte, voluntario, para el registro formal y la entrega de ayuda — nunca una condición para ser rescatado.

## 3. Qué datos se recogen, para qué y con qué base legal

| Capacidad | Datos | Finalidad | Base legal orientativa |
|---|---|---|---|
| Reporte SOS público | Ubicación GPS, descripción, foto/adjunto opcional | Coordinar respuesta de emergencia inmediata | Protección de un interés vital del titular o de terceros (situación de emergencia); no requiere autorización previa |
| Personas/animales desaparecidos | Nombre, descripción, última ubicación conocida, foto | Búsqueda, deduplicación y coordinación | Interés vital / consentimiento de quien reporta |
| Reencuentro (`FEATURE_REUNIFICATION`) | Teléfono (solo como hash HMAC irreversible), estado de la solicitud | Notificar a un posible familiar sin exponer si el número existe, inició sesión o su ubicación | Consentimiento explícito y versionado del solicitante; el número objetivo nunca se persiste en texto plano |
| Mascotas seguras (`FEATURE_PET_SAFETY`) | Datos del animal, documento/microchip del tutor (hash HMAC), fotos | Reunificación de mascotas con sus tutores | Consentimiento del reportante |
| Expediente de damnificado | Identidad, documento (hash HMAC, se muestran solo los últimos caracteres), GPS, foto/video de documento y de presencia (liveness), consentimiento versionado | Verificación humana para registro formal y distribución de ayuda | Consentimiento explícito, previo, expreso e informado (ver `examples/affected-consent-v1.txt`) |
| Funcionarios/operadores | Nombre, entidad, celular, rol | Autenticación OTP y trazabilidad de acciones operacionales | Ejecución de funciones públicas / relación contractual o de voluntariado |
| WhatsApp (si está habilitado) | Mensajes entrantes por Cloud API | Canal alterno de reporte | Consentimiento implícito al iniciar contacto |

## 4. Lo que el sistema nunca hace

1. No expone teléfono, documento, evidencia, datos médicos ni coordenadas exactas en vistas públicas.
2. No permite que quien busca un reencuentro sepa si el teléfono objetivo existe, inició sesión, leyó el aviso o está localizado; tampoco si el destinatario ignoró, bloqueó o reportó abuso.
3. No usa IA, similitud, liveness o *scoring* para negar automáticamente un rescate, una ayuda vital o un derecho — toda decisión de identidad o ayuda la toma una persona autorizada y queda auditada.
4. No guarda el número de documento ni el teléfono objetivo en texto plano donde la política exige HMAC; el secreto del HMAC vive fuera de Git y fuera del estado de Terraform.
5. No comparte datos sensibles con terceros salvo mandato legal u orden judicial expresa.

## 5. Cómo se protegen los datos

- Documentos, fotos y videos se guardan cifrados en almacenamiento privado (S3/KMS o MinIO on-prem), nunca en buckets públicos, mediante URLs presignadas de corta duración.
- El acceso operacional exige autenticación OTP, autorización por rol (RBAC) y queda auditado (actor, entidad, marca de tiempo).
- La sincronización offline usa cifrado de extremo a extremo (`SecureEnvelope`: RSA-OAEP-256 + AES-256-GCM + firma ECDSA) cuando `FEATURE_SECURE_ENVELOPE` está activo; si no lo está, el cliente opera en línea y no persiste PII sin cifrar en el dispositivo.
- Toda evidencia sensible pasa por antimalware antes de poder usarse en un expediente o descargarse.

## 6. Retención

Los siguientes son **valores técnicos por defecto del repositorio, no una decisión jurídica**. La entidad operadora debe fijar y documentar su propia política de retención antes de recibir datos reales, y ajustar las variables correspondientes:

- Evidencia de expediente de damnificado: `EVIDENCE_RETENTION_DAYS` (por defecto 90 días).
- Evidencia de mascotas: `PET_EVIDENCE_RETENTION_DAYS` (por defecto 30 días).
- Solicitud de reencuentro: `REUNIFICATION_REQUEST_TTL_DAYS` (por defecto 14 días).
- Copias de seguridad: deben seguir la misma política institucional; un *lifecycle* del bucket por sí solo no borra copias de respaldo externas.

## 7. Derechos del titular (Ley 1581 de 2012, art. 8)

Toda persona cuyos datos se traten tiene derecho a:

1. Conocer, actualizar y rectificar sus datos personales.
2. Solicitar prueba de la autorización otorgada, salvo en los casos exceptuados por el artículo 10 de la Ley 1581 de 2012 (p. ej. datos requeridos para proteger un interés vital durante una emergencia).
3. Ser informado del uso dado a sus datos, previa solicitud.
4. Presentar quejas ante la Superintendencia de Industria y Comercio (SIC) por infracciones a la ley.
5. Revocar la autorización y/o solicitar la supresión del dato cuando no exista un deber legal o contractual que obligue a conservarlo.
6. Acceder de forma gratuita a sus datos.

Para expedientes de damnificados existe además un camino explícito de **corrección** (`NEEDS_INFO`) y **apelación** (`REJECTED → APPEAL`) resuelto siempre por una persona autorizada.

**Canal para ejercer estos derechos:** [COMPLETAR — correo o formulario del Responsable/Encargado].

## 8. Menores de edad y personas vulnerables

Los reportes de personas desaparecidas y reencuentro pueden involucrar menores de edad u otras personas en situación de vulnerabilidad. La entidad operadora debe definir, antes de habilitar estas capacidades con datos reales, un protocolo específico de revisión conforme a la Ley 1098 de 2006 (Código de la Infancia y la Adolescencia) y coordinarlo con la autoridad competente (ICBF u homóloga). `docs/operations/PRODUCTION_GATES.md` exige esta revisión antes del Gate 2.

## 9. Transferencias y encargados

[COMPLETAR: si se usan proveedores externos — AWS Cognito, AWS Rekognition, WhatsApp Cloud API, etc. — deben listarse aquí como Encargados del Tratamiento, con su ubicación y si implican transferencia internacional de datos, conforme al Capítulo V de la Ley 1581 de 2012.]

## 10. Vigencia y versionado

Este aviso debe versionarse igual que los textos de consentimiento del sistema (ver `examples/affected-consent-v1.txt`, versión `affected-consent-v1`), de modo que quede registro de qué versión aceptó cada persona. Versión de este borrador: `privacy-notice-draft-v1` — [FECHA DE APROBACIÓN PENDIENTE].
