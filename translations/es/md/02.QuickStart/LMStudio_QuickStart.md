# **Usando Phi-3 en LM Studio**

[LM Studio](https://lmstudio.ai) es una aplicación para llamar SLM y LLM en una aplicación de escritorio local. Permite a los usuarios utilizar fácilmente diferentes modelos y soporta computación acelerada usando NVIDIA/AMD GPU/Apple Silicon. A través de LM Studio, los usuarios pueden descargar, instalar y ejecutar varios LLM y SLM de código abierto basados en Hugging Face para probar el rendimiento del modelo localmente sin necesidad de codificar.

## **1. Instalación**

![LMStudio](../../../../translated_images/LMStudio.87422bdb03d330dc05137ba237dd0cb43f7964245b848a466ab1730de93bc4db.es.png)

Puedes elegir instalar en Windows, Linux, macOS a través del sitio web de LM Studio [https://lmstudio.ai/](https://lmstudio.ai/)

## **2. Descargar Phi-3 en LM Studio**

LM Studio llama a modelos de código abierto en formato gguf cuantizado. Puedes descargarlo directamente desde la plataforma proporcionada por LM Studio Search UI, o puedes descargarlo tú mismo y especificar que se llame en el directorio relevante.

***Buscamos Phi3 en LM Studio Search y descargamos el modelo gguf de Phi-3***

![LMStudioSearch](../../../../translated_images/LMStudio_Search.1e577e0f69f336fc26e56653eeec2a20b90c3895cc4aa2ff05b6ec51059f12fd.es.png)

***Gestionar los modelos descargados a través de LM Studio***

![LMStudioLocal](../../../../translated_images/LMStudio_Local.55f9d6f61eb27f0f37fc4833599aa43fa45a66dfc20444ba1419a922b60b5005.es.png)

## **3. Chatear con Phi-3 en LM Studio**

Seleccionamos Phi-3 en LM Studio Chat y configuramos la plantilla de chat (Preset - Phi3) para comenzar el chat local con Phi-3

![LMStudioChat](../../../../translated_images/LMStudio_Chat.1bdc3a8f804f12d9548b386448c1642b741c10816576973155a90ef55f8a9c8d.es.png)

***Nota***:

a. Puedes configurar parámetros a través de la Configuración Avanzada en el panel de control de LM Studio

b. Debido a que Phi-3 tiene requisitos específicos de plantilla de chat, se debe seleccionar Phi-3 en Preset

c. También puedes configurar diferentes parámetros, como el uso de GPU, etc.

## **4. Llamar a la API de Phi-3 desde LM Studio**

LM Studio soporta el despliegue rápido de servicios locales, y puedes construir servicios de modelo sin necesidad de codificar.

![LMStudioServer](../../../../translated_images/LMStudio_Server.917c115e12599e7698ce323085ce4f8bdb020665656bbe90edca2d45a7de932d.es.png)

Este es el resultado en Postman

![LMStudioPostman](../../../../translated_images/LMStudio_Postman.4481aa4873ecaae0e05032f539090897002fc9aca9da5d1336fb28776f4c45a7.es.png)

**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando servicios de traducción automática basados en IA. Si bien nos esforzamos por lograr precisión, tenga en cuenta que las traducciones automáticas pueden contener errores o imprecisiones. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda la traducción humana profesional. No nos hacemos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.