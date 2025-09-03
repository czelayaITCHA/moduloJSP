# Componente Gestión de Ordenes

## 1.- Crear carpeta de ordenes dentro components, en el proyecto Orders-app (proyecto react), tal como se muestra en la siguiente imágen

<img width="257" height="212" alt="image" src="https://github.com/user-attachments/assets/20cb943a-30cf-49d3-81ee-f0f9d2c6afb7" />

## 2.- Agregar o modificar opción de *Gestión Ordenes* en el Siderbar.jsx

```JavaScript
 {(user?.role === "ADMIN" || user?.role === "MESERO/A" || user?.role === "COCINERO" || user?.role === "CAJERO") && (
  <li>
    <Link
      to="/ordenes"
      className="p-4 hover:bg-blue-800 cursor-pointer flex items-center"
      onClick={handleMenuClick}
    >
      <FaBorderStyle className="mr-2" />
      Gestión Ordenes
    </Link>
  </li>
)}

```
## 3.- Crear *ordenService.js* dentro de la carpeta services del proyecto frontend (orders-app)
En este archivo programar funciones exportables para la gestión de las ordenes

```JavaScript
import axios from "axios";
import { urlBase } from "../../utils/config";


const buildHeaders = (token) => ({
  headers: {
    Authorization: token ? `Bearer ${token}` : "",
    "Content-Type": "application/json",
  },
});

export const getMenus = async (token) => {
  const res = await axios.get(`${urlBase}/menus`, buildHeaders(token));
  return res.data;
};

export const getMesas = async (token) => {
  const res = await axios.get(`${urlBase}/mesas`, buildHeaders(token));
  return res.data;
};

export const getClientes = async (token) => {
  const res = await axios.get(`${urlBase}/clientes`, buildHeaders(token));
  return res.data;
};

export const createOrden = async (dto, token) => {
  const res = await axios.post(`${url}/ordenes`, dto, buildHeaders(token));
  return res.data;
};

```

## 3.- Crear componentes de las vistas de acuerdo al rol del usuario
En la carpeta ordenes vamos a crear los archivos de los diferentes componentes para gestionar las ordenes, por el momento serán solo archivos con algun texto, luego se programará lógica real de cada componente

### 3.1 Crear vista para el rol de 'MESERO/A' compMeseroView.jsx




