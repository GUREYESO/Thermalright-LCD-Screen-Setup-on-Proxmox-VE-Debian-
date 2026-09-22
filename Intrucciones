# Thermalright LCD Screen Setup on Proxmox VE (Debian)

Guía paso a paso para configurar la pantalla LCD integrada del disipador **Thermalright (Vision MAX / ChiZhu Tech `87ad:70db`)** en un servidor sin entorno gráfico **Proxmox VE**, utilizando la herramienta CLI `thermalright-trcc-linux`.

---

## 🛠️ Requisitos e Instalación de Dependencias

Instalar los paquetes base del sistema y preparar el entorno virtual de Python:

```bash
# Dependencias del sistema
apt update
apt install -y lm-sensors libgl1 p7zip-full python3-venv python3-pip

# Entorno virtual de Python e instalación de Pillow
python3 -m venv /root/trcc-env
/root/trcc-env/bin/pip install pillow
```

---

## 🖼️ Preparación de la Imagen de Fondo ($480 \times 854$ px)

La pantalla vertical del disipador requiere una resolución nativa de **$480 \times 854$ píxeles** para evitar franjas negras recortadas.

Script en Python para centrar el escudo en un lienzo blanco completo:

```bash
/root/trcc-env/bin/python3 -c "
from PIL import Image, ImageChops
img = Image.open('/root/escudo.jpg').convert('RGB')
diff = ImageChops.difference(img, Image.new('RGB', img.size, (255, 255, 255)))
bbox = diff.getbbox()
cropped = img.crop(bbox) if bbox else img
cropped.thumbnail((420, 420), Image.Resampling.LANCZOS)
canvas = Image.new('RGB', (480, 854), (255, 255, 255))
canvas.paste(cropped, ((480 - cropped.width) // 2, (854 - cropped.height) // 2))
canvas.save('/root/escudo_full.jpg')
"
```

---

## ⚙️ Configuración del Fondo y Capas en TRCC

1. **Asignar la imagen de fondo:**
   ```bash
   trcc display background 87ad:70db /root/escudo_full.jpg
   trcc display overlay 87ad:70db true
   ```

2. **Añadir los elementos de texto y métricas:**
   ```bash
   trcc display overlay-add 87ad:70db clock
   trcc display overlay-add 87ad:70db metric
   trcc display overlay-add 87ad:70db metric
   trcc display overlay-add 87ad:70db metric
   ```

3. **Alineación, centrado y formato de métricas:**
   ```bash
   # Reloj (Superior)
   trcc display overlay-update 87ad:70db el_6612f685 --bold --x 225 --y 55 --size 36

   # Temperatura de CPU (Inferior)
   trcc display overlay-update 87ad:70db el_984147d9 --bold --metric "cpu:temp" --format "CPU: {value:.1f} °C" --x 220 --y 650 --size 28

   # Uso de CPU (Inferior)
   trcc display overlay-update 87ad:70db el_10da9f41 --bold --metric "cpu:usage" --format "USO: {value:.1f} %" --x 235 --y 695 --size 28

   # Uso de Memoria RAM (Inferior)
   trcc display overlay-update 87ad:70db el_ae772f55 --bold --metric "memory:percent" --format "RAM: {value:.1f} %" --x 235 --y 740 --size 28
   ```

---

## 🔄 Servicio Systemd para Refresco en Tiempo Real

Para mantener las métricas actualizadas en vivo frame a frame, se configura el servicio utilizando el modo **`display play`**.

1. **Crear `/etc/systemd/system/trcc.service`:**
   ```ini
   [Unit]
   Description=Thermalright TRCC Live Display Ticker
   After=multi-user.target

   [Service]
   Type=simple
   ExecStart=/usr/local/bin/trcc display play 87ad:70db
   Restart=always
   RestartSec=3

   [Install]
   WantedBy=multi-user.target
   ```

2. **Habilitar e iniciar el servicio:**
   ```bash
   systemctl daemon-reload
   systemctl enable trcc
   systemctl start trcc
   ```

---

## 🔍 Comandos de Diagnóstico Útiles

* **Verificar lecturas de sensores en el servidor:** `trcc sensors`
* **Ver estado del servicio:** `systemctl status trcc`
* **Previsualización en terminal ANSI:** `trcc display test-lcd 87ad:70db`
