# Informe: Enfoques para Armar Módulos

En este documento se describen en detalle dos alternativas de arquitectura para gestionar un producto frontend con módulos que pueden tener variantes específicas por cliente (A, B, C). La primera opción es un monolito modular con variantes internas, donde todo el código vive en un único repositorio y se emplean subcarpetas o feature flags para activar las variantes según el tenant. La segunda propuesta es una arquitectura de micro-frontends, donde cada módulo (p. ej. “Clientes”) se despliega como una unidad independiente (remote) que expone sus variantes y un host (o “shell”) orquesta dinámicamente qué bundle cargar basándose en la configuración por tenant. Al final se incluye una tabla comparativa y un ejemplo ampliado de cómo implementar el host dinámico que, leyendo un manifiesto JSON, carga con React.lazy los remotes adecuados.

---

## 1. Monolito modular con variantes internas

Un monolito es una arquitectura en la que toda la funcionalidad de la aplicación se encuentra en un único código base. Los módulos están integrados en un solo proyecto, lo que facilita la gestión centralizada y la sincronización de dependencias. Este enfoque permite que todas las partes de la aplicación compartan un mismo entorno de desarrollo, lo que simplifica el proceso de construcción, pruebas e implementación. Además, al no requerir comunicación entre servicios separados, se elimina la sobrecarga de red, lo que puede resultar en un mejor desempeño en términos de tiempo de respuesta.

Sin embargo, a medida que el proyecto crece, un monolito puede volverse más difícil de mantener debido al aumento en la complejidad del código y la posibilidad de que los cambios en una parte del sistema afecten inadvertidamente a otras. También puede presentar desafíos en términos de escalabilidad, ya que todos los equipos deben trabajar en el mismo repositorio, lo que puede generar conflictos y ralentizar el desarrollo. Por último, el tamaño del bundle generado puede ser considerablemente mayor, ya que incluye todas las variantes y funcionalidades, incluso aquellas que no son necesarias para ciertos usuarios o clientes.

### 1.1 Estructura y configuración (Ejemplo)

```
/monolito-app
├─ /src
│  ├─ /modules
│  │  ├─ /clientes
│  │  │  ├─ core/               ← lógica común
│  │  │  ├─ variantA/           ← extensiones para Empresa A
│  │  │  └─ variantB/           ← extensiones para Empresa B
│  │  ├─ /campañas
│  │  │  ├─ core/
│  │  │  ├─ variantA/
│  │  │  └─ variantB/
│  │  └─ …otros módulos…
│  ├─ /config
│  │  └─ tenants.json           ← mapea tenantId → variantes
│  └─ index.tsx                 ← punto de entrada React
└─ package.json
```

### Tenants

```json
{
  "empresaA": { "clientes": "variantA", "campañas": "variantA" },
  "empresaB": { "clientes": "variantB", "campañas": "core" }
}
```

La app, tras autenticar, lee tenantId, selecciona la variante y renderiza, por ejemplo: 

```typescript
import ClientesCore from './modules/clientes/core'
import ClientesA    from './modules/clientes/variantA'

const Clientes = tenantConfig.clientes === 'variantA'
  ? ClientesA
  : ClientesCore
```

### 1.2 Despliegue y CI/CD

* **Pipeline único:** un solo build empaqueta todo (npm run build) y despliega el bundle completo.
* **Feature flags** o configuración runtime para alternar variantes sin redeploy. 
msmechatronics.medium.com

### Ventajas
- **Simplicidad**: Fácil de desarrollar, desplegar y probar en etapas iniciales. Una sola base de código y un único repositorio facilita la sincronización de versiones y dependencias.
- **Menor complejidad**: No requiere herramientas adicionales para la comunicación entre módulos.
- **Consistencia**: testing e integración continúan cubriendo todo el producto en un solo flujo. 
- **Desempeño**: No hay sobrecarga de red entre módulos, ya que todo está en un solo lugar.

### Desventajas
- **Escalabilidad limitada**: Dificultad para escalar equipos y proyectos grandes.
- **Tamaño del bundle:** incluye todas las variantes, aunque no se usen, lo que afecta el tiempo de carga. 
- **Riesgo de regresiones:** cambios en el core pueden impactar inadvertidamente variantes específicas. 
- **Mantenimiento complicado**: A medida que crece, el código puede volverse difícil de manejar.
- **Despliegues lentos**: Cualquier cambio requiere desplegar toda la aplicación.

---

## 2. Micro-frontends con módulos independientes

### Descripción
El enfoque de microfrontend divide la aplicación en módulos independientes que se desarrollan, despliegan y mantienen por separado. Este paradigma permite que cada módulo sea autónomo, lo que significa que puede tener su propio ciclo de vida de desarrollo, pipeline de CI/CD y equipo responsable. Además, cada módulo puede ser desarrollado utilizando diferentes tecnologías o frameworks si es necesario, lo que brinda flexibilidad para elegir la herramienta más adecuada para cada caso.

