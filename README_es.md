### Reviviendo Xeon Phi 5110P para IA y HPC en distribuciones modernas de Linux

En el sector de educación básica de Lima, Perú, hay una creciente demanda de soluciones de IA rentables para apoyar el aprendizaje personalizado, evaluaciones automatizadas y sistemas de tutoría inteligente. Sin embargo, desplegar modelos de IA, como modelos con 7B y 14B parámetros, requiere recursos computacionales sustanciales, a menudo fuera del alcance financiero de las instituciones con presupuestos limitados.

El Intel Xeon Phi 5110P ofrece una alternativa potencial de bajo costo para cargas de trabajo de IA debido a sus altas capacidades de computación paralela. Sin embargo, su adopción en sistemas operativos modernos, como AlmaLinux 8.1, se ve obstaculizada por la falta de controladores y utilidades fácilmente disponibles. Dado que Intel ha descontinuado el soporte para la arquitectura Xeon Phi, las distribuciones de Linux más recientes ya no incluyen los componentes de software necesarios para utilizar plenamente estos coprocesadores.

Este desafío limita a educadores y desarrolladores a reutilizar el hardware Xeon Phi existente para inferencia y entrenamiento de IA. Abordar este problema requiere desarrollar o adaptar el soporte de controladores y utilidades para garantizar la compatibilidad total con AlmaLinux 8.1. Este esfuerzo permitiría a las instituciones de bajo presupuesto aprovechar el hardware más antiguo para herramientas educativas basadas en IA, fomentando la inclusión tecnológica y la innovación en el sector educativo de Lima.

Esta investigación explora la viabilidad de recompilar los controladores del Xeon Phi 5110P en AlmaLinux 8.1 para admitir cargas de trabajo de IA en el sector educativo de Lima. AlmaLinux 8.1 se elige por su soporte a largo plazo hasta 2029, compatibilidad binaria con RHEL y estabilidad, lo que lo convierte en una solución práctica para superar la ausencia de controladores oficiales. Garantizar la compatibilidad con los marcos de trabajo modernos de IA proporcionaría recursos computacionales de bajo costo y escalables, permitiendo a las instituciones educativas maximizar sus inversiones en hardware existente para soluciones de aprendizaje basadas en IA.

**Revisión de la literatura**

Keijser (2024) proporciona una guía completa sobre la implementación de la Manycore Platform Software Stack (MPSS) para Xeon Phi en CentOS 8, abordando los desafíos de compatibilidad y configuración en sistemas operativos modernos. Este estudio se basa en el trabajo de Keijser adaptado para AlmaLinux 8.1, aprovechando su compatibilidad binaria con RHEL y soporte extendido. La metodología descrita por Keijser (2024) ha sido una referencia crítica para adaptar el software MPSS en AlmaLinux, garantizando una solución funcional para entornos educativos en los EE. UU.

**Instalación del módulo mic.ko en AlmaLinux 8.10**

Primero, puedes optar por usar el módulo precompilado proporcionado en GitHub. Si prefieres o necesitas una compilación personalizada, asegúrate de seguir las instrucciones para compilar el módulo por ti mismo, especialmente si tu versión del kernel es diferente. Después de compilar con éxito el módulo, procede a instalar el módulo recién compilado.

   **Paso 1: Instalación del módulo compilado**

Para instalar el módulo precompilado, usa el siguiente comando:

```rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/releases/download/v0.1.0-alpha/mpss-modules-4.18.0-553.40.1.el8_10.x86_64-3.8.6-8.x86_64.rpm```

Este comando registra el módulo `mic.ko` en el sistema. Nota: Si tu versión del kernel es diferente de la mencionada aquí, necesitarás seguir los pasos de la siguiente sección para construir el módulo desde el código fuente.

   **Paso 2: Crear un enlace simbólico para mic.ko**

Navega al directorio del módulo del kernel y crea un enlace simbólico para el módulo `mic.ko`:

```
cd /lib/modules/4.18.0-553.40.1.el8_10.x86_64/weak-updates/
ln -s /lib/modules/4.18.0-553.40.1.el8_10.x86_64/extra/mic.ko ./mic.ko
```

