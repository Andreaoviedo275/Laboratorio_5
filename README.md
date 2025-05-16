# Laboratorio_5: Variabilidad de la Frecuencia Cardíaca con Transformada Wavelet

📌 Descripción
Este laboratorio tiene como objetivo analizar la variabilidad de la frecuencia cardíaca (HRV) mediante el procesamiento de señales ECG. Se aplican técnicas de análisis en el dominio del tiempo y la frecuencia, incluyendo la Transformada Wavelet Continua (CWT) para obtener espectrogramas detallados en el tiempo-frecuencia.

🎯 Objetivos

- Cargar y procesar señales ECG simuladas.
  
- Detectar picos R y calcular los intervalos R-R.
  
- Evaluar la variabilidad de la frecuencia cardíaca (HRV) en el dominio del tiempo (media, SDNN, RMSSD).
  
- Visualizar la dinámica de frecuencia con Transformada Wavelet.
  
- Calcular potencia espectral en bandas LF y HF.

🛠 Fundamento Teórico

1. Actividad simpática y parasimpática del sistema nervioso autónomo:
   
    El sistema nervioso autónomo (SNA) regula funciones involuntarias del cuerpo, como la frecuencia cardíaca, la presión arterial, la digestión y la respiración. Tiene dos ramas principales:
    
    - Simpática: prepara al cuerpo para situaciones de “lucha o huida” (estrés). Aumenta la frecuencia cardíaca, dilata los bronquios, y redirige la sangre a los músculos.
    
    - Parasimpática: promueve el estado de “reposo y digestión”. Disminuye la frecuencia cardíaca, estimula la actividad digestiva y favorece el descanso.
    
    Ambas ramas actúan de manera complementaria y se equilibran constantemente para mantener la homeostasis.
   
  
3.  Efecto de la actividad simpática y parasimpática en la frecuencia cardíaca:
   
    - Estimulación simpática: incrementa la frecuencia cardíaca al liberar noradrenalina, que actúa sobre los receptores β1 del nodo sinoauricular (SA) en el corazón.
    
    - Estimulación parasimpática: reduce la frecuencia cardíaca mediante la acetilcolina, que actúa sobre receptores muscarínicos del nodo SA.
    
    - El balance entre estas dos influencias determina la frecuencia cardíaca en un momento dado.
    
  
4. Variabilidad de la frecuencia cardíaca (HRV):
     
    La HRV (Heart Rate Variability) es la variación en el tiempo entre latidos cardíacos sucesivos, medida comúnmente como fluctuaciones en los intervalos R-R del ECG (intervalo entre dos ondas R consecutivas).
    
    - Importancia:
      
      - Refleja el estado funcional del SNA.
      
      - Alta HRV: mayor influencia parasimpática, buena adaptación fisiológica.
      
      - Baja HRV: dominancia simpática, menor flexibilidad del sistema cardiovascular (asociada a estrés, enfermedades, etc).
      
      - Frecuencias de interés (dominio espectral):
        
    
    Se analizan con transformadas como Fourier o Wavelet. Las bandas de frecuencia típicas son:
    
      - ULF (Ultra Low Frequency): < 0.003 Hz
      
      - VLF (Very Low Frequency): 0.003 – 0.04 Hz
    
      - LF (Low Frequency): 0.04 – 0.15 Hz → influencias simpáticas y parasimpáticas.
      
      - HF (High Frequency): 0.15 – 0.4 Hz → reflejo principalmente parasimpático, asociado a la respiración.
      
      - LF/HF ratio: usado para estimar el balance simpático/parasimpático.
  
5. Transformada Wavelet
   
    Definición:
    
    La Transformada Wavelet (WT) es una herramienta matemática para analizar señales no estacionarias o transitorias, como las biológicas. Descompone una señal en diferentes escalas de tiempo y frecuencia, permitiendo observar detalles locales.
    
    Usos en señales biológicas:
    
    - Detección de eventos transitorios en ECG o EEG.
    
    - Análisis de HRV (especialmente cuando la señal varía en el tiempo).
    
    - Eliminación de ruido y artefactos.
    
    - Compresión y reconstrucción de señales.
    
    Tipos de wavelets utilizadas:
    
    - Daubechies (db4, db6, etc.): muy usadas en ECG y HRV por su similitud con la morfología de las señales.
    
    - Symlet: versiones más simétricas de Daubechies.
    
    - Coiflet: útiles para señales con propiedades suaves.
    
    - Haar: la más simple; útil para análisis rápidos o enseñanza.
    
    - Morlet: frecuentemente usada en análisis espectral de señales EEG o HRV.