En términos de arquitectura, los microfrontends se integran en un "host" o "shell" que actúa como orquestador. Este host es responsable de cargar dinámicamente los módulos según la configuración específica del cliente o tenant. Por ejemplo, un cliente puede requerir una variante personalizada de un módulo, mientras que otro cliente utiliza la versión estándar. Esta configuración se gestiona a través de un manifiesto o archivo JSON que define qué módulos y variantes deben cargarse para cada cliente.

Una de las principales ventajas de este enfoque es la escalabilidad organizacional. Los equipos pueden trabajar de manera independiente en diferentes módulos sin interferir entre sí, lo que reduce los conflictos en el código y acelera el desarrollo. Además, los despliegues independientes permiten actualizar o corregir errores en un módulo sin necesidad de redeployar toda la aplicación.

Sin embargo, este enfoque también introduce desafíos. La comunicación entre módulos requiere herramientas y patrones específicos, como Module Federation en Webpack o plugins similares en Vite. Además, mantener una experiencia de usuario consistente en toda la aplicación puede ser complicado, especialmente si los módulos utilizan diferentes tecnologías. Para abordar esto, es fundamental contar con una biblioteca de componentes compartida que garantice la coherencia en el diseño y los estilos.

En términos de desempeño, los microfrontends pueden optimizar la carga inicial de la aplicación, ya que los usuarios solo descargan los módulos que necesitan. Sin embargo, esta ventaja puede verse contrarrestada por la sobrecarga de red introducida por la comunicación entre módulos y la carga dinámica de recursos.

### 1.1 Estructura y configuración (Ejemplo)

```
/shell-app (HOST)
  └─ vite.config.ts (o webpack MF)   ← configura remotes y shared
  └─ src/App.tsx        	         ← carga dinámicamente

/modulo-clientes
  ├─ /core
  │   └─ remoteEntry.js   ← variante core (Boostivity)
  ├─ /variantA
  │   └─ remoteEntry.js   ← variante Empresa A
  └─ /variantB
      └─ remoteEntry.js   ← variante Empresa B

/modulo-campañas
  ├─ /core
  │   └─ remoteEntry.js   ← variante core (Boostivity)
  ├─ /variantA
  │   └─ remoteEntry.js   ← variante Empresa A
  └─ /variantB
      └─ remoteEntry.js   ← variante Empresa B
```

- Cada remote se builda con Vite + vite-plugin-federation o Webpack Module Federation, y luego se publica en un CDN.

### 2.2 Manifiesto de tenants

Un endpoint o archivo JSON por tenant define qué URL usar para cada módulo:

```json
// manifest-empresa-A.json
{
  "modules": {
    "clientes":   "https://cdn.miempresa.com/clientes/variantA/remoteEntry.js",
    "campañas":   "https://cdn.miempresa.com/campañas/core/remoteEntry.js",
    "encuestas":  "https://cdn.miempresa.com/encuestas/core/remoteEntry.js"
  }
}
```

La app host realiza el fetch() y carga remotes según ese manifest. 

### 2.3 CI/CD por módulo

- Cada repo de módulo tiene su propio pipeline: npm run build → remoteEntry.js → CDN.
- El host solo redispone cuando su propio código o la orquestación cambia. 




### Ventajas
- **Escalabilidad**: Equipos independientes pueden trabajar en diferentes módulos simultáneamente.
- **Flexibilidad tecnológica**: Permite usar diferentes frameworks o herramientas en cada módulo.
- **Despliegues independientes**: Los módulos pueden actualizarse sin afectar a toda la aplicación.
- **Optimización de bundle:** el cliente descarga solo el código de las variantes que realmente usa 

