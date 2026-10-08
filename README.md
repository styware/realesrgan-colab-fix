# Real-ESRGAN en Google Colab — instalación corregida + upscale de carpetas de video

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/styware/realesrgan-colab-fix/blob/main/4k_Video_Upscaler_Colab_(Real_ESRGAN).ipynb

> **English summary:** Colab notebook that installs [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) (+ BasicSR, GFPGAN, facexlib) on the latest Colab runtimes, where the standard install fails with `metadata-generation-failed` / `KeyError: '__version__'` on Python 3.13. It also upscales a whole **folder of videos** and resumes where it left off.

## ¿Para qué sirve?

Si intentaste instalar Real-ESRGAN en Colab y te salieron errores como estos, este notebook los resuelve:

- `error: metadata-generation-failed` / `python setup.py egg_info did not run successfully`
- `KeyError: '__version__'`
- `ModuleNotFoundError: No module named 'basicsr'` justo después de instalar
- `torch requires setuptools<82, but you have setuptools 84`

Además incluye una celda para **escalar una carpeta completa de videos** (por ejemplo de 720p a 1080p), no solo un video a la vez.

## Cómo usarlo

1. Pulsa el botón **Open in Colab** de arriba.
2. En Colab: **Entorno de ejecución → Cambiar tipo de entorno de ejecución → GPU** (probado con L4).
3. Ejecuta la **celda 1 (instalación)**. Al final debe imprimir `Imports OK`.
4. Configura y ejecuta la **celda 2 (upscale)**:
   - `video_path`: un video **o una carpeta** con videos.
   - `output_dir`: carpeta de salida. Usa una ruta de Google Drive, porque `/content` se borra al cerrar la sesión.
   - `resolution` y `model`: ver abajo.

Si Colab se desconecta, vuelve a ejecutar la celda 2: salta los videos que ya tienen resultado.

## Qué estaba fallando (y cómo se corrigió)

| Problema | Causa | Solución en el notebook |
|---|---|---|
| `metadata-generation-failed` | Los `setup.py` de BasicSR, facexlib, GFPGAN y Real-ESRGAN usan `setup_requires=['cython','numpy','torch']`, que intenta compilar esas librerías en un entorno aislado. | Se elimina esa línea con `sed` antes de instalar. |
| `KeyError: '__version__'` | En **Python 3.13**, `exec()` ya no crea variables locales, y esos `setup.py` hacen `return locals()['__version__']`. | Se reemplaza por la lectura directa del archivo `VERSION`. |
| `ModuleNotFoundError: basicsr` tras instalar | Una instalación con `pip install -e` no es visible en un kernel ya iniciado hasta reiniciarlo. | Se añaden las rutas de los repos a `sys.path` en la misma sesión. |
| Conflicto con `torch` | `pip install --upgrade setuptools` subía setuptools a una versión que torch no admite. | No se actualiza setuptools. |
| Instalaciones que traen otro `basicsr` desde PyPI | `gfpgan` y `facexlib` lo piden como dependencia. | Se instala con `--no-deps --no-build-isolation` y las dependencias se instalan a mano. |

## Velocidad: elige bien el modelo

`RealESRGAN_x4plus` siempre procesa a 4× internamente y luego reduce. Para video es muy lento. En una prueba (video de 720×1280 → 1080×1920, 192 frames, GPU L4):

| Modelo | Velocidad |
|---|---|
| `RealESRGAN_x4plus` | ~3.6 s por frame |
| `realesr-animevideov3` | ~1.7 frames por segundo (192 frames en 1 min 51 s) |

Recomendación: `realesr-animevideov3` para animación/ilustración y `realesr-general-x4v3` para video realista. Los tiempos dependen del video y de la GPU: son una referencia, no una garantía.

## Versión probada

- Runtime de Colab: **2026.07** (la más reciente al 7 de octubre de 2026)
- GPU: L4
- Python: **X.XX** *(copia aquí la versión que imprime la celda 1)*

Colab actualiza sus imágenes con frecuencia. Si algo falla en el futuro, abre un *issue* con el error completo y la versión del runtime.

## Créditos

Este repositorio **no contiene el código de Real-ESRGAN**: el notebook lo clona y le aplica pequeños parches al instalarlo. Todo el mérito del modelo es de sus autores:

- [Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN)
- [BasicSR](https://github.com/XPixelGroup/BasicSR)
- [GFPGAN](https://github.com/TencentARC/GFPGAN)
- [facexlib](https://github.com/xinntao/facexlib)

Consulta la licencia de cada uno antes de usar los resultados en proyectos comerciales. Este repositorio no está afiliado a ellos.

## Licencia

El notebook y este README: MIT (ver `LICENSE`).