Esto asegura que el sistema reconozca el módulo en la ubicación correcta.

   **Paso 3: Cargar y verificar el módulo**

Finalmente, carga el módulo `mic.ko` en el kernel usando:

```modprobe mic```

Para confirmar que el módulo se cargó correctamente, verifica con:

```lsmod | grep mic```

Si el módulo está correctamente instalado y cargado, deberías ver `mic` en la salida.

**Compilación del módulo mic.ko en AlmaLinux 8.10**

El Intel Xeon Phi 5110P requiere el módulo de kernel `mic.ko` para funcionar correctamente, pero dado que el soporte oficial para la Manycore Platform Software Stack (MPSS) ha sido descontinuado, es necesario construir e instalar este módulo manualmente. Esta guía describe el proceso para compilar e instalar el módulo `mic.ko` en AlmaLinux 8.10.

  **Paso 1: Preparar el sistema**

Antes de comenzar, asegúrate de que tu sistema tenga instaladas las herramientas de desarrollo necesarias. Comienza instalando los paquetes requeridos:

```
dnf groupinstall -y "Development Tools"

dnf install -y rpm-build kernel-devel-$(uname -r) kernel-headers-$(uname -r) elfutils-libelf-devel gcc make dnf-utils
```

Esto instala los compiladores esenciales, archivos de desarrollo del kernel y utilidades necesarias para construir módulos del kernel.

   **Paso 2: Configurar el entorno de construcción de RPM**

A continuación, crea la estructura de directorios necesaria para construir los paquetes RPM:

```mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}```

Esto asegura que todos los directorios necesarios existan antes de proceder con el proceso de construcción de RPM.

   **Paso 3: Instalar el RPM de código fuente**

Ahora, instala el paquete RPM de código fuente de MPSS:

```rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/releases/download/v0.1.0-alpha/mpss-modules-3.8.6-8.src.rpm```

Esto extrae los archivos fuente en el directorio `rpmbuild`, que se utilizarán para compilar el módulo.

   **Paso 4: Editar el archivo SPEC**

Navega al directorio `SPECS` y abre el archivo SPEC para editarlo:

```
cd ~/rpmbuild/SPECS
vi mpss-modules-3.8.6-rhel85.spec
```

En este paso, verifica que la versión del kernel coincida con la que tienes actualmente en tu sistema. Puedes verificar la versión de tu kernel con:

```uname -r```

Por ejemplo, si el resultado es `4.18.0-553.40.1.el8_10.x86_64`, actualiza el archivo SPEC en consecuencia.

   **Paso 5: Aplicar el parche del kernel**

Copia el archivo de parche del kernel adecuado al directorio `SOURCES`:

```cp /root/rpmbuild/SOURCES/mpss-modules-4.18.0-553.37.1.el8_10.x86_64.patch /root/rpmbuild/SOURCES/mpss-modules-4.18.0-553.40.1.el8_10.x86_64.patch```

Esto asegura que el parche esté alineado con la versión del kernel actualmente instalada.

   **Paso 6: Construir el paquete RPM**

Una vez que hayas realizado las modificaciones necesarias, construye el paquete RPM con:

```rpmbuild -bb mpss-modules-3.8.6-rhel85.spec```

Este comando compilará el módulo del kernel y lo empaquetará en un RPM instalable.

   **Conclusión**

Siguiendo estos pasos, has compilado, instalado y cargado correctamente el módulo `mic.ko` en AlmaLinux 8.10. Este proceso permite el uso continuo del hardware Xeon Phi 5110P a pesar de la descontinuación del soporte oficial. Con este módulo en su lugar, ahora puedes explorar aplicaciones de IA y HPC utilizando este hardware.

**Referencias**

Keijser, J. J. (2023). *MPSS for Xeon Phi 5110P KNC drivers and utilities* (v3.8.6). GitHub. https://github.com/jjkeijser/mpss

Keijser, J. J. (2022). *Xeon Phi MPSS en CentOS 8*. Recuperado de https://jjkeijser.github.io/mpss/xeon-phi-mpss-centos8.html

xdsopl. (2015). mpss-modules [GitHub repository]. GitHub. Retrieved February 26, 2025, from https://github.com/xdsopl/mpss-modules
