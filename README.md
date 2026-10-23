# Módulo 12 · Del modelo al contenedor

**Virtualización y Despliegue en la Nube — ERA 3**

En este laboratorio llevas el modelo de sensación térmica que entrenaste en Colab hasta un servicio web que corre en la nube.

| Paso | Dónde | Qué se hace |
|---|---|---|
| 1 | Colab | Entrenar y exportar el modelo ✅ |
| **2** | **Codespaces** | **Probar la API sin Docker** ← estamos aquí |
| 3 | Codespaces | Construir la imagen y ejecutar el contenedor |
| 4 | Nube | Publicar la imagen |

## Estructura del repositorio

```
.
├── .devcontainer/
│   └── devcontainer.json   ← describe la máquina del Codespace (Python + Docker)
├── app/
│   ├── main.py             ← la API (FastAPI)
│   ├── modelo.joblib       ← el modelo entrenado en Colab
│   ├── metadata.json       ← ficha técnica del modelo
│   └── requirements.txt    ← versiones exactas de las librerías
└── README.md
```

> El repositorio ya trae un modelo de respaldo. Si no terminaste el Paso 1, puedes continuar con ese.

---

## Paso 2 — Probar la API sin Docker

La pregunta de este paso es: **¿funciona mi código?** Todavía no usamos Docker. Así, si algo falla más adelante, sabremos que el problema es del contenedor y no de la API.

### 2.1 Crea tu copia del repositorio

1. En la página de este repositorio, pulsa **Use this template → Create a new repository**.
2. Créalo en **tu** cuenta. Así el Codespace usará tu cuota gratuita.

### 2.2 Abre el Codespace

1. En **tu** repositorio: botón verde **Code → pestaña Codespaces → Create codespace on main**.
2. Espera a que termine de prepararse. La primera vez tarda unos minutos, porque está instalando las librerías de `requirements.txt`.

Al terminar verás VS Code en el navegador, con el explorador de archivos a la izquierda y una terminal abajo.

### 2.3 Sube tus archivos del Paso 1 (opcional)

Si entrenaste tu propio modelo, arrastra tus tres archivos (`modelo.joblib`, `metadata.json` y `requirements.txt`) a la carpeta `app/` y acepta reemplazar los existentes.

Luego reinstala las librerías, porque tus versiones pueden ser distintas a las del respaldo:

```bash
pip install -r app/requirements.txt
```

### 2.4 Arranca la API

```bash
cd app
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

Debes ver en la terminal:

```
✅ Modelo cargado: Polinómica grado 2
INFO:     Uvicorn running on http://0.0.0.0:8000
```

Aparecerá un aviso abajo a la derecha: **Open in Browser**. Ábrelo. Si no lo ves, ve a la pestaña **PORTS** (junto a TERMINAL) y abre la dirección del puerto `8000`.

### 2.5 Prueba la API

Agrega `/docs` al final de la dirección que se abrió. Verás la documentación interactiva que FastAPI genera sola.

1. Abre **GET /** → *Try it out* → *Execute*. Revisa qué modelo está cargado y en qué rangos se entrenó.
2. Abre **POST /predecir** → *Try it out*, escribe:
   ```json
   { "temperatura": 30, "humedad": 80 }
   ```
   y pulsa *Execute*. Debes obtener una sensación térmica cercana a **37 °C**.
3. Ahora prueba con `"temperatura": 42`. ¿Qué responde la API? ¿Por qué crees que se diseñó así?
4. Prueba con `"temperatura": "calor"`. ¿Quién detectó el error: tu código o FastAPI?

También puedes probar desde una **segunda terminal** (botón **+** en el panel de terminales):

```bash
curl -X POST localhost:8000/predecir \
     -H "Content-Type: application/json" \
     -d '{"temperatura": 30, "humedad": 80}'
```

Fíjate en el campo `atendido_por` de la respuesta y anota su valor. Lo volverás a mirar en el Paso 3.

### 2.6 Detén la API

En la terminal donde corre `uvicorn`, pulsa **Ctrl + C**.

### ✅ Lista de verificación

- [ ] La terminal muestra `✅ Modelo cargado`.
- [ ] `/docs` abre en el navegador.
- [ ] `/predecir` responde ≈ 37 °C para 30 °C y 80 %.
- [ ] Una temperatura fuera de rango devuelve un error **422** con un mensaje claro.
- [ ] Anotaste el valor de `atendido_por`.

### Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `Error loading ASGI app. Could not import module "main"` | Ejecutaste `uvicorn` fuera de la carpeta `app/` | `cd app` y repite |
| `ModuleNotFoundError: No module named 'sklearn'` (u otra librería) | Las librerías no se instalaron | `pip install -r app/requirements.txt` desde la raíz |
| `InconsistentVersionWarning` o error al cargar `modelo.joblib` | Tu `requirements.txt` no coincide con las versiones con que se entrenó el modelo | Sube los tres archivos del **mismo** Colab y reinstala |
| La página del puerto no abre | El servidor no está corriendo o escucha en `127.0.0.1` | Verifica que usaste `--host 0.0.0.0` |
| `Address already in use` | Ya hay otro `uvicorn` corriendo | Ciérralo con Ctrl + C en la otra terminal |

### Para pensar antes del Paso 3

1. El Codespace ya tenía Python y tú instalaste las librerías a mano. Si un compañero tuviera otra versión de Python o de scikit-learn, ¿obtendría el mismo resultado?
2. ¿Qué tendrías que escribirle a alguien para que ejecute tu API en su propio computador? Esa lista de instrucciones es, casi literalmente, lo que será el **Dockerfile**.

---

### Al terminar la sesión

Detén tu Codespace para no gastar cuota: pulsa **F1** (o **Ctrl + Shift + P**), escribe `Stop Current Codespace` y presiona Enter. También puedes detenerlo desde [github.com/codespaces](https://github.com/codespaces), en el menú **⋯** de tu Codespace. Al final del curso puedes borrarlo desde [github.com/codespaces](https://github.com/codespaces).
