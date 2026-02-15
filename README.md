# fourier-signal-analysis
import numpy as np
import matplotlib.pyplot as plt

# 1. Configuración de la señal
# Aumentamos fs para mejorar la resolución en frecuencia
fs = 1000                      # Frecuencia de muestreo (Hz)
t = np.arange(-0.5, 0.5, 1/fs) # Tiempo de -0.5 a 0.5 segundos (1000 muestras)

# 2. Generación de Señales Elementales (Punto 2 de la actividad)
# Señal A: Pulso Rectangular (Ancho de 0.2s)
rect = np.where(np.abs(t) < 0.1, 1, 0)

# Señal B: Onda Senoidal (50 Hz)
f0 = 50
seno = np.sin(2 * np.pi * f0 * t)

# Señal C: Función Escalón Unitario (Heaviside)
escalon = np.where(t >= 0, 1, 0)

# 3. Función para procesar la FFT (Transformada Rápida de Fourier)
def procesar_fourier(señal, fs):
    n = len(señal)
    # Frecuencias centradas
    freq = np.fft.fftfreq(n, 1/fs)
    # Calculamos FFT y normalizamos por el número de muestras
    fft_val = np.fft.fft(señal) / n
    
    # Desplazamos para tener la frecuencia 0 (DC) al centro del espectro
    f_shift = np.fft.fftshift(freq)
    fft_shift = np.fft.fftshift(fft_val)
    
    magnitud = np.abs(fft_shift)
    # Filtro para evitar ruido de fase en magnitudes despreciables
    fase = np.angle(fft_shift)
    fase[magnitud < (np.max(magnitud) * 0.001)] = 0 
    
    return f_shift, magnitud, fase

# 4. Verificación de Propiedades (Punto 2 de la actividad)

# --- PROPIEDAD DE LINEALIDAD ---
# Si x(t) = a*rect + b*seno, entonces FFT(x) = a*FFT(rect) + b*FFT(seno)
señal_suma = rect + seno
f_suma, mag_suma, _ = procesar_fourier(señal_suma, fs)

# --- PROPIEDAD DE DESPLAZAMIENTO EN EL TIEMPO ---
# Desplazamos el pulso rectangular 0.1s a la derecha
t_shift = 0.1
rect_desplazado = np.where(np.abs(t - t_shift) < 0.1, 1, 0)
f_desp, mag_desp, fase_desp = procesar_fourier(rect_desplazado, fs)

# 5. Visualización de Resultados
fig, axs = plt.subplots(4, 2, figsize=(14, 18))
plt.subplots_adjust(hspace=0.5, wspace=0.3)

# --- SECCIÓN 1: SEÑALES ELEMENTALES ---
# Pulso Rectangular
axs[0, 0].plot(t, rect, color='blue')
axs[0, 0].set_title("Señal: Pulso Rectangular")
axs[0, 0].grid(True)

axs[0, 1].plot(f_rect := procesar_fourier(rect, fs)[0], procesar_fourier(rect, fs)[1], color='red')
axs[0, 1].set_title("Espectro Magnitud: Función Sinc")
axs[0, 1].set_xlim(-50, 50)
axs[0, 1].grid(True)

# Onda Senoidal
axs[1, 0].plot(t, seno, color='blue')
axs[1, 0].set_title("Señal: Onda Senoidal (50Hz)")
axs[1, 0].set_xlim(-0.05, 0.05)
axs[1, 0].grid(True)

axs[1, 1].plot(f_seno := procesar_fourier(seno, fs)[0], procesar_fourier(seno, fs)[1], color='red')
axs[1, 1].set_title("Espectro Magnitud: Impulsos en ±50Hz")
axs[1, 1].set_xlim(-100, 100)
axs[1, 1].grid(True)

# Función Escalón
axs[2, 0].plot(t, escalon, color='blue')
axs[2, 0].set_title("Señal: Función Escalón")
axs[2, 0].grid(True)

axs[2, 1].plot(f_esc := procesar_fourier(escalon, fs)[0], procesar_fourier(escalon, fs)[1], color='red')
axs[2, 1].set_title("Espectro Magnitud: Escalón")
axs[2, 1].set_xlim(-50, 50)
axs[2, 1].grid(True)

# --- SECCIÓN 2: VERIFICACIÓN DE PROPIEDADES ---
# Linealidad (Suma de señales)
axs[3, 0].plot(f_suma, mag_suma, color='purple')
axs[3, 0].set_title("Propiedad: Linealidad (Suma de Rect + Seno)")
axs[3, 0].set_xlim(-70, 70)
axs[3, 0].grid(True)

# Desplazamiento (Comparación de Fase)https://www.onlinegdb.com/online_c_compiler#tab-stdin
axs[3, 1].plot(f_desp, fase_desp, color='green', label='Desplazada 0.1s')
axs[3, 1].plot(f_rect, procesar_fourier(rect, fs)[2], color='black', alpha=0.3, label='Original')
axs[3, 1].set_title("Propiedad: Desplazamiento (Cambio en Fase)")
axs[3, 1].set_xlim(-20, 20)
axs[3, 1].legend()
axs[3, 1].grid(True)

print("Simulación avanzada finalizada con verificación de propiedades.")
plt.show()