### Desventajas
- **Complejidad**: Requiere herramientas para la orquestación y comunicación entre módulos. Configuracion de host robusto.
- **Desempeño**: Puede haber sobrecarga de red debido a la comunicación entre módulos.
- **Curva de aprendizaje**: Los desarrolladores necesitan familiarizarse con nuevas herramientas y patrones.
- Consistencia de UX: mantener estilos y diseño coherente requiere una buena librería de componentes compartida  (Nos da pie a hacer lo que siempre dije que queria hacer, un modulo npm de codetria. Al menos para Backoffice's)
- Mayor overhead: configurar Module Federation o variantes similares en Vite/Webpack añade curva de aprendizaje para una correcta implementacion y va a llevar un tiempo mayor inicial.


---

## Ejemplo Dinamico para imports

Aun con microfrontends tenemos un problema. El de que no nos salvamos de traer e importar manualmente todos los moduleFederation que creemos. Pero, se me ocurrio una alternativa (que no teestee, pero tengo toda la fe de que si se puede implementar), de importar dinamicamente los modulos incluso en el Shell host. Para hacer totalmente dinamica la configuracion de modulos / permisos. Solo vamos a necesitar crear un archivo json para proveer en cada cliente (empresa) el cual en su estructura tenga el tenantID, y los links de los CDN de cada modulo con su variante especifica. Incluso en un sidebar como tenemos los backoffice, podemos implementar React Lazy para hacer aun mas eficiente el paquete y la carga inicial del servidor para los clientes, asi solo van a descargar lo que necesitan, y nos evitamos proveer todo el JS antes de empezar a usar la app. 

En este escenario el Sidebar va a definirse en el shell (Host), porque es quien orquesta qué micro-frontend (MFE) vamos a mostrar:

1. El host es responsable tanto de la configuración (fetch del manifest según tenant) como de la navegación básica (sidebar, rutas, layout general).

2. Cada opción del sidebar simplemente cambia una pieza de estado (o ruta) en el host. Al hacer click, el host carga dinámicamente solo ese MFE.

```
/shell-app (host)
 ├─ src/
 │   ├─ App.tsx          ← sidebar + área de renderizado de MFEs
 │   ├─ sidebar/         ← componente Sidebar
 │   └─ manifest.ts      ← helper para fetch del manifest
 └─ vite.config.ts       ← configuración de remotes (vacíos/dummy)
 
 Módulos remotos (MFE1, MFE2…) se mantienen en repos propios, exponen sus remoteEntry.js.
```

###  Patrón de carga bajo demanda (On-Click)

#### 1. Estado del modulo activo:

```typescript
type ModuleKey = 'modulo1' | 'modulo2';

export const App = () => {
  const [manifest, setManifest] = useState<Record<ModuleKey,string>>({});
  const [active, setActive]   = useState<ModuleKey | null>(null);

  useEffect(() => {
    fetch(`/manifests/manifest-${tenantId}.json`)
      .then(r => r.json())
      .then(m => setManifest(m.modules));
  }, []);

  return (
    <div className="app-container">
      <Sidebar onSelect={setActive}/>
      <div className="main-area">
        {active && (
          <ModuleLoader
            name={active}
            url={manifest[active]}
          />
        )}
      </div>
    </div>
  );
}

- Active es el módulo seleccionado.
- Sólo cuando active tiene valor, montamos el MFE.

```

#### 2. Componente genérico ModuleLoader

```typescript
import React, { Suspense, lazy, useMemo } from 'react';

interface Props {
  name: string;
  url: string;
}

export const ModuleLoader = ({ name, url }: Props) => {
  // Creamos una fábrica de imports dinámicos basados en URL
  const RemoteComponent = useMemo(() => {
    return lazy(() =>
      import(/* @vite-ignore */ url).then(remote => ({
        default: remote[name]
      }))
    );
  }, [name, url]);

  return (
    <Suspense fallback={<div>Cargando {name}…</div>}>
      <RemoteComponent />
    </Suspense>
  );
}
```

#### 3. Sidebar

```typescript

- El Sidebar sólo dispara onSelect.
- No necesita conocer URLs ni internals de los remotes, sólo las keys (identificadores). (Asi incluso lo hacemos aun mas robusto y seguro)

interface SidebarProps {
  onSelect: (moduleKey: ModuleKey) => void;
}

export const Sidebar = ({ onSelect }: SidebarProps) => {
  return (
    <nav className="sidebar">
      <ul>
        <li onClick={() => onSelect('modulo1')}>
          Módulo 1
        </li>
        <li onClick={() => onSelect('modulo2')}>
          Módulo 2
        </li>
      </ul>
    </nav>
  );
}
```

#### Integracion con Rutas en lugar de estados manuales:

```typescript
// App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function App() {
  // fetch manifest…
  return (
    <BrowserRouter>
      <Sidebar/>
      <Routes>
        <Route path="/" element={<Navigate to="/modulo1"/>}/>
        <Route
          path="/modulo1"
          element={
            <ModuleLoader
              name="modulo1"
              url={manifest.modulo1}
            />
          }
        />
        <Route
          path="/modulo2"
          element={
            <ModuleLoader
              name="modulo2"
              url={manifest.modulo2}
            />
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

#### Flujo completo - Resumen
1. Arranca el host → fetch del manifest del tenant → guardamos 

```json
{ 
    modulo1: url1, 
    modulo2: url2 
}
```

2. Usuario ve el sidebar y hace click en “Módulo 2”.

3. setActive('modulo2') → el host monta
```typescript
<ModuleLoader name="modulo2" url={url2}/>
```
4. React.lazy + Suspense descargan y renderizan el MFE2.

5. De este modo, no estás cargando todos los módulos simultáneamente, sino solo el que el usuario solicita al navegar por el sidebar.
