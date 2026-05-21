# RAG Chatbot — Qwen + FAISS + Gradio

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![Qwen](https://img.shields.io/badge/Qwen-1.5--0.5B-purple)
![FAISS](https://img.shields.io/badge/Vector_Search-FAISS-orange)
![Gradio](https://img.shields.io/badge/UI-Gradio-yellow?logo=gradio)
![License](https://img.shields.io/badge/License-MIT-green)

> Chatbot de recuperación aumentada (RAG) sobre documentos propios.  
> Soporta PDF, DOCX, TXT y XLSX. Respuestas generadas con Qwen1.5-0.5B-Chat a partir de contexto recuperado semánticamente con FAISS.

---

## ¿Qué hace este proyecto?

Permite hacer preguntas en lenguaje natural sobre cualquier documento que el usuario suba — PDF, Word, texto plano o Excel. El sistema no improvisa: cada respuesta está fundamentada en fragmentos reales del documento.

**Pipeline completo:**

```
Documento subido (PDF / DOCX / TXT / XLSX)
    ↓
Extracción y fragmentación en chunks con control de tamaño
    ↓
Embeddings semánticos — all-MiniLM-L6-v2 (sentence-transformers)
    ↓
Índice vectorial FAISS — búsqueda por similitud coseno
    ↓
Recuperación de chunks relevantes a la pregunta
    ↓
Generación de respuesta — Qwen1.5-0.5B-Chat
    ↓
Interfaz conversacional — Gradio
```

---

## ¿Por qué importa?

Este proyecto demuestra la capacidad de construir un sistema RAG funcional de extremo a extremo — desde la ingesta de documentos hasta la interfaz de usuario — sin depender de APIs externas de pago.

Lo que se trabaja aquí va más allá de "usar LangChain con dos líneas":
- **Fragmentación controlada:** el tamaño y solapamiento de los chunks afecta directamente la calidad de las respuestas; se implementa con control explícito de caracteres
- **Búsqueda semántica real:** FAISS permite recuperación por similitud vectorial, no por palabras clave
- **Modelo local:** Qwen1.5-0.5B-Chat corre completamente en Colab sin costos de API — el sistema es autocontenido
- **Multi-formato:** la misma pipeline maneja cuatro tipos de archivo distintos sin cambiar el flujo

**Hallazgo clave:** ajustar el número máximo de caracteres por chunk mejora significativamente la coherencia de las respuestas al dar al modelo más contexto continuo sin exceder la ventana de entrada.

---

## Stack técnico

| Componente | Herramienta |
|-----------|-------------|
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` |
| Índice vectorial | FAISS |
| Generación | Qwen1.5-0.5B-Chat (HuggingFace) |
| Interfaz | Gradio |
| Parseo PDF | PyMuPDF / pdfplumber |
| Parseo DOCX | python-docx |
| Parseo XLSX | openpyxl |

---

## Formatos soportados

| Formato | Librería | Estrategia de extracción |
|---------|---------|--------------------------|
| `.pdf` | PyMuPDF | Extracción página a página |
| `.docx` | python-docx | Párrafos y tablas |
| `.txt` | built-in | Lectura directa |
| `.xlsx` | openpyxl | Filas como texto serializado |

---

## Estructura del repositorio

```
rag-chatbot-qwen-faiss/
│
├── RAG_Chatbot.ipynb            ← Notebook principal (Colab-ready)
├── README.md
├── requirements.txt
└── sample_docs/                 ← Documentos de prueba opcionales
```

---

## Instalación y ejecución

```bash
# Opción recomendada — Google Colab
# Abrir RAG_Chatbot.ipynb en Colab
# El notebook instala dependencias e inicia Gradio automáticamente

# Local
git clone https://github.com/byron-velasco/rag-chatbot-qwen-faiss.git
cd rag-chatbot-qwen-faiss
pip install -r requirements.txt
jupyter notebook RAG_Chatbot.ipynb
```

**requirements.txt**
```
transformers>=4.35
sentence-transformers
faiss-cpu
gradio
pymupdf
python-docx
openpyxl
torch
accelerate
```

---

## Decisiones metodológicas

**¿Por qué FAISS y no una búsqueda simple por keywords?**  
La búsqueda por palabras clave falla cuando el usuario pregunta de forma diferente a como está escrito el documento. FAISS compara representaciones semánticas — "¿cuál es el costo?" encuentra fragmentos que mencionan "precio", "tarifa" o "inversión" sin que aparezca la palabra exacta.

**¿Por qué Qwen1.5-0.5B y no un modelo más grande?**  
El objetivo era un sistema funcional en Google Colab sin GPU dedicada. Qwen1.5-0.5B-Chat corre en CPU con tiempos aceptables y produce respuestas coherentes cuando el contexto recuperado es relevante. Para producción con más recursos, el modelo se puede reemplazar sin cambiar el resto de la pipeline.

**¿Por qué control explícito del tamaño de chunk?**  
Chunks muy pequeños pierden contexto; chunks muy grandes exceden la ventana del modelo. Parametrizar el tamaño máximo de caracteres permite calibrar este tradeoff según el tipo de documento y el modelo usado.

**¿Por qué multi-formato?**  
En escenarios reales la documentación no viene en un solo formato. Soportar PDF, DOCX, TXT y XLSX desde la misma pipeline hace el sistema directamente utilizable sin conversiones previas.

---

## Licencia

MIT — código libre para uso, modificación y distribución.