6. Filtros IIR

    Los filtros IIR (respuesta infinita al impulso) permiten limpiar señales biológicas de ruido, interferencias y artefactos sin requerir grandes órdenes.



📌 Parte I – Análisis de HRV con Transformada Wavelet

🔹 Paso 1: Carga de datos y estadísticas básicas

        informacion = np.loadtxt("ecg_5min_simulado_estudiantes.csv", delimiter=",", skiprows=1)
        tiempo = informacion[:, 0]
        ecg_filtrado = informacion[:, 1]
       
* Carga de un archivo CSV que contiene dos columnas: tiempo y voltaje ECG.

* Separamos en tiempo (s) y ecg_filtrado (mV).

        sampling_rate = 1000
        duration = tiempo[-1]
        quantization_bits = 12
        quantization_levels = 2 ** quantization_bits

* Se define la frecuencia de muestreo.

* Se calcula duración total y niveles de cuantificación (4096 niveles en 12 bits).

🔹 Paso 2: Cálculo de medidas estadísticas

        media = np.mean(ecg_filtrado)
        mediana = np.median(ecg_filtrado)
        varianza = np.var(ecg_filtrado)
        desviacion = np.std(ecg_filtrado)
        coef_variacion = desviacion / media

![image](https://github.com/user-attachments/assets/31b629a8-0144-404d-b0fd-e4838b765490)

- Media cercana a 0 mV: indica una señal bien centrada.

- Desviación estándar baja (0.02182 mV): poca dispersión del voltaje.

- Coeficiente de variación elevado: debido a la media cercana a cero (matemáticamente normal en señales ECG con picos y fases positivas/negativas).

🔹 Paso 3: Detección de picos R

        altura_minima = 0.5
        distancia_minima = int(0.6 * sampling_rate)
        picos_R, _ = find_peaks(ecg_filtrado, height=altura_minima, distance=distancia_minima)

- Se busca una separación mínima de 0.6 segundos entre picos para evitar falsos positivos.
  
- Se detectan los picos R sobre la señal ECG filtrada. Estos representan los latidos del corazón.

![image](https://github.com/user-attachments/assets/67f6770f-0022-428c-9ef8-8064bc6c2111)

* Línea azul: Señal ECG limpia.

* Puntos rojos: Picos R detectados correctamente.

* Eje X: Tiempo en segundos.

* Eje Y: Voltaje en milivoltios (mV).

* La regularidad o irregularidad entre los picos indica variabilidad entre latidos (HRV).

🔹 Paso 4: Intervalos R-R

        intervalos_RR = np.diff(tiempo[picos_R])
        tiempos_RR = tiempo[picos_R][1:]

- Se calcula el tiempo entre picos consecutivos (diferencia entre tiempos R).

🔹 Paso 5: HRV en dominio del tiempo

        media_RR = np.mean(intervalos_RR)
        desviacion_RR = np.std(intervalos_RR)
        rmssd = np.sqrt(np.mean(np.square(np.diff(intervalos_RR))))

- Se analiza la variabilidad de los intervalos entre picos R (R-R), para extraer tres métricas: media, SDNN (desviación estándar) y RMSSD.

-  SDNN: desviación estándar de R-R

- RMSSD: raíz del promedio de las diferencias al cuadrado de intervalos R-R adyacentes

![image](https://github.com/user-attachments/assets/053a7ed8-07dc-40d9-840b-1b09b8fe4bfc)

- Línea verde con puntos: Variación de intervalos R-R.

- Línea azul punteada: Media de los intervalos.

- Líneas rojas punteadas: +1 y -1 desviación estándar.

Colores:

- Azul: equilibrio promedio.

- Rojo: límites normales de variabilidad.

- Una alta SDNN y RMSSD indican buena adaptabilidad fisiológica (menor estrés).

Justificación fisiológica:

- SDNN alta → buena variabilidad

- RMSSD alta → cuerpo relajado

![image](https://github.com/user-attachments/assets/53d81d33-2c15-470a-b105-6f8e58635520)

- Media R-R = 37.03 s: promedio del tiempo entre latidos.

- SDNN = 18.44 s: variación saludable entre los intervalos.

- RMSSD = 37.17 s: muy buena recuperación del cuerpo, alto tono vagal.

🔹 Paso 6: Transformada Wavelet

        cwt_result, freqs = pywt.cwt(intervalos_RR, scales, wavelet, sampling_period=1)

- Se aplica la CWT sobre los intervalos R-R, usando la wavelet compleja Morlet (cmor).
- La Transformada Wavelet Continua permite representar cómo varían las frecuencias a lo largo del tiempo (análisis tiempo-frecuencia).

![image](https://github.com/user-attachments/assets/393fc2bd-3e30-4f81-8188-ad0df538c6fe)

- Color: Amplitud (potencia espectral).

  - Rojo: mayor potencia (alta contribución).
  
  - Azul: baja potencia.

- Eje Y (frecuencia): en Hz.

- Líneas horizontales punteadas:

  - Azul: banda LF (0.04–0.15 Hz).
  
  - Roja: banda HF (0.15–0.4 Hz).

- Importancia:
  
  - LF (baja frecuencia): modulaciones simpáticas y parasimpáticas.
  
  - HF (alta frecuencia): modulación parasimpática (actividad respiratoria).
    
  - Cambios en color a lo largo del tiempo indican variaciones dinámicas en HRV.

🔹 Paso 7: Potencia espectral en bandas LF y HF

        lf_mask = (freqs >= 0.04) & (freqs <= 0.15)
        hf_mask = (freqs > 0.15) & (freqs <= 0.4)

- Se separan las frecuencias en las bandas fisiológicas.

        power_lf = np.abs(cwt_result[lf_mask, :])**2
        power_hf = np.abs(cwt_result[hf_mask, :])**2

![image](https://github.com/user-attachments/assets/b4ae203e-55db-40a8-b7d7-e93872c05bca)

- Arriba: Potencia en banda LF.

  - Colormap Blues: cuanto más oscuro, mayor potencia.

- Abajo: Potencia en banda HF.

  - Colormap Reds: rojo intenso indica alta potencia.

- Eje X: Tiempo.

- Eje Y: Frecuencia.

📌 Parte II – Filtrado ECG con Filtro IIR

🔹 Paso 1: Señal ECG ruidosa

        def synthetic_ecg_(...):
            # Picos cardíacos
            # Ruido blanco
            # Interferencia 50-60 Hz
            # Deriva de línea base
            # Artefactos de movimiento
            
- Esta función crea una señal realista con errores típicos de la adquisición.

🔹 Paso 2: Diseño del filtro IIR Butterworth

        def butter_bandpass(lowcut=0.5, highcut=35.0, fs=1000.0, order=4):
            nyq = 0.5 * fs
            ...
            b, a = butter(order, [low, high], btype='band')
            return b, a

- Filtro pasa banda [0.5–35 Hz]

- Elimina ruido, deriva e interferencia eléctrica.

🔹 Paso 3: Aplicación del filtro

        filtered_ecg = filtfilt(b, a, raw_ecg)

- filtfilt() aplica el filtro sin fase.

![Figure 2025-05-15 202108](https://github.com/user-attachments/assets/04efe926-c2fe-41df-a1be-d1a9bcb95466)

- Línea gris punteada: Señal original con ruido y artefactos.

- Línea azul: Señal filtrada con el filtro IIR Butterworth.

- El filtrado elimina eficazmente:

  - La deriva de línea base (muy baja frecuencia)

  - La interferencia eléctrica (alta frecuencia)

🔹 Paso 4: Ecuación en diferencias del filtro

        for i, bi in enumerate(b): print(...)
        for i, ai in enumerate(a): print(...)

![image](https://github.com/user-attachments/assets/9635c4bb-f330-48bb-9323-c23e9913461c)

🔹 Paso 5: Ecuación explícita

        terms_b = [f"{b[i]:+.3e}·x[n-{i}]" for i in range(len(b))]
        terms_a = [f"{-a[i]:+.3e}·y[n-{i}]" for i in range(1, len(a))]

![image](https://github.com/user-attachments/assets/670a5eb7-c56f-41c5-b248-d8dae80a115e)

❤️ Estimación de Frecuencia Cardíaca

- Después del filtrado, se detectan los picos R de la señal limpia usando find_peaks. La frecuencia cardíaca se calcula con:

        heart_rate = 60 / np.mean(np.diff(r_peaks) / sampling_rate)

- Este valor representa la frecuencia promedio en los 10 segundos simulados.
  
💫 Conclusión
- La HRV analizada con transformada wavelet aporta información rica en el tiempo y la frecuencia.

- El filtro IIR Butterworth limpia eficazmente la señal ECG ruidosa.

- El análisis conjunto permite evaluar salud cardíaca, relajación, adaptación y fatiga.

- Se cubren todos los requisitos de la guía, incluyendo espectrogramas, filtros, HRV y ecuaciones en diferencias.
