# Facephi Challenge 2026: Identity Validation Pipeline
> **Participante:** Santiago Álvarez Geanta
>
> **Grado:** Ingeniería en Inteligencia Artificial
>
> **Correo UA:** sag82@alu.ua.es
---

## 1. Visión General
El objetivo de este desarrollo es crear un sistema de validación de identidad que no solo extraiga texto (OCR), sino que actúe como un **perito digital**. El pipeline se divide en tres capas críticas:
1. **Seguridad Física:** Detección de fraude (Anti-spoofing).
2. **Extracción Resiliente:** OCR con simulación de Monte Carlo.
3. **Integridad Lógica:** Validación cruzada VIZ/MRZ y Checksums ICAO.

---

## 2. Innovaciones Técnicas

### Módulo Anti-Spoofing (Análisis de Frecuencia)
Para prevenir ataques de presentación (fotos a pantallas), implementamos un análisis en el **Dominio de la Frecuencia** mediante la Transformada Rápida de Fourier (FFT).
* **Detección de Moiré:** Identifica patrones de interferencia invisibles al ojo humano pero detectables como picos de energía geométrica en el espectro.
* **Filtro de Altas Frecuencias:** El sistema ignora el centro del espectro (formas generales) para enfocarse en el ruido de rejilla de los píxeles digitales.

[Image of Fast Fourier Transform (FFT) moire pattern detection on a digital screen]

### MRZ: Consenso por Monte Carlo
El OCR tradicional falla ante reflejos. Nuestra solución aplica:
* **Aumentación de Datos In-situ:** Generamos 10 variaciones de la imagen (contraste, brillo, rotación afín).
* **Majority Voting:** Un sistema de votación a nivel de carácter que colapsa las 10 inferencias en una única cadena de texto estadísticamente pura.
* **Corrección por Familia:** Uso de máscaras semánticas para corregir confusiones comunes (ej. 'O' por '0') según la posición definida por la normativa ICAO.

### Cross-Validation (VIZ vs MRZ)
El sistema actúa como árbitro de integridad:
* Compara el nombre, apellidos y DNI extraídos de la cara frontal (**VIZ**) con los datos codificados en el reverso (**MRZ**).
* Utiliza **Fuzzy Matching** (Distancia de Levenshtein) para permitir una tolerancia de lectura del 70%, corrigiendo automáticamente errores de segmentación.

---

## 3. Validación de Atributos Físicos (ISO 7810)
No basta con leer los datos; el documento debe ser real. El módulo `DataValidation` verifica:
* **Ratio de Aspecto ID-1:** Comprobación de dimensiones estándar (1.58).
* **Detección Biométrica:** Localización de rostro mediante Haar Cascades.
* **Análisis de Textura (Chip):** El chip metálico se valida mediante su baja varianza de textura y geometría cuadrada, diferenciándolo de simples manchas de impresión.
* **Ink Ratio (Firma):** Verificación de presencia de trazos en la zona de rúbrica.

[Image of a DNI document with bounding boxes highlighting the chip, face photo, and signature area]

---

## 4. Lógica de Negocio e Integridad ICAO
Implementación estricta del estándar **ICAO Document 9303**:
* **Algoritmo de Pesos 7-3-1:** Validación de los dígitos de control de todos los campos críticos.
* **Suma de Control Compuesta:** Garantiza que ninguna de las tres líneas del MRZ haya sido alterada.
* **Validación de Edad y Vigencia:** Cruce biográfico para asegurar que la caducidad del documento cumple con la normativa española según la edad del titular.

---

## 5. Decision Engine (Veredicto Final)
El sistema emite un juicio basado en la gravedad de los hallazgos:
* **VALIDADO:** Todos los controles superados.
* **REVISIÓN MANUAL:** Incoherencias menores o falta de atributos no críticos (ej. firma).
* **RECHAZADO:** Fallo en Checksums, Moiré detectado o inconsistencia total de datos.

---

## Mejoras sin Implementar
1. Implementación de **Deep Learning para segmentación de CLI** (Imagen láser cambiante).
2. Verificación de microtextos y tintas OVI (ópticamente variables).
3. Verificación extra de datos:
    - lugar de nacimiento
    - nacionalidad
    - número de equipo
    - domicilio
    - CAN (Clave numérica para sesión digital)
    - banderas