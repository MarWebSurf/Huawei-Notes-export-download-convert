# Huawei-Notes-export-download-convert
script python para convertir todas las notas de block de notas de Huawei a archivos de texto.txt
import os
import time
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options

# Configurar opciones de Chrome
chrome_options = Options()
chrome_options.add_argument("--headless")  # Para no mostrar el navegador (opcional)
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")

# Ruta del ChromeDriver (en la misma carpeta que el script)
service = Service('chromedriver.exe')  # Asegúrate de que este archivo está en la misma carpeta que el script

# Crear una instancia del navegador
driver = webdriver.Chrome(service=service, options=chrome_options)

# Ruta de la carpeta que contiene los archivos HTML, obtenida dinámicamente
carpeta_html = os.path.dirname(__file__)  # Obtiene la ruta del directorio donde se encuentra el script

# Crear la carpeta de salida para los archivos .txt en la misma carpeta que el script
carpeta_salida_txt = os.path.join(carpeta_html, "txt_output")
os.makedirs(carpeta_salida_txt, exist_ok=True)

# Recorrer los archivos en la carpeta y en sus subcarpetas
for root, _, archivos in os.walk(carpeta_html):
    for nombre_archivo in archivos:
        if nombre_archivo.lower().endswith('.html'):  # Asegúrate de que el nombre del archivo es en minúsculas
            ruta_archivo = os.path.join(root, nombre_archivo)
            
            # Abrir el archivo HTML en el navegador
            print(f"Abrir el archivo: {ruta_archivo}")
            driver.get(f"file:///{ruta_archivo.replace(os.sep, '/')}")  # Formatear la ruta para el navegador
            
            # Esperar un momento para asegurarse de que la página cargue
            time.sleep(2)  # Ajusta el tiempo según sea necesario
            
            # Copiar todo el texto visible de la página
            try:
                texto = driver.find_element(By.TAG_NAME, "body").text
            except Exception as e:
                print(f"Error al extraer texto del archivo {ruta_archivo}: {e}")
                continue
            
            # Imprimir el contenido extraído para depuración
            print(f"Texto extraído (primeros 100 caracteres): {texto[:100]}")
            
            if texto:  # Solo proceder si hay texto extraído
                # Tomar los primeros 8 caracteres del nombre de la carpeta
                nombre_carpeta = os.path.basename(root)  # Obtener el nombre de la carpeta actual
                primeros_caracteres = nombre_carpeta[:8]
                
                # Comenzar a construir el nuevo nombre de archivo
                nuevo_nombre = primeros_caracteres + " "  # Agregar un espacio
                
                # Usar los caracteres del texto desde el índice 18 en adelante
                if len(texto) > 18:  # Asegúrate de que hay suficientes caracteres
                    caracteres_restantes = texto[18:]  # Tomar desde el carácter 19 en adelante
                    
                    # Completar hasta 35 caracteres
                    espacio_disponible = 35 - len(nuevo_nombre)
                    nuevo_nombre += caracteres_restantes[:espacio_disponible]  # Agregar los caracteres restantes hasta completar 35
                
                # Limpiar el nombre de archivo de caracteres no válidos
                nuevo_nombre = "".join(c for c in nuevo_nombre if c.isalnum() or c in (' ', '_')).rstrip()  # Limpiar el nombre
                print(f"Nombre del archivo limpio: {nuevo_nombre}")

                # Guardar el texto en un archivo .txt en la carpeta de salida
                try:
                    with open(os.path.join(carpeta_salida_txt, nuevo_nombre + ".txt"), 'w', encoding='utf-8') as archivo_txt:
                        archivo_txt.write(texto)
                    print(f"Archivo guardado en carpeta común: {nuevo_nombre}.txt")
                except Exception as e:
                    print(f"Error al guardar el archivo {nuevo_nombre}: {e}")
            else:
                print(f"No se encontró texto en el archivo {ruta_archivo}")

# Cerrar el navegador
driver.quit()

print("Proceso completado.")
