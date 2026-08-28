# Taller Práctico Procesamiento del lenguaje natural con modelos probabilísticos

Este taller práctico permite integrar los principales conocimientos desarrollados en el curso mediante la construcción de un asistente básico para la redacción de mensajes bancarios. La solución recibirá un texto escrito por un usuario, detectará posibles errores ortográficos, propondrá palabras corregidas y sugerirá cómo continuar el mensaje a partir de las palabras ingresadas.

Para desarrollar la solución, los participantes aplicarán técnicas de corrección automática basadas en probabilidades y distancia mínima de edición, así como modelos de lenguaje construidos con n-gramas. También utilizarán conceptos de vocabulario, frecuencias de palabras, probabilidades condicionales, suavizado, tratamiento de palabras desconocidas y evaluación del modelo mediante perplejidad.

Como complemento, el notebook permitirá analizar las palabras desde una representación probabilística y vectorial, identificando términos relacionados mediante embeddings entrenados con el modelo Continuous Bag of Words, CBOW. Todos estos componentes corresponden a los contenidos cubiertos en el curso.

## Estructura

- `CapituloXX/README.md`: guía de laboratorio por capítulo.

## Lista de laboratorios

### Capítulo 1

- [Construcción de un asistente probabilístico para corregir y autocompletar mensajes bancarios](Capitulo01/README.md#construcción-de-un-asistente-probabilístico-para-corregir-y-autocompletar-mensajes-bancarios)
  - Descripción: Construir un prototipo de procesamiento del lenguaje natural que corrija errores ortográficos y sugiera la continuación de mensajes, aplicando modelos probabilísticos, distancia mínima de edición, n-gramas y representaciones vectoriales de palabras.

Los participantes trabajarán en un notebook con mensajes bancarios sintéticos. Construirán un corrector ortográfico basado en distancia mínima de edición y probabilidades de palabras, y después desarrollarán un sistema de autocompletado con modelos de lenguaje de n-gramas. Finalmente, integrarán ambas funciones en un prototipo que reciba un mensaje, sugiera correcciones y proponga las palabras más probables para continuarlo.
  - Duración estimada: 105 min

## Flujo de colaboración

- Trabajar en `changes_course`.
- Crear Pull Request hacia `main`.
- Merge por `Squash and merge`.
