### Reviviendo Xeon Phi 5110P para IA y HPC en distribuciones modernas de Linux
![image](https://github.com/user-attachments/assets/9b96f1f2-4ff7-40d8-8ef9-6211b9510597)
En el sector de educación básica de Lima, Perú, hay una creciente demanda de soluciones de IA rentables para apoyar el aprendizaje personalizado, evaluaciones automatizadas y sistemas de tutoría inteligente. Sin embargo, desplegar modelos de IA, como modelos con 7B y 14B parámetros, requiere recursos computacionales sustanciales, a menudo fuera del alcance financiero de las instituciones con presupuestos limitados.

El Intel Xeon Phi 5110P ofrece una alternativa potencial de bajo costo para cargas de trabajo de IA debido a sus altas capacidades de computación paralela. Sin embargo, su adopción en sistemas operativos modernos, como AlmaLinux 8.10 (AlmaLinux, 2025), se ve obstaculizada por la falta de controladores y utilidades fácilmente disponibles. Dado que Intel ha descontinuado el soporte para la arquitectura Xeon Phi, las distribuciones de Linux más recientes ya no incluyen los componentes de software necesarios para utilizar plenamente estos coprocesadores.

Este desafío limita a educadores y desarrolladores a reutilizar el hardware Xeon Phi existente para inferencia y entrenamiento de IA. Abordar este problema requiere desarrollar o adaptar el soporte de controladores y utilidades para garantizar la compatibilidad total con AlmaLinux 8.10. Este esfuerzo permitiría a las instituciones de bajo presupuesto aprovechar el hardware más antiguo para herramientas educativas basadas en IA, fomentando la inclusión tecnológica y la innovación en el sector educativo de Lima.

Esta investigación explora la viabilidad de recompilar los controladores del Xeon Phi 5110P en AlmaLinux 8.10 para admitir cargas de trabajo de IA en el sector educativo de Lima. AlmaLinux 8.10 se elige por su soporte a largo plazo hasta 2029, compatibilidad binaria con RHEL y estabilidad, lo que lo convierte en una solución práctica para superar la ausencia de controladores oficiales. Garantizar la compatibilidad con los marcos de trabajo modernos de IA proporcionaría recursos computacionales de bajo costo y escalables, permitiendo a las instituciones educativas maximizar sus inversiones en hardware existente para soluciones de aprendizaje basadas en IA.

**Revisión de la literatura**

Keijser (2024) proporciona una guía completa sobre la implementación de la Manycore Platform Software Stack (MPSS) para Xeon Phi en CentOS 8, abordando los desafíos de compatibilidad y configuración en sistemas operativos modernos. Este estudio se basa en el trabajo de Keijser adaptado para AlmaLinux 8.1, aprovechando su compatibilidad binaria con RHEL y soporte extendido. La metodología descrita por Keijser (2024) ha sido una referencia crítica para adaptar el software MPSS en AlmaLinux, garantizando una solución funcional para entornos educativos en Perú.

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

   **Paso 4: Instalación de Librerías Requeridas y el Daemon**  

Para ejecutar el daemon **MPSS (mpss-daemon)**, es necesario instalar sus dependencias requeridas. Asegúrate de tener **MPSS 3.8.6** instalado, ya que este proporciona los paquetes RPM necesarios. Instalar las dependencias necesarias para ejecutar el daemon MPSS: 

```sh
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif0-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif-dev-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh /mnt/centosroot/root/mpss-3.8.6/libscif-doc-3.8.6-1.glibc2.12.x86_64.rpm
rpm -ivh mpss-daemon-3.8.6-4.el8.x86_64.rpm
rpm -ivh https://github.com/ruribe17/Xeon-Phi-5110P-KNC-Drivers-Utilities-for-AlmaLinux-8.1/blob/main/mpss-daemon-3.8.6-4.el8.x86_64.rpm
```

Con esto, se garantizará que el daemon **MPSS** tenga todas las dependencias necesarias para funcionar correctamente en **MPSS 3.8.6**. 

### Paso 5: Orden de Instalación de MPSS 3.8.6 RPM  

1. Instalar el paquete **libmicmgmt0**:  
   ```
   rpm -ivh glibc2.12pkg-libmicmgmt0-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
2. Instalar los paquetes de desarrollo y documentación de **libmicmgmt**:  
   ```
   rpm -ivh glibc2.12pkg-libmicmgmt-doc-3.8.6-1.glibc2.12.x86_64.rpm glibc2.12pkg-libmicmgmt-dev-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
3. Instalar el paquete **mpss-miccheck-bin**:  
   ```
   rpm -ivh mpss-miccheck-bin-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
4. Crear un enlace simbólico para Python:  
   ```
   sudo ln -s /usr/bin/python3 /usr/bin/python
   ```  
5. Instalar el paquete **libmicaccesssdk0**:  
   ```
   rpm -ivh glibc2.12pkg-libmicaccesssdk0-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
6. Instalar el paquete de desarrollo **libmicaccesssdk-dev**:  
   ```
   rpm -ivh glibc2.12pkg-libmicaccesssdk-dev-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
7. Instalar **libodmdebug0** y su paquete de desarrollo:  
   ```
   rpm -ivh glibc2.12pkg-libodmdebug0-3.8.6-1.glibc2.12.x86_64.rpm glibc2.12pkg-libodmdebug-dev-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
8. Instalar **libsettings0** y su paquete de desarrollo:  
   ```
   rpm -ivh glibc2.12pkg-libsettings0-3.8.6-1.glibc2.12.x86_64.rpm glibc2.12pkg-libsettings-dev-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
9. Instalar **mpss-flash**:  
   ```
   rpm -ivh glibc2.12pkg-mpss-flash-3.8.6-1.glibc2.12.x86_64.rpm
   ```  
10. Instalar los paquetes relacionados con el kernel:  
    ```
    rpm -ivh glibc2.12pkg-mpss-memdiag-kernel-3.8.6-1.glibc2.12.x86_64.rpm glibc2.12pkg-mpss-rasmm-kernel-3.8.6-1.glibc2.12.x86_64.rpm
    ```  
11. Instalar **mpss-micmgmt** y su paquete de Python:  
    ```
    rpm -ivh mpss-micmgmt-3.8.6-1.glibc2.12.x86_64.rpm mpss-micmgmt-python-3.8.6-1.glibc2.12.x86_64.rpm
    ```  
    🔴 **Nota de error**: Se detectó un problema de compatibilidad con Python (`SyntaxError: invalid syntax`). Se recomienda verificar la compatibilidad con Python 2 o modificar el script.  
12. Instalar **mpss-miccheck**:  
    ```
    rpm -ivh mpss-miccheck-3.8.6-1.glibc2.12.x86_64.rpm
    ```  
    🔴 **Nota de error**: Se detectó un error similar en la sintaxis de Python. Se requiere una corrección.  
13. Instalar paquetes adicionales de MPSS:  
    ```
    rpm -ivh mpss-3.8.6/mpss-micmgmt-doc-3.8.6-1.glibc2.12.x86_64.rpm mpss-boot-files-3.8.6-1.glibc2.12.x86_64.rpm mpss-coi-3.8.6-1.glibc2.12.x86_64.rpm mpss-3.8.6/mpss-coi-doc-3.8.6-1.glibc2.12.x86_64.rpm mpss-coi-dev-3.8.6-1.glibc2.12.x86_64.rpm
    ```  
14. Revisar compatibilidad antes de instalar los paquetes del daemon y la licencia:  
    ```
    rpm -ivh mpss-daemon-dev-3.8.6-1.glibc2.12.x86_64.rpm  # Comprobar compatibilidad con mpss-daemon
    rpm -ivh mpss-license-3.8.6-1.glibc2.12.x86_64.rpm      # Verificar su función
    ```  
15. Instalar paquetes para la GUI y headers del módulo:  
    ```
    rpm -ivh mpss-micsmc-gui-3.8.6-1.glibc2.12.x86_64.rpm mpss-modules-headers-3.8.6-1.glibc2.12.x86_64.rpm  # Asegurar compatibilidad con módulos modificados
    ```  
16. Instalar los componentes restantes de MPSS:  
    ```
    rpm -ivh mpss-mpm-3.8.6-1.glibc2.12.x86_64.rpm mpss-mpm-doc-3.8.6-1.glibc2.12.x86_64.rpm mpss-myo-3.8.6-1.glibc2.12.x86_64.rpm mpss-myo-dev-3.8.6-1.glibc2.12.x86_64.rpm mpss-myo-doc-3.8.6-1.glibc2.12.x86_64.rpm mpss-core-3.8.6-1.glibc2.12.x86_64.rpm
    ```  

### Paso 6: Instalación de Scripts de Red  

Ejecutar el siguiente comando para instalar los scripts de red necesarios para configurar direcciones IP dinámicas:  
```
sudo dnf install network-scripts -y
```  
Esto asegura que los comandos `ifup` y `ifdown` estén disponibles para configurar las direcciones IP.  

---

### Paso 7: Inicializar Configuración por Defecto para Xeon Phi  

```
micctrl --initdefaults
```  
🔴 **Nota**: Si aparece el siguiente error:  
```
[Error] mic0: Create failed for /etc/ssh/rsa1 keys: Unknown error 255
```  
Proceda a iniciar el servicio MPSS manualmente.  

---

### Paso 8: Iniciar y Habilitar el Servicio MPSS  

```
systemctl start mpss
systemctl enable mpss
```  
Esto iniciará la pila de software **Manycore Platform Software Stack (MPSS)** y asegurará que se ejecute en cada inicio del sistema.

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
