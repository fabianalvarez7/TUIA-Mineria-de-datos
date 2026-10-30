# Entorno virtual y Kernel de Jupyter — Guía de referencia

Guía para no confundirse con el kernel de Jupyter en los trabajos prácticos de
**Minería de Datos (TUIA - UNR)**.

---

## 1. La idea central: venv y kernel son cosas DISTINTAS

Es el malentendido más común, y por eso vale la pena tenerlo claro antes de seguir.

| | Entorno virtual (`venv`) | Kernel de Jupyter |
|---|---|---|
| **Qué es** | Una carpeta con su propia copia de Python + librerías | Un "puente" que conecta Jupyter con un Python concreto |
| **Dónde vive** | `TUIA-Mineria-de-datos/.venv/` | Se registra en Jupyter al arrancar el kernel |
| **Qué resuelve** | "Cada proyecto tiene sus propias librerías, no se pisan entre sí" | "Este notebook se ejecuta con ESTE Python" |
| **Se crea con** | `python3 -m venv .venv` | `python3 -m ipykernel install ...` |

**El kernel SIEMPRE apunta a un Python.** La pregunta que hay que hacerse es: *¿a cuál?*

- Si el kernel apunta al Python del sistema → no necesitás venv (y conviene no usarlo).
- Si el kernel apunta al Python del venv → entonces sí necesitás el venv.

---

## 2. Por qué conviene el venv en esta materia

Este TP usa `scikit-learn`, `seaborn`, `pandas` y `matplotlib`. Esas librerías
**no vienen con Python**. Hay dos caminos:

- **Sin venv** → `pip install` contra el Python del sistema → te pollúa el Python
  global. Si otro TP necesita otra versión de scikit-learn, se rompe todo.
- **Con venv** → las librerías viven en la carpeta del proyecto. Borrás la carpeta
  y empezás de cero. Cero sorpresas.

### Requisito de versión (importante)

El `tp.ipynb` usa `OrdinalEncoder(...).set_output(transform='pandas')`, que
**requiere scikit-learn >= 1.2**. Con una versión vieja vas a obtener un error
confuso del tipo `AttributeError: 'OrdinalEncoder' object has no attribute
'set_output'`.

Además, el notebook usa `inplace=True` sobre slices de `train_test_split`, lo que
genera `SettingWithCopyWarning`. Funciona, pero en pandas 3.0 ese comportamiento
va a cambiar.

---

## 3. Puesta en marcha (una sola vez por proyecto)

Desde la raíz del repo (`TUIA-Mineria-de-datos/`, donde están `TP1`, `Unidad1...`, etc.):

```bash
# 1. Crear el entorno
python3 -m venv .venv

# 2. Activarlo
source .venv/bin/activate
# En Windows: .venv\Scripts\activate

# 3. Instalar las librerías que pide el TP
pip install numpy pandas matplotlib seaborn scikit-learn jupyterlab ipykernel
```

El prompt debe cambiar a `(.venv)` al principio de la línea. **Esa es la señal de
que el entorno quedó activado.**

```bash
# 4. Registrar un kernel que apunte a este venv
python -m ipykernel install --user --name tuia-mineria --display-name "TUIA Mineria (venv)"
```

> Tip: el paso 4 es "belt and suspenders" (defensa en capas). Hacélo siempre y te
> olvidás del tema, sin importar desde dónde abras Jupyter.

### Levantar Jupyter

```bash
jupyter lab
```

En el dropdown **Kernel** de la esquina superior derecha del notebook vas a ver
`TUIA Mineria (venv)` además de `Python 3`. Elegí el del venv.

---

## 4. La trampa clásica

- **Si abrís Jupyter con `jupyter lab` DESDE el venv activado** → el kernel que
  aparece por defecto ya es el del venv. No necesitás registrar nada.
- **Si abrís Jupyter desde fuera del venv** → te aparece el Python del sistema, y
  ahí sí tenés que usar el kernel registrado en el paso 4.

Por eso el paso 4 es la solución que funciona en los dos casos.

---

## 5. Verificación infalible

Cuando algo falle, lo primero es correr esto en una celda nueva:

```python
import sys
print(sys.executable)
```

- `.../TUIA-Mineria-de-datos/.venv/bin/python` → **estás en el venv** ✅
- `/opt/homebrew/bin/python3` (u `/usr/bin/python3`) → **no estás en el venv** ❌

Este es **el** verificador. `sys.executable` te dice exactamente qué Python está
ejecutando el notebook.

Comprobación complementaria:

```python
import sklearn, pandas, numpy, seaborn
print(sklearn.__version__, pandas.__version__, numpy.__version__, seaborn.__version__)
```

---

## 6. Tabla de problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'sklearn'` | Kernel apunta al Python del sistema | Cambiar el kernel en el dropdown a `TUIA Mineria (venv)` |
| `AttributeError: 'OrdinalEncoder' object has no attribute 'set_output'` | scikit-learn < 1.2 | `pip install -U scikit-learn` **con el venv activado** |
| `SettingWithCopyWarning` al hacer `inplace=True` | Asignación sobre un slice de `train_test_split` | Usar `.copy()` explícito o `inplace=False` con reasignación |
| `jupyter: command not found` | JupyterLab no está en el PATH del venv | `pip install jupyterlab` con el venv activado, o usar `python -m jupyterlab` |
| El kernel no aparece en el dropdown | El kernel no se registró o quedó con otro nombre | Repetir el paso 4 y reiniciar Jupyter Lab |
| `pip install` instala en otro sitio | El venv no está activado al momento de instalar | Activar primero (`.venv` en el prompt), luego instalar |

---

## 7. Desactivar y limpiar

```bash
# Desactivar el entorno
deactivate

# Eliminar el entorno por completo (empezar de cero)
rm -rf .venv
```

El `.venv` **no** se versiona en git. Si el repo tiene `.gitignore`, agregá:

```gitignore
.venv/
```

---

## 8. Comandos de referencia

```bash
python3 -m venv .venv              # crear entorno
source .venv/bin/activate          # activar (macOS/Linux)
deactivate                         # desactivar
pip install <paquete>              # instalar (CON el venv activado)
pip list                           # ver qué hay instalado
pip freeze > requirements.txt     # fijar versiones
python -m jupyterlab               # lanzar JupyterLab
which python                       # a qué Python apunta ahora
python --version                   # versión de Python
```

---

## 9. Nota sobre la entrega del TP

El `tp.ipynb` **ya tiene las salidas guardadas** de la ejecución del compañero.
Eso permite revisar y corregir el contenido sin necesidad de ejecutar el notebook.

De todas formas, para verificar los números o hacer pruebas, conviene ejecutarlo
con el kernel del venv. Si vas a reejecutar todo, hacelo con **Run All** desde
arriba, porque el notebook depende del orden de las celdas (muchas variables se
definen en celdas anteriores y se reutilizan después).
