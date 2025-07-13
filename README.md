# -Laboratorios-del-modulo-VI
Practica 3 -Instalacion de IDS snort 
# =====================================================
# GUÍA COMPLETA DE SNORT 3 EN RED HAT (RHEL 8/CentOS 8)
# =====================================================

# === SECCIÓN 1: PREPARACIÓN DEL SISTEMA ===

# 1. Actualización del sistema:
sudo dnf update && sudo yum update
# EXPLICACIÓN: Actualiza todos los paquetes del sistema usando DNF (nuevo) y YUM (antiguo) para garantizar compatibilidad.

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
# EXPLICACIÓN: Instala el repositorio EPEL que contiene paquetes adicionales necesarios para Snort.

sudo subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms
# EXPLICACIÓN: Habilita el repositorio CodeReady Builder que incluye herramientas de desarrollo críticas.

sudo yum upgrade -y
# EXPLICACIÓN: Aplica todas las actualizaciones pendientes de forma automática (-y).

# === SECCIÓN 2: CONFIGURACIÓN DE BIBLIOTECAS ===

sudo nano /etc/ld.so.conf.d/local.conf
# EXPLICACIÓN: Edita el archivo de configuración de bibliotecas. Debes agregar:
/usr/local/lib
/usr/local/lib64
# RAZÓN: Estas líneas aseguran que el sistema encuentre bibliotecas instaladas manualmente.

# === SECCIÓN 3: INSTALACIÓN DE DEPENDENCIAS ===

sudo dnf install flex bison gcc gcc-c++ make cmake autoconf libtool git nano unzip wget libpcap-devel pcre-devel libdnet-devel hwloc-devel openssl-devel zlib-devel luajit-devel pkgconfig pkgconf libunwind-devel libnfnetlink-devel libnetfilter_queue-devel libmnl-devel xz-devel gperftools libuuid-devel hyperscan hyperscan-devel -y
# EXPLICACIÓN: Instala 25+ paquetes esenciales:
# - Herramientas de compilación (gcc, make, cmake)
# - Bibliotecas para procesamiento de red (libpcap, libnfnetlink)
# - Soporte para patrones (hyperscan, pcre)
# - Optimización de memoria (gperftools)

# === SECCIÓN 4: INSTALACIÓN DE LIBDAQ ===

git clone https://github.com/snort3/libdaq.git
# EXPLICACIÓN: Descarga libDAQ (biblioteca de captura de paquetes para Snort 3).

cd libdaq/
./bootstrap
# EXPLICACIÓN: Prepara el entorno de compilación (genera scripts de configuración).

./configure && make && sudo make install
# EXPLICACIÓN: Compila e instala libDAQ en el sistema (3 pasos en 1 comando).

sudo ldconfig
# EXPLICACIÓN: Actualiza la caché de bibliotecas del sistema.

# === SECCIÓN 5: INSTALACIÓN DE SNORT 3 ===

git clone https://github.com/snort3/snort3.git
cd snort3
# EXPLICACIÓN: Descarga el código fuente de Snort 3.

export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig:$PKG_CONFIG_PATH
# EXPLICACIÓN: Configura rutas para que el sistema encuentre bibliotecas personalizadas.

export CFLAGS="-O3"
export CXXFLAGS="-O3 -fno-rtti"
# EXPLICACIÓN: Habilita optimización máxima (-O3) y desactiva RTTI para mejor rendimiento.

./configure_cmake.sh --prefix=/usr/local/snort --enable-tcmalloc
# EXPLICACIÓN: Prepara la compilación con:
# --prefix: Instala Snort en /usr/local/snort
# --enable-tcmalloc: Usa el allocator de Google para mejor manejo de memoria.

cd build/
make -j$(nproc)
# EXPLICACIÓN: Compila Snort usando todos los núcleos del CPU (-j$(nproc)).

sudo make -j$(nproc) install
# EXPLICACIÓN: Instala los archivos compilados en el sistema.

sudo ldconfig
# EXPLICACIÓN: Vuelve a actualizar la caché de bibliotecas.

/usr/local/snort/bin/snort -V
# EXPLICACIÓN: Verifica la instalación mostrando la versión.

sudo ln -s /usr/local/snort/bin/snort /usr/bin/snort
# EXPLICACIÓN: Crea un acceso directo para ejecutar Snort desde cualquier lugar.

# === SECCIÓN 6: CONFIGURACIÓN ===

sudo nano /usr/local/snort/etc/snort/snort.lua
# EXPLICACIÓN: Edita el archivo de configuración principal. Cambia:
HOME_NET = '192.168.0.0/24'
# RAZÓN: Define qué red proteger (ajusta según tu configuración).

snort -T -c /usr/local/snort/etc/snort/snort.lua
# EXPLICACIÓN: Prueba la configuración (-T = test mode).

sudo ip link set dev ens160 promisc on
# EXPLICACIÓN: Habilita modo promiscuo en la interfaz de red para capturar todo el tráfico.

# === SECCIÓN 7: REGLAS PERSONALIZADAS ===

sudo nano /usr/local/snort/etc/snort/local.rules
# EXPLICACIÓN: Añade reglas personalizadas. Ejemplo básico:
alert icmp any any -> any any (msg:"ICMP Detectado"; sid:1000001;)
# FORMATO: acción protocolo origen puerto -> destino puerto (opciones)

# === SECCIÓN 8: EJECUCIÓN ===

sudo snort -c /usr/local/snort/etc/snort/snort.lua -R /usr/local/snort/etc/snort/local.rules -i ens160 -A alert_fast -s 65535 -k none
# EXPLICACIÓN DETALLADA:
# -c: Archivo de configuración
# -R: Archivo de reglas
# -i: Interfaz de red
# -A alert_fast: Muestra alertas en formato simplificado
# -s 65535: Tamaño máximo de paquetes
# -k none: Ignora checksums inválidos

# =====================================================
# NOTAS FINALES:
# 1. Reemplaza 'ens160' con tu interfaz de red (usa ip a para verla)
# 2. Para monitorear en tiempo real: usa -A console
# 3. Documentación oficial: https://www.snort.org/
