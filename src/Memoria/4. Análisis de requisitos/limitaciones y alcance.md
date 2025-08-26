## **Limitaciones y alcance**

### **Alcance incluido**

#### **5.1.1 Funcionalidades Core**

- Compilador completo para el repertorio de instrucciones y comandos BASIC usados por el SHARP PC-1500. Encargado del analisis del código reporta los errores léxicos, sintácticos y semánticos, además de generar la respresentación intemedia y el codigo máquina compatible con el procesador LH5801.
- Emulador funcional del procesador LH5801 y sistema completo. Incluye la ROM autentica de la calculadora necesaria para obtener una ejecución precisa del sistema original.
- Interfaz gráfica que replica la experiencia del usuario original. El código escrito principalmente en Rust utiliza la librería de código abierto *iced*, simple de usar pero con todas las utilidades necesarias para generar una replica completa de la interfaz del SHARP PC-1500. 
--TO DO descripción de la GUI usada en Rust--
- Herramientas de depuración básicas pero efectivas
- Documentación completa del usuario y técnica

#### **5.1.2 Compatibilidad de Software**

- Soporte para programas BASIC estándar del PC-1500
- Compatibilidad con las funciones matemáticas y de strings principales
- Simulación de dispositivos de E/S básicos

#### **5.1.3 Plataformas Objetivo**

- Sistemas operativos de escritorio modernos (Windows, MacOs, Linux)
- Arquitectura de procesador x86-64
- Interfaces gráficas estándar (Qt, GTK, o similar)
